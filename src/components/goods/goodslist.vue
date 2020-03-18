<template>
  <el-card class="box-card">
    <!-- 1.面包屑 -->
    <my-bread level1="商品管理" level2="商品列表"></my-bread>
    <!-- 2.搜索 -->
    <el-row class="searchRow">
      <el-col>
        <el-input
          v-model="query"
          class="inputSearch"
          clearable
          placeholder="请输入内容"
          @clear="loadGoodsList()"
        >
          <el-button @click="serchGoods()" slot="append" icon="el-icon-search"></el-button>
        </el-input>
        <el-button type="success" @click="$router.push({name:'goodsadd'})">添加商品</el-button>
      </el-col>
    </el-row>
    <!-- 3.表格 -->
    <el-table height="300px" :data="goodslist" style="width: 100%">
      <el-table-column type="index" label="#" width="60"></el-table-column>
      <el-table-column prop="goods_name" label="商品名称" width="80"></el-table-column>
      <el-table-column prop="goods_price" label="商品价格(元)"></el-table-column>
      <el-table-column prop="goods_weight" label="商品重量"></el-table-column>
      <el-table-column label="创建日期">
        <template slot-scope="scope">{{scope.row.add_time | fmtdate}}</template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            size="mini"
            plain
            type="primary"
            icon="el-icon-edit"
            circle
            @click="showEditGoodsDia(scope.row)"
          ></el-button>
          <el-button
            size="mini"
            plain
            type="danger"
            icon="el-icon-delete"
            circle
            @click="showDeleGoodsMsgBox(scope.row.id)"
          ></el-button>          
        </template>
      </el-table-column>
    </el-table>
    <!-- 4.分页 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
  </el-card>
</template>
<script>
export default {
  data() {
    return {
      query: "",
      //表格绑定的数据
      goodslist: [],
      //分页相关数据
      total: -1,
      pagenum: 1,
      pagesize: 2
    };
  },
  created() {
    this.getUseList();
  },
  methods: {
    //分页相关的方法
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.pagesize = val;
      //回到第一页
      this.pagenum = 1;
      this.getUseList();
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.pagenum = val;
      this.getDoodsList();
    },
    //获取商品列表的请求
    async getDoodsList() {
      const res = await this.$http.get(
        `goods?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`
      )
      //console.log(res)
      
      const {
        meta: { status, msg },
        data: { goodslist, total }
      } = res.data;
      if (status === 200) {
        //1.给表格数据赋值
        this.goodslist = goodslist;
        //2.给total赋值
        this.total = total;
        //3.提示
        this.$message.success(msg);
      } else {
        //提示
        this.$message.warning(msg);
      }
    },
    //搜索商品
    serchGoods() {
      // 按照input绑定的query参数 发请求
      this.getDoodsList()
    },
    //清空搜索框 重新获取数据
    loadUserList() {
      this.getDoodsList();
    },
    //删除用户-打开消息盒子(config)
    showDeleGoodsMsgBox(goodsId) {
      this.$confirm("删除用户?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          //发送删除请求  :id----> 用户id
          //1.data中找userId ---- 行不通
          //2.把userId以showDeleUserMsgBox参数传进来
          const res = await this.$http.delete(`goods/${goodsId}`)
          //console.log(res)

          if (res.data.meta.status === 200) {
            this.pagenum = 1;
            //更新视图
            this.getDoodsList();
            //提示
            this.$message({
              type: "success",
              message: res.data.meta.msg
            })
          }
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          })
        })
    },
  }
};
</script>
<style>
.box-card {
  height: 100%;
}
.searchRow {
  margin-top: 20px;
}
.inputSearch {
  width: 300px;
}
</style>