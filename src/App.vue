<template>
  <div id="app">
    <div>---------有翻页--------------</div>
    <pdf watermark="翻页水印翻页水印" />
    <div>---------不翻页--------------</div>
    <pdfNoPage watermark="测试水印测试水印"/>
  </div>
</template>

<script>
import pdf from './components/pdf.vue'
import pdfNoPage from './components/pdfNoPage.vue'

export default {
  name: 'App',
  components: {
    pdf,
    pdfNoPage
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
