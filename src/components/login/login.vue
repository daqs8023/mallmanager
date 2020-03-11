<template>
    <div class="login-wrap">
        <!-- 登录表单 -->
        <el-form 
        class="login-form"
        label-position="top" 
        label-width="80px" 
        :model="formdata">
        <h2>用户登录</h2>
            <el-form-item label="用户名">
                <el-input v-model="formdata.name"></el-input>
            </el-form-item>
            <el-form-item label="密码">
                <el-input v-model="formdata.password" show-password></el-input>
            </el-form-item>
            <el-button class="login-btn" type="primary" @click.prevent="handleLogin">登录</el-button>
        </el-form>
    </div>
</template>
<script>
export default {
    data () {
        return {
            formdata: {
                username: '',
                password: ''
            }
        }    
    },
    methods:{
        async handleLogin(){
            //希望 让异步操作的代码 看起来像同步代码
            //ES7 async+await
            const res =await this.$http.post('login',this.formdata)
            
            const {
                data,
                meta:{msg,status}
            } =res.data

                            
            if(status ===200){
                //登录成功 
                //0.保存token
                localStorage.seItem('token',data.token)
                //1.跳转home
                this.router.push({name:'home'})
                //2.提示成功
                this.$message.success(msg)
            }else{
                //不成功
                //1.提示信息
                this.$message.warning(msg)
            }             
        }
    }
}
</script>
<style>
.login-wrap{
    height: 100%;
    background: #324152;
    /*弹性盒布局*/
    display: flex;
    justify-content: center;
    align-items: center;
}
.login-wrap .login-form{
    width: 400px;
    background: #fff;
    border-radius:5px ;
    padding: 30px;
}
.login-wrap .login-btn{
    width: 100%;
}
</style>