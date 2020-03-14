<template>
  <!-- 头部 -->
  <el-container class="container">
    <el-header class="header">
      <el-row>
        <el-col :span="4">
          <div class="grid-content bg-purple">
            <img src="../../assets/images/logo.png" alt="无法显示图片" />
          </div>
        </el-col>
        <el-col :span="18" class="middle">
          <h3>电商后台管理系统</h3>
        </el-col>
        <el-col :span="2">
          <div class="grid-content bg-purple">
            <a class="loginout" href="#" @click.prevent="handleSignout()">退出</a>
          </div>
        </el-col>
      </el-row>
    </el-header>

    <el-container>
      <el-aside class="aside" width="200px">
        <!-- 侧边栏导航el-menu -->
        <!-- 开启路由模式 -->
        <el-menu
        :router="true"
        :unique-opened="true">
          <el-submenu v-for="(item1,index) in menus" :key="index" :index="''+item1.order">
            <template slot="title">
              <i class="el-icon-location"></i>
              <span>{{item1.authName}}</span>
            </template>
            <el-menu-item v-for="(item2,index) in item1.children" :key="index" :index="item2.path">
              <i class="el-icon-menu"></i>
              <span>{{item2.authName}}</span>
            </el-menu-item>
          </el-submenu>          
        </el-menu>
      </el-aside>
      <el-main class="main">
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>
<script>
export default {    
    data(){
      return{
        menus:[]
      }
    },
    created(){
      this.getMenus()
    },
    methods:{
        handleSignout(){
            //1.清除token
            localStorage.clear()
            //2.提示
            this.$message.success('退出成功')
            //3.来到login组件
            this.router.push({name:'login'})
        },
        //获取导航数据
        async getMenus(){
          const res = await this.$http.get(`menus`)
          //console.log(res)
          this.menus=res.data.data
        }
    }
};
</script>
<style>
.container {
  height: 100%;
}
.header {
  background-color: #b3c0d1;
}
.aside {
  background-color: #d3dce6;
}
.main {
  background-color: #e9eef3;
}
/*头部样式*/
.middle {
  text-align: center;
  color: #fff;
}
.loginout {
  line-height: 60px;
  text-decoration: none;
}
</style>