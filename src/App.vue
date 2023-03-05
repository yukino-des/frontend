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
  name: "农作物病虫害检测",
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
