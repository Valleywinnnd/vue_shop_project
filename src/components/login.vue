<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 头像区域 -->
      <div class="login_head_pic">
        <img src="../assets/logo.png" alt="头像" />
      </div>
      <!-- 登录表单区域 -->
      <el-form
        ref="loginFormRef"
        label-width="0px"
        class="login_form"
        :model="loginForm"
        :rules="loginRules"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input
            prefix-icon="iconfont icon-user"
            v-model="loginForm.username"
          ></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="loginForm.password"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮区域 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      /* 数据绑定 */
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      /* 验证规则 */
      loginRules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 12, message: '长度在 3 到 12 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 5, max: 12, message: '长度在 5 到 12 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    /* 重置输入 */
    resetForm() {
      //   console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    /* 表单预验证 */
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) {
          return false
        } else {
          /* 获取返回信息 */
          const { data: res } = await this.$http.post('/login', this.loginForm)
          if (res.meta.status !== 200) {
            return this.$message.error('登录出错')
          } else {
            this.$message.success('登录成功')
            // 1.将登录成功的token保存到客户端的sessionStorage中
            //   1.1 项目中除了登录之外的其他API接口需要登录之后才能生效
            //   1.2 token只能在当前网站打开期间生效，所以存储在sessionStorage中
            window.sessionStorage.setItem('token', res.data.token)
            // 2.通过编程式导航跳转到主页，/home
            this.$router.push('/home')
          }
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;

  .login_head_pic {
    position: absolute;
    top: 0;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 130px;
    height: 130px;
    border: 1px solid #ddd;
    background-color: #fff;
    padding: 10px;
    border-radius: 50%;
    box-shadow: 0 0 10px #ddd;

    img {
      width: 100%;
      height: 100%;
      background-color: #eee;
      border: 1px solid #eee;
      border-radius: 50%;
    }
  }
}
.login_form {
  box-sizing: border-box;
  position: absolute;
  bottom: 0;
  padding: 0 20px;
  width: 100%;
}
.btns {
  display: flex;
  justify-content: flex-end;
}
</style>
