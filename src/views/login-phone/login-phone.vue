<!-- eslint-disable no-console -->
<script lang="ts" setup>
import { ref } from 'vue'
// 引入接口 api
import { getCaptcha, phoneLogin } from '@/api'
const emits = defineEmits(['close-login', 'loginSuccess'])

const phone = ref('')
const code = ref('')
const sendCaptchaStatus = ref(true)
const phoneErr = ref('')
const codeErr = ref('')
const sucTip = ref('')
const num = ref(60)

function close() {
  // emits('close-login')
}
// 验证手机号
function _checkPhone() {
  const reg = /^\d{11}$/
  if (!reg.test(phone.value)) {
    phoneErr.value = '请输入正确格式的手机号'
    return false
  }
  else {
    return true
  }
}
// 倒计时
function captchaCountdown() {
  num.value = 60
  const Timer = setInterval(() => {
    if (num.value === 0) {
      clearInterval(Timer)
      sendCaptchaStatus.value = true
    }
    else {
      num.value = --num.value
    }
  }, 1000)
}
function focusFn() {
  phoneErr.value = ''
}
function inputCode(e) {
  code.value = e.detail.value
  codeErr.value = ''
}
function inputNumber(e) {
  phone.value = e.detail.value
  sucTip.value = ''
  phoneErr.value = ''
  codeErr.value = ''
}
// 发送验证码
function sendCaptcha() {
  if (!phone.value)
    return

  if (!_checkPhone()) {
    phoneErr.value = '请输入正确格式的手机号'
    return
  }
  sendCaptchaStatus.value = false
  sucTip.value = ''
  codeErr.value = ''
  getCaptcha(phone.value)
    .then((res: any) => {
      if (res.err_code === 0) {
        sendCaptchaStatus.value = false
        sucTip.value = res.msg
        captchaCountdown()
      }
      else if (res.err_code === 2007) {
        sendCaptchaStatus.value = true
        sucTip.value = ''
        codeErr.value = res.err_msg
      }
      else if (res.err_msg.includes('频繁')) {
        sendCaptchaStatus.value = true
        sucTip.value = ''
        codeErr.value = res.err_msg
      }
      else if (res.err_msg.includes('短信')) {
        sendCaptchaStatus.value = true
        sucTip.value = ''
        codeErr.value = res.err_msg
      }
      else {
        if (res.err_code === 308) {
          sendCaptchaStatus.value = true
          sucTip.value = ''
          codeErr.value = '验证码发送失败！请检查手机号是否正确！'
        }
        else {
          sendCaptchaStatus.value = true
          sucTip.value = ''
          codeErr.value = res.err_msg || '服务出错，code94'
        }
      }
    })
    .catch(() => {
      sendCaptchaStatus.value = true
      sucTip.value = ''
      codeErr.value = '短信发送失败，请重试'
    })
}

// 点击登录
function login() {
  if (!phone.value || !code.value)
    return

  if (!_checkPhone()) {
    phoneErr.value = '请输入正确格式的手机号'
    return
  }
  if (!/\d{4}$/.test(code.value)) {
    codeErr.value = '请输入正确格式的验证码'
    return
  }
  phoneLogin({ phone: phone.value, secret: code.value }).then((res: any) => {
    console.log('---手机号登录结果', res)
    if (res.err_code === 0) {
      // showToast({
      //   icon: 'none',
      //   title: '登录成功',
      // })
      sessionStorage.setItem('wwid', res.base_info.wwid)
      sessionStorage.setItem('phone', res.base_info.phone)
      sessionStorage.setItem('token', res.token)
      emits('loginSuccess')
      // 回到首页,暂时改成tools隐藏
      // reLaunch({
      //   url:'/pages/home-new/home-new'
      // });
    }
    else if (res.code == 40401500) {
      // showModal({
      //   title: '登录异常',
      //   content: res.message || '登录失败',
      //   showCancel: false,
      //   confirmText: '知道了',
      //   confirmColor: '#5A92F4',
      // })
    }
    else {
      if (res.err_msg && res.err_msg.includes('验证码'))
        codeErr.value = '验证码不正确，请检查短信内容'

      else
        codeErr.value = '验证码不正确，请检查短信内容'
    }
  }).catch((err) => {
    console.log(err)

    // showToast({
    //   icon: 'none',
    //   title: '登录失败请重试',
    // })
  })
}
</script>

<template>
  <!--   手机号登录 -->
  <div class="login-phone-content" @click="() => { }">
    <div class="login-title">
      验证码登录
    </div>
    <div class="desc">
      使用手机号快速登录，无账号将自动注册
    </div>
    <div :class="!!phoneErr ? 'phone-box err-border' : 'phone-box'">
      <input
        always-embed="true" class="phone" :value="phone" maxlength="11" placeholder-style="phone-plcholder" type="number"
        placeholder="请输入手机号" @focus="focusFn" @input="inputNumber"
      >
    </div>
    <div class="err-tip">
      {{ phoneErr }}
    </div>
    <div class="box">
      <div :class="!!codeErr ? 'login-code-box err-border' : 'login-code-box'">
        <input
          always-embed="true" class="code" :value="code" placeholder-style="phone-plcholder" type="text"
          placeholder="请输入验证码" @input="inputCode"
        >
      </div>
      <div v-if="sendCaptchaStatus" :class="!!phone ? 'code-send not-disable' : 'code-send'" @click="sendCaptcha">
        获取验证码
      </div>
      <div v-else class="code-send not-disable">
        {{ num }}s后重新发送
      </div>
    </div>
    <div v-if="!!codeErr" class="err-tip">
      {{ codeErr }}
    </div>
    <div v-else class="err-tip suc-tip">
      {{ sucTip }}
    </div>
    <div :class="(!!phone && !!code) ? 'button can-login' : 'button'" @click="login">
      登录
    </div>
  </div>
</template>

<style lang="less">
.login-phone-content {
  width: 100vw;
  background-color: #ffffff;
  padding: 25px 25px;
  box-sizing: border-box;

  .login-title {
    height: 25px;
    font-size: 18px;
    font-family: PingFang SC;
    font-weight: 500;
    color: #333333;
    line-height: 25px;
  }

  .desc {
    height: 21px;
    font-size: 15px;
    font-family: PingFang SC;
    font-weight: 400;
    color: #666666;
    line-height: 21px;
    margin-top: 15px;
  }

  .phone-box {
    border: 1px solid #CCCCCC;
    margin-top: 60px;
    width: 325px;
    height: 42px;
    border-radius: 21px;
  }

  .phone {
    width: 100%;
    height: 100%;
    background: #FFFFFF;
    border-radius: 21px;
    padding: 0 15px;
    box-sizing: border-box;
    color: #333333;
    font-size: 15px;
  }

  .phone-plcholder {
    font-size: 15px;
    font-family: PingFang SC;
    font-weight: 400;
    color: #999999;
  }

  .phone:focus {
    border-color: #3583FB;
  }

  .box {
    // margin-top: 10px;
    display: flex;
    justify-content: space-between;
  }

  .login-code-box {
    width: 208px;
    height: 42px;
    border-radius: 21px;
    border: 1px solid #CCCCCC;
    background: #FFFFFF;

  }

  .code {
    width: 100%;
    height: 100%;
    border-radius: 21px;
    font-size: 15px;
    font-family: PingFang SC;
    font-weight: 400;
    padding: 0 15px;
    box-sizing: border-box;
    color: #333333;
    font-size: 15px;
  }

  .code-send {
    width: 107px;
    height: 42px;
    background: #EBEBEB;
    border-radius: 21px;
    font-size: 15px;
    font-family: PingFang SC;
    font-weight: 400;
    color: #999999;
    line-height: 42px;
    text-align: center;
  }

  .not-disable {
    background: #2D89FF linear-gradient(90deg, #23C0FF 0%, #2384FF 100%);
    color: #FFFFFF;
  }

  .num {
    background: #ffffff;
    border: 1px solid #cccccc;
    color: #333333;
  }

  .button {
    width: 325px;
    height: 42px;
    background: linear-gradient(90deg, #23C0FF 0%, #2384FF 100%);
    border-radius: 21px;
    opacity: 0.39;

    font-size: 15px;
    font-family: PingFang SC;
    font-weight: 400;
    color: #FFFFFF;
    line-height: 42px;
    text-align: center;
    height: 44px;
    margin-top: 20px;
  }

  .err-tip {
    height: 18px;
    font-size: 13px;
    font-family: PingFang SC;
    font-weight: 400;
    color: #E02020;
    line-height: 18px;
    margin-left: 15px;
  }

  .suc-tip {
    color: #6DD400;
  }

  .can-login {
    opacity: 1;
    background: linear-gradient(90deg, #23C0FF 0%, #2384FF 100%);
    color: #FFFFFF;
  }

  .err-border {
    border-color: #E02020;
  }
}
</style>
