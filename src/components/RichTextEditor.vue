<!-- 富文本组件 -->
<template>
  <div style="border: 1px solid #ccc">
    <Toolbar style="border-bottom: 1px solid #ccc" :editor="editorRef" :defaultConfig="toolbarConfig" :mode="mode" />
    <Editor style="height: 500px; overflow-y: hidden;" v-model="valueHtml" :defaultConfig="editorConfig" :mode="mode"
      @onCreated="handleCreated" @onChange="handleChange" />
  </div>
</template>

<script setup>
import '@wangeditor/editor/dist/css/style.css' // 引入 css

import { onBeforeUnmount, ref, shallowRef, onMounted, inject, reactive } from 'vue'
import { Editor, Toolbar } from '@wangeditor/editor-for-vue'

const server_url = inject("server_url")
const mode = ref("simple")
const props = defineProps({
  modelValue: {
    type: String,
    default: ""
  },
  articleId: {
    type: Number,
    default: 0
  }
})
// 编辑器实例，必须用 shallowRef
const editorRef = shallowRef()

// 内容 HTML
const valueHtml = ref('')
const aId = ref(0)
let initFinished = false
const imgList1 = ref([])  // 存储打开编辑器时的图片列表
const imgList2 = ref([])  // 存储关闭编辑器时的图片列表


var changeCount = 0;   // 改变次数计数，因为第二次获取到的是文章本来的图片

const emit = defineEmits(["update:model-value", "completed"])


onMounted(() => {
  setTimeout(() => {
    valueHtml.value = props.modelValue;
    aId.value = props.articleId;
  }, 50)
  initFinished = true
})

// const toolbarConfig = {}
// const editorConfig = { placeholder: '请输入内容...' }
const toolbarConfig = { excludeKeys: ["uploadVideo"] };
const editorConfig = { placeholder: '请输入内容...' };
editorConfig.MENU_CONF = {}
editorConfig.MENU_CONF['uploadImage'] = {
  base64LimitSize: 10 * 1024, // 10kb
  server: server_url + '/upload/rich_editor_upload',
  allowedFileTypes: ['image/*'],
  meta: {
    articleId: aId.value
  },
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
  // 对比imgList1和imgList2的差异，其中imgList1有而imgList2无的图片就是被删除的图片
  for (let img1 of imgList1.value) {
    for (let img2 of imgList2.value) {
      // 比较img1是否能在imgList2中找到
      if (img1.src === img2.src) {
        // 找到了，就从imgList1中删除
        imgList1.value.splice(imgList1.value.indexOf(img1), 1)
        break
      }
    }
  }
  // imgList1中剩下的就是被删除的图片

  // 组建销毁时会触发completed事件
  emit("completed", imgList1.value)

  if (editor == null) return
  editor.destroy()
})

const handleCreated = (editor) => {
  if (initFinished) {
    editorRef.value = editor // 记录 editor 实例，重要！
  }
}

const handleChange = (editor) => {
  if (changeCount <= 1) {
    // console.log(valueHtml.value)
    emit("update:model-value", valueHtml.value)
    imgList1.value = editor.getElemsByType('image')  // 在此处获取编辑器最初的图片列表
  } else {
    imgList2.value = editor.getElemsByType('image')  // 在此处获取编辑器最新的图片列表
  }
  changeCount++;
}




</script>

<style lang="scss" scoped>

</style>