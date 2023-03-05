<template>
  <div id="head">
    <h1>{{ head }}</h1>
  </div>
</template>
<script>
export default {
  name: "head",
  data() {
    return {
      head: "农作物病虫害检测",
    };
  },
  methods: {},
};
</script>
<style scoped>
#head {
  padding: 0 0 0 160px;
  width: 50%;
  margin: 10px auto;
  font-size: 1.2em;
}
</style>