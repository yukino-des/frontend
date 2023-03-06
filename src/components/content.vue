<template>
  <div id="content">
    <h1 style="font-size: 3em">农业病虫害检测</h1>
    <el-card shadow="always">
      <div slot="header">
        <el-button type="success" v-on:click="preUpload">上传图像、视频、压缩包
          <input ref="upload" style="display: none" type="file" @change="upload"/>
        </el-button>
        <el-button type="error" v-on:click="download('model.pth')">下载权值</el-button>
        <el-button type="error" v-on:click="download('model.onnx')">下载模型</el-button>
        <el-button type="error" v-on:click="download('summary.txt')">下载summary</el-button>
      </div>
      <el-card shadow="always">
        <el-row type="flex" justify="center">
          <!-- 左图像框 -->
          <el-image :src="urlL" class="image" :preview-src-list="arrL"></el-image>
          <!-- 中图像框 -->
          <el-image :src="urlM" class="image" :preview-src-list="arrM"></el-image>
          <!-- 右图像框 -->
          <el-image :src="urlR" class="image" :preview-src-list="arrR"></el-image>
        </el-row>
      </el-card>
      <!-- 检测结果 -->
      <el-table :data="infoArr" border style="width: 100%" :header-cell-style="{'text-align':'center'}"
                :cell-style="{'text-align':'center'}" element-loading-text="请等待"
                element-loading-spinner="el-icon-loading" lazy>
        <el-table-column label="目标种类">
          <template v-slot:default="scope">
            <span>{{ scope.row[2] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="目标置信度">
          <template #default="scope">
            <span>{{ scope.row[1] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="边界框尺寸">
          <template #default="scope">
            <span>{{ scope.row[0] }}</span>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import axios from "axios"

export default {
  name: "content",
  data() {
    return {
      url: "http://0.0.0.0:8080",
      urlL: "",
      urlM: "",
      urlR: "",
      arrL: [],
      arrM: [],
      arrR: [],
      infoArr: [],
    }
  },
  created: function () {
    document.title = "农作物病虫害检测"
  },
  methods: {
    preUpload() { // 上传图像、视频
      this.$refs.upload.click()
    },
    upload(e) {
      let file = e.target.files[0]
      if (window.URL !== undefined) this.urlL = window.URL.createObjectURL(file)
      else this.urlL = window.webkitURL.createObjectURL(file)
      let param = new FormData()
      param.append("file", file, file.name)
      axios.post(this.url + "/image", param, {
        headers: {"Content-Type": "multipart/form-data"},
      }).then((response) => {
        this.urlL = response.data.imageUrl
        this.urlM = response.data.imageOutUrl
        this.urlR = response.data.imageHeatmapUrl
        this.arrL.push(this.urlL)
        this.arrM.push(this.urlM)
        this.arrR.push(this.urlR)
        // window.console.log(this.urlR)
        let classes = Object.keys(response.data.targetInfo)
        this.infoArr = []
        for (let i = 0; i < classes.length; i++) {
          response.data.targetInfo[classes[i]][2] = classes[i]
          this.infoArr.push(response.data.targetInfo[classes[i]])
        }
        this.notice()
      })
    },
    notice() {
      this.$notify({title: "检测成功", message: "点击查看大图", duration: 3000})
    },
    download(filename) {
      axios.get(this.url + "/data/" + filename).then((response) => {
        const blob = new Blob([response.data])
        const elink = document.createElement('a')
        elink.download = filename
        elink.style.display = "none"
        elink.href = URL.createObjectURL(blob)
        document.body.appendChild(elink)
        elink.click()
        URL.revokeObjectURL(elink.href)
        document.body.removeChild(elink)
      }).catch((err) => {
        window.console.log(err)
      })
    },
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

.image {
  width: 400px;
  height: 400px;
  border: 3px solid pink;
  border-radius: 30px;
  margin: 20px;
}
</style>