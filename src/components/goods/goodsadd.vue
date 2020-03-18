<template>
    <el-card class="box-card">
        <!-- 1.面包屑 -->
        <my-bread level1="商品管理" level2="商品列表"></my-bread>
        <!-- 提示 el-alert -->
        <el-alert class="alert" title="天机商品信息" type="success" center show-icon></el-alert>
        <!-- 步骤条 el-steps -->
        <el-steps :active="active" finish-status="success" simple style="margin-top: 20px">
            <el-step title="基本信息" ></el-step>
            <el-step title="商品参数" ></el-step>
            <el-step title="商品属性" ></el-step>
            <el-step title="商品图片" ></el-step>
            <el-step title="商品内容" ></el-step>
        </el-steps>
        <!-- 最外层是 el-form -->
        <el-form label-position="top" label-width="80px" :model="form" style="height: 400px;overflow:auto">
            <!-- tabs标签页 el-tabs -->
            <el-tabs @tab-click="tabChange()" v-model="active" tab-position="left">
                <el-tab-pane name="1" label="基本信息">
                    <el-form-item label="商品名称">
                        <el-input v-model="form.goods_name"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格">
                        <el-input v-model="form.goods_price"></el-input>
                    </el-form-item>
                    <el-form-item label="商品重量">
                        <el-input v-model="form.goods_weight"></el-input>
                    </el-form-item>
                    <el-form-item label="商品数量">
                        <el-input v-model="form.goods_number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品分类">
                        <!-- 级别选择器 el-cascader -->
                        <el-cascader
                        clearable
                        expand-trigger="hover"
                        :options="options"
                        v-model="selectedOptions"
                        :props="defaultProp"
                        @change="handleChange"></el-cascader>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane name="2" label="商品参数">
                    <!-- 显示的是该三级分类的商品参数 -->
                    <el-form-item
                    v-for="(item1,i) in arrDyparams" 
                    :key="i"
                    :label="item1.attr_name">
                        <el-checkbox-group v-model="item1.attr_vals">
                            <el-checkbox 
                            border
                            v-for="(item2,i) in item1.attr_vals"
                            :key="i"
                            :label="item2"></el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane name="3" label="商品属性">
                    <!-- 静态参数的数据 -->
                    <el-form-item
                    v-for="(item,i) in arrStaticparams"
                    :label="item.attr_name"
                    :key="i">
                        <el-input v-model="item.attr_vals"></el-input>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane name="4" label="商品图片">
                    <el-form-item>
                        <!-- baseUrl 是axios发的请求 -->
                        <el-upload
                        action="http://127.0.0.1:8888/api/private/v1/upload"
                        :headers="headers"
                        :on-preview="handlePreview"
                        :on-remove="handleRemove"
                        :on-success="handleSuccess"
                        list-type="picture">
                            <el-button size="small" type="primary">点击上传</el-button>
                        <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
                        </el-upload>
                    </el-form-item>                    
                </el-tab-pane>
                <el-tab-pane name="5" label="商品内容">
                    <el-form-item>
                        <!-- 表单元素 -->
                        <el-button type="primary" @click="addGoods()">添加商品</el-button>
                        <!-- 富文本 -->
                        <quill-editor v-model="form.goods_introduce"></quill-editor>
                    </el-form-item>    
                </el-tab-pane>
            </el-tabs>
        </el-form>
        
    </el-card>    
</template>
<script>
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
 
import { quillEditor } from 'vue-quill-editor'

export default {
    data(){
        return{
            active:'1',
            //添加商品的表单数据
            form:{
                goods_name:'',
                goods_cat:'',
                goods_price:'',
                goods_number:'',
                goods_weight:'',
                goods_introduce:'',
                pics:[],
                attrs:[]
            },
            //级联选择器绑定的数据
            options:[],
            selectedOptions:[],
            defaultProp:{
                label:'cat_name',
                value:'cat_id',
                children:'children'
            },
            //动态参数的数据数组
            arrDyparams:[],
            arrStaticparams:[],
            headers:{
                Authorization:localStorage.getItem("token")
            }
        }
    },
    components: {
        quillEditor
    },
    created(){
        this.getGoodCate()
    },
    methods:{
        //级联选择器@change触发的方法
        handleChange(){

        },
        //获取三级分类的信息
        async getGoodCate() {
            const res = await this.$http.get(`categories?type=3`)
            //console.log(res)
            this.options = res.data.data
        },
        //点击不同的tab时
        async tabChange(){
            //如果点击的是第二个tab 同时 三级分类要有值
            if(this.active === '2'){
                if(this.selectedOptions.length !== 3){
                    //提示
                    this.$message.warning('请选择商品的三级分类')
                    return
                }
                //获取动态参数的数据
                //id -> 分类id
                const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=many`)
                //console.log(res)
                this.arrDyparams = res.data.data
                //this.arrDyparams每个对象.attr_vals字符串->数组->v-for
                this.arrDyparams.forEach(item => {
                    //并不是所有的三级分类都有动态参数 -> "" -> [] -> v-for报错
                    item.attr_vals=
                    item.attr_vals.length === 0
                    ? [] : item.attr_vals.trim().split(',')
                })
            }else if(this.active === '3'){
                if(this.selectedOptions.length !== 3){
                    //提示
                    this.$message.warning('请选择商品的三级分类')
                    return
                }
                //获取静态参数的数据
                const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=only`)
                //console.log(res)
                this.arrStaticparams = res.data.data
            }
        },
        // 图片上传时的相关方法
        //file 形参里面是当前操作的图片的相关信息(图片名/图片路径)
        handlePreview(file){
            
        },
        handleRemove(file){
            console.log("移除----"+file)
            //file.response.data.tem_path 图片临时上传的路径
            //先获取该图片的索引
            //findIndex() 遍历 把符合条件的元素的索引进行返回
            let Index = this.form.pics.findIndex((item) =>{
                return item.pic === file.response.data.tem_path
            })
            this.form.pics.splice(Index,1)
        },
        handleSuccess(file){
            console.log("成功----"+file)
            //file.data.tem_path 图片临时上传的路径
            this.form.pics.push({
                pic:file.data.tmp_path
            })
        },
        //添加商品-发送请求
        async addGoods(){
            //在发请求之前 处理this.form中的未处理数据
            this.form.goods_cat=this.selectedOptions.join(',')

            //attrs[{attr_id:?,attr_vals:?}]
            //动态参数数组
            // 遍历+取值+ 放在一个新数组中
            //console.log(this.arrDyparams)
            let arr1=this.arrDyparams.map((item)=>{
                return {attr_id:item.attr_id,attr_vals:item.attr_vals}
            })
            //console.log(arr1)

            //静态参数数组
            //console.log(this.arrStaticparams)
            let arr2=this.arrStaticparams.map((item)=>{
                return {attr_id:item.attr_id,attr_vals:item.attr_vals}
            })
            //console.log(arr2)

            this.form.attrs=[...arr1,...arr2]

            const res = await this.$http.get(`goods`,this.form)
            //console.log(res)
            //回到商品列表
            this.$router.push({name:"goods"})
        }
    }
}
</script>
<style>
.alert{
    margin-top: 20px;
}
.ql-editor{
    min-height: 300px;
}
</style>