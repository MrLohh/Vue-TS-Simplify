<template>
  <div class="login-container">
    <vue-particle-line>
      <div class="login-box">
        <h2>Admin-Login</h2>
        <p>Vue+TypeScript联合实践</p>
        <el-form ref="form" :model="loginData" :rules="loginRules" label-width="0px">
          <el-form-item label="" prop="username">
            <el-input v-model="loginData.username" prefix-icon="el-icon-user"></el-input>
          </el-form-item>
          <el-form-item label="" prop="password">
            <el-input type="password" v-model="loginData.password" prefix-icon="el-icon-lock"></el-input>
          </el-form-item>
          <el-form-item label="" prop="captcha">
            <el-row>
              <el-col :span="13">
                <el-input v-model="loginData.captcha" prefix-icon="el-icon-lock"></el-input>
              </el-col>
              <el-col :span="11">
                <img src="http://127.0.0.1:7001/imageCaptcha" ref="captchaImage" alt="" @click="updateCaptcha">
              </el-col>
            </el-row>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit" style="width: 100px; background:transparent; color: #409EFF"
                       icon="el-icon-s-promotion">登录
            </el-button>
          </el-form-item>
        </el-form>
        <p>没有账号,去<span style="color: deepskyblue;cursor: pointer" @click="toRegister">注册</span> or ↓</p>
        <p>------------- 第三方登录 ------------- </p>
        <ul>
          <!--        <li class="iconfont icon-qq" style="color: dodgerblue"></li>-->
          <!--        <li class="iconfont icon-weixin" style="color: greenyellow"></li>-->
          <!--        <li class="iconfont icon-weibo" style="color: red"></li>-->
          <li class="iconfont icon-github" style="color: #fff">
            <a href="http://127.0.0.1:7001/passport/github"></a>
          </li>
        </ul>
      </div>
    </vue-particle-line>
  </div>
</template>

<script lang="ts">
  import {Vue, Component, Ref} from 'vue-property-decorator'
  import {ElForm} from "element-ui/types/form";
  import {loginUser} from "@/api";
  import {mapActions} from "vuex";

  @Component({
    name: 'Login',
    methods: {
      ...mapActions([
        'commitSetRoles'
      ])
    }
  })
  export default class Login extends Vue {
    commitSetRoles!: (payload: []) => any;
    private loginData = {
      username: '',
      email: '',
      phone: '',
      password: '',
      captcha: '',
      type: 'normal',
    };
    private loginRules = {
      username: [
        {validator: this.validateUsername, trigger: 'blur'}
      ],
      password: [
        {validator: this.validatePassword, trigger: 'blur'}
      ],
      captcha: [
        {validator: this.validateCaptcha, trigger: 'blur'}
      ]
    };

    private validateUsername(rule: any, value: string, callback: any) {
      // rule 规则  value 需要校验的数据 callback 判断成功或失败
      const normalReg = /^[a-zA-Z][a-zA-Z0-9_]{8,20}$/;
      const emailReg = /^([A-Za-z0-9_\-\.])+\@(163.com|qq.com)$/;
      const phoneReg = /^(?:(?:\+|00)86)?1(?:(?:3[\d])|(?:4[5-7|9])|(?:5[0-3|5-9])|(?:6[5-7])|(?:7[0-8])|(?:8[\d])|(?:9[1|8|9]))\d{8}$/;
      if (!value) {
        callback(new Error('用户名不能为空'));
      } else if (emailReg.test(value)) {
        this.loginData.email = this.loginData.username;
        this.loginData.type = 'email';
        callback();
      } else if (phoneReg.test(value)) {
        this.loginData.phone = this.loginData.username;
        this.loginData.type = 'phone';
        callback();
      } else if (normalReg.test(value)) {
        this.loginData.type = 'normal';
        callback();
      } else {
        callback(new Error('用户名不正确'));
      }
    }

    private validatePassword(rule: any, value: string, callback: any) {
      const reg = /^(?![0-9]+$)(?![a-z]+$)(?![A-Z]+$)(?!([^(0-9a-zA-Z)])+$).{8,20}$/;
      if (!value) {
        callback(new Error('请输入密码'));
      } else if (value.length < 8) {
        callback(new Error('密码至少8位'));
      } else if (!reg.test(value)) {
        callback(new Error('包含数字,英文,字符中的两种以上'));
      } else {
        callback();
      }
    }

    private validateCaptcha(rule: any, value: string, callback: any) {
      const reg = /^[A-Za-z0-9]{4}$/;
      if (!value) {
        callback(new Error('请输入验证码'));
      } else if (value.length < 4) {
        callback(new Error('验证码为4位'));
      } else if (!reg.test(value)) {
        callback(new Error('验证码格式不正确'));
      } else {
        callback();
      }
    }

    @Ref() readonly captchaImage!: HTMLImageElement;

    private updateCaptcha() {
      this.captchaImage.src = `http://127.0.0.1:7001/imageCaptcha?r=${Math.random()}`
    }

    @Ref() readonly form!: ElForm;
    private onSubmit() {
      this.form.validate((valid) => {
        if (valid) {
          loginUser(this.loginData)
            .then((res: any) => {
              if (res.status === 200) {
                sessionStorage.setItem('userInfo', JSON.stringify(res.data.data));
                (this as any).$message.success('登录成功');
                this.$router.push('/admin');
                sessionStorage.setItem('roles', JSON.stringify(res.data.data.roles))
                // this.commitSetRoles(res.data.data.roles)
              } else {
                (this as any).$message.error(res.data.msg);
                // console.log(res.data.msg, '1111111111111111111111')
                this.updateCaptcha();
              }
            })
            .catch((err: any) => {
              (this as any).$message.error(err.response.data.msg);
              this.updateCaptcha();
            })
        } else {
          (this as any).$message.error('请完善登录信息');
        }
      })
    }

    private toRegister() {
      this.$router.push('/register');
    }
  }
</script>

<style lang="scss" scoped>
  .login-container {
    width: 100%;
    height: 100%;
    /*background: url("../assets/bg.jpg") no-repeat;*/
    background-color: #000;
    background-size: 100% 100%;

    .login-box {
      width: 400px;
      height: 465px;
      border-radius: 10px;
      /*background: rgba(255,255,255,.8);*/
      background: transparent;
      position: absolute;
      left: 0;
      top: 0;
      right: 0;
      bottom: 0;
      margin: auto;
      padding: 0 60px;
      box-sizing: border-box;
      z-index: 666;
      /*opacity: 0.8;*/
      h2 {
        font-weight: bold;
        color: #efefef;
      }

      p {
        text-align: center;
        color: #d1d1d1;
      }

      ul {
        display: flex;
        justify-content: space-around;
        list-style: none;
        padding: 0;

        li {
          font-size: 30px;
          position: relative;

          a {
            display: block;
            width: 100%;
            height: 100%;
            position: absolute;
            left: 0;
            top: 0;
          }
        }
      }

      .el-form {
        margin: 0 auto;
        text-align: center;

        .el-form-item {
          height: 40px;

          .el-button {
            margin-top: 10px;
          }

          .el-row {
            height: 40px;

            .el-col {
              height: 40px;
            }
          }
        }
      }

      h2 {
        text-align: center;
      }
    }
  }
</style>
