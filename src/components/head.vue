<template>
  <div id="head">
    <h1>{{ msg }}</h1>
  </div>
</template>
<script>
export default {
  name: "head",
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
#head {
  padding: 0 0 0 160px;
  width: 50%;
  margin: 10px auto;
}

h1 {
  color: #21b3b9;
  font-size: 2.3em;
}
</style>