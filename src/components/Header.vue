<template>
  <div id="header">
    <div class="edition">
      <span style="color: #21b3b9; font-weight: bold">
        <i class="el-icon-star-off" style="font-size: 23px"></i>MobileNetV2 + YOLOv4
      </span>
      <span>
        <i style="font-size: 23px"></i>Author: github.com/yukino-des
      </span>
    </div>
    <div id="word">
      <h1>{{ msg }}</h1>
    </div>
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
  padding: 30px 110px 0 150px;
  width: 90%;
  margin: 10px auto;
}

#word {
  margin-left: 45%;
  margin-top: -35px;
  margin-bottom: 37px;
  height: 60px;
  line-height: 3.2em;
}

h1 {
  color: #21b3b9;
  font-size: 2.3em;
}

.edition span i {
  float: left;
  margin-right: 10px;
}

i,
input,
label {
  vertical-align: middle;
}

i {
  border: 0;
  display: block;
  cursor: pointer;
}

.edition span {
  float: left;
  font-size: 16px;
  color: #999999;
  line-height: 24px;
  margin-right: 40px;
}
</style>


