<template>
  <div style="margin:0 auto;width: 80%;text-align: left">
    <el-upload
        :show-file-list="false"
        drag
        :before-upload="beforeUpload"
        style="display:inline-block"
    >
      <img v-if="imageUrl" :src="imageUrl" class="avatar">
      <template v-else>
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <div class="el-upload__tip" slot="tip">只能上传jpg/png文件</div>
      </template>
    </el-upload>
    <span v-if="imageWidth > 0" style="margin-left:50px">
       图片大小：{{imageWidth}}x{{imageHeight}}
    </span>
    <el-collapse v-model="activeNames" style="margin-top: 20px">
      <el-collapse-item title="高级配置" name="1">
        <div>
          <div >
            <label>ascii字符的组成字符: </label>
            <el-input v-model="chars" size="mini" style="width: 380px"></el-input>
          </div>
          <div style="margin-top: 20px">
            <label>生成的ascii字符大小(越接近图片宽度越精细): </label>
            <el-input v-model="size" size="mini" style="width: 100px"></el-input>
          </div>
        </div>
      </el-collapse-item>
    </el-collapse>
    <div style="margin-top: 20px">
      <el-button @click="clickGenerate" type="primary">生成</el-button>
      <el-button @click="copy">复制</el-button>
    </div>
    <pre id="result">

    </pre>
  </div>
</template>

<script>

export default {
  data() {
    return {
      imageUrl: '',
      size: 50,
      chars: '$@B%8&WM#*oahkbdpqwmZO0QLCJUYXzcvunxrjft/|()1{}[]?-_+~<>i!lI;:,\"^`\'.',
      imageWidth: 0,
      imageHeight : 0
    }
  },
  methods: {
    copy() {
      var copy = function (e) {
        e.preventDefault();
        console.log('copy');
        var text = document.getElementById('result').textContent
        if (e.clipboardData) {
          e.clipboardData.setData('text/plain', text);
        } else if (window.clipboardData) {
          window.clipboardData.setData('Text', text);
        }
      }
      window.addEventListener('copy', copy);
      document.execCommand('copy');
      window.removeEventListener('copy', copy);
    },
    getChar(gray) {
      const ascii_char = (this.chars + " ").split("")
      let length = ascii_char.length
      let unit = (256.0 + 1) / length
      return ascii_char[parseInt(gray / unit)]
    },

    clickGenerate: async function () {
      if (!this.imageUrl) {
        this.$alert('请先选择图片')
      }
      let canvas = document.createElement('canvas');
      canvas.width = this.size;
      let image = new Image();
      image.src = this.imageUrl
      await new Promise((r1) => {
        image.onload = function () {
          r1()
        }
      })
      canvas.height = canvas.width * image.height / image.width;
      image.width = canvas.width
      image.height = canvas.height
      const ctx = canvas.getContext('2d');
      ctx.drawImage(image, 0, 0, image.width, image.height);
      let imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      let data = imgData.data;
      let res = ""
      for (let i = 0; i < data.length; i += 4) {
        let gray = Math.ceil(data[i] * 0.2126 + data[i + 1] * 0.7152 + data[i + 2] * 0.0722)
        /* data[i] = 255-data[i];
         data[i + 1] = 255-data[i+1];
         data[i + 2] = 255-data[i+2];*/
        res += this.getChar(gray)
        if (parseInt(i / 4) % canvas.width === 0) {
          res += "\n"
        }
      }
      ctx.putImageData(imgData, 0, 0);
      console.log(res)
      document.getElementById('result').textContent = res
      //document.body.appendChild(canvas)
    },
    beforeUpload: async function (file) {
      console.log('before upload', file)
      this.imageUrl = file.path;
      let image = new Image();
      image.src = this.imageUrl
      await new Promise((r1) => {
        image.onload = function () {
          r1()
        }
      })
      this.imageWidth = image.width
      this.imageHeight = image.height

    }
  }
}

</script>

<style scoped>
.result {

}

.avatar {
  width: 100%;
  height: 100%;
  display: block;
}
</style>
