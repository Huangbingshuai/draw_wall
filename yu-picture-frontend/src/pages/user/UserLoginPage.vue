<template>
  <div id="userLoginPage">
    <router-link class="auth-brand" to="/"><img src="/favicon.svg" alt="" />汇图壁纸网</router-link>
    <span class="eyebrow">WALLPAPER COLLECTION</span>
    <h2 class="title">欢迎回来</h2>
    <div class="desc">登录后继续整理你的灵感与壁纸空间。</div>
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
      <div class="tips">
        没有账号？
        <RouterLink to="/user/register">去注册</RouterLink>
      </div>
      <a-form-item>
        <a-button type="primary" html-type="submit" style="width: 100%">登录</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>
<script lang="ts" setup>
import { reactive } from 'vue'
import { userLoginUsingPost } from '@/api/userController.ts'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { message } from 'ant-design-vue'
import router from '@/router' // 用于接受表单输入的值

// 用于接受表单输入的值
const formState = reactive<API.UserLoginRequest>({
  userAccount: '',
  userPassword: '',
})

const loginUserStore = useLoginUserStore()

/**
 * 提交表单
 * @param values
 */
const handleSubmit = async (values: any) => {
  const res = await userLoginUsingPost(values)
  // 登录成功，把登录态保存到全局状态中
  if (res.data.code === 0 && res.data.data) {
    await loginUserStore.fetchLoginUser()
    message.success('登录成功')
    router.push({
      path: '/',
      replace: true,
    })
  } else {
    message.error('登录失败，' + res.data.message)
  }
}
</script>

<style scoped>
#userLoginPage {
  max-width: 440px; margin: 4vh auto; padding: 38px 40px 28px;
  background: white; border: 1px solid var(--brand-line); border-radius: 20px;
  box-shadow: var(--brand-shadow);
}
.auth-brand { display: inline-flex; align-items: center; gap: 9px; margin-bottom: 42px; color: var(--brand-ink); font-family: var(--brand-display-font); font-size: 19px; font-weight: 700; text-decoration: none; }
.auth-brand img { width: 36px; height: 36px; }
.eyebrow { display: block; margin-bottom: 10px; color: var(--brand-primary); font-size: 10px; letter-spacing: .16em; }

.title {
  text-align: left; margin-bottom: 8px; font-family: var(--brand-display-font); font-size: 30px;
}

.desc {
  text-align: center;
  color: var(--brand-muted); margin-bottom: 28px;
}

.tips {
  color: var(--brand-muted);
  text-align: right;
  font-size: 13px;
  margin-bottom: 16px;
}
@media (max-width: 520px) { #userLoginPage { margin: 0; padding: 26px 22px; border-radius: 16px; } .auth-brand { margin-bottom: 34px; } }
</style>
