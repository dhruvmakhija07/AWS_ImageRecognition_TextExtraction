<template>
  <div class="image-analysis">
    <section class="acknowledge">
      <!-- This section contains the checkbox to acknowledge the warning about using appropriate photos. -->
      <input type="checkbox" id="warning" v-model="warningAccepted" />
      <label for="warning">I understand and accept the conditions listed above for image labeling and text extraction.</label>

      <br />

      <!-- This button only appears when the checkbox is ticked. -->
      <button class="button" v-if="warningAccepted" @click="openFilePicker">Choose a photo</button>

      <!-- Input field to be shown only if the user accepts the terms and conditions -->
      <input type="file" ref="fileInput" id="input" v-if="warningAccepted" @change="onFileSelected" accept="image/*" style="display: none" />

      <!-- This button only appears when the user has selected a photo that is under 5MB. -->
      <button v-show="!noFileSelected" class="button" @click="analysePhotowithRekognition">Analyze with Amazon Rekognition</button>
    </section>

    <main id="container">
      <article id="content">
        <!-- This container only appears when a file is successfully loaded -->
        <figure v-show="!noFileSelected" id="photo-container">
          <!-- Display the selected image to the user -->
          <img id="userPhoto" :src="userPhoto" class="photoFrame" width="720" />
        </figure>

        <!-- This section only appears when the Amazon Rekognition API has been successfully called -->
        <section v-if="rekognitionSuccess" class="text-container instructions" id="rekognition">
          <h2>Object Detection with <span class="text-gradient">Amazon Rekognition</span></h2>
          <table style="width:100%">
            <thead>
              <tr>
                <th>Object</th>
                <th>Confidence</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="objectDetected in objectsDetected.Labels" :key="objectDetected.Name">
                <!-- Name of the object detected -->
                <td><span class="text-gradient">{{ objectDetected.Name }}</span></td>

                <!-- Confidence of the object detected -->
                <td>({{ parseFloat(objectDetected.Confidence).toFixed(1) }}%)</td>
              </tr>
            </tbody>
          </table>
        </section>
      </article>
    </main>
  </div>
</template>

<script>
import { ref } from 'vue';
import { RekognitionClient, DetectLabelsCommand } from "@aws-sdk/client-rekognition";

export default {
  name: 'AnalyzePhoto',
  setup() {
    const warningAccepted = ref(false);
    const noFileSelected = ref(true);
    const uimage_bytes = ref(null);
    const rekognitionClient = ref(null);
    const rekognitionSuccess = ref(false);
    const objectsDetected = ref(null);
    const userPhoto = ref(null);
    const confidence = 70;
    const numLabels = 10;

    const openFilePicker = () => {
      const fileInput = document.getElementById('input');
      fileInput.click();
    };

    const onFileSelected = (event) => {
      const imageFile = event.target.files[0];
      if (imageFile) {
        if (imageFile.size > 5242880) {
          alert("Your file is too large, please select a file that is 5MB or less.");
          return;
        }

        const fileReader = new FileReader();
        fileReader.onload = (e) => {
          const imagesrc = e.target.result;
          if (!processImage(imagesrc)) {
            alert("There was an issue processing the photo, please try a different photo.");
            return;
          }

          rekognitionSuccess.value = false;
          noFileSelected.value = false;
        };

        fileReader.readAsDataURL(imageFile);
      }
    };

    const processImage = (imagesrc) => {
      let image = null;
      let jpg = true;
      try {
        image = atob(imagesrc.split("data:image/jpeg;base64,")[1]);
      } catch (e) {
        jpg = false;
      }
      if (jpg == false) {
        try {
          image = atob(imagesrc.split("data:image/png;base64,")[1]);
        } catch (e) {
          alert("Please select a .jpg or .png file.");
          noFileSelected.value = true;
          return false;
        }
      }
      const length = image.length;
      const imageBytes = new ArrayBuffer(length);
      uimage_bytes.value = new Uint8Array(imageBytes);
      for (let i = 0; i < length; i++) {
        uimage_bytes.value[i] = image.charCodeAt(i);
      }
      userPhoto.value = imagesrc;
      return true;
    };

    const analysePhotowithRekognition = async () => {
      if (!rekognitionClient.value) {
        try {
          const client = new RekognitionClient({
            region: import.meta.env.VITE_AWS_REGION,
            credentials: {
              accessKeyId: import.meta.env.VITE_AWS_ACCESS_KEY_ID,
    secretAccessKey: import.meta.env.VITE_AWS_SECRET_ACCESS_KEY,
sessionToken: import.meta.env.VITE_AWS_SESSION_TOKEN,

            },
          });
          console.log(client);
          rekognitionClient.value = client;
        } catch (e) {
          console.log(e);
        }
      }

      const params = {
        Image: { Bytes: uimage_bytes.value },
        MaxLabels: numLabels,
        MinConfidence: confidence,
      };

      try {
        const query = new DetectLabelsCommand(params);
        const response = await rekognitionClient.value.send(query);
        objectsDetected.value = response;
        console.log(response);
        rekognitionSuccess.value = true;
      } catch (e) {
        console.log(e);
        alert("There was an error calling detect labels, please check the console for details.");
      }
    };

    return {
      warningAccepted,
      noFileSelected,
      uimage_bytes,
      rekognitionClient,
      rekognitionSuccess,
      objectsDetected,
      userPhoto,
      confidence,
      numLabels,
      openFilePicker,
      onFileSelected,
      analysePhotowithRekognition,
    };
  },
};
</script>