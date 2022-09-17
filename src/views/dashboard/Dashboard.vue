<template>
    <div class="main-panel">
        <div class="menus">
            <!-- <div v-for="menu in menus" @click="toPage(menu)">
                {{ menu.label }}
            </div> -->
            <n-menu
                :options="menus"
                :default-expand-all="true"
            />
        </div>
        <div style="padding:20px;width:100%">
            <router-view></router-view>
        </div>
    </div>
    <div class="title">后台管理系统</div>
</template>

<script setup>
import { useRouter, RouterLink } from 'vue-router'
import { h } from "vue";
const router = useRouter()


//菜单
// let menus = [
//     { name: "文章管理", href: "/dashboard/article" },
//     { name: "分类管理", href: "/dashboard/category" },
//     { name: "退出", href: "logout" },
// ];
let menus = [
    {
        label: () => h(
        RouterLink,
        {
            to: {
            path: "/dashboard/article"
            }
        },
        { default: () => "文章管理" }
    ),
    key: "/dashboard/article",
  },
  {
    label: () => h(
        RouterLink,
        {
            to: {
                path: "/dashboard/category"
            }
        },
        { default: () => "分类管理" },
    ),
    key: "/dashboard/category",
  },
  {
    label: () => h(
        RouterLink,
        {
            to: {
            path: "/logout"
            }
        },
        { default: () => "退出" }
    ),
    key: "/logout",
  }
 
]

//路由跳转
// const toPage = (value) => {
//     console.log(value); 
//     // if (menu.href === 'logout') {
//     //     router.push("/login")
//     // } else {
//     //     router.push(menu.key)
//     // }
// }




</script>

<style lang="scss" scoped>
.main-panel {
    display: flex;
    color: #64676a;
    max-width: 1500px;
    margin: 0 auto;
}

.menus {
    padding: 20px 0;
    box-sizing: border-box;
    line-height: 55px;
    text-align: center;
    width: 180px;
    height: 95vh;
    border-right: 1px solid #dadada;

    div {
        cursor: pointer;

        &:hover {
            color: #fd760e;
        }
    }
}

.title {
    font-size: 65px;
    font-weight: bold;
    text-align: right;
    position: fixed;
    color: rgba(0, 0, 0, 20%);
    right: calc((100vw - 1500px) / 2);
    bottom: 20px;
}
</style>