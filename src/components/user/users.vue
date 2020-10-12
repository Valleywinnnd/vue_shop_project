<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索区域 -->
    <el-card class="box-card">
      <el-row :gutter="20">
        <el-col :span="8"
          ><div>
            <el-input
              placeholder="请输入内容"
              v-model="queryUser.query"
              clearable
              @clear="getUserList"
            >
              <el-button
                slot="append"
                icon="el-icon-search"
                @click="getUserList"
              ></el-button>
            </el-input>
          </div>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addUDialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addUDialogVisible"
      width="50%"
      @close="addFormReset"
    >
      <el-form
        :model="addForm"
        :rules="addUserRules"
        ref="addUserFromRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addUDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editUserFrom"
        :rules="editUserRules"
        ref="editFromRef"
        label-width="70px"
      >
        <el-form-item label="用户名">
          <el-input v-model="editUserFrom.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editUserFrom.email"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input v-model="editUserFrom.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser(editUserFrom.id)"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 用户展示区域 -->
    <el-table :data="userlist" stripe border>
      <el-table-column type="index" label="#"> </el-table-column>
      <el-table-column prop="username" label="姓名"> </el-table-column>
      <el-table-column prop="email" label="邮箱"> </el-table-column>
      <el-table-column prop="mobile" label="电话"> </el-table-column>
      <el-table-column prop="role_name" label="角色"> </el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.mg_state"
            @change="userStateChange(scope.row)"
          >
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" width="180px">
        <template slot-scope="scope">
           <!-- 修改用户按钮 -->
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="queryUserInfo(scope.row.id)"
          ></el-button>
          <!-- 删除用户按钮 -->
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="removeUserById(scope.row.id)"
          ></el-button>
          <!-- 文字提示 -->
          <el-tooltip
            class="item"
            effect="dark"
            content="分配角色"
            placement="top"
            :enterable="false"
          >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
            ></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页区域 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryUser.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryUser.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  data() {
    /* 校验邮箱的验证规则 */
    var checkEmail = (rule, value, callback) => {
      const regEmail = /^[a-zA-Z0-9_.-]+@[a-zA-Z0-9-]+(\.[a-zA-Z0-9-]+)*\.[a-zA-Z0-9]{2,6}$/
      if (regEmail.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的邮箱'))
    }
    /* 校验手机号规则 */
    var checkMoblie = (rule, value, callback) => {
      const regMobile = /^(((13[0-9]{1})|(14[0-9]{1})|(15[0-9]{1})|(16[0-9]{1})|(18[0-9]{1})|(19[0-9]{1})|(17[0-9]{1}))+\d{8})$/
      if (regMobile.test(value)) {
        return callback()
      }
      callback(new Error('请输入合法的手机号'))
    }
    return {
      /* 获取到的信息 */
      queryUser: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      /* 用户列表 */
      userlist: [],
      /* 用户个数总和 */
      total: 0,
      /* 添加用户dialog显示与否 */
      addUDialogVisible: false,
      /* 用户表单 */
      addForm: {
        username: '',
        password: '',
        emial: '',
        mobile: ''
      },
      /* 用户验证规则 */
      addUserRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 12,
            message: '用户名在 3 到 12 位之间',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 5,
            max: 16,
            message: '用户名在 5 到 12 位之间',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMoblie }
        ]
      },
      /* 修改用户dialog是否可见 */
      editDialogVisible: false,
      /* 修改用户表单 */
      editUserFrom: {},
      /* 修改用户验证规则 */
      editUserRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMoblie }
        ]
      }
    }
  },

  created() {
    this.getUserList()
  },
  methods: {
    /* 获取用户列表 */
    async getUserList() {
      const { data: res } = await this.$http.get('users', {
        params: this.queryUser
      })
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户信息失败')
      }
      this.userlist = res.data.users
      this.total = res.data.total
    },
    /* 改变一行显示个数 */
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryUser.pagesize = newSize
      this.getUserList()
    },
    /* 改变页数 */
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryUser.pagenum = newPage
      this.getUserList()
    },
    /* 改变用户状态 */
    async userStateChange(userInfo) {
      // console.log(userInfo)
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      )
      // console.log(res)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('状态更新失败')
      }
      this.$message.success('状态更新成功')
    },
    /* 重置添加用户表单项 */
    addFormReset() {
      this.$refs.addUserFromRef.resetFields()
    },
    /* 添加用户 */
    addUser() {
      this.$refs.addUserFromRef.validate(async valide => {
        // console.log(valide)
        if (!valide) return false
        const { data: res } = await this.$http.post('users', this.addForm)
        // console.log(res)
        if (res.meta.status !== 201) return this.$message.error('用户创建失败')
        this.$message.success('用户创建成功')
        this.addUDialogVisible = false
        this.getUserList()
      })
    },
    /* 修改获取的用户 */
    async queryUserInfo(id) {
      const { data: res } = await this.$http.get('users/' + id)
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取用户失败')
      this.editUserFrom = res.data
      // console.log(this.editUserFrom)
      this.editDialogVisible = true
    },
    /* 重置验证结果 */
    editDialogClosed() {
      this.$refs.editFromRef.resetFields()
    },
    /* 修改用户 */
    editUser(id) {
      this.$refs.editFromRef.validate(async valide => {
        if (!valide) return false
        const { data: res } = await this.$http.put('users/' + id, {
          email: this.editUserFrom.email,
          mobile: this.editUserFrom.mobile
        })
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('更新数据失败')
        this.getUserList()
        this.editDialogVisible = false
        this.$message.success('更新数据成功')
      })
    },
    /* 删除用户 */
    async removeUserById(id) {
      // console.log('点击了')
      const confirmRes = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmRes !== 'confirm') return this.$message.info('已取消删除')
      // console.log('确认删除')
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除用户成功')
      this.getUserList()
    }
  }
}
</script>

<style lang="less" scoped>
</style>
