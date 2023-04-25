<!-- eslint-disable no-console -->
<script  lang="ts">
import { ref } from 'vue'

export default {
  setup() {
    const formData = ref({
      phoneNumber: '',
      username: '',
      password: '',
      confirmPassword: '',
      verificationCode: '',
    })

    const formRules = ref({
      phoneNumber: [
        { required: true, message: '请输入手机号' },
        { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确' },
      ],
      username: [
        { required: true, message: '请输入用户名' },
        { min: 4, max: 16, message: '用户名长度在4到16个字符之间' },
      ],
      password: [
        { required: true, message: '请输入密码' },
        { min: 6, max: 32, message: '密码长度在6到32个字符之间' },
      ],
      confirmPassword: [
        { required: true, message: '请再次输入密码' },
        {
          validator: (rule: any, value: string) => {
            return value === formData.value.password
          },
          message: '两次输入的密码不一致',
        },
      ],
      verificationCode: [{ required: true, message: '请输入验证码' }],
    })

    const isSignUp = ref(false)
    const countdown = ref(0)

    const handleSubmit = () => {
      console.log('Form submitted:', formData.value)
      // 调用API进行登录或注册
    }

    const sendVerificationCode = () => {
      if (countdown.value > 0)
        return
      console.log('Sending verification code to:', formData.value.phoneNumber)
      // 调用API发送验证码
      countdown.value = 60
      const timer = setInterval(() => {
        countdown.value--
        if (countdown.value === 0)
          clearInterval(timer)
      }, 1000)
    }

    return {
      formData,
      formRules,
      isSignUp,
      countdown,
      handleSubmit,
      sendVerificationCode,
    }
  },
}
</script>

<template>
  <n-config-provider>
    <n-space vertical>
      <n-card :style="{ width: '320px', margin: 'auto' }">
        <template #header>
          <div class="header">
            <h3 v-if="!isSignUp">
              登录
            </h3>
            <h3 v-else>
              注册
            </h3>
            <n-button v-if="!isSignUp" type="text" @click="isSignUp = true">
              注册
            </n-button>
          </div>
        </template>
        <n-space vertical align="center">
          <n-form :model="formData" :rules="formRules" @submit="handleSubmit">
            <n-form-item label="手机号" prop="phoneNumber">
              <n-input v-model="formData.phoneNumber" placeholder="请输入手机号" />
            </n-form-item>
            <n-form-item v-if="isSignUp" label="用户名" prop="username">
              <n-input v-model="formData.username" placeholder="请输入用户名" />
            </n-form-item>
            <n-form-item v-if="isSignUp" label="密码" prop="password">
              <n-input v-model="formData.password" placeholder="请输入密码" type="password" />
            </n-form-item>
            <n-form-item v-if="isSignUp" label="确认密码" prop="confirmPassword">
              <n-input v-model="formData.confirmPassword" placeholder="请再次输入密码" type="password" />
            </n-form-item>
            <n-form-item label="验证码" prop="verificationCode">
              <n-input-group>
                <n-input v-model="formData.verificationCode" placeholder="请输入验证码" />
                <n-button :disabled="countdown > 0" @click="sendVerificationCode">
                  {{ countdown > 0 ? `${countdown}秒后可重发` : '发送验证码' }}
                </n-button>
              </n-input-group>
            </n-form-item>
            <n-form-item>
              <n-button type="primary" native-type="submit" :disabled="!formData.verificationCode">
                {{ isSignUp ? '注册' : '登录' }}
              </n-button>
            </n-form-item>
          </n-form>
          <div v-if="!isSignUp" style="text-align: center;">
            <n-button type="text" @click="isSignUp = false">
              切换密码登录
            </n-button>
          </div>
        </n-space>
      </n-card>
    </n-space>
  </n-config-provider>
</template>

<style scoped>
    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
</style>
