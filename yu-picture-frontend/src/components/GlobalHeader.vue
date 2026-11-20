<template>
  <header id="globalHeader">
    <router-link class="brand-link" to="/" aria-label="汇图壁纸网首页">
      <img class="brand-mark" :src="brandMark" alt="" />
      <span class="brand-name">汇图壁纸网</span>
    </router-link>

    <nav class="desktop-nav" aria-label="主导航">
      <a-menu
        v-model:selectedKeys="current"
        mode="horizontal"
        :items="items"
        @click="doMenuClick"
      />
    </nav>

    <div class="header-actions">
      <a-dropdown v-if="loginUserStore.loginUser.id" placement="bottomRight">
        <button class="account-trigger" type="button" aria-label="打开账户菜单">
          <a-avatar :src="loginUserStore.loginUser.userAvatar" />
          <span class="account-name">{{ loginUserStore.loginUser.userName ?? '我的账户' }}</span>
          <DownOutlined class="account-chevron" />
        </button>
        <template #overlay>
          <a-menu>
            <a-menu-item key="/my_space" @click="router.push('/my_space')">
              <UserOutlined />
              我的空间
            </a-menu-item>
            <a-menu-item key="logout" @click="doLogout">
              <LogoutOutlined />
              退出登录
            </a-menu-item>
          </a-menu>
        </template>
      </a-dropdown>
      <a-button v-else class="login-button" type="primary" href="/user/login">登录</a-button>

      <a-dropdown class="mobile-nav" :trigger="['click']" placement="bottomRight">
        <a-button aria-label="打开导航菜单" shape="circle">
          <template #icon><MenuOutlined /></template>
        </a-button>
        <template #overlay>
          <a-menu :selected-keys="current" :items="items" @click="doMenuClick" />
        </template>
      </a-dropdown>
    </div>
  </header>
</template>

<script lang="ts" setup>
import { computed, h, ref } from 'vue'
import { useRouter } from 'vue-router'
import {
  DownOutlined,
  HomeOutlined,
  LogoutOutlined,
  MenuOutlined,
  PictureOutlined,
  SearchOutlined,
  UploadOutlined,
  UserOutlined,
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import type { MenuProps } from 'ant-design-vue'
import brandMark from '@/assets/brand-mark.svg'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { userLogoutUsingPost } from '@/api/userController.ts'

const loginUserStore = useLoginUserStore()
const router = useRouter()

const originItems = [
  { key: '/', icon: () => h(HomeOutlined), label: '壁纸画廊', title: '壁纸画廊' },
  { key: '/search_picture', icon: () => h(SearchOutlined), label: '以图搜图', title: '以图搜图' },
  { key: '/add_picture', icon: () => h(UploadOutlined), label: '上传壁纸', title: '上传壁纸' },
  { key: '/admin/userManage', label: '用户管理', title: '用户管理' },
  { key: '/admin/pictureManage', label: '图片管理', title: '图片管理' },
  { key: '/admin/spaceManage', label: '空间管理', title: '空间管理' },
]

const items = computed(() =>
  originItems.filter((menu) => {
    if (menu.key.startsWith('/admin/')) {
      return loginUserStore.loginUser.userRole === 'admin'
    }
    return true
  }) as MenuProps['items'],
)

const current = ref<string[]>([])
router.afterEach((to) => {
  current.value = [to.path]
})

const doMenuClick = ({ key }: { key: string }) => {
  router.push({ path: key })
}

const doLogout = async () => {
  const res = await userLogoutUsingPost()
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({ userName: '未登录' })
    message.success('已退出登录')
    await router.push('/user/login')
    return
  }
  message.error('退出登录失败，' + res.data.message)
}
</script>

<style scoped>
#globalHeader {
  display: flex;
  align-items: center;
  gap: 36px;
  min-height: 76px;
}

.brand-link {
  display: inline-flex;
  flex: 0 0 auto;
  align-items: center;
  gap: 10px;
  color: var(--brand-ink);
  text-decoration: none;
}

.brand-mark {
  width: 42px;
  height: 42px;
}

.brand-name {
  font-family: var(--brand-display-font);
  font-size: 21px;
  font-weight: 700;
  letter-spacing: 0.025em;
  white-space: nowrap;
}

.desktop-nav {
  flex: 1 1 auto;
  min-width: 0;
}

.desktop-nav :deep(.ant-menu) {
  color: var(--brand-muted);
  background: transparent;
  border: 0;
}

.desktop-nav :deep(.ant-menu-item) {
  padding-inline: 14px;
}

.header-actions {
  display: flex;
  flex: 0 0 auto;
  align-items: center;
  gap: 12px;
}

.account-trigger {
  display: inline-flex;
  align-items: center;
  gap: 9px;
  padding: 5px 8px 5px 5px;
  color: var(--brand-ink);
  background: transparent;
  border: 0;
  border-radius: 999px;
  cursor: pointer;
}

.account-trigger:hover {
  background: var(--brand-canvas);
}

.account-name {
  max-width: 140px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.account-chevron {
  color: var(--brand-muted);
  font-size: 11px;
}

.login-button {
  min-width: 86px;
}

.mobile-nav {
  display: none;
}

@media (max-width: 900px) {
  #globalHeader {
    gap: 18px;
  }

  .brand-name {
    font-size: 18px;
  }

  .desktop-nav :deep(.ant-menu-item) {
    padding-inline: 9px;
  }
}

@media (max-width: 680px) {
  #globalHeader {
    min-height: 64px;
    gap: 12px;
  }

  .brand-mark {
    width: 36px;
    height: 36px;
  }

  .brand-name {
    font-size: 17px;
  }

  .desktop-nav {
    display: none;
  }

  .header-actions {
    margin-left: auto;
    gap: 8px;
  }

  .mobile-nav {
    display: inline-flex;
  }

  .account-name,
  .account-chevron {
    display: none;
  }
}
</style>
