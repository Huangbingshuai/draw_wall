<template>
  <div class="share-modal">
    <a-modal v-model:visible="visible" :title="title" :footer="false" @cancel="closeModal">
      <div class="share-copy-block">
      <h4>复制分享链接</h4>
      <a-typography-link copyable>
        {{ link }}
      </a-typography-link>
      </div>
      <div class="share-qr-block"><h4>手机扫码查看</h4><a-qrcode :value="link || ''" /></div>
    </a-modal>
  </div>
</template>
<script lang="ts" setup>
import { ref } from 'vue'

interface Props {
  title: string;
  link: string;
}

const props = withDefaults(defineProps<Props>(), {
  title: "分享图片",
  link: '',
})

// 是否可见
const visible = ref(false)

// 打开弹窗
const openModal = () => {
  visible.value = true
}

// 关闭弹窗
const closeModal = () => {
  visible.value = false;
}

// 暴露函数给父组件
defineExpose({
  openModal,
})
</script>
<style scoped>
.share-copy-block { padding: 16px; background: var(--brand-canvas); border-radius: 12px; }
.share-copy-block h4, .share-qr-block h4 { margin: 0 0 10px; color: var(--brand-ink); }
.share-copy-block :deep(.ant-typography) { overflow-wrap: anywhere; }
.share-qr-block { display: flex; flex-direction: column; align-items: center; padding: 22px 8px 6px; }
</style>
