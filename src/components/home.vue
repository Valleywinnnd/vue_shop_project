<template>
  <el-container class="home_container">
    <!-- 头部布局 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="logo" />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 折叠菜单 -->
        <div class="toggle_menu" @click="toggleMenu">|||</div>
        <!-- 侧边栏菜单区 -->
        <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409EFF"
          :unique-opened="true"
          :collapse="isCollapse"
          :collapse-transition="false"
          router
          :default-active="menuActive"
        >
        <!-- 一级菜单 -->
          <el-submenu :index="item.id+''" v-for=" item in menulist" :key="item.id">
            <template slot="title">
              <!-- 图标 -->
              <i :class="iconObj[item.id]"></i>
              <!-- 文本 -->
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+ subItem.path+''" v-for="subItem in item.children" :key="subItem.id" @click="setMenuActive('/'+ subItem.path+'')">
              <i class="el-icon-menu"></i>
              <span>{{subItem.authName}}</span>
              </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右边主题 -->
      <el-main>
        <!-- 路由占位符 -->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menulist: [],
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-lock_fill',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      menuActive: ''
    }
  },
  created() {
    this.getMenuList()
    this.menuActive = window.sessionStorage.getItem('menuPathActive')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    /* 获取菜单数据 */
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取菜单出错')
      this.menulist = res.data
    },
    toggleMenu() {
      this.isCollapse = !this.isCollapse
    },
    /* 将二级菜单地址存储到session中 */
    setMenuActive(menuPathActive) {
      window.sessionStorage.setItem('menuPathActive', menuPathActive)
      this.menuActive = menuPathActive
    }
  }
}
</script>

<style lang="less" scoped>
.home_container {
  padding: 0;
  margin: 0;
  height: 100%;
}
.el-header {
  display: flex;
  padding-left: 0;
  justify-content: space-between;
  background-color: #373d41;
  align-items: center;
  font-size: 20px;
  color: #fff;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}

.el-aside {
  background-color: #333744;
}

.el-main {
  background-color: #eaedfa;
}

.el-menu {
  border-right: 0;
  span {
    margin-left: 15px;
  }
}
.toggle_menu {
  width: 100%;
  height: 24px;
  color: #fff;
  font-size: 10px;
  background-color: #4a5064;
  text-align: center;
  line-height: 24px;
  letter-spacing: 0.2em;
}
</style>
