<template>
  <div class="userLoginPwd">
    <div class="title-info" style="background:#673ab7;">
      <div class="title-icon">
        <a-icon type="key" class="title-item-icon"></a-icon>&nbsp;使用账号密码登录
      </div>
    </div>
    <a-spin :spinning="showLoading" size="large">
      <div class="team-logo-container">
        <img src="./unique.jpg" class="team-logo" alt="team-logo">
      </div>
      <h1>登录</h1>
      <div class="login-container">
        <a-input-group>
          <a-input addonBefore="昵称" v-model.trim="nickname"/>
          <a-input addonBefore="密码" v-model.trim="pwd" type="password" @keyup.enter="login"/>
        </a-input-group>
      </div>
      <div class="login-btn-container">
        <a-button
          icon="login"
          type="primary"
          @click="login"
          id="login"
          :disabled="loginBtnDisabled"
        >登录</a-button>
      </div>
      <div>
        <router-link to="/user/login/wx">
          <a-button icon="wechat" type="primary">使用企业微信登录</a-button>
        </router-link>
      </div>
    </a-spin>
  </div>
</template>
<script>
import crypto from "crypto";
export default {
    data() {
        return {
            nickname: "",
            pwd: "",
            loginBtnDisabled: false,
            showLoading: false
        };
    },
    computed: {
        pwd_md5() {
            const md5 = crypto.createHash("md5");
            return md5.update(this.pwd).digest("hex");
        }
    },
    methods: {
        async login(e) {
            if (!(e instanceof KeyboardEvent) && !(e instanceof MouseEvent))
                return;
            this.showLoading = true;
            this.loginBtnDisabled = true;
            document.getElementById("login").focus();
            const nickname = this.nickname;
            const pwd = this.pwd_md5;
            const result = await this.$ajax.post(this.$urls.login, {
                nickname: nickname,
                pwd: pwd
            });
            const response = result.data;
            if (response.code !== 1) {
                const modal = this.$error();
                modal.update({
                    title: "登录错误",
                    content: response.msg
                });
                localStorage.removeItem("token");
                localStorage.removeItem("uid");
            } else {
                const token = response.msg.token;
                const uid = response.msg.uid;
                localStorage.setItem("token", token);
                localStorage.setItem("uid", uid);
                this.$store.commit("setLoginStatus", true);
                this.$store.dispatch("checkLoginStatus");
                this.$notification.open({
                    message: "登录",
                    description: "登录成功，欢迎回来！",
                    icon: <a-icon type="smile" style="color: #108ee9" />
                });
                this.$router.push({ path: "/user/my/info" });
            }
            this.loginBtnDisabled = false;
            this.showLoading = false;
        }
    }
};
</script>
<style scoped>
.login-btn-container {
    margin: 12px 0 36px 0;
}
.team-logo-container {
    margin: 48px auto;
    text-align: center;
}
.team-logo {
    width: 140px;
    height: 140px;
    border-radius: 70px;
}
.userLoginPwd {
    text-align: center;
}
h1 {
    user-select: none;
}
@media screen and (min-width: 800px) {
    .login-container {
        width: 35%;
    }
}
@media screen and (max-width: 800px) {
    .login-container {
        width: 80%;
    }
}
.login-container span {
    margin: 6px auto;
}
.login-container {
    margin: 6px auto;
}
</style>
