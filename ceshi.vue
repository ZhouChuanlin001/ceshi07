// 界面
<template>
<div>
<input type="file" @change="getBase64">
</div>
</template>
//js代码
<script>
export default {
  methods: {
    getBase64 (e) {
      // 选择的文件
      let file = e.target.files[0]
      console.log(file.name) // 文件名称，有需求可处理
      console.log(file.type) // 文件类型，有需求可处理
      // 判断文件是否读取完毕，读取完毕后执行
      if (window.FileReader) {
        let reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = e => {
          let base64String = e.target.result
          // 此处可对该base64进行获取赋值传入后端
          console.log('bese64编码：', base64String)
        }
      }
    }
  }
}
</script>