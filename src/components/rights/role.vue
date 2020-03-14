<template>
  <el-card class="box-card">
    <!-- 1.面包屑 -->
    <my-bread level1="权限管理" level2="角色列表"></my-bread>
    <!-- 2.添加按钮 -->
    <el-row class="addrolebtn">
      <el-col>
        <el-button type="info" @click="showAddRoleDia()">添加角色</el-button>
      </el-col>
    </el-row>
    <!-- 3.表格 -->
    <el-table :data="roleList" style="width: 100%">
      <el-table-column type="expand" width="40">
        <template slot-scope="scope">
          <el-row v-for="(item1,i) in scope.row.children" :key="i">
            <el-col :span="4">
              <!-- 传角色id和权限id -->
              <el-tag closable @close="deleRight(scope.row,item1.id)">{{item1.authName}}</el-tag>
              <i class="el-icon-arrow-right"></i>
            </el-col>
            <el-col :span="20">
              <el-row v-for="(item2,i) in item1.children" :key="i">
                <el-col :span="4">
                  <el-tag
                    closable
                    @close="deleRight(scope.row,item2.id)"
                    type="success"
                  >{{item2.authName}}</el-tag>
                  <i class="el-icon-arrow-right"></i>
                </el-col>
                <el-col :span="20">
                  <el-tag
                    v-for="(item3,i) in item2.children"
                    :key="i"
                    closable
                    @close="deleRight(scope.row,item3.id)"
                    type="warning"
                  >{{item3.authName}}</el-tag>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
          <!-- 无权限的提示 -->
          <span v-if="scope.row.children.length === 0">未分配权限</span>
        </template>
      </el-table-column>
      <el-table-column type="index" label="#" width="80"></el-table-column>
      <el-table-column prop="roleName" label="角色名称" width="120"></el-table-column>
      <el-table-column prop="roleDesc" label="角色描述" width="300"></el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <el-button
            size="mini"
            plain
            type="primary"
            icon="el-icon-edit"
            circle
            @click="showEditRoleDia(scope.row)"
          ></el-button>
          <el-button
            size="mini"
            plain
            type="danger"
            icon="el-icon-delete"
            circle
            @click="showDeleRoleMsgBox(scope.row.id)"
          ></el-button>
          <el-button
            size="mini"
            plain
            type="success"
            icon="el-icon-check"
            circle
            @click="showSetRightDia(scope.row)"
          ></el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 修改权限的对话框 -->
    <el-dialog title="修改权限" :visible.sync="dialogFormVisibleRight">
      <!-- 
          树形结构 
          data->数据源[]
          show-checkbox -> 选择框
          node-key 每个节点的唯一标识 通常是data数据中key名id
          default-expanded-keys 默认展开 [要展开的节点的id]
          default-checked-keys [要选择的节点的id]
          props 配置项{label,children}
          label 节点的文字标题 和 children 节点的子节点
          值都来源于data绑定的数据源中的该数据的key名'label' 'children'
      -->
      <el-tree
        ref="tree"
        :data="treeList"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="arrcheck" 
        :props="defaultProps"
      ></el-tree>

      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisibleRight = false">取 消</el-button>
        <el-button type="primary" @click="setRoleRight()">确 定</el-button>
      </div>
    </el-dialog>
  </el-card>
</template>
<script>
export default {
  data() {
    return {
      roleList: [],
      dialogFormVisibleRight: false,
      //树形结构的数据
      treeList:[],
      defaultProps:{
        label:'authName',
        children:'children'
      },
      expanded:[],
      arrcheck:[],
      currRoleId:-1      
    };
  },
  created() {
    this.getRoleList();
  },
  methods: {
    async getRoleList() {
      const res = await this.$http.get(`roles`)
      //console.log(res)
      this.roleList = res.data.data
    },    
    //取消权限
    async deleRight(role, rightId) {
      //roleId 当前角色的id
      //rightId 要删除的权限id
      const res = await this.$http.delete(`roles/${role.id}/right/${rightId}`);
      //console.log(res)
      //删除成功 返回200 和 该角色的剩余权限
      role.children = res.data.data;
    },
    //修改/分配权限-显示对话框
    async showSetRightDia(role) {
      //console.log(role)
      //给currRoleId赋值
      this.currRoleId = role.id
      //获取树形结构的权限数据
      const res = await this.$http.get(`rights/tree`)
      //console.log(res)
      this.treeList = res.data.data
      //目的：展开所有相，获取所有节点的id
      /*let arrtemp1 = []
      this.treeList.forEach(item1 => {
        arrtemp1.push(item1.id)
        item1.children.forEach(item2 => {
          arrtemp1.push(item2.id)
            item2.children.forEach(item2 => {
            arrtemp1.push(item3.id)
          }) 
        }) 
      })*/

      //获取当前角色role的权限id
      let arrtemp2 = []
      role.children.forEach(item1 => {
        //arrtemp2.push(item1.id)
        item1.forEach(item2 => {
          //arrtemp2.push(item2.id)
          item2.forEach(item3 => {
            arrtemp2.push(item3.id)
          })
        })
      })
      this.arrcheck=arrtemp2
      this.dialogFormVisibleRight = true
    },
    //修改权限-发送请求
    async setRoleRight(){
      //roleId 当前要修改权限的角色id
      //rids 树形节点中 所有全选和版选的label的id[]
      //获取全选的id的数组arr1 getCheckedKeys()      
      let arr1=this.$refs.tree.getCheckedKeys()

      //获取半选的id的数组arr2 getHalfCheckedKeys()
      let arr2=this.$refs.tree.getHalfCheckedKeys()
      //ES6 展开运算符 ...数组或者对象
      let arr=[...arr1,...arr2]

      const res = await this.$http.post(`roles/${this.currRoleId}/rights`,{rids:arr.join(',')})
      //console.log(res)

      //更新视图
      this.getRoleList()
      //关闭对话框
      this.dialogFormVisibleRight = false
    }
  }
  
};
</script>
<style>
.addrolebtn {
  margin-top: 20px;
}
</style>
