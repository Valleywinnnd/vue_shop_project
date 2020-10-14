<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <!-- 提示文本框 -->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 步骤条区域 -->
      <el-steps :active="activeIndex - 0" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tabs导航栏区域 -->
      <el-form
        ref="addGoodsRef"
        :model="addGoodsFrom"
        :rules="addFormRules"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          tab-position="left"
          v-model="activeIndex"
          :before-leave="beforeLeaveItem"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addGoodsFrom.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addGoodsFrom.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addGoodsFrom.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addGoodsFrom.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addGoodsFrom.goods_cat"
                :options="cateList"
                :props="cateProps"
                expand-trigger="hover"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableDate"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                  border
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableDate"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3"
            ><el-upload
              action="http://127.0.0.1:8888/api/private/v1/upload"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headersObj"
              :on-success="handleSuccess"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload></el-tab-pane
          >
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="addGoodsFrom.goods_introduce"></quill-editor>
            <!-- 添加商品 -->
            <el-button type="primary" class="add_btn" @click="add"
              >添加商品</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览Dialog -->
    <el-dialog title="图片预览" :visible.sync="imageDialogVisible" width="50%">
      <img :src="previewPath" alt="" class="preImage" />
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      /* 激活步骤 */
      activeIndex: '0',
      /* 添加商品列表 */
      addGoodsFrom: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        /* 商品分类 */
        goods_cat: [],
        /* 图片路径 */
        pics: [],
        /* 商品的详情描述 */
        goods_introduce: '',
        // 添加商品的属性
        attrs: []
      },
      /* 添加商品验证规则 */
      addFormRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品名称',
            trigger: 'blur'
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品价格',
            trigger: 'blur'
          }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品重量',
            trigger: 'blur'
          }
        ],
        goods_number: [
          {
            required: true,
            message: '请输入商品数量',
            trigger: 'blur'
          }
        ],
        goods_cat: [
          {
            required: true,
            message: '请选择商品分类',
            trigger: 'blur'
          }
        ]
      },
      /* 商品分类列表数据 */
      cateList: [],
      /* 指定多选列表显示的是哪个值 */
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      /* 动态参数列表 */
      manyTableDate: [],
      /* 静态属性面板 */
      onlyTableDate: [],
      /* 上传图片的绝对路径请求地址 */
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      /* 上传图像的headerOBJ */
      headersObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      /* 预览图片地址 */
      previewPath: '',
      /* 图片预览窗口显示与否 */
      imageDialogVisible: false,
      /* 添加商品时的attr数组 */
      attrs: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    /* 获取商品分类列表 */
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) this.$message.error('获取商品分类数据失败')
      //   console.log(res)
      this.cateList = res.data
    },
    /* 商品分类选择完成后的函数 */
    handleChange() {
      //   console.log(this.addGoodsFrom.goods_cat)
      //   /* 控制级联选择器的选择范围 */
      //   if (this.addGoodsFrom.goods_cat.length !== 3) {
      //     this.addGoodsFrom.goods_cat = []
      //   }
    },
    /* 离开tabs时判断是否选中商品分类 */
    beforeLeaveItem(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addGoodsFrom.goods_cat.length === 0) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    /* tabs栏点击 */
    async tabClicked() {
      //   console.log(this.activeIndex)
      /* 证明访问的动态参数面板 */
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品信息失败')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals === 0 ? [] : item.attr_vals.split(',')
        })
        this.manyTableDate = res.data
        console.log(res.data)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品静态属性失败')
        }
        this.onlyTableDate = res.data
      }
    },
    /* 图片预览 */
    handlePreview(file) {
      this.previewPath = file.response.data.url
      //   console.log(this.previewPath)
      this.imageDialogVisible = true
    },
    /* 图片删除 */
    handleRemove(file) {
      // 获取将要移除的图片的路径
      const filePath = file.response.data.tmp_path
      //   console.log(filePath)
      const i = this.addGoodsFrom.pics.findIndex(x => {
        x.pic = filePath
      })
      // 在pics中移除该图片的路径
      this.addGoodsFrom.pics.splice(i, 1)
      //   console.log(this.addGoodsFrom)
    },
    /* 图片上传成功 */
    handleSuccess(response) {
      //   console.log(response)
      const picPath = { pic: response.data.tmp_path }
      this.addGoodsFrom.pics.push(picPath)
      //   console.log(this.addGoodsFrom.pics)
      //   console.log(this.addGoodsFrom)
    },
    /* 添加商品 */
    add() {
      /* 表单预验证 */
      this.$refs.addGoodsRef.validate(async valide => {
        if (!valide) {
          return this.$message.error('请填写必要的表单项')
        }

        const form = _.cloneDeep(this.addGoodsFrom)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableDate.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addGoodsFrom.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableDate.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addGoodsFrom.attrs.push(newInfo)
        })
        // console.log(this.addGoodsFrom.attrs)
        form.attrs = this.addGoodsFrom.attrs
        console.log(form)
        const { data: res } = await this.$http.post('goods', form)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('商品添加失败')
        }
        this.$message.success('商品添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    /* 获取商品的id */
    cateId() {
      if (this.addGoodsFrom.goods_cat.length === 3) {
        return this.addGoodsFrom.goods_cat[2]
      } else if (this.addGoodsFrom.goods_cat === 2) {
        return this.addGoodsFrom.goods_cat[1]
      }
      return null
    }
  }
}
</script>

<style lang="less">
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.preImage {
  width: 100%;
}
</style>
