<template>
  <main>
    <h2><span class="text-gradient">AWS Machine Learning</span> Workshop</h2>
    <p class="instructions">
      This web app analyses photos using Amazon Rekognition<br/>
      Please ensure the photo is 5MB or less in size.<br/>
      Please do <strong>not</strong> choose a photo that contains any people, faces, or sensitive data.<br/>
      Please do <strong>not</strong> choose an offensive photo.<br/>
      <i>If you have no photos on this device, some photos are supplied in the <code>public</code> folder.</i><br/>     
    </p>     
    
    <!-- The component that renders all the content of the main page -->
    <AnalyzePhoto/>
  </main>
</template>

<script>
import AnalyzePhoto from "./components/AnalyzePhoto.vue";

export default {
  components: {
    AnalyzePhoto,
  },
};
</script>

<style>
/* CSS styles for the App.vue component */
</style>
