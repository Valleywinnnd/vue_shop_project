<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 搜索商品和添加商品区域 -->
      <el-row :gutter="20">
        <!-- 搜索商品区域 -->
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <!-- 添加商品区域 -->
        <el-col :span="4">
          <el-button type="primary" @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- 表格区域 -->
      <el-table :data="goodsList">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column
          prop="goods_price"
          label="商品价格(元)"
          width="95px"
        ></el-table-column>
        <el-table-column
          prop="goods_weight"
          label="商品重量"
          width="70px"
        ></el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="160px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <!-- 操作商品区域 -->
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeGoods(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 20, 50, 100]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      /* 查询商品所携带的信息 */
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      /* 商品列表 */
      goodsList: [],
      /* 商品总数 */
      total: 0
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    /* 获取商品列表 */
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('商品查询失败')
      //   console.log(res.data.goods)
      this.goodsList = res.data.goods
      this.total = res.data.total
      //   this.$message.success('商品查询成功')
      //   console.log(this.goodsList)
    },
    /* 改变页面所展示数据条数 */
    handleSizeChange(size) {
      this.queryInfo.pagesize = size
      this.getGoodsList()
    },
    /* 改变当前页码值 */
    handleCurrentChange(page) {
      this.queryInfo.pagenum = page
      this.getGoodsList()
    },
    /* 删除商品 */
    async removeGoods(id) {
      const confirmRes = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)

      if (confirmRes !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('goods/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除商品失败')
      this.$message.success('商品删除成功')
      this.getGoodsList()
    },
    /* 添加商品 */
    goAddPage() {
      this.$router.push('/goods/add')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
