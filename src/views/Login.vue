<template>
  <div class="login-wrap">
    <div class="ms-login">
      <div class="ms-title">LTE网络干扰分析系统</div>
      <el-form
        :model="formdata"
        :rules="rules"
        ref="login"
        label-width="0px"
        class="ms-content"
      >
        <el-form-item prop="username">
          <el-input v-model="formdata.username" placeholder="用户名">
            <template #prepend>
              <el-button icon="el-icon-user"></el-button>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            type="password"
            placeholder="密码"
            v-model="formdata.password"
            @keyup.enter="handleLogin"
          >
            <template #prepend>
              <el-button icon="el-icon-lock"></el-button>
            </template>
          </el-input>
        </el-form-item>
        <div class="login-btn">
          <el-button-group>
            <el-button type="primary" @click="handleLogin">登录</el-button>
            <el-button type="primary" @click="handleRegister">用户注册</el-button>
          </el-button-group>
        </div>
      </el-form>
    </div>
  </div>
</template>

<script>
import { BACKEND } from "../utils/backend";
import request from "../utils/request";
export default {
  data() {
    return {
      formdata: {
        username: "",
        password: "",
      },
      rules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }],
      },
    };
  },
  created() {
    this.$store.commit("clearTags");
  },
  methods: {
    handleLogin() {
      this.$refs.login.validate((valid) => {
        if (valid) {
          request({
            url: `${BACKEND}/dataservice/account/login?username=${this.formdata.username}&password=${this.formdata.password}`,
            method: "post",
          }).then((response) => {
            if (response.success) {
              this.$message.success("登录成功");
              localStorage.setItem("usertype", response.message);
              localStorage.setItem("username", this.formdata.username);
              this.$router.push("/");
            } else {
              this.$message.error(response.message);
            }
          });
        } else {
          this.$message.error("请输入账号和密码");
          return false;
          // this.$message.success("登录成功");
          // localStorage.setItem("usertype", response.message);
          // localStorage.setItem("username", this.formdata.username);
          // this.$router.push("/");
        }
      });
    },
    handleRegister() {
      this.$refs.login.validate((valid) => {
        if (valid) {
          request({
            url: `${BACKEND}/dataservice/account/register?username=${this.formdata.username}&password=${this.formdata.password}`,
            method: "post",
          }).then((response) => {
            if (response.success) {
              this.$message.success("用户注册成功，等待管理员审核");
            } else {
              this.$message.error(response.message);
            }
          });
        } else {
          this.$message.error("请输入账号和密码");
          return false;
        }
      });
    },
  },
};
</script>

<style scoped>
.login-wrap {
  position: relative;
  width: 100%;
  height: 100%;
  background-image: url(../assets/img/login-bg.jpg);
  background-size: 100%;
}
.ms-title {
  width: 100%;
  line-height: 50px;
  text-align: center;
  font-size: 20px;
  color: #fff;
  border-bottom: 1px solid #ddd;
}
.ms-login {
  position: absolute;
  left: 50%;
  top: 50%;
  width: 350px;
  margin: -190px 0 0 -175px;
  border-radius: 5px;
  background: rgba(80, 80, 80, 0.3);
  overflow: hidden;
}
.ms-content {
  padding: 30px 30px;
}
.login-btn {
  text-align: center;
}
.login-btn button {
  width: 100%;
  height: 36px;
  margin-bottom: 10px;
}
.login-tips {
  font-size: 12px;
  line-height: 30px;
  color: #fff;
}
</style>
