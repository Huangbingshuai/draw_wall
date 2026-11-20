<template>
  <div id="basicLayout" :class="{ 'is-workspace': isWorkspaceRoute, 'is-auth': isAuthRoute }">
    <a-layout class="app-layout">
      <a-layout-header v-if="!isAuthRoute" class="site-header">
        <div class="site-header-inner">
          <GlobalHeader />
        </div>
      </a-layout-header>
      <a-layout class="site-body">
        <GlobalSider v-if="showWorkspaceSider" class="workspace-sider" />
        <a-layout-content
          :class="[
            'site-content',
            isWorkspaceRoute ? 'site-content--workspace' : 'site-content--gallery',
          ]"
        >
          <main class="site-content-inner">
            <router-view />
          </main>
        </a-layout-content>
      </a-layout>
      <a-layout-footer v-if="!isAuthRoute" class="site-footer">
        <span>汇图壁纸网</span>
        <span aria-hidden="true">·</span>
        <span>为每一块屏幕收藏好风景</span>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useRoute } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'

const route = useRoute()
const loginUserStore = useLoginUserStore()

const isAuthRoute = computed(() => ['/user/login', '/user/register'].includes(route.path))
const isWorkspaceRoute = computed(() => {
  const workspacePrefixes = [
    '/add_picture',
    '/add_space',
    '/my_space',
    '/space/',
    '/space_analyze',
    '/spaceUserManage/',
    '/admin/',
    '/user_exchange_vip',
  ]
  return workspacePrefixes.some((prefix) => route.path.startsWith(prefix))
})
const showWorkspaceSider = computed(
  () => isWorkspaceRoute.value && Boolean(loginUserStore.loginUser.id),
)
</script>
