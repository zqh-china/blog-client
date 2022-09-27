<template>
  <n-tabs v-model:value="tabValue" justify-content="start" type="line">
      <n-tab-pane name="list" tab="文章列表">
      <div v-for="blog in blogListInfo" style="margin-bottom:15px">
        <n-card :title="blog.title">
          {{ blog.content }}

          <template #footer>
            <n-space align="center">
              <div>发布时间：{{ blog.create_time }}</div>
              
              <n-button color="#42aeff" @click="toUpdate(blog)">修改</n-button>
              <n-button color="#ff5722" @click="toDelete(blog)">删除</n-button>
            </n-space>
          </template>
        </n-card>
      </div>
      <n-pagination @click="toPage(pageInfo.page)" v-model:page="pageInfo.page" :page-count="pageInfo.pageCount" />
      </n-tab-pane>

    
    
     <n-tab-pane name="add" tab="添加文章">
      <n-form>
        <n-form-item label="标题">
          <n-input v-model:value="addArticle.title" placeholder="请输入标题" />
        </n-form-item>
        <n-form-item label="分类">
          <n-select v-model:value="addArticle.category_id" :options="categortyOptions" />
        </n-form-item>
        <n-form-item label="内容">
          <rich-text-editor v-model="addArticle.content"></rich-text-editor>
        </n-form-item>
        <n-form-item label="">
          <n-button @click="add">提交</n-button>
        </n-form-item>
      </n-form>
      </n-tab-pane>

      <!-- 这里禁用tab页，防止直接点击晋入修改，所有修改均通过文章的链接进入 -->
      <n-tab-pane name="update" tab="修改文章" :disabled="true">
      <n-form>
        <n-form-item label="标题">
          <n-input v-model:value="updateArticle.title" placeholder="请输入标题" />
        </n-form-item>
        <n-form-item label="分类">
          <n-select v-model:value="updateArticle.category_id" :options="categortyOptions" />
        </n-form-item>
        <n-form-item label="内容">
            <rich-text-editor v-model="updateArticle.content" @image-change="handleImageChange"></rich-text-editor>
        </n-form-item>
        <n-form-item label="">
          <n-button @click="update">提交</n-button>
        </n-form-item>
      </n-form>
    </n-tab-pane>
  </n-tabs> 
</template>

<script setup>
import { ref, reactive, inject, onMounted } from 'vue'
import RichTextEditor from '../../components/RichTextEditor.vue'

const message = inject("message")
const axios = inject("axios")
const server_url = inject("server_url")

//文章添加数据
const addArticle = reactive({
  category_id: 0,
  title: "",
  content: "",
})

//文章修改数据
const updateArticle = reactive({
  id: 0,
  category_id: 0,
  title: "",
  content: "",
})

//分类选项
const categortyOptions = ref([])
const blogListInfo = ref([])
//标签页
const tabValue = ref("list")

//分页数据
const pageInfo = reactive({
  page: 1,
  pageSize: 3,
  pageCount: 0,
  count: 0,
})

onMounted(() => {
  loadBlogs()
  loadCategorys()
})



//读取博客列表
const loadBlogs = async () => {
  let res = await axios.post(`/blogs/search?page=${pageInfo.page}&pageSize=${pageInfo.pageSize}`)
  // console.log(res)
  let temp_rows = res.data.data.rows;
  for (let row of temp_rows) {
    row.content += "..."
    let d = new Date(row.create_time)
    row.create_time = `${d.getFullYear()}年${d.getMonth() + 1}月${d.getDate()}日`
  }
  blogListInfo.value = temp_rows;
  pageInfo.count = res.data.data.count;
  pageInfo.pageCount = parseInt(pageInfo.count / pageInfo.pageSize) + (pageInfo.count % pageInfo.pageSize > 0 ? 1 : 0)
}

//读取分类
const loadCategorys = async () => {
  let res = await axios.get("/category/list")
  categortyOptions.value = res.data.rows.map((item) => {
    return {
      label: item.name,
      value: item.id
    }
  })
  // console.log(categortyOptions.value)
}

const add = async () => {
  let res = await axios.post("/blogs/_token/add", addArticle)
  if (res.data.code === 200) {
    message.info(res.data.msg)
    addArticle.title = ""
    addArticle.category_id = 0
    addArticle.content = ""

    tabValue.value = "list"
    loadBlogs()
  } else {
    message.error(res.data.msg)
  }
}

const toPage = async (page) => {
  pageInfo.page = page;
  loadBlogs()

}

const toUpdate = async (blog) => {
  
  tabValue.value = "update"
  console.log(tabValue.value)
  let res = await axios.get("/blogs/detail?id=" + blog.id)
  updateArticle.id = blog.id
  updateArticle.title = res.data.rows[0].title
  updateArticle.content = res.data.rows[0].content
  updateArticle.category_id = res.data.rows[0].category_id
}

const update = async () => {
  await deleteImage()
  let res = await axios.put("/blogs/_token/update", updateArticle)
  if (res.data.code === 200) {
    message.info(res.data.msg)
    loadBlogs()
    tabValue.value = "list"
  } else {
    message.error(res.data.msg)
  }
}

const toDelete = async (blog) => {
  // 删除文章之前，先删除文章中的图片
  // 获取文章内容
  let resContent = await axios.get("/blogs/detail?id=" + blog.id)
  let content = resContent.data.rows[0].content
 
  // 正则匹配内容中以网站server_url开头的图片地址
  let reg = new RegExp(`src="${server_url}/upload/.*?"`, "g")
  let imageList = content.match(reg)
  // console.log(imageList)
  // 对图片重新映射取图片url
  imageList = imageList.map((item) => {
    return {
      src: item.replace(`src="`, "").replace(/"/g, "")
    }
  })
  // console.log(imageList)
  // 删除图片
  let delRes = await axios.delete("/blogs/_token/deleteImages", {
    data: {
      images: imageList
    }
  })
  if (delRes.data.code === 200) {
    // 删除文章
    let res = await axios.delete("/blogs/_token/delete?id=" + blog.id)
    if (res.data.code === 200) {
      message.info(res.data.msg)
      loadBlogs()
    } else {
      message.error(res.data.msg)
    }
  } else {
    message.error(delRes.data.msg)
  }
}

// 用于记录编辑器已删除的图片
const deletedImages = ref([])

// 处理图片改变
const handleImageChange = (imageList) => {
  deletedImages.value = imageList
  
}

// 删除图片
const deleteImage = () => {
  setTimeout(async () => {
    if (deletedImages.value.length === 0) {
    return
  }
  console.log(deletedImages.value)
  let res = await axios.delete("/blogs/_token/deleteImages", {
    data: {
      images: deletedImages.value
    }
  })
  if (res.data.code === 200) {
    message.info(res.data.msg)
  } else {
    message.error(res.data.msg)
  }

  }, 2000)  // 延时2s是为了等待富文本编辑器组件销毁
  
}


</script>

<style lang="scss" scoped>
</style>