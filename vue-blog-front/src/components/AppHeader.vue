<template>
    <div class="nav">

        <n-space>
            <n-menu v-model:value="activeKey" mode="horizontal" :options="menuOptions" />
            <n-input-group>
                <n-input :style="{ width: '70%' }" v-model:value="keyword" placeholder="请输入关键字" />
                <n-button type="primary" ghost @click="$emit('search', keyword, selectedCategory)">搜索</n-button>
            </n-input-group>
        </n-space>
    </div>
</template>

<script setup>
import { computed, inject, onMounted, reactive, ref, defineComponent, h } from 'vue';
import { useRoute, useRouter, RouterLink } from 'vue-router';
import { NIcon } from "naive-ui";
import {
    PersonOutline as PersonIcon,
    HomeOutline as HomeIcon,
    MenuOutline as MenuIcon,
    PricetagsOutline as PricetagsIcon,
} from "@vicons/ionicons5";

const activeKey = ref(null)
const menuOptions = ref([])

const homeMenuOtion = {
    label: () => h(
        RouterLink,
        {
            to: {
                path: "/"
            }
        },
        { default: () => "首页" }
    ),
    key: "home",
    icon: renderIcon(HomeIcon)
};
const tagMenuOtion = {
    label: () => h(
        RouterLink,
        {
            to: {
                path: "/tags"
            }
        },
        { default: () => "标签" }
    ),
    key: "tags",
    icon: renderIcon(PricetagsIcon)
};
const aboutMenuOption = {
    label: () => h(
        RouterLink,
        {
            to: {
                path: "/page/about"
            }
        },
        { default: () => "关于我" }
    ),
    key: "about",
    icon: renderIcon(PersonIcon)
}

const axios = inject("axios")
const router = useRouter()
// 文章列表
const articleListInfo = ref([])
// 选中的分类
const selectedCategory = ref(0)
// 分类选项
const categoryOptions = ref([])
// 搜索关键词
const keyword = ref("")

const categoryName = computed(() => {
    //获取选中的分类
    let selectedOption = categoryOptions.value.find((option) => {
        return option.value === selectedCategory.value
    })
    //返回分类的名称
    return selectedOption ? selectedOption.label : ""
})

onMounted(async () => {
   await loadCategories();
   const categoryMenuOption = {
        label: "分类",
        key: "category",
        icon: renderIcon(MenuIcon),
        children: categoryOptions.value.map(m=>({label:m.label, key:m.value}))
    };
    menuOptions.value = [homeMenuOtion, categoryMenuOption, tagMenuOtion, aboutMenuOption]
})
/**
 * 获取分类列表
 */
const loadCategories = async () => {
    let res = await axios.get("/api/category/list")
    console.log(res)
    categoryOptions.value = res.data.data.map((item) => {
        return {
            label: item.name,
            value: item.id
        }
    })
}


// 查询和分页数据
const pageInfo = reactive({
    page: 1,
    pageSize: 10,
    totalPage: 0,
    total: 0,
    categoryId: 0,
})
/**
 * 选中分类
 */
const searchByCategory = (categoryId) => {
    pageInfo.categoryId = categoryId;
    loadArticles()
}
function renderIcon(icon) {
    return () => h(NIcon, null, { default: () => h(icon) });
}
</script>

<style lang="scss" scoped>
.nav {
    display: flex;
    padding-top: 16px;
}
</style>