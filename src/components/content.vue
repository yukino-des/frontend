<template>
  <div id="content">
    <el-dialog title="模型预测中" :visible.sync="dialogTableVisible"
               :show-close="false"
               :close-on-press-escape="false"
               :append-to-body="true"
               :close-on-click-modal="false"
               :center="true">
      <el-progress :percentage="percentage"></el-progress>
      <span slot="footer" class="dialog-footer">请等待</span>
    </el-dialog>
    <div id="ct">
      <div id="image1">
        <el-card id="image0" class="box_card"
                 style="border-radius: 8px; width: 800px; height: 360px; margin-bottom: -30px">
          <div class="image_preview0">
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
                               type="success"
                               icon="el-icon-upload"
                               class="d_button"
                               v-on:click="upload0">上传
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
              <span style="color: white">原始图像</span>
            </div>
          </div>
          <!-- right box -->
          <div class="image_preview1">
            <div v-loading="loading"
                 element-loading-text="Please be patient."
                 element-loading-spinner="el-icon-loading">
              <el-image :src="url2"
                        class="image"
                        :preview-src-list="srcList2"
                        style="border-radius: 3px 3px 0 0">
                <div slot="error">
                  <div slot="placeholder" class="error">等待上传</div>
                </div>
              </el-image>
            </div>
            <div class="img_info" style="border-radius: 0 0 5px 5px">
              <span style="color: white">检测图像</span>
            </div>
          </div>
        </el-card>
      </div>
      <div id="info">
        <el-card style="border-radius: 8px">
          <div slot="header" class="clearfix">
            <span>检测结果</span>
            <el-button style="margin-left: 30px" v-show="!showButton" type="success" icon="el-icon-upload"
                       class="d_button" v-on:click="upload1">重新选择
              <input ref="upload2" style="display: none" name="file" type="file" @change="update"/>
            </el-button>
          </div>
          <el-table :data="featureList1" height="400" border style="width: 750px; text-align: center"
                    v-loading="loading" element-loading-text="请等待"
                    element-loading-spinner="el-icon-loading" lazy>
            <el-table-column label="目标种类" width="250px">
              <template v-slot:default="scope">
                <span>{{ scope.row[2] }}</span>
              </template>
            </el-table-column>
            <el-table-column label="目标边界框" width="250px">
              <template #default="scope">
                <span>{{ scope.row[0] }}</span>
              </template>
            </el-table-column>
            <el-table-column label="目标置信度" width="250px">
              <template #default="scope">
                <span>{{ scope.row[1] }}</span>
              </template>
            </el-table-column>
          </el-table>
        </el-card>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios"

export default {
  name: "content",
  data() {
    return {
      serverUrl: "http://0.0.0.0:8081",
      active: 0,
      url1: "",
      url2: "",
      srcList1: [],
      srcList2: [],
      featureList1: [],
      featureList2: [],
      featList: [],
      url: "",
      visible: false,
      loading: false,
      table: false,
      showButton: true,
      percentage: 0,
      fullscreenLoading: false,
      dialogTableVisible: false,
    }
  },
  created: function () {
    document.title = "农作物病虫害检测"
  },
  methods: {
    upload0() {
      this.$refs.upload.click()
    },
    upload1() {
      this.$refs.upload2.click()
    },
    next() {
      this.active++
    },
    getObjectURL(file) {
      let url = null
      if (window.createObjcectURL !== undefined) {
        url = window.createOjcectURL(file)
      } else if (window.URL !== undefined) {
        url = window.URL.createObjectURL(file)
      } else if (window.webkitURL !== undefined) {
        url = window.webkitURL.createObjectURL(file)
      }
      return url
    },
    update(e) {
      this.percentage = 0
      this.dialogTableVisible = true
      this.url1 = ""
      this.url2 = ""
      this.srcList1 = []
      this.srcList2 = []
      this.featureList1 = []
      this.featList = []
      this.fullscreenLoading = true
      this.loading = true
      this.showButton = false
      let file = e.target.files[0]
      this.url1 = this.$options.methods.getObjectURL(file)
      let param = new FormData()
      param.append("file", file, file.name)
      const timer = setInterval(() => {
        this.f()
      }, 30)
      let config = {
        headers: {"Content-Type": "multipart/form-data"},
      }
      axios.post(this.serverUrl + "/image", param, config)
          .then((response) => {
            this.percentage = 100
            clearInterval(timer)
            this.url1 = response.data.image_url
            this.srcList1.push(this.url1)
            this.url2 = response.data.draw_url
            this.srcList2.push(this.url2)
            this.fullscreenLoading = false
            this.loading = false
            this.featList = Object.keys(response.data.image_info)
            for (let i = 0; i < this.featList.length; i++) {
              response.data.image_info[this.featList[i]][2] = this.featList[i]
              this.featureList1.push(response.data.image_info[this.featList[i]])
            }
            this.featureList1.push(response.data.image_info)
            this.featureList2 = this.featureList1[0]
            this.dialogTableVisible = false
            this.percentage = 0
            this.notice()
          })
    },
    f() {
      if (this.percentage + 33 < 99) {
        this.percentage = this.percentage + 33
      } else {
        this.percentage = 99
      }
    },
    notice() {
      this.$notify({
        title: "检测成功！",
        message: "点击查看大图",
        duration: 3000,
        type: "success",
      })
    }
  },
  mounted() {
  },
}
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

#ct {
  display: flex;
  height: 100%;
  width: 100%;
  flex-wrap: wrap;
  justify-content: center;
  margin: 0 0 0 auto;
  max-width: 1800px;
}

#image0 {
  width: 90%;
  height: 40%;
  margin: 0 180px 0 auto;
  border-radius: 4px;
}

#image1 {
  margin-bottom: 60px;
  margin-left: 20px;
  margin-top: 5px;
}

.image {
  width: 275px;
  height: 260px;
  background: white;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

.img_info {
  height: 30px;
  width: 275px;
  text-align: center;
  background-color: dodgerblue;
  line-height: 30px;
}

.image_preview0 {
  width: 250px;
  height: 290px;
  margin: 20px 40px;
  float: left;
}

.image_preview1 {
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

.d_button {
  padding: 10px 16px !important;
}

#content {
  width: 85%;
  height: 800px;
  background-color: white;
  margin: 15px auto;
  display: flex;
  min-width: 1200px;
}

#info {
  margin-top: 10px;
  margin-right: 160px;
}
</style>