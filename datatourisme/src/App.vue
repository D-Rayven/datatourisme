<template>
  <div id="app">
    <Map />
  </div>
</template>

<script>
import Map from "./components/Map.vue";

export default {
  name: "App",
  components: {
    Map,
  },
};
</script>

<style>
#app {
  position: relative;
  height: 750px;
  width: 100%;
}
</style>
