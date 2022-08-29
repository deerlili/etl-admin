<template>
  <div class="login-container">
    <el-form ref="loginForm" :model="loginForm" :rules="loginRules" class="login-form">
      <h1 class="loginTitle">数据抽取平台</h1>
      <p class="loginSystem">ETL-ADMIN</p>
      <el-form-item prop="username">
        <el-input
          ref="username"
          v-model="loginForm.username"
          placeholder="账号"
          name="username"
          type="text"
          tabindex="1"
          autocomplete="on"
        ><svg-icon slot="prefix" icon-class="peoples" />
        </el-input>
      </el-form-item>
      <el-form-item prop="password">
        <el-input
          :key="passwordType"
          ref="password"
          v-model="loginForm.password"
          :type="passwordType"
          placeholder="密码"
          name="password"
          tabindex="2"
          autocomplete="on"
          @keyup.native="checkCapslock"
          @keyup.enter.native="handleLogin"
        ><svg-icon slot="prefix" icon-class="password"/></el-input>
        <span class="show-pwd" @click="showPwd">
          <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
        </span>
      </el-form-item>
      <!-- <el-form-item>
        <drag-verify
          ref="dragVerifyRef"
          text="请按住滑块拖动解锁"
          success-text="验证通过"
          handler-icon="el-icon-d-arrow-right"
          success-icon="el-icon-circle-check"
          :width="435"
          style="margin-left: -100px;"
          handler-bg="#F5F7FA"
          :is-passing.sync="isPassing"
          @update:isPassing="updateIsPassing"
        />
      </el-form-item> -->
      <el-form-item class="loginButtonWrapper">
        <el-button class="loginButton" style="width:100%;" type="primary" :disabled="submitDisabled" @click.native.prevent="handleLogin">登录</el-button>
      </el-form-item>
    </el-form>
    <div style="position: absolute;bottom: 0;width: 100%;background: rgba(204, 204, 204, 0.3);height: 35px;">
      <p style="text-align: center;line-height: 5px;margin-top: 15px">版权所有©重庆半山智能科技有限公司 2019-2022 技术支持电话:17629371303 (<a href="http://www.freepik.com">Designed by fullvector/Freepik</a>)</p>
    </div>
  </div>
</template>
<script>

export default {
  name: 'Login',
  data() {
    const validatePassword = (rule, value, callback) => {
      if (value.length < 6) {
        callback(new Error('请输入您的密码(密码不能少于6位)'))
      } else {
        callback()
      }
    }
    return {
      loginForm: {
        username: '',
        password: ''
      },
      loginRules: {
        password: [{ required: true, trigger: 'blur', validator: validatePassword }]
      },
      passwordType: 'password',
      redirect: undefined,
      otherQuery: {}
    }
  },
  watch: {
    $route: {
      handler: function(route) {
        const query = route.query
        if (query) {
          this.redirect = query.redirect
          this.otherQuery = this.getOtherQuery(query)
        }
      },
      immediate: true
    },
    //  已验证通过时，若重新输入用户名或密码，滑动解锁恢复原样
    'loginForm.username'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    },
    'loginForm.password'() {
      this.isPassing = false
      this.$refs.dragVerifyRef.reset()
    }
  },
  created() {
  },
  mounted() {
    if (this.loginForm.username === '') {
      this.$refs.username.focus()
    } else if (this.loginForm.password === '') {
      this.$refs.password.focus()
    }
  },
  destroyed() {
  },
  methods: {
    checkCapslock({ shiftKey, key } = {}) {
      if (key && key.length === 1) {
        if (shiftKey && (key >= 'a' && key <= 'z') || !shiftKey && (key >= 'A' && key <= 'Z')) {
          this.capsTooltip = true
        } else {
          this.capsTooltip = false
        }
      }
      if (key === 'CapsLock' && this.capsTooltip === true) {
        this.capsTooltip = false
      }
    },
    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
      this.$nextTick(() => {
        this.$refs.password.focus()
      })
    },
    handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true
          this.$store.dispatch('user/login', this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || '/', query: this.otherQuery })
              this.loading = false
            })
            .catch(() => {
              this.loading = false
            })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    getOtherQuery(query) {
      return Object.keys(query).reduce((acc, cur) => {
        if (cur !== 'redirect') {
          acc[cur] = query[cur]
        }
        return acc
      }, {})
    }
  }
}
</script>

<style lang="scss" rel="stylesheet/scss">
.login-container {
  background-size: cover;
  display: flex;
  height: 100%;
  width: 100%;
  justify-content: right;
  align-items: center;
  background-image: url("../../assets/common/back.jpg");
}
</style>

<style lang="scss" scoped>
$bg:#2d3a4b;
$dark_gray:#889aa4;
$light_gray:#eee;

.login-container {
  min-height: 100%;
  width: 100%;
  text-align: center;
  overflow: hidden;
  background-size: cover;
  background-position: center;
  position: relative;
  .loginTitle {
    margin-bottom: 10px;
    font-weight: 400;
    font-size: 25px;
    color: #2F3FB0;
  }
  .loginSystem {
    font-weight: 300;
    font-size: 20px;
    color: rgb(63, 160, 144);
  }
  .login-form {
    border-radius: 6px;
    margin-right: 9%;
    margin-top: 6%;
    background: #ffffff;
    width: 450px;
    padding: 25px 25px 5px 25px;
    .el-input {
      height: 38px;
      input {
        height: 38px;
      }
    }

  }

  .tips {
    font-size: 14px;
    color: #fff;
    margin-bottom: 10px;

    span {
      &:first-of-type {
        margin-right: 16px;
      }
    }
  }

  .svg-container {
    padding: 6px 5px 6px 15px;
    color: $dark_gray;
    vertical-align: middle;
    width: 30px;
    display: inline-block;
  }

  .title-container {
    position: relative;

    .title {
      font-size: 26px;
      color: $light_gray;
      margin: 0px auto 40px auto;
      text-align: center;
      font-weight: bold;
    }
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 7px;
    font-size: 16px;
    color: $dark_gray;
    cursor: pointer;
    user-select: none;
  }

  .thirdparty-button {
    position: absolute;
    right: 0;
    bottom: 6px;
  }

  @media only screen and (max-width: 470px) {
    .thirdparty-button {
      display: none;
    }
  }
}
</style>
