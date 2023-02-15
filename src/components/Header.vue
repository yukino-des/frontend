<template>
  <div id="header">
    <h1>{{ msg }}</h1>
  </div>
</template>
<script>
export default {
  name: "header",
  data() {
    return {
      msg: "Plant Disease Detection",
      activeIndex: "1",
    };
  },
  methods: {},
};
</script>
<style scoped>
#header {
  padding: 0 0 0 160px;
  width: 50%;
  margin: 10px auto;
}

h1 {
  color: #21b3b9;
  font-size: 2.3em;
}
</style>