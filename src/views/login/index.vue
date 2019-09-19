/* eslint-disable no-useless-return */
/* eslint-disable no-undef */
/* eslint-disable no-undef */
<template>
  <div class="login-warp">
    <div class="login-form-warp">
      <div class="login-head">
        <img src="./logo_index.png" alt />
      </div>
      <div class="login-form">
        <!--表单验证：
        rules配置验证规则
        将需要验证的字段通过prop属性配置到el-form-item组件上
        ref 获取表单组件，可以手动调用表单组件的验证方法
        -->
        <el-form :rules="rules" ref="ruleForm" :model="form">
          <el-form-item prop="mobile">
            <el-input v-model="form.mobile" placeholder="手机号"></el-input>
          </el-form-item>
          <el-form-item prop="code">
            <el-col :span="12">
              <el-input v-model="form.code" placeholder="验证码"></el-input>
            </el-col>
            <el-col :span="10" :offset="2">
              <el-button type="primary" @click="handleSendCode">获取验证码</el-button>
            </el-col>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" class="btn-login" @click="handleLogin">登录</el-button>
          </el-form-item>
        </el-form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import '@/vendor/gt.js';
export default {
  name: 'AppLogin',
  data () {
    return {
      form: {
        mobile: '',
        code: '',
        captchaObj: null
      },
      rules: {
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { len: 11, message: '长度必须11位', trigger: 'blur' }
        ],
        code: [
          { required: true, message: '请输入验证码', trigger: 'blur' },
          { len: 6, message: '长度6个字符', trigger: 'blur' }
        ]
      }
    };
  },
  methods: {
    handleLogin () {
      this.$refs['ruleForm'].validate(vaild => {
        if (!vaild) {
          return
        }
        this.submitLogin()
      })
    },
    submitLogin () {
      axios({
        method: 'POST',
        url: 'http://ttapi.research.itcast.cn/mp/v1_0/authorizations',
        data: this.form
      }).then(res => {
        // >=200$<=400的状态码都会进入这里
        // Element中提供的Message消息提示组件，这也是组件调用的一种形式
        this.$message({
          message: '登录成功',
          type: 'success'
        });
        // 建议路由跳转都使用name去跳转，路由传参非常方便
        this.$router.push({
          name: 'home'
        })
      }).catch(err => {
        // >=400的状态码都会进入这里
        // console.dir(err) 打印出来的结果总response中有status
        if (err.response.status === 400) {
          this.$message.error('错了哦，这是一条错误消息');
        }
      })
    },

    handleSendCode () {
      const { mobile } = this.form;
      if (this.captchaObj) {
        return this.captchaObj.verify();
      }
      axios({
        method: 'GET',
        url: `http://ttapi.research.itcast.cn/mp/v1_0/captchas/${mobile}`
      }).then(res => {
        const data = res.data.data;
        window.initGeetest(
          {
            // 以下配置参数来自服务端 SDK
            gt: data.gt,
            challenge: data.challenge,
            offline: !data.success,
            new_captcha: true,
            product: 'bind'
          },
          captchaObj => {
            this.captchaObj = captchaObj;
            // 这里可以调用验证实例 captchaObj 的实例方法
            // console.log(captchaObj);
            captchaObj
              .onReady(function () {
                // 验证码ready之后才能调用verify方法显示验证码
                captchaObj.verify(); // 显示验证码
              })
              .onSuccess(function () {
                // console.log('验证成功了')
                const {
                  geetest_challenge: challenge,
                  geetest_seccode: seccode,
                  geetest_validate: validate
                } = captchaObj.getValidate();
                axios({
                  method: 'GET',
                  url: `http://ttapi.research.itcast.cn/mp/v1_0/sms/codes/${mobile}`,
                  parmas: {
                    challenge,
                    seccode,
                    validate
                  }
                }).then(res => {
                  console.log(res.data);
                });
              });
          }
        );
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
