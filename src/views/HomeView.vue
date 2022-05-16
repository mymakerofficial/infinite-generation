<template>
  <main>
    <RenderContainer />
    <!--<div style="position: absolute; top: 50vh; left: 50vw; background: white; width: 10px; height: 10px; border-radius: 100%;transform: translate(-50%, -50%)"></div>-->
  </main>
</template>
<script>
import RenderContainer from "../components/RenderContainer.vue";
export default {
  name: "HomeView",
  components: { RenderContainer },
};
</script>
