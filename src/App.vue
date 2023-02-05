<template>
  <div id="app">
    <app-header></app-header>
    <app-content></app-content>
  </div>
</template>

<script>
import Header from "./components/Header";
import Content from "./components/Content";

export default {
  name: "PlantDiseaseDetection",
  data() {
    return {};
  },
  components: {
    "app-header": Header,
    "app-content": Content,
  },
  methods: {},
};
</script>
