<template>
  <div style="display: inline-block; margin-bottom: 60px;">
    <img class="main-img" src="" alt="">
    <div class="res">
      <div class="main-bot">
        <input class="login-input" @keyup.13="doLogin" v-model="loginForm.userName" type="text" placeholder="请输入帐号" />
        <input class="login-input" @keyup.13="doLogin" v-model="loginForm.password" type="password" placeholder="请输入密码" />
        <div class="image-code-div">
          <input type="text" v-model="loginForm.captchaCode" placeholder="验证码">
          <img v-lazy="imageCode" :key="imageCode" @click="getImage" />
        </div>
        <div class="main-password">
          <input id="button" type="checkbox" class="button" @click="rememberMeFn" :checked="rememberMe" value="Bike" title="" />
          <label for="button" class="remMe">记住我</label>
          <a class="fog" @click="loadPage('ResetPwdEmail')">忘记密码?</a>
        </div>
        <button class="login" @click="doLogin">登录</button>
        <div class="other">
          <hr />
          <p>其他方式登录</p>
          <hr />
        </div>
        <div class="last">
          <a href="#"><img src="@/assets/images/login/git1.png" alt=""></a>
          <a href="#"><img src="@/assets/images/login/git2.png" alt=""></a>
          <a href="#"><img src="@/assets/images/login/git3.png" alt=""></a>
          <a href="#"><img src="@/assets/images/login/git4.png" alt=""></a>
          <a href="#"><img src="@/assets/images/login/git5.png" alt=""></a>
        </div>
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
export default {
  data() {
    return {
      imageCode: "",
      deviceId: "",
      loginForm: {
        userName: "",
        password: "",
        captchaCode: ""
      }
    };
  },
  mounted() {
    this.getImage();
    if (this.rememberMe) {
      this.loginForm.userName = this.$store.getters.getLoginName;
    }
  },
  methods: {
    doLogin() {
      let validateResult = this.formValidate();
      if (!validateResult.status) {
        this.errorMsg(validateResult.msg, true);
        return;
      }
      this.login();
    },
    login() {
      let userName = this.loginForm.userName;
      let password = this.loginForm.password;
      this.$http({
        method: "POST",
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",
          deviceId: this.deviceId
        },
        url: "/aus/auth/form",
        auth: {
          username: "tgcloud-microservice-uac",
          password: "tgcloudClientSecret"
        },
        params: {
          username: userName,
          password: password,
          imageCode: this.loginForm.captchaCode
        }
      })
        .then(res => {
          if (res && res.code === 200) {
            this.$store.dispatch("update_auth_token", res.result);
            if (this.$store.getters.getRememberMe) {
              this.$store.dispatch("update_user_info", this.loginForm.userName);
            }
            window.location.href = this.redirectUri;
            return;
          }
          this.getImage();
        })
        .catch(err => {
          console.log(err);
        });
    },
    getImage() {
      this.deviceId = new Date().getTime();
      let that = this;
      that
        .$http({
          method: "GET",
          url: "aus/auth/code/image",
          headers: {
            deviceId: that.deviceId
          }
        })
        .then(res => {
          that.imageCode = "data:image/jpg;base64," + res.result;
        });
    },
    rememberMeFn() {
      this.$store.dispatch("update_remember_me");
    },
    formValidate() {
      let result = {
        status: false,
        msg: ""
      };
      if (!this.validate(this.loginForm.userName, "require")) {
        result.msg = "用户名不能为空";
        return result;
      }
      if (!this.validate(this.loginForm.password, "require")) {
        result.msg = "密码不能为空";
        return result;
      }
      if (!this.validate(this.loginForm.captchaCode, "require")) {
        result.msg = "验证码不能为空";
        return result;
      }
      // 通过验证，返回正确提示
      result.status = true;
      result.msg = "验证通过";
      return result;
    }
  },
  computed: {
    rememberMe() {
      return this.$store.getters.getRememberMe;
    },
    redirectUri() {
      return this.$store.getters.getRedirectUri;
    }
  }
};
</script>
<style rel="stylesheet/scss" lang="scss">
.image-code-div {
  width: 320px;
  height: 40px;
  margin-bottom: 14px;
  input {
    width: 210px;
    height: 40px;
    outline: none;
    font-size: 14px;
    text-indent: 1em;
    border-radius: 2px;
    background: #f3f6f8;
    border: 1px solid #dce3e8;
  }
  img {
    width: 106px !important;
    height: 40px;
    border: 1px solid #dce4e6;
    padding-left: 0;
    float: right;
  }
}

.main-img {
  width: 375px;
  margin: 27px 52.5px;
  float: left;
}

.main .res {
  float: right;
  width: 420px;
  margin: 0 30px;
  background-color: #ffffff;
  margin-top: 27px;
  .main-bot {
    width: 320px;
    margin: 20px 50px 30px;
    .login-input {
      width: 100%;
      height: 40px;
      margin-bottom: 20px;
      outline: 0;
      font-size: 14px;
      color: #a1a1a2;
      text-indent: 1em;
      background-color: #f3f6f8;
      border: 1px solid #dce3e8;
    }
    .button {
      width: 15px;
      height: 15px;
      vertical-align: middle;
    }
    .main-password {
      height: 20px;
      line-height: 20px;
      margin-bottom: 10px;
      .remMe {
        vertical-align: middle;
        cursor: pointer;
        color: #7e8c8d;
      }
    }
    .login {
      border: 0;
      outline: 0;
      width: 100%;
      height: 40px;
      color: #ffffff;
      cursor: pointer;
      font-size: 16px;
      margin-bottom: 20px;
      background-color: #fe7300;
    }
  }
  .other {
    width: 100%;
    height: 20px;
    line-height: 20px;
    margin-bottom: 20px;
    hr {
      width: 90px;
      display: inline-block;
      vertical-align: middle;
    }
    p {
      width: 136px;
      color: #9b9b9b;
      text-align: center;
      display: inline-block;
    }
  }
  .last {
    width: 100%;
    img {
      width: 42px;
      height: 42px;
      margin: 0 12px;
    }
    a:first-child img {
      margin-left: 0;
    }
    a:last-child img {
      margin-right: 0;
    }
  }
}

.fog {
  float: right;
  cursor: pointer;
}
</style>
