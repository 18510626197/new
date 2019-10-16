@ -0,0 +1,45 @@
<template>
    <div style="text-align: center;color: #FFFFFF;">
        <h1><i class="el-icon-loading"></i> 正在跳转到MyCard用户中心进行登录</h1>
    </div>
</template>

<script>
    export default {
        name: "SignIn",
        created() {
            this.getSignUrl();
        },
        methods:{
            getSignUrl: function(){
                window.console.log(this.GLOBAL_API.apiUrl);
                this.axios.get(this.GLOBAL_API.apiUrl + 'base/authentication/getAuthenticationURL').then(function(res){
                    if(res.data.errcode === 0){
                        window.location.href = res.data.data.url;
                    }else{
                        this.$notify({
                            title: '服务器异常',
                            type: "warning",
                            message: "服务器或者您的网络发生异常，无法获取登录地址。"
                        })
                    }

                    /*
                    if(res.data.errcode === 0){
                        window.location.href = res.data.data;
                    }*/
                }).catch(function(err){
                    this.$notify({
                        title: '服务器异常',
                        type: "warning",
                        message: "服务器或者您的网络发生异常，无法获取登录地址。" + err
                    })
                })
            }
        }
    }
</script>

<style scoped>

</style>