<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 订单搜索区域 -->
      <el-row>
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @close="closeGetOrder"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="searchOrder"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 表格展示区域 -->
      <el-table :data="orderlist">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column
          label="订单编号"
          prop="order_number"
          width="300px"
        ></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="order_pay">
          <template slot-scope="scope">
            <el-tag type="success" v-if="scope.row.order_pay == 1"
              >已付款</el-tag
            >
            <el-tag type="danger" v-if="scope.row.order_pay == 0"
              >未付款</el-tag
            >
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send"></el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showBox"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 修改地址Dialog -->
    <el-dialog
      title="修改地址"
      :visible.sync="addressDialogVisible"
      width="50%"
      @close="closeBox"
    >
      <el-form
        :model="addressRuleForm"
        :rules="addressRules"
        ref="addressRef"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="选择省/市" prop="city">
          <el-cascader
            v-model="addressRuleForm.city"
            :options="Citydata"
            @change="handleChange"
          ></el-cascader
        ></el-form-item>
        <el-form-item label="详细地址" prop="address">
          <el-input v-model="addressRuleForm.address"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addressDialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>
    <!-- 物流进度对话框 -->
    <el-dialog title="物流进度" :visible.sync="progressVisible" width="50%">
      <!-- 时间线组件 -->
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in progressInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>

<script>
import Citydata from './citydata.js'

export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 订单列表
      orderlist: [],
      total: 0,
      // 城市数据
      Citydata,
      addressDialogVisible: false,
      // 地址数据表单
      addressRuleForm: {
        address: '',
        city: []
      },
      addressRules: {
        city: [{ required: true, message: '请选择城市', trigger: 'blur' }],
        address: [{ required: true, message: '请输入地址', trigger: 'blur' }]
      },
      progressVisible: false,
      progressInfo: []
    }
  },
  created() {
    this.getOrderlist()
  },
  methods: {
    async getOrderlist() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      //   console.log(res)
      if (res.meta.status !== 200) return this.$message.error('查询订单失败')
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    // 页面尺寸改变时
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderlist()
    },
    // 页数发生变化时
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getOrderlist()
    },
    // showBox
    showBox() {
      this.addressDialogVisible = true
    },
    // 关闭closebox时
    closeBox() {
      this.$refs.addressRef.resetFields()
      this.addressDialogVisible = false
    },
    handleChange() {},
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败')
      }
      this.progressInfo = res.data
      this.progressVisible = true
      console.log(res.data)
    },
    closeGetOrder() {
      this.queryInfo.query = ''
      this.getOrderlist()
    },
    searchOrder() {
      //   console.log(this.queryInfo.query)
      this.getOrderlist()
      console.log(123)
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
