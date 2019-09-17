/* eslint-disable no-undef */
/* eslint-disable no-undef */
<template>
  <div class="login-warp">
    <div class="login-form-warp">
      <div class="login-head">
        <img src="./logo_index.png" alt />
      </div>
      <div class="login-form">
        <el-form ref="form" :model="form">
          <el-form-item>
            <el-input v-model="form.mobile" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item>
            <el-col :span="12">
              <el-input v-model="form.code" placeholder="验证码"></el-input>
            </el-col>
            <el-col :span="10" :offset="2">
              <el-button type="primary" @click="handleSendCode">获取验证码</el-button>
            </el-col>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" class="btn-login" @click="onSubmit">登录</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import '@/vendor/gt.js'
export default {
  name: 'AppLogin',
  data () {
    return {
      form: {
        mobile: '13683109553',
        code: ''
      }
    };
  },
  methods: {
    onSubmit () {
      console.log('submit!');
    },
    handleSendCode () {
      const { mobile } = this.form;
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        const data = res.data.data
        window.initGeetest({
          // 以下配置参数来自服务端 SDK
          gt: data.gt,
          challenge: data.challenge,
          offline: !data.success,
          new_captcha: true
        }, function (captchaObj) {
          // 这里可以调用验证实例 captchaObj 的实例方法
          console.log(captchaObj)
        })
      });
    }
  }
};
</script>

<style lang='less' scoped>
.login-warp {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: url(../../assets/login_bg.jpg);
  background-size: 100%;
  .login-form-warp {
    background-color: #fff;
    padding: 50px;
    .login-head {
      display: flex;
      justify-content: center;
      margin-bottom: 5px;
      img {
        width: 200px;
      }
    }
    .btn-login {
      width: 100%;
      border-radius: 10px;
    }
  }
}
</style>
