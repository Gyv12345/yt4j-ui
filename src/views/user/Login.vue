<template>
  <div class="main">
    <a-form ref="formLogin" :form="form" @submit="handleSubmit">
      <a-alert v-if="isLoginError" type="error" showIcon message="账号或密码不正确，请重新输入！" />
      <a-form-item class="form-item">
        <a-input
          size="large"
          type="text"
          placeholder="请输入账号"
          v-decorator="[
            'username',
            { rules: [{ required: true, message: '账号不能为空' }], validateTrigger: 'change' }
          ]"
        >
          <a-icon slot="prefix" type="user" class="form-icon" />
        </a-input>
      </a-form-item>
      <a-form-item>
        <a-input-password
          size="large"
          placeholder="请输入密码"
          v-decorator="[
            'password',
            { rules: [{ required: true, message: '密码不能为空' }], validateTrigger: 'blur' }
          ]"
        >
          <a-icon slot="prefix" type="lock" />
        </a-input-password>
      </a-form-item>
      <a-form-item>
        <a-button
          size="large"
          type="primary"
          htmlType="submit"
          class="login-button"
          :loading="state.loginBtn"
          :disabled="state.loginBtn"
        >
          确定
        </a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script>
import { encode } from '@/utils/rsa'
import { mapActions } from 'vuex'

export default {
  data () {
    return {
      loginBtn: false,
      isLoginError: false,
      form: this.$form.createForm(this),
      state: {
        loginBtn: false,
        smsSendBtn: false
      }
    }
  },
  methods: {
    ...mapActions(['Login']),
    handleSubmit (e) {
      e.preventDefault()
      const {
        form: { validateFields },
        state,
        Login
      } = this

      state.loginBtn = true
      validateFields((err, values) => {
        if (!err) {
          const loginParams = { ...values }
          delete loginParams.username
          loginParams['username'] = values.username
          loginParams.password = encode(values.password)
          Login(loginParams)
            .then(() => this.loginSuccess())
            .catch(err => this.requestFailed(err))
            .finally(() => {
              state.loginBtn = false
            })
        } else {
          setTimeout(() => {
            state.loginBtn = false
          }, 600)
        }
      })
    },
    loginSuccess () {
      this.$router.push({ path: '/sys-user/list' })
      this.isLoginError = false
    },
    requestFailed (err) {
      this.isLoginError = true
      this.$notification['error']({
        message: '错误',
        description: ((err.response || {}).data || {}).message || '请求出现错误，请稍后再试',
        duration: 4
      })
    }
  }
}
</script>

<style lang='less' scoped>
.form-item {
  margin: 24px 0;

  .form-icon {
    color: rgba(0, 0, 0, 0.25);
  }
}

.login-button {
  width: 100%;
}
</style>
