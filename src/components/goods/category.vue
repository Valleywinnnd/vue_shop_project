<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <!-- 添加商品区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addCateDialog">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        :selection-type="false"
        :expand-type="false"
        class="treeTable"
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i
            class="el-icon-error"
            v-if="scope.row.cat_delete === true"
            style="color: red"
          ></i>
        </template>
        <template slot-scope="scope" slot="order">
          <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag v-if="scope.row.cat_level === 1" type="success">二级</el-tag>
          <el-tag v-if="scope.row.cat_level === 2" type="warning">三级</el-tag>
        </template>
        <template slot="option" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="removeCate(scope.row)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加商品分类Dialog -->
    <el-dialog
      title="添加商品分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateRules"
        ref="addCateRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="parentcatelist"
            expand-trigger="hover"
            :props="pCateprops"
            @change="parentCateChange"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 携带的查询参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 分类列表
      catelist: [],
      // 商品分类总数
      total: 0,
      // 定义表格展示时属性
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 指定为模板类型
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          // 指定为模板类型
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          // 指定为模板类型
          type: 'template',
          template: 'option'
        }
      ],
      // 添加商品分类dialog是否显示
      addCateDialogVisible: false,
      // 添加商品分类的输入表单
      addCateForm: {
        cat_name: '',
        cat_level: 0,
        cat_pid: 0
      },
      // 添加商品分类时的验证规则
      addCateRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类参数
      parentcatelist: [],
      pCateprops: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 被选中的父级元素id数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求商品分类信息失败')
      }
      //   console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 改变一页显示个数
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 改变当前页数
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 添加商品
    async addCateDialog() {
      this.addCateDialogVisible = true
      // 查询二级父级分类
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求分级分类参数失败')
      }
      this.parentcatelist = res.data
      console.log(this.parentcatelist)
    },
    // 选择父级分类参数时
    parentCateChange(selectValue) {
      //   console.log(selectValue)
      if (selectValue.length > 0) {
        // 父级的分类id
        this.addCateForm.cat_pid = selectValue[selectValue.length - 1]
        // 为当前分类等级赋值
        this.addCateForm.cat_level = selectValue.length
      } else {
        // 父级的分类id
        this.addCateForm.cat_pid = 0
        // 为当前分类等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类
    addCate() {
      //   console.log(this.addCateForm)
      this.$refs.addCateRef.validate(async valide => {
        if (!valide) {
          return false
        } else {
          const { data: res } = await this.$http.post(
            'categories',
            this.addCateForm
          )
          if (res.meta.status !== 201) {
            return this.$message.error('添加分类失败')
          }
          this.$message.success('添加分类成功')
          this.getCateList()
          this.addCateDialogVisible = false
        }
      })
    },
    // 对话框关闭时清空选择
    addCateDialogClosed() {
      this.$refs.addCateRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 删除分类
    async removeCate(cate) {
      const result = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      //   console.log(result)
      if (result !== 'confirm') {
        return this.$message.info('已取消删除操作')
      } else {
        const { data: res } = await this.$http.delete(
          'categories/' + cate.cat_id
        )
        if (res.meta.status !== 200) {
          return this.$message.error('删除分类失败')
        } else {
          this.$message.success('删除分类成功')
          this.getCateList()
        }
      }
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
