<template>
  <div id="userRegisterPage">
    <router-link class="auth-brand" to="/"><img src="/favicon.svg" alt="" />汇图壁纸网</router-link>
    <span class="eyebrow">START YOUR COLLECTION</span>
    <h2 class="title">创建你的账户</h2>
    <div class="desc">收藏风景，也收藏每一次灵感。</div>
    <a-form :model="formState" name="basic" autocomplete="off" @finish="handleSubmit">
      <a-form-item name="userAccount" :rules="[{ required: true, message: '请输入账号' }]">
        <a-input v-model:value="formState.userAccount" placeholder="请输入账号" />
      </a-form-item>
      <a-form-item
        name="userPassword"
        :rules="[
          { required: true, message: '请输入密码' },
          { min: 8, message: '密码长度不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.userPassword" placeholder="请输入密码" />
      </a-form-item>
      <a-form-item
        name="checkPassword"
        :rules="[
          { required: true, message: '请输入确认密码' },
          { min: 8, message: '确认密码长度不能小于 8 位' },
        ]"
      >
        <a-input-password v-model:value="formState.checkPassword" placeholder="请输入确认密码" />
      </a-form-item>
      <div class="tips">
        已有账号？
        <RouterLink to="/user/login">去登录</RouterLink>
      </div>
      <a-form-item>
        <a-button type="primary" html-type="submit" style="width: 100%">注册</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>
<script lang="ts" setup>
import { reactive } from 'vue'
import { userRegisterUsingPost } from '@/api/userController.ts'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { message } from 'ant-design-vue'
import router from '@/router' // 用于接受表单输入的值

// 用于接受表单输入的值
const formState = reactive<API.UserRegisterRequest>({
  userAccount: '',
  userPassword: '',
  checkPassword: '',
})

const loginUserStore = useLoginUserStore()

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: any) => {
  // 校验两次输入的密码是否一致
  if (values.userPassword !== values.checkPassword) {
    message.error('两次输入的密码不一致')
    return
  }
  const res = await userRegisterUsingPost(values)
  // 注册成功，跳转到登录页面
  if (res.data.code === 0 && res.data.data) {
    message.success('注册成功')
    router.push({
      path: '/user/login',
      replace: true,
    })
  } else {
    message.error('注册失败，' + res.data.message)
  }
}
</script>

<style scoped>
#userRegisterPage {
  max-width: 440px; margin: 2vh auto; padding: 34px 40px 24px;
  background: white; border: 1px solid var(--brand-line); border-radius: 20px;
  box-shadow: var(--brand-shadow);
}
.auth-brand { display: inline-flex; align-items: center; gap: 9px; margin-bottom: 30px; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 19px; font-weight: 700; text-decoration: none; }
.auth-brand img { width: 36px; height: 36px; }
.eyebrow { display: block; margin-bottom: 10px; color: var(--brand-primary); font-size: 10px; letter-spacing: .16em; }

.title {
  text-align: left; margin-bottom: 8px; font-family: var(--brand-display-font); font-size: 30px;
}

.desc {
  text-align: center;
  color: var(--brand-muted); margin-bottom: 24px;
}

.tips {
  color: var(--brand-muted);
  text-align: right;
  font-size: 13px;
  margin-bottom: 16px;
}
@media (max-width: 520px) { #userRegisterPage { margin: 0; padding: 26px 22px; border-radius: 16px; } .auth-brand { margin-bottom: 24px; } }
</style>
