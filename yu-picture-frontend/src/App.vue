<template>
  <div id="app">
    <a-config-provider
      :locale="zhCN"
      :theme="{
        token: {
          colorPrimary: '#315FE8',
          colorText: '#17252D',
          colorTextSecondary: '#66747A',
          colorBorder: '#DCE3E3',
          borderRadius: 10,
          fontFamily: 'PingFang SC, Noto Sans SC, Microsoft YaHei, sans-serif',
        },
      }"
    >
      <BasicLayout />
    </a-config-provider>
  </div>
</template>

<script setup lang="ts">
import zhCN from 'ant-design-vue/es/locale/zh_CN';
import dayjs from 'dayjs';
import 'dayjs/locale/zh-cn';
dayjs.locale('zh-cn');

import BasicLayout from '@/layouts/BasicLayout.vue'
// import { healthUsingGet } from '@/api/mainController.ts'
// import {useLoginUserStore} from "@/stores/useLoginUserStore.ts";

// 已经改为在权限校验文件中获取
// const loginUserStore = useLoginUserStore()
// loginUserStore.fetchLoginUser()

// healthUsingGet().then((res) => {
//   console.log(res)
// })
</script>

<style scoped></style>
