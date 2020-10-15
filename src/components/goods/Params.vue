<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        title="注意:只允许为第三季分类设置相关参数!"
        type="warning"
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row>
        <el-col>
          <span>选择商品分类: </span>
          <!-- 级联菜单 -->
          <el-cascader
            expand-trigger="hover"
            :options="catelist"
            v-model="selectedCateKeys"
            :props="cateProps"
            @change="cateHandleChange"
          >
          </el-cascader>
        </el-col>
      </el-row>

      <!-- tabs 标签栏区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数区域 -->
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="BtnDisabled"
            @click="addParamsDialogVisible = true"
            >添加参数</el-button
          >
          <!-- 动态参数列表 -->
          <el-table :data="manyParamsDate" border stripe>
            <el-table-column type="expand" label=""></el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template>
                <el-button type="primary" size="mini" icon="el-icon-edit"
                  >修改</el-button
                >
                <el-button type="danger" size="mini" icon="el-icon-delete"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性区域 -->
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="BtnDisabled"
            @click="addParamsDialogVisible = true"
            >添加属性</el-button
          >
          <!-- 静态属性列表 -->
          <el-table :data="onlyParamsDate" border stripe>
            <el-table-column type="expand" label=""></el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            ></el-table-column>
            <el-table-column label="操作">
              <template>
                <el-button type="primary" size="mini" icon="el-icon-edit"
                  >修改</el-button
                >
                <el-button type="danger" size="mini" icon="el-icon-delete"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>

      <!-- 添加参数Dialog -->
      <el-dialog
        :title="'添加' + getTextTitle"
        :visible.sync="addParamsDialogVisible"
        width="50%"
        @close="addParamsDialogClosed"
      >
        <el-form
          ref="addParmasRef"
          :model="addParamsForm"
          :rules="addParamsRules"
        >
          <el-form-item :label="getTextTitle" prop="attr_name">
            <el-input v-model="addParamsForm.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addParamsDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类返回菜单
      catelist: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的级联
      selectedCateKeys: [],
      // 选中中的tab栏
      activeName: 'many',
      isBtnDisabled: true,
      // 动态参数
      manyParamsDate: [],
      // 静态属性
      onlyParamsDate: [],
      // 是否显示添加参数对话框
      addParamsDialogVisible: false,
      // 添加参数表单
      addParamsForm: { attr_name: '' },
      addParamsRules: {
        attr_name: [{ required: true, message: '请填写内容', trigger: 'blur' }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 3 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      } else {
        // console.log(res.data)
        this.catelist = res.data
      }
    },
    // 选择商品分类后
    cateHandleChange() {
      //   console.log(this.selectedCateKeys)
      if (this.selectedCateKeys.length !== 3) {
        this.$message.error('请选择三级分类')
        this.selectedCateKeys = []
        this.manyParamsDate = []
        this.onlyParamsDate = []
        return false
      }
      //   console.log(this.selectedCateKeys)
      this.getCateDate()
    },
    // 切换标签栏
    handleTabClick() {
      console.log(this.activeName)
      this.getCateDate()
    },
    /* 获取相关分类参数 */
    async getCateDate() {
      if (this.selectedCateKeys.length !== 3) {
        return false
      } else {
        const { data: res } = await this.$http.get(
          `categories/${this.getId}/attributes`,
          {
            params: { sel: this.activeName }
          }
        )
        if (res.meta.status !== 200) return this.$message.error('获取参数失败')
        if (this.activeName === 'many') {
          this.manyParamsDate = res.data
        } else {
          this.onlyParamsDate = res.data
        }
      }
    },
    // dialog关闭时 重置参数
    addParamsDialogClosed() {
      this.$refs.addParmasRef.resetFields()
    },
    // 添加参数或属性
    addParams() {
      this.$refs.addParmasRef.validate(async valide => {
        if (!valide) return false
        const { data: res } = await this.$http.post(
          `categories/${this.getId}/attributes`,
          {
            attr_name: this.addParamsForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加参数成功')
        this.getCateDate()
        this.addParamsDialogVisible = false
      })
    }
  },
  computed: {
    BtnDisabled() {
      if (this.selectedCateKeys.length === 3) {
        return false
      } else {
        return true
      }
    },
    // 获取三级分类ID
    getId() {
      if (this.selectedCateKeys.length !== 3) {
        return false
      } else {
        return this.selectedCateKeys[2]
      }
    },
    // 动态获取标题
    getTextTitle() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else if (this.activeName === 'only') {
        return '静态属性'
      }
      return false
    }
  }
}
</script>

<style lang="less" scope>
</style>
