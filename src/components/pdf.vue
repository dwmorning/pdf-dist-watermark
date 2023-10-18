<template>
  <div class="main-container">
    <input type="file" ref="fielinput" @change="uploadFile" />
    <div ref="canvasCont" class="canvas-container">
      <canvas ref="myCanvas" class="pdf-container"></canvas>
    </div>
    <div class="pagination-wrapper">
      <button @click="clickPre">上一页</button>
      <span>第{{ pageNo }} / {{ pdfPageNumber }}页</span>
      <button @click="clickNext">下一页</button>
    </div>
  </div>
</template>
 
<script>
const pdfJS = require("pdfjs-dist");
 
pdfJS.GlobalWorkerOptions.workerSrc = require("pdfjs-dist/build/pdf.worker.entry");
export default {
  props: {
    watermark: {
      type: String,
      default: "水印文字水印文字水印文字",
    },
  },
  mounted() {},
  data() {
    return {
      pageNo: null,
      pdfPageNumber: null,
      renderingPage: false,
      pdfData: null, // PDF的base64
      scale: 1, // 缩放值
      width: "",
      height: "",
    };
  },
  methods: {
    uploadFile() {
      let inputDom = this.$refs.fielinput;
      let file = inputDom.files[0];
      let reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        let data = atob(
          reader.result.substring(reader.result.indexOf(",") + 1)
        );
        this.loadPdfData(data);
      };
    },
    loadPdfData(data) {
      // 引入pdf.js的字体
      let CMAP_URL = "https://unpkg.com/pdfjs-dist@2.0.943/cmaps/";
      //读取base64的pdf流文件
      this.pdfData = pdfJS.getDocument({
        data: data, // PDF base64编码
        cMapUrl: CMAP_URL,
        cMapPacked: true,
      });
      this.renderPage(1);
    },
 
    // 根据页码渲染相应的PDF
    renderPage(num) {
      this.renderingPage = true;
      this.pdfData.promise.then((pdf) => {

        this.pdfPageNumber = pdf.numPages;
 
        pdf.getPage(num).then((page) => {
          // 获取DOM中为预览PDF准备好的canvasDOM对象
          let canvas = this.$refs.myCanvas;
          let ctx = canvas.getContext("2d");
 
          // 获取页面比率
          let ratio = this._getRatio(ctx);
          
          // 根据页面宽度和视口宽度的比率就是内容区的放大比率
          let dialogWidth = this.$refs["canvasCont"].offsetWidth;
          let pageWidth = page.view[2] * ratio;
          let scale = dialogWidth / pageWidth;
          let viewport = page.getViewport({ scale });
          

          // 记录内容区宽高，后期添加水印时需要
          this.width = viewport.width * ratio;
          this.height = viewport.height * ratio;
 
          canvas.width = this.width;
          canvas.height = this.height;
 
          // 缩放比率
          ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
 
          let renderContext = {
            canvasContext: ctx,
            viewport: viewport,
          };
          page.render(renderContext).promise.then(() => {
            this.renderingPage = false;
            this.pageNo = num;
 
            // 添加水印
            this._renderWatermark();
          });
        });
      });
    },
    // 计算角度
    _getRatio(ctx) {
      let dpr = window.devicePixelRatio || 1;
      let bsr =
        ctx.webkitBackingStorePixelRatio ||
        ctx.mozBackingStorePixelRatio ||
        ctx.msBackingStorePixelRatio ||
        ctx.oBackingStorePixelRatio ||
        ctx.backingStorePixelRatio ||
        1;
      return dpr / bsr;
    },
 
    // 在画布上渲染水印
    _renderWatermark() {
      let canvas = this.$refs.myCanvas;
      let ctx = canvas.getContext("2d");
      // 平铺水印
      let pattern = ctx.createPattern(this._initWatermark(), "repeat");
      ctx.rect(0, 0, this.width, this.height);
      ctx.fillStyle = pattern;
      ctx.fill();
    },
 
    // 生成水印图片
    _initWatermark() {
      let canvas = document.createElement("canvas");
      canvas.width = 200;
      canvas.height = 200;
      let ctx = canvas.getContext("2d");
      ctx.rotate((-18 * Math.PI) / 180);
      ctx.font = "10px Vedana";
      ctx.fillStyle = "rgba(0, 0, 0, 0.8)";
      ctx.textAlign = "left";
      ctx.textBaseline = "middle";
      ctx.fillText(this.watermark, 10, 100);
      return canvas;
    },
 
    clickPre() {
      if (!this.renderingPage && this.pageNo && this.pageNo > 1) {
        this.renderPage(this.pageNo - 1);
      }
    },
    clickNext() {
      if (
        !this.renderingPage &&
        this.pdfPageNumber &&
        this.pageNo &&
        this.pageNo < this.pdfPageNumber
      ) {
        this.renderPage(this.pageNo + 1);
      }
    },
  },
};
</script>
 
<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.canvas-container {
  width: 100%;
  height: 100%;
  border: 1px dashed black;
  position: relative;
  display: flex;
  justify-content: center;
}
.pdf-container {
  width: 100%;
  height: 100%;
}
 
.pagination-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
