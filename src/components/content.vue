<template>
  <div id="content">
    <h1 style="font-size: 3em; text-align: center">农业病害检测</h1>
    <el-card shadow="always">
      <div slot="header">
        <el-button type="success" v-on:click="preUpload">上传图像、视频
          <input ref="upload" style="display: none" type="file" @change="upload"/>
        </el-button>
        <el-button type="warning" v-on:click="download('data/model.pth', {})">下载权值</el-button>
        <el-button type="warning" v-on:click="download('data/cache/model.onnx', {})">下载模型</el-button>
        <el-button type="warning" v-on:click="download('data/cache/summary.txt', {})">下载summary</el-button>
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
        <el-table-column label="类别">
          <template #default="scope">
            <span>{{ scope.row[2] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="置信度">
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
      url: "http://0.0.0.0:2475/", // 后端服务器监听2475号端口
      urlL: "",
      urlM: "",
      urlR: "",
      arrL: [],
      arrM: [],
      arrR: [],
      infoArr: []
    }
  },
  created: function () {
    document.title = "农业病害检测"
  },
  methods: {
    preUpload() { // 上传图像、视频
      this.$refs.upload.click()
    },

    upload(e) { // 上传图像、视频
      const file = e.target.files[0]
      let param = new FormData()
      param.append("file", file, file.name)
      const arr = file.name.toLowerCase().split(".")
      const extend_name = arr[arr.length - 1]
      if (extend_name === "jpg" || extend_name === "jpeg" || extend_name === "png") {
        axios.post(this.url + "file", param, {
          headers: {"Content-Type": "multipart/form-data"}
        }).then((response) => {
          this.urlL = response.data.imageUrl
          this.urlM = response.data.imageOutUrl
          this.urlR = response.data.imageHeatmapUrl
          this.arrL.push(this.urlL)
          this.arrM.push(this.urlM)
          this.arrR.push(this.urlR)
          let classes = Object.keys(response.data.targetInfo)
          this.infoArr = []
          for (let i = 0; i < classes.length; i++) {
            response.data.targetInfo[classes[i]][2] = classes[i]
            this.infoArr.push(response.data.targetInfo[classes[i]])
          }
          this.$notify({title: "检测成功", message: "点击查看图像", duration: 3000})
        })
      } else if (extend_name === "avi" || extend_name === "mov" || extend_name === "mp4") {
        axios.post(this.url + "file", param, {
          headers: {"Content-Type": "multipart/form-data"}
        }).then((response) => {
          window.console.log(response.data)
          this.download(response.data.videoPath, {
            responseType: "blob" // responseType设置为blob二进制流类型
          })
        })
      } else {
        this.$notify({title: "上传失败", message: "文件格式错误", duration: 3000})
      }
    },

    download(filePath, config) {
      const arr = filePath.split("/")
      const filename = arr[arr.length - 1]
      let title = "下载成功"
      axios.get(this.url + filePath, config).then((response) => {
        const blob = new Blob([response.data])
        const eLink = document.createElement("a")
        eLink.download = filename
        eLink.style.display = "none"
        eLink.href = URL.createObjectURL(blob)
        document.body.appendChild(eLink)
        eLink.click()
        URL.revokeObjectURL(eLink.href)
        document.body.removeChild(eLink)
      }).catch((err) => {
        window.console.log(err === null)
        title = "下载失败"
      }).finally(() => {
        this.$notify({"title": title, "message": filename, "duration": 3000})
      })
    }
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