<template>
  <div class="box" v-loading="loading">
    <div class="imgBox"><div class="imgContainer"></div></div>
    <div class="loginBox">
      <div class="loginText">信息登记系统门卫登录</div>
      <div class="loginFrom">
        <el-form @submit.prevent="login">
          <!-- 密码登录 -->
          <div>
            <!-- 用户名 -->
            <el-form-item class="inputContainer">
              <el-input
                prefix-icon="el-icon-user"
                v-model="mineID"
                placeholder="请输入工号"
                maxlength="11"
              ></el-input>
            </el-form-item>
            <!-- 密码 -->
            <el-form-item class="inputContainer">
              <el-input
                prefix-icon="el-icon-lock"
                placeholder="请输入密码"
                v-model="pwd"
                show-password
              ></el-input>
            </el-form-item>
            <div class="loginBtn">
              <el-button
                style="width:100%"
                type="primary"
                round
                size="medium"
                @click="login('/home')"
                >登录</el-button
              >
            </div>
            <div style="margin-top:20px;margin-left:60px;font-size:12px;">
              <el-link @click="$router.replace('/orderclient')"
                >点击返回预约官网</el-link
              >
              <el-link type="info" disabled> | </el-link>
              <el-link type="primary" :underline="false" @click="changeAdmin()"
                >我是门卫管理员</el-link
              >
            </div>
          </div>
        </el-form>
      </div>
    </div>
    <el-dialog
      title="管理员登录"
      :visible.sync="dialogAdminVisible"
      width="20%"
      style="margin-top:50px;"
    >
      <el-form>
        <el-form-item>
          <el-input
            v-model="adminName"
            style="width:200px;margin-left:80px;"
            prefix-icon="el-icon-user"
            placeholder="请输入管理员账号"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-input
            v-model="adminPwd"
            style="width:200px;margin-left:80px;"
            prefix-icon="el-icon-lock"
            placeholder="请输入密码"
            show-password
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button
          @click="
            dialogAdminVisible = false;
            loading = false;
          "
          >取 消</el-button
        >
        <el-button type="primary" @click="loginAdmin('/administrator')"
          >登 录</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>
<script>
import "./login.css";
import { reqSendCode, reqSmsLogin, reqPwdLogin,reqAdminLogin } from "../../api";
export default {
  data() {
    return {
      // loginWay: true, //登录方式
      mineID: "", //用户名
      pwd: "", //密码
      // cache: "",//图形验证码
      phone: "", //手机号
      code: "", //短信验证码
      computeTime: 0, //倒计时
      loading: false,
      dialogAdminVisible: false,
      adminName: "",
      adminPwd: "",
    };
  },
  computed: {
    rightPhone() {
      return /^1\d{10}$/.test(this.phone);
    },
  },
  methods: {
    changeAdmin() {
      this.dialogAdminVisible = true;
      this.loading = true;
    },
    //异步登录
    async loginAdmin(path) {
      let result;
      const { adminName, adminPwd } = this;
      // 发送ajax请求密码登陆
      result = await reqAdminLogin({ adminName, adminPwd });
      // 根据结果数据处理
      if (result.code === 0) {
        // 跳转
        this.$router.replace(path);
        this.adminName = "";
        this.adminPwd = "";
      } else {
        // 显示警告提示
        const msg = result.msg;
        this.$message.error(msg);
      }
      this.dialogAdminVisible = false;
    },
    //异步登录
    async login(path) {
      // console.log(this.mineID,this.pwd);
      let result;
      // 前端验证
      // if (this.loginWay) {
      //密码登录
      const { mineID, pwd } = this;
      if (!this.mineID) {
        this.$message.error("用户名不能为空，请检查！");
        return;
      } else if (!this.pwd) {
        this.$message.error("密码不能为空，请检查！");
        return;
      }
      // 发送ajax请求密码登陆
      result = await reqPwdLogin({ mineID, pwd });

      // } else {
      //   //短信登录
      //   const { rightPhone, phone, code } = this;
      //   if (!this.rightPhone) {
      //     this.$message.error("手机号输入有误，请检查");
      //     return;
      //   } else if (!/^\d{6}$/.test(code)) {
      //     this.$message.error("验证码输入有误，请检查");
      //     return;
      //   }
      //   // 发送ajax请求短信登陆
      //   result = await reqSmsLogin(phone, code);
      // }
      // 停止计时
      if (this.computeTime) {
        this.computeTime = 0;
        clearInterval(this.intervalId);
        this.intervalId = undefined;
      }
      // 根据结果数据处理
      if (result.code === 0) {
        const user = result.data;
        // 将user保存到vuex的state
        this.$store.dispatch("recordUser", user);
        // 去首页
        this.$router.replace(path);
      } else {
        // 显示警告提示
        const msg = result.msg;
        this.$message.error(msg);
      }
    },
    // 异步获取短信验证码
    async getCode() {
      // 如果当前没有计时
      if (!this.computeTime) {
        // 启动倒计时
        this.computeTime = 60;
        this.intervalId = setInterval(() => {
          this.computeTime--;
          if (this.computeTime <= 0) {
            // 停止计时
            clearInterval(this.intervalId);
          }
        }, 1000);
        // 发送ajax请求(向指定手机号发送验证码短信)
        const result = await reqSendCode(this.phone);
        if (result.code === 1) {
          // 显示提示
          this.$message.error(result.msg);
          // 停止计时
          if (this.computeTime) {
            this.computeTime = 0;
            clearInterval(this.intervalId);
            this.intervalId = undefined;
          }
        }
      }
    },
  },
  beforeRouteEnter(to, from, next) {
    // ...
    window.document.body.style.backgroundColor = "#0c82dc";
    next();
  },
  beforeRouteLeave(to, from, next) {
    // ...
    this.loading = true;
    window.document.body.style.backgroundColor = "";
    next();
  },
};
</script>
