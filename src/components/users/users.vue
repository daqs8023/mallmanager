<template>
  <div>
    <el-card class="box-card">
      <!-- 1.面包屑 -->
      <my-bread level1="用户管理" level2="用户列表"></my-bread>
      <!-- 2.搜索 -->
      <el-row class="searchRow">
        <el-col>
          <el-input
            placeholder="请输入内容"
            v-model="query"
            class="inputSearch"
            clearable
            @clear="loadUserList()"
          >
            <el-button @click="serchUser()" slot="append" icon="el-icon-search"></el-button>
          </el-input>
          <el-button type="success" @click="showAddUserDia()">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 3.表格 -->
      <el-table height="300px" :data="userlist" style="width: 100%">
        <el-table-column type="index" label="#" width="60"></el-table-column>
        <el-table-column prop="username" label="姓名" width="80"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <!-- 如果单元格内现实的内容不是字符串(文本)
        需要给被显示的内容外层包裹一个template-->

        <!-- template内部要用数据 设置slot-scope属性
        该属性的值是要用数据create_time的数据源userlist-->

        <!-- slot-scope的值userlist其实就是el-table绑定的数据userlist
        userlist.row -> 数组中的每个对象-->

        <!-- slot-scope="scope"中的scope其实就是数据源userlist -->
        <el-table-column label="创建时间">
          <template slot-scope="scope">{{scope.row.create_time | fmtdate}}</template>
        </el-table-column>
        <el-table-column label="用户状态">
          <template slot-scope="scope">
            <el-switch 
            @change="changeMgState(scope.row)"
            v-model="scope.row.mg_state" 
            active-color="#13ce66" 
            inactive-color="#ff4949">
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              plain
              type="primary"
              icon="el-icon-edit"
              circle
              @click="showEditUserDia(scope.row)"
            ></el-button>
            <el-button
              size="mini"
              plain
              type="danger"
              icon="el-icon-delete"
              circle
              @click="showDeleUserMsgBox(scope.row.id)"
            ></el-button>
            <el-button 
            size="mini" 
            plain 
            type="success" 
            icon="el-icon-check" 
            circle
            @click="showSetUserRoleDia(scope.row)"
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

      <!-- 对话框-添加用户的对话框 -->
      <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
        <el-form :model="form">
          <el-form-item label="用户名" label-width="100px">
            <el-input v-model="form.username" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码" label-width="100px">
            <el-input v-model="form.password" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" label-width="100px">
            <el-input v-model="form.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="电话" label-width="100px">
            <el-input v-model="form.mobile" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
          <el-button type="primary" @click="addUser()">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 对话框-编辑用户的对话框 -->
      <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
        <el-form :model="form">
          <el-form-item label="用户名" label-width="100px">
            <el-input v-model="form.username" autocomplete="off" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" label-width="100px">
            <el-input v-model="form.email" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="电话" label-width="100px">
            <el-input v-model="form.mobile" autocomplete="off"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisibleEdit = false">取 消</el-button>
          <el-button type="primary" @click="editUser()">确 定</el-button>
        </div>
      </el-dialog>

      <!-- 对话框-分配角色的对话框 -->
      <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRol">
        <el-form :model="form">
          <el-form-item label="用户名" label-width="100px">
            {{currUsername}}
          </el-form-item>
          <el-form-item label="角色" label-width="100px">
              <!-- 如果select的绑定的数据的值 和 option的value一样 就会显示该option的label值-->
            <el-select v-model="currRoleId">
                <el-option label="请选择" :value="-1"></el-option>
                <el-option 
                v-for="(item,i) in roles" 
                :key="i"
                :label="item.roleName"
                :value="item.id"
                ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogFormVisibleRol = false">取 消</el-button>
          <el-button type="primary" @click="setRole()">确 定</el-button>
        </div>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      query: "",
      //表格绑定的数据
      userlist: [],
      //分页相关数据
      total: -1,
      pagenum: 1,
      pagesize: 2,
      //添加对话框的属性
      dialogFormVisibleAdd: false,
      dialogFormVisibleEdit: false,
      dialogFormVisibleRol:false,
      //添加用户的表单数据
      form: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      //分配角色
      currRoleId:-1,
      currUserId:-1,
      currUsername:'',
      //保存所有角色数据
      roles:[]
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
      this.getUseList();
    },
    //获取用户列表的请求
    async getUseList() {
      const res = await this.$http.get(
        `users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`
      )
      //console.log(res)
      
      const {
        meta: { status, msg },
        data: { users, total }
      } = res.data;
      if (status === 200) {
        //1.给表格数据赋值
        this.userlist = users;
        //2.给total赋值
        this.total = total;
        //3.提示
        this.$message.success(msg);
      } else {
        //提示
        this.$message.warning(msg);
      }
    },
    //搜索用户
    serchUser() {
      // 按照input绑定的query参数 发请求
      this.getUseList()
    },
    //清空搜索框 重新获取数据
    loadUserList() {
      this.getUseList();
    },
    //添加用户-显示对话框
    showAddUserDia() {
      //清空文本框
      //this.form={}
      //或者遍历对象--清空
      for (const key in this.form) {
        if (this.form.hasOwnProperty(key)) {
          this.form[key] = "";
        }
      }
      this.dialogFormVisibleAdd = true
    },
    //添加用户-发送请求
    async addUser() {
      //1.关闭对话框
      this.dialogFormVisibleAdd = false;

      const res = await this.$http.post(`users`, this.form)
      //console.log(res)
      const {
        meta: { status, msg },
        data: { users, total }
      } = res.data;
      if (status === 201) {
        //2.提示
        this.$message.success(msg)
        //3.更新视图
        this.getUseList();
      } else {
        //提示
        this.$message.warning(msg)
      }
    },
    //删除用户-打开消息盒子(config)
    showDeleUserMsgBox(userId) {
      this.$confirm("删除用户?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          //发送删除请求  :id----> 用户id
          //1.data中找userId ---- 行不通
          //2.把userId以showDeleUserMsgBox参数传进来
          const res = await this.$http.delete(`users/${userId}`)
          //console.log(res)

          if (res.data.meta.status === 200) {
            this.pagenum = 1;
            //更新视图
            this.getUseList();
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
    //编辑用户-显示对话框
    showEditUserDia(user) {
      //获取用户数据
      this.form = user
      this.dialogFormVisibleEdit = true
    },
    //编辑用户-发送请求
    async editUser() {
      //1.关闭对话框
      this.dialogFormVisibleEdit = false;

      const res = await this.$http.put(`users/${this.form.id}`, this.form)
      //console.log(res)
      const {
        meta: { status, msg },
        data: { users, total }
      } = res.data;
      if (status === 200) {
        //2.提示
        this.$message.success(msg);
        //3.更新视图
        this.getUseList();
      } else {
        //提示
        this.$message.warning(msg);
      }
    },
    //修改状态
    async changeMgState(user){
        //发送请求
        const res = await this.$http.put(`users/${user.id}/state/${user.mg_status}`)
        //console.log(res)
    },
    //分配角色-打开对话框
    async showSetUserRoleDia(user){
        this.currUsername=user.username
        //给currUserId赋值
        this.currUserId=user.id

        //获取所有的角色
        const res1 = await this.$http.get(`roles`)
        //console.log(res1)
        this.roles = res1.data.data

        //获取当前用户的角色id -> rid
        const res = await this.$http.get(`users/${user.id}`)
        //console.log(res)        
        this.currRoleId=res.data.data.rid

        this.dialogFormVisibleRol=true
    },
    //分配角色-发送请求
    async setRole(){
        //:id 要修改的用户id值
        //请求体中 rid 修改的新的角色id
        const res = await this.$http.put(`users/${this.currUserId}/role`, {rid:this.currRoleId})
        //console.log(res)  
        //关闭对话框
        this.dialogFormVisibleRol=false
        const {
            meta: { status, msg },
            data: { users, total }
        } = res.data;
        if (status === 200) {
            //2.提示
            this.$message.success(msg);
            //3.更新视图
            this.getUseList();
        } else {
            //提示
            this.$message.warning(msg);
        }
    }
  }
}
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