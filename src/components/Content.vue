<template>
  <div id="Content">
    <el-dialog title="AI Predicting"
               :visible.sync="dialogTableVisible"
               :show-close="false"
               :close-on-press-escape="false"
               :append-to-body="true"
               :close-on-click-modal="false"
               :center="true">
      <el-progress :percentage="percentage"></el-progress>
      <span slot="footer" class="dialog-footer">Please Waiting</span>
    </el-dialog>

    <div id="CT">
      <div id="CT_image">
        <el-card id="CT_image_1"
                 class="box-card"
                 style="border-radius: 8px; width: 800px; height: 360px; margin-bottom: -30px;">
          <div class="demo-image__preview1">
            <div v-loading="loading"
                 element-loading-text="Uploading"
                 element-loading-spinner="el-icon-loading">
              <el-image :src="url_1"
                        class="image_1"
                        :preview-src-list="srcList"
                        style="border-radius: 3px 3px 0 0">
                <div slot="error">
                  <div slot="placeholder" class="error">
                    <el-button v-show="showbutton"
                               type="primary"
                               icon="el-icon-upload"
                               class="download_bt"
                               v-on:click="true_upload">
                      Upload Image
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
            <div class="img_info_1" style="border-radius: 0 0 5px 5px">
              <span style="color: white">Original Image</span>
            </div>
          </div>
          <div class="demo-image__preview2">
            <div v-loading="loading"
                 element-loading-text="Processing, please be patient ^_^"
                 element-loading-spinner="el-icon-loading">
              <el-image :src="url_2"
                        class="image_1"
                        :preview-src-list="srcList1"
                        style="border-radius: 3px 3px 0 0">
                <div slot="error">
                  <div slot="placeholder" class="error">{{ wait_return }}</div>
                </div>
              </el-image>
            </div>
            <div class="img_info_1" style="border-radius: 0 0 5px 5px">
              <span style="color: white">Detection Result</span>
            </div>
          </div>
        </el-card>
      </div>
      <div id="info_patient">
        <el-card style="border-radius: 8px">
          <div slot="header" class="clearfix">
            <span>Detection Target</span>
            <el-button style="margin-left: 35px"
                       v-show="!showbutton"
                       type="primary"
                       icon="el-icon-upload"
                       class="download_bt"
                       v-on:click="true_upload2">
              Reselect Image
              <input ref="upload2"
                     style="display: none"
                     name="file"
                     type="file"
                     @change="update"/>
            </el-button>
          </div>
          <el-tabs v-model="activeName">
            <el-tab-pane label="Detected Target" name="first">
              <el-table :data="feature_list"
                        height="390"
                        border
                        style="width: 750px; text-align: center"
                        v-loading="loading"
                        element-loading-text="The data is being processed, please be patient ^_^"
                        element-loading-spinner="el-icon-loading"
                        lazy>
                <el-table-column label="Target Category" width="250px">
                  <template slot-scope="scope">
                    <span>{{ scope.row[2] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="Target Size" width="250px">
                  <template slot-scope="scope">
                    <span>{{ scope.row[0] }}</span>
                  </template>
                </el-table-column>
                <el-table-column label="Confidence" width="250px">
                  <template slot-scope="scope">
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
  name: "Content",
  data() {
    return {
      server_url: "http://127.0.0.1:8081",
      activeName: "first",
      active: 0,
      centerDialogVisible: true,
      url_1: "",
      url_2: "",
      textarea: "",
      srcList: [],
      srcList1: [],
      feature_list: [],
      feature_list_1: [],
      feat_list: [],
      url: "",
      visible: false,
      wait_return: "Waiting For Upload...",
      wait_upload: "Waiting For Upload...",
      loading: false,
      table: false,
      isNav: false,
      showbutton: true,
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
    true_upload() {
      this.$refs.upload.click();
    },
    true_upload2() {
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
      this.url_1 = "";
      this.url_2 = "";
      this.srcList = [];
      this.srcList1 = [];
      this.wait_return = "";
      this.wait_upload = "";
      this.feature_list = [];
      this.feat_list = [];
      this.fullscreenLoading = true;
      this.loading = true;
      this.showbutton = false;
      let file = e.target.files[0];
      this.url_1 = this.$options.methods.getObjectURL(file);
      let param = new FormData();
      param.append("file", file, file.name);
      const timer = setInterval(() => {
        this.myFunc();
      }, 30);
      let config = {
        headers: {"Content-Type": "multipart/form-data"},
      };
      axios
          .post(this.server_url + "/upload", param, config)
          .then((response) => {
            window.console.log(response.data)
            this.percentage = 100;
            clearInterval(timer);
            this.url_1 = response.data.image_url;
            this.srcList.push(this.url_1);
            this.url_2 = response.data.draw_url;
            this.srcList1.push(this.url_2);
            this.fullscreenLoading = false;
            this.loading = false;
            this.feat_list = Object.keys(response.data.image_info);
            for (let i = 0; i < this.feat_list.length; i++) {
              response.data.image_info[this.feat_list[i]][2] = this.feat_list[i];
              this.feature_list.push(response.data.image_info[this.feat_list[i]]);
            }
            this.feature_list.push(response.data.image_info);
            this.feature_list_1 = this.feature_list[0];
            this.dialogTableVisible = false;
            this.percentage = 0;
            this.notice1();
          });
    },
    myFunc() {
      if (this.percentage + 33 < 99) {
        this.percentage = this.percentage + 33;
      } else {
        this.percentage = 99;
      }
    },
    drawChart() {
    },
    notice1() {
      this.$notify({
        title: "Predicted Success",
        message: "Click to view a larger image",
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

<style>
p {
  font-size: 15px !important;
}
</style>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both;
}

.box-card {
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

#CT_image_1 {
  width: 90%;
  height: 40%;
  /* padding: 0 auto; */
  margin: 0 180px 0 auto;
  border-radius: 4px;
}

#CT_image {
  margin-bottom: 60px;
  margin-left: 30px;
  margin-top: 5px;
}

.image_1 {
  width: 275px;
  height: 260px;
  background: #ffffff;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.img_info_1 {
  height: 30px;
  width: 275px;
  text-align: center;
  background-color: #21b3b9;
  line-height: 30px;
}

.demo-image__preview1 {
  width: 250px;
  height: 290px;
  margin: 20px 60px;
  float: left;
}

.demo-image__preview2 {
  width: 250px;
  height: 290px;

  margin: 20px 460px;
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

#Content {
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