<template>
  <div id="app">
    <app-head></app-head>
    <app-content></app-content>
  </div>
</template>

<script>
import head from "./components/head";
import content from "./components/content";

export default {
  name: "PlantDiseaseDetection",
  data() {
    return {};
  },
  components: {
    "app-head": head,
    "app-content": content,
  },
  methods: {},
};
</script>
