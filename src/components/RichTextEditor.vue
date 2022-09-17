<!-- 富文本组件 -->
<template>
  <div style="border: 1px solid #ccc">
    <Toolbar style="border-bottom: 1px solid #ccc" :editor="editorRef" :defaultConfig="toolbarConfig" :mode="mode" />
    <Editor style="height: 500px; overflow-y: hidden;" v-model="valueHtml" :defaultConfig="editorConfig" :mode="mode"
      @onCreated="handleCreated" @onChange="handleChange"/>
  </div>
</template>

<script setup>
import '@wangeditor/editor/dist/css/style.css' // 引入 css

import { onBeforeUnmount, ref, shallowRef, onMounted, inject } from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

const server_url = inject("server_url")
const mode = ref("simple")
const props = defineProps({
  modelValue: {
    type: String,
    default: ""
  }
})
// 编辑器实例，必须用 shallowRef
const editorRef = shallowRef()

// 内容 HTML
const valueHtml = ref('')
let initFinished = false

const emit = defineEmits(["update:model-value"])

onMounted(() => {
  setTimeout(() => {
    valueHtml.value = props.modelValue;
  }, 500)
  initFinished = true
})

// const toolbarConfig = {}
// const editorConfig = { placeholder: '请输入内容...' }
const toolbarConfig = {excludeKeys: ["uploadVideo"]};
const editorConfig = {placeholder: '请输入内容...'};
editorConfig.MENU_CONF = {}
editorConfig.MENU_CONF['uploadImage'] = {
  base64LimitSize: 10 * 1024, // 10kb
  server: server_url + '/upload/rich_editor_upload',
}
editorConfig.MENU_CONF['insertImage'] = {
  parseImageSrc: (src) => {
    // console.log(src)
    if (src.indexOf("http") !== 0) {
      // console.log(`${server_url}${src}`);
      return `${server_url}${src}`
    }
    return src
  }
}


// 组件销毁时，也及时销毁编辑器
onBeforeUnmount(() => {
  const editor = editorRef.value
  if (editor == null) return
  editor.destroy()
})

const handleCreated = (editor) => {
  editorRef.value = editor // 记录 editor 实例，重要！
}

const handleChange = (editor) => {
  if (initFinished) {
    // console.log(valueHtml.value)
    emit("update:model-value", valueHtml.value)
  }
};


</script>

<style lang="scss" scoped>
</style>