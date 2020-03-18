<template>
    <el-card class="box-card">
        <!-- 1.面包屑 -->
        <my-bread level1="商品管理" level2="商品列表"></my-bread>
        <el-row>
            <el-col :span="24" class="searchArea">
                <el-button type="success" @click="addGoodsCate()">添加分类</el-button>
            </el-col>
        </el-row>
        <!-- 3.表格 -->
        <el-table height="450px" :data="list" style="width: 100%">
            <!-- <el-table-column label="分类名称" prop="cat_name" width="60"> -->
            <!-- 
                treeKey -> nodeKey -> 节点唯一标识 id
                parentKey -> 父节点的id
                levelKey -> 当前节点的级别
                childKey -> 子节点
             -->
            <el-tree-grid 
            prop="cat_name" 
            label="分类名称" 
            treeKey="cat_id" 
            parentKey="cat_pid" 
            levelKey ="cat_level"
            childKey="children">
                
            </el-tree-grid>   
            <el-table-column label="级别" width="100">
                <template slot-scope="scope">
                    <span v-if="scope.row.cat_level === 0">一级</span>
                    <span v-else-if="scope.row.cat_level === 1">二级</span>
                    <span v-else-if="scope.row.cat_level === 2">三级</span>
                </template>
            </el-table-column>
            <el-table-column label="是否有效" width="100">
                <template slot-scope="scope">
                    <span v-if="scope.row.cat_deleted === false">有效</span>
                    <span v-else-if="scope.row.cat_deleted === true">无效</span>
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
                    @click="showEditCateDia(scope.row)"
                ></el-button>
                <el-button
                    size="mini"
                    plain
                    type="danger"
                    icon="el-icon-delete"
                    circle
                    @click="showDeleCateMsgBox(scope.row.id)"
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
        :page-sizes="[5, 10, 15, 20]"
        :page-size="pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        ></el-pagination>


        <!-- 添加分类对话框 -->
        <el-dialog title="添加分类" :visible.sync="dialogFormVisibleAdd">
            <el-form :model="form">
                <el-form-item label="分类名称" label-width="140px">
                <el-input v-model="form.cat_name" autocomplete="off"></el-input>
                </el-form-item>
                <el-form-item label="分类" label-width="140px">
                    <!-- 级别选择器 el-cascader -->
                    <el-cascader
                    expand-trigger="hover"
                    change-on-select
                    clearable
                    :options="caslist"
                    v-model="selectedOptions"
                    :props="defaultProp"
                    ></el-cascader>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
                <el-button type="primary" @click="addCate()">确 定</el-button>
            </div>
        </el-dialog>
    </el-card>    
</template>
<script>
//引入element-tree-grid
var ElTreeGrid = require("element-tree-grid")
export default {
    //组件名 <el-tree-grid>
    components:{
        ElTreeGrid
    },
    data(){
        return{
            list:[],
            //分页相关数据
            pagenum: 1,
            pagesize: 10,
            total: 1,            
            //添加对话框的属性
            dialogFormVisibleAdd: false,
            form:{
                cat_pid:-1,
                cat_name:"",
                cat_level:-1
            },
            //级联选择器绑定的数据
            caslist:[],
            selectedOptions:[],
            defaultProp:{
                label:'cat_name',
                value:'cat_id',
                children:'children'
            },
        }
    },
    methods:{
        //添加分类-发送请求
        async addCate(){
            //三种情况
            //1.一级分类 selectedOptions.length===0 则 cat_pid=0 cat_level=0
            //2.二级分类 selectedOptions.length===1 则 cat_pid=selectedOptions[0] cat_level=1
            //2.三级分类 selectedOptions.length===2 则 cat_pid=selectedOptions[1] cat_level=2
            if(this.selectedOptions.length===0){
                this.form.cat_pid=0
                this.form.cat_level=0
            }else if(this.selectedOptions.length===1){
                this.form.cat_pid=this.selectedOptions[0]
                this.form.cat_level=1
            }else if(this.selectedOptions.length===2){
                this.form.cat_pid=this.selectedOptions[1]
                this.form.cat_level=2
            }
            const res = await this.$http.post(`categories`,this.form)
            //console.log(res)

            //更新视图
            this.getGoodsCate()
            //关闭对话框
            this.dialogFormVisibleAdd = false
            //清空form
            this.form={}
        },
        //添加分类-显示对话框
        async addGoodsCate(){
            //获取二级分类的数据
            const res = await this.$http.get(`categories?type=2`)
            //console.log(res)
            this.caslist = res.data.data
            this.dialogFormVisibleAdd = true
        },
        //分页相关的方法
        handleSizeChange(val) {
            console.log(`每页 ${val} 条`);
            this.pagesize = val;
            //回到第一页
            this.pagenum = 1;
            this.getGoodsCate();
        },
        handleCurrentChange(val) {
            console.log(`当前页: ${val}`);
            this.pagenum = val;
            this.getGoodsCate();
        },
        //获取所有分类
        async getGoodsCate(){
            const res = await this.$http.get(
                `categories?type=3&pagenum=${this.pagenum}&pagesize=${this.pagesize}`
            )
            //console.log(res)
            this.list = res.data.data.result
            this.total=res.data.data.total
        }
    }
}
</script>
<style>
.searchRow {
  margin-top: 20px;
}
</style>