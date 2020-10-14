<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 添加角色 按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 角色列表展示区域 -->
      <el-table :data="rolesList" border strip>
        <!-- 展开页 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightsById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <!-- 二级权限渲染 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightsById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限渲染 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightsById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引页 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <!-- 角色操作列 -->
        <el-table-column label="操作" width="400px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="editRoleDialog(scope.row)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              @click="removeRoleById(scope.row.id)"
              >删除</el-button
            >
            <el-button type="warning" icon="el-icon-setting" @click="showSetRightsDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色Dialog -->
    <el-dialog
      title="添加角色"
      :visible.sync="addRoleDialogVisible"
      width="30%"
    >
      <el-form :model="addRoleForm" :rules="addRules" ref="addRoleFormRef">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addRoleForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addRoleForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resetRole">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色Dialog -->
    <el-dialog title="修改用户" :visible.sync="editRoleVisible" width="30%">
      <el-form
        :model="editList"
        :rules="addRules"
        ref="editRoleRef"
        label-width="100px"
      >
        <el-form-item label="角色id" prop="roleDesc">
          <el-input v-model="editList.id" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editList.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editList.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="resetEditRole">取 消</el-button>
        <el-button type="primary" @click="editRole(editList.id)"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 分配权限Dialog -->
    <el-dialog
      title="权限分配"
      :visible.sync="rightsDialogVisible"
      width="30%"
      @close="setClearDefKeys"
    >
      <!-- 权限树 -->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all
      :default-checked-keys="defKeys" ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotKeys"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      /* 角色列表 */
      rolesList: [],
      /* 添加角色Dialog是否可见 */
      addRoleDialogVisible: false,
      /* 添加角色 */
      addRoleForm: {},
      /* 角色验证 */
      addRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      /* 修改角色Dialog显示与否 */
      editRoleVisible: false,
      /* 修改角色表单 */
      editList: {},
      /* 权限分配Dialog是否显示 */
      rightsDialogVisible: false,
      /* 权限树 */
      rightsList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      /* 默认勾选的权限 */
      defKeys: [],
      /* 即将分配权限的角色id */
      roleId: 0
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      // console.log(res)
      this.rolesList = res.data
    },
    /* 重置用户输入 */
    resetRole() {
      this.$refs.addRoleFormRef.resetFields()
      this.addRoleDialogVisible = false
    },
    /* 添加用户 */
    addRole() {
      this.$refs.addRoleFormRef.validate(async valide => {
        if (!valide) return false
        const { data: res } = await this.$http.post('roles', this.addRoleForm)
        //   console.log(res)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败')
        this.addRoleDialogVisible = false
        this.getRolesList()
        this.$message.success('添加角色成功')
      })
    },
    /* 修改用户 */
    async editRoleDialog(list) {
      this.editRoleVisible = true
      //   console.log(list)
      this.editList = list
    },
    /* 重置编辑Dialog */
    resetEditRole() {
      this.$refs.editRoleRef.resetFields()
      this.editRoleVisible = false
    },
    /* 编辑角色 */
    editRole(id) {
      this.$refs.editRoleRef.validate(async valide => {
        if (!valide) return false
        const { data: res } = await this.$http.put('roles/' + id, {
          roleName: this.editList.roleName,
          roleDesc: this.editList.roleDesc
        })
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error('修改用户失败')
        this.getRolesList()
        this.editRoleVisible = false
        this.$message.success('修改用户成功')
      })
    },
    /* 删除角色 */
    async removeRoleById(id) {
      const removeRes = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (removeRes !== 'confirm') return this.$message.info('取消删除')
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除角色失败')
      this.getRolesList()
      this.$message.success('删除角色成功')
    },
    /* 移除权限 */
    async removeRightsById(role, rid) {
      const removeRights = await this.$confirm(
        '此操作将永久删除该角色, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (removeRights !== 'confirm') {
        return this.$message.error('已取消删除权限')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rid}`
      )
      if (res.meta.status !== 200) return this.$message.error('权限移除失败')
      this.$message.success('权限移除成功')
      // 避免页面重新加载
      role.children = res.data
    },
    /* 权限分配 */
    async showSetRightsDialog(role) {
      /* 要分配权限的角色id */
      this.roleId = role.id
      // 获取权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      this.rightsList = res.data
      this.$message.success('获取权限成功')
      this.getLeafKeys(role, this.defKeys)
      this.rightsDialogVisible = true
    },
    /* 通过递归形式来后去所有权限的三级id,并保存到defKeys数组中 */
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    /* 清空defKeys权限数组 */
    setClearDefKeys() {
      this.defKeys = []
    },
    /* 拿到所有全选中和半选中的权限id */
    async allotKeys() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const id = this.roleId
      //   console.log(id)
      //   console.log(keys)
      const rids = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${id}/rights`, { rids: rids })
      //   console.log(res)
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.getRolesList()
      this.rightsDialogVisible = false
      this.$message.success('分配权限成功')
    }
  }
}
</script>

<style lang="less">
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
// 垂直居中
.vcenter {
  display: flex;
  align-items: center;
}
</style>
