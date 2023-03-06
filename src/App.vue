<template>
  <div id="app">
    <app-content></app-content>
  </div>
</template>

<script>
import content from "./components/content"

export default {
  name: "农业病虫害检测",
  data() {
    return {}
  },
  components: {
    "app-content": content
  },
  methods: {}
}
</script>