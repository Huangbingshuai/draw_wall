<template>
  <div id="homePage">
    <section class="gallery-intro">
      <div class="intro-copy">
        <span class="eyebrow">汇图 · 壁纸画廊</span>
        <h1>把喜欢的风景，<br /><em>收藏进日常。</em></h1>
        <p>发现、整理并分享值得珍藏的屏幕风景。</p>
      </div>
      <div class="intro-index" aria-hidden="true"><span>H</span><small>WALLPAPER<br />COLLECTION</small></div>
    </section>
    <div class="search-bar">
      <a-input-search
        v-model:value="searchParams.searchText"
        placeholder="搜索壁纸名称、主题或灵感…"
        enter-button="搜一搜"
        size="large"
        @search="doSearch"
      />
    </div>
    <section class="gallery-filters" aria-label="筛选壁纸">
      <a-tabs v-model:active-key="selectedCategory" @change="doSearch">
        <a-tab-pane key="all" tab="全部风景" />
        <a-tab-pane v-for="category in categoryList" :tab="category" :key="category" />
      </a-tabs>
      <div v-if="tagList.length" class="tag-bar">
        <span class="filter-label">精选标签</span>
        <a-space :size="[0, 8]" wrap>
        <a-checkable-tag
          v-for="(tag, index) in tagList"
          :key="tag"
          v-model:checked="selectedTagList[index]"
          @change="doSearch"
        >
          {{ tag }}
        </a-checkable-tag>
        </a-space>
      </div>
    </section>
    <div class="results-heading"><span>画廊精选</span><span>{{ total }} 张壁纸</span></div>
    <PictureList :dataList="dataList" :loading="loading" />
    <a-pagination v-if="total > searchParams.pageSize"
      style="text-align: right"
      v-model:current="searchParams.current"
      v-model:pageSize="searchParams.pageSize"
      :total="total"
      @change="onPageChange"
    />
  </div>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'
import {
  listPictureTagCategoryUsingGet,
  listPictureVoByPageUsingPost,
} from '@/api/pictureController.ts'
import { message } from 'ant-design-vue'
import PictureList from '@/components/PictureList.vue' // 定义数据

// 定义数据
const dataList = ref<API.PictureVO[]>([])
const total = ref(0)
const loading = ref(true)

// 搜索条件
const searchParams = reactive<API.PictureQueryRequest>({
  current: 1,
  pageSize: 12,
  sortField: 'createTime',
  sortOrder: 'descend',
})

// 获取数据
const fetchData = async () => {
  loading.value = true
  // 转换搜索参数
  const params = {
    ...searchParams,
    tags: [] as string[],
  }
  if (selectedCategory.value !== 'all') {
    params.category = selectedCategory.value
  }
  // [true, false, false] => ['java']
  selectedTagList.value.forEach((useTag, index) => {
    if (useTag) {
      params.tags.push(tagList.value[index])
    }
  })
  const res = await listPictureVoByPageUsingPost(params)
  if (res.data.code === 0 && res.data.data) {
    dataList.value = res.data.data.records ?? []
    total.value = res.data.data.total ?? 0
  } else {
    message.error('获取数据失败，' + res.data.message)
  }
  loading.value = false
}

// 页面加载时获取数据，请求一次
onMounted(() => {
  fetchData()
})

// 分页参数
const onPageChange = (page: number, pageSize: number) => {
  searchParams.current = page
  searchParams.pageSize = pageSize
  fetchData()
}

// 搜索
const doSearch = () => {
  // 重置搜索条件
  searchParams.current = 1
  fetchData()
}

// 标签和分类列表
const categoryList = ref<string[]>([])
const selectedCategory = ref<string>('all')
const tagList = ref<string[]>([])
const selectedTagList = ref<boolean[]>([])

/**
 * 获取标签和分类选项
 * @param values
 */
const getTagCategoryOptions = async () => {
  const res = await listPictureTagCategoryUsingGet()
  if (res.data.code === 0 && res.data.data) {
    tagList.value = res.data.data.tagList ?? []
    categoryList.value = res.data.data.categoryList ?? []
  } else {
    message.error('获取标签分类列表失败，' + res.data.message)
  }
}

onMounted(() => {
  getTagCategoryOptions()
})
</script>

<style scoped>
#homePage { padding-bottom: 20px; }
.gallery-intro { position: relative; display: flex; align-items: center; justify-content: space-between; min-height: 270px; margin: 0 0 30px; padding: 42px 54px; overflow: hidden; background: #e9eee9; border-radius: 22px; }
.intro-copy { position: relative; z-index: 1; }
.eyebrow { color: var(--brand-primary); font-size: 12px; font-weight: 700; letter-spacing: .14em; }
.gallery-intro h1 { margin: 18px 0 12px; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: clamp(32px, 4vw, 48px); font-weight: 600; line-height: 1.2; letter-spacing: -.04em; }
.gallery-intro h1 em { color: var(--brand-primary); font-style: normal; }
.gallery-intro p { margin: 0; color: var(--brand-muted); font-size: 15px; }
.intro-index { display: flex; align-items: center; gap: 16px; color: rgb(49 95 232 / 32%); transform: rotate(-8deg); }
.intro-index span { font-family: var(--brand-display-font); font-size: 160px; line-height: .8; }
.intro-index small { color: var(--brand-muted); font-size: 9px; letter-spacing: .14em; line-height: 1.7; }
#homePage .search-bar { max-width: 720px; margin: 0 auto 32px; }
#homePage .search-bar :deep(.ant-input), #homePage .search-bar :deep(.ant-input-search-button) { height: 52px; }
.gallery-filters { margin-bottom: 24px; }
.gallery-filters :deep(.ant-tabs-nav) { margin-bottom: 10px; }
.tag-bar { display: flex; align-items: baseline; gap: 14px; flex-wrap: wrap; padding-top: 4px; }
.filter-label { color: var(--brand-muted); font-size: 12px; }
.results-heading { display: flex; justify-content: space-between; align-items: center; margin: 24px 0 16px; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 21px; font-weight: 600; }
.results-heading span + span { color: var(--brand-muted); font-family: var(--brand-ui-font); font-size: 12px; font-weight: 400; }
@media (max-width: 640px) {
  .gallery-intro { min-height: 220px; padding: 30px 24px; border-radius: 16px; }
  .gallery-intro h1 { font-size: 32px; }
  .intro-index { position: absolute; right: -24px; bottom: 0; opacity: .18; }
  .intro-index span { font-size: 110px; }
  .gallery-intro p { max-width: 230px; }
  #homePage .search-bar { margin-bottom: 20px; }
  .results-heading { margin-top: 16px; }
}
</style>
