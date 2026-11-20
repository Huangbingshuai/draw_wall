<template>
  <div id="searchPicturePage">
    <header class="page-heading">
      <div>
        <span class="eyebrow">视觉探索</span>
        <h1>以图搜图</h1>
        <p>从这张壁纸出发，寻找相似的灵感。</p>
      </div>
    </header>

    <section v-if="!pictureId" class="missing-source">
      <a-empty description="先从壁纸画廊选择一张图片，再使用以图搜图" />
      <div class="missing-source-action">
        <router-link to="/"><a-button type="primary">返回壁纸画廊</a-button></router-link>
      </div>
    </section>

    <template v-else>
      <a-alert v-if="detailError" class="search-error" type="error" show-icon :message="detailError" />
      <div v-else-if="detailLoading" class="detail-loading"><a-spin size="large" /></div>
      <template v-else-if="picture.url">
        <h2 class="section-title">搜索来源</h2>
        <a-card class="source-picture">
          <template #cover>
            <img :alt="picture.name || '待识别的壁纸'" :src="picture.thumbnailUrl ?? picture.url" />
          </template>
          <a-card-meta :title="picture.name || '未命名壁纸'" />
        </a-card>

        <div class="results-heading">
          <h2 class="section-title">识图结果</h2>
          <span v-if="!searchLoading && !searchError">{{ dataList.length }} 张相似图片</span>
        </div>
        <a-alert
          v-if="searchError"
          class="search-error"
          type="error"
          show-icon
          :message="searchError"
        />
        <a-list
          v-else-if="searchLoading || dataList.length > 0"
          :grid="{ gutter: 16, xs: 1, sm: 2, md: 3, lg: 4, xl: 5, xxl: 6 }"
          :data-source="dataList"
          :loading="searchLoading"
        >
          <template #renderItem="{ item: result }">
            <a-list-item class="result-item">
              <a
                class="result-link"
                :href="result.fromUrl"
                target="_blank"
                rel="noopener noreferrer"
                :aria-label="`打开图片来源：${result.name || '未命名图片'}`"
              >
                <a-card class="result-picture" hoverable>
                  <template #cover>
                    <img :alt="result.name || '相似图片'" :src="result.thumbUrl" />
                  </template>
                  <a-card-meta :title="result.name || '查看图片来源'" />
                </a-card>
              </a>
            </a-list-item>
          </template>
        </a-list>
        <a-empty
          v-if="!searchLoading && !searchError && dataList.length === 0"
          description="暂时没有找到相似图片"
        />
      </template>
    </template>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue'
import {
  getPictureVoByIdUsingGet,
  searchPictureByPictureUsingPost,
} from '@/api/pictureController.ts'
import { useRoute } from 'vue-router'

const route = useRoute()

const pictureId = computed(() => {
  const value = route.query?.pictureId
  if (typeof value !== 'string') {
    return undefined
  }
  const id = Number(value)
  return Number.isInteger(id) && id > 0 ? id : undefined
})

const picture = ref<API.PictureVO>({})
const dataList = ref<API.ImageSearchResult[]>([])
const detailLoading = ref(false)
const searchLoading = ref(false)
const detailError = ref('')
const searchError = ref('')

const fetchPictureDetail = async (id: number) => {
  detailLoading.value = true
  detailError.value = ''
  try {
    const res = await getPictureVoByIdUsingGet({ id })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
      return true
    }
    detailError.value = '获取图片详情失败，' + (res.data.message || '请稍后重试')
  } catch (error: unknown) {
    detailError.value = '获取图片详情失败：' + (error instanceof Error ? error.message : '请稍后重试')
  } finally {
    detailLoading.value = false
  }
  return false
}

const fetchResultData = async (id: number) => {
  searchLoading.value = true
  searchError.value = ''
  try {
    const res = await searchPictureByPictureUsingPost({ pictureId: id })
    if (res.data.code === 0 && res.data.data) {
      dataList.value = res.data.data
    } else {
      searchError.value = '获取识图结果失败，' + (res.data.message || '请稍后重试')
    }
  } catch (error: unknown) {
    searchError.value = '获取识图结果失败：' + (error instanceof Error ? error.message : '请稍后重试')
  } finally {
    searchLoading.value = false
  }
}

watch(
  pictureId,
  async (id) => {
    picture.value = {}
    dataList.value = []
    detailError.value = ''
    searchError.value = ''
    if (!id) {
      detailLoading.value = false
      searchLoading.value = false
      return
    }
    if (await fetchPictureDetail(id)) {
      await fetchResultData(id)
    }
  },
  { immediate: true },
)
</script>

<style scoped>
#searchPicturePage { min-height: 62vh; margin-bottom: 16px; }
.eyebrow { color: var(--brand-primary); font-size: 11px; letter-spacing: .14em; }
.page-heading h1 { margin: 6px 0; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 30px; }
.page-heading p { margin: 0; color: var(--brand-muted); }
.missing-source { margin: 56px auto; padding: 40px 20px; background: var(--brand-surface); border: 1px dashed var(--brand-line); border-radius: var(--brand-radius); }
.missing-source-action { display: flex; justify-content: center; margin-top: 18px; }
.detail-loading { display: grid; min-height: 280px; place-items: center; }
.search-error { margin: 28px 0; }
.section-title { margin: 28px 0 12px; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 21px; }
.source-picture { width: min(100%, 360px); margin-bottom: 28px; overflow: hidden; }
.source-picture :deep(.ant-card-cover img), .result-picture :deep(.ant-card-cover img) { display: block; width: 100%; height: 210px; object-fit: cover; background: #edf0ed; }
.results-heading { display: flex; align-items: baseline; justify-content: space-between; gap: 16px; }
.results-heading span { color: var(--brand-muted); font-size: 12px; }
.result-item { padding: 0; }
.result-link { display: block; width: 100%; color: inherit; text-decoration: none; }
.result-picture { overflow: hidden; }
.result-picture :deep(.ant-card-cover img) { transition: transform .3s ease; }
.result-picture:hover :deep(.ant-card-cover img) { transform: scale(1.04); }
@media (max-width: 600px) {
  .missing-source { margin: 32px auto; padding: 28px 12px; }
  .source-picture :deep(.ant-card-cover img), .result-picture :deep(.ant-card-cover img) { height: 180px; }
}
@media (prefers-reduced-motion: reduce) { .result-picture :deep(.ant-card-cover img) { transition: none; } }
</style>
