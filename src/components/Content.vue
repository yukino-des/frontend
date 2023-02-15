<template>
  <div id="content">
    <el-dialog title="AI Detecting" :visible.sync="dialogTableVisible"
               :show-close="false"
               :close-on-press-escape="false"
               :append-to-body="true"
               :close-on-click-modal="false"
               :center="true">
      <el-progress :percentage="percentage"></el-progress>
      <span slot="footer" class="dialog-footer">Please waiting.</span>
    </el-dialog>
    <div id="CT">
      <div id="CT_image2">
        <el-card id="CT_image1" class="box_card"
                 style="border-radius: 8px; width: 800px; height: 360px; margin-bottom: -30px;">
          <!-- left box -->
          <div class="demo_image_preview1">
            <div v-loading="loading"
                 element-loading-text="Uploading"
                 element-loading-spinner="el-icon-loading">
              <el-image :src="url1"
                        class="image"
                        :preview-src-list="srcList1"
                        style="border-radius: 3px 3px 0 0">
                <div slot="error">
                  <div slot="placeholder" class="error">
                    <el-button v-show="showButton"
                               type="primary"
                               icon="el-icon-upload"
                               class="download_bt"
                               v-on:click="trueUpload1">upload
                      <input ref="upload"
                             style="display: none"
                             name="file"
                             type="file"
                             @change="update"/>
                    </el-button>
                  </div>
                </div>
              </el-image>
            </div>
            <div class="img_info" style="border-radius: 0 0 5px 5px">
              <span style="color: white">original</span>
            </div>
          </div>
          <!-- right box -->
          <div class="demo_image_preview2">
            <div v-loading="loading"
                 element-loading-text="Please be patient."
                 element-loading-spinner="el-icon-loading">
              <el-image :src="url2"
                        class="image"
                        :preview-src-list="srcList2"
                        style="border-radius: 3px 3px 0 0">
                <div slot="error">
                  <div slot="placeholder" class="error">{{ waitReturn }}</div>
                </div>
              </el-image>
            </div>
            <div class="img_info" style="border-radius: 0 0 5px 5px">
              <span style="color: white">detected</span>
            </div>
          </div>
        </el-card>
      </div>
      <div id="info_patient">
        <el-card style="border-radius: 8px">
          <div slot="header" class="clearfix">
            <span>Detection Details</span>
            <el-button style="margin-left: 35px" v-show="!showButton" type="primary" icon="el-icon-upload"
                       class="download_bt" v-on:click="trueUpload2">Reselect
              <input ref="upload2" style="display: none" name="file" type="file" @change="update"/>
            </el-button>
          </div>
          <el-tabs v-model="activeName">
            <el-tab-pane label="Detected Targets" name="first">
              <el-table :data="featureList1" height="390" border style="width: 750px; text-align: center"
                        v-loading="loading" element-loading-text="Please be patient."
                        element-loading-spinner="el-icon-loading" lazy>
                <el-table-column label="Target Category" width="250px">
                  <template v-slot:default="scope">
                    <span>{{ scope.row[2] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="Target Size" width="250px">
                  <!--`v-slot:default`, simplify to `#default`-->
                  <template #default="scope">
                    <span>{{ scope.row[0] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="Confidence" width="250px">
                  <template #default="scope">
                    <span>{{ scope.row[1] }}</span>
                  </template>
                </el-table-column>
              </el-table>
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "content",
  data() {
    return {
      serverUrl: "http://127.0.0.1:8081",
      activeName: "first",
      active: 0,
      centerDialogVisible: true,
      url1: "",
      url2: "",
      textarea: "",
      srcList1: [],
      srcList2: [],
      featureList1: [],
      featureList2: [],
      featList: [],
      url: "",
      visible: false,
      waitReturn: "waiting for upload...",
      waitUpload: "waiting for upload...",
      loading: false,
      table: false,
      showButton: true,
      percentage: 0,
      fullscreenLoading: false,
      opacitys: {
        opacity: 0,
      },
      dialogTableVisible: false,
    };
  },
  created: function () {
    document.title = "Plant Disease Detect";
  },
  methods: {
    trueUpload1() {
      this.$refs.upload.click();
    },
    trueUpload2() {
      this.$refs.upload2.click();
    },
    next() {
      this.active++;
    },
    getObjectURL(file) {
      let url = null;
      if (window.createObjcectURL !== undefined) {
        url = window.createOjcectURL(file);
      } else if (window.URL !== undefined) {
        url = window.URL.createObjectURL(file);
      } else if (window.webkitURL !== undefined) {
        url = window.webkitURL.createObjectURL(file);
      }
      return url;
    },
    update(e) {
      this.percentage = 0;
      this.dialogTableVisible = true;
      this.url1 = "";
      this.url2 = "";
      this.srcList1 = [];
      this.srcList2 = [];
      this.waitReturn = "";
      this.waitUpload = "";
      this.featureList1 = [];
      this.featList = [];
      this.fullscreenLoading = true;
      this.loading = true;
      this.showButton = false;
      let file = e.target.files[0];
      this.url1 = this.$options.methods.getObjectURL(file);
      let param = new FormData();
      param.append("file", file, file.name);
      const timer = setInterval(() => {
        this.f();
      }, 30);
      let config = {
        headers: {"Content-Type": "multipart/form-data"},
      };
      axios
          .post(this.serverUrl + "/upload", param, config)
          .then((response) => {
            this.percentage = 100;
            clearInterval(timer);
            this.url1 = response.data.image_url;
            this.srcList1.push(this.url1);
            this.url2 = response.data.draw_url;
            this.srcList2.push(this.url2);
            this.fullscreenLoading = false;
            this.loading = false;
            this.featList = Object.keys(response.data.image_info);
            for (let i = 0; i < this.featList.length; i++) {
              response.data.image_info[this.featList[i]][2] = this.featList[i];
              this.featureList1.push(response.data.image_info[this.featList[i]]);
            }
            this.featureList1.push(response.data.image_info);
            this.featureList2 = this.featureList1[0];
            this.dialogTableVisible = false;
            this.percentage = 0;
            this.notice();
          });
    },
    f() {
      if (this.percentage + 33 < 99) {
        this.percentage = this.percentage + 33;
      } else {
        this.percentage = 99;
      }
    },
    drawChart() {
    },
    notice() {
      this.$notify({
        title: "Predicted Success!",
        message: "Click to view larger size",
        duration: 0,
        type: "success",
      });
    },
  },
  mounted() {
    this.drawChart();
  },
};
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.clearfix:before .clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}

.box_card {
  width: 680px;
  height: 200px;
  border-radius: 8px;
  margin-top: -20px;
}

#CT {
  display: flex;
  height: 100%;
  width: 100%;
  flex-wrap: wrap;
  justify-content: center;
  margin: 0 0 0 auto;
  max-width: 1800px;
}

#CT_image1 {
  width: 90%;
  height: 40%;
  margin: 0 180px 0 auto;
  border-radius: 4px;
}

#CT_image2 {
  margin-bottom: 60px;
  margin-left: 20px;
  margin-top: 5px;
}

.image {
  width: 275px;
  height: 260px;
  background: #ffffff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.img_info {
  height: 30px;
  width: 275px;
  text-align: center;
  background-color: #21b3b9;
  line-height: 30px;
}

.demo_image_preview1 {
  width: 250px;
  height: 290px;
  margin: 20px 40px;
  float: left;
}

.demo_image_preview2 {
  width: 250px;
  height: 290px;
  margin: 20px 440px;
}

.error {
  margin: 100px auto;
  width: 50%;
  padding: 10px;
  text-align: center;
}

div {
  display: block;
}

.download_bt {
  padding: 10px 16px !important;
}

#content {
  width: 85%;
  height: 800px;
  background-color: #ffffff;
  margin: 15px auto;
  display: flex;
  min-width: 1200px;
}

#info_patient {
  margin-top: 10px;
  margin-right: 160px;
}
</style>