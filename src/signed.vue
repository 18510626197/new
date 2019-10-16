@ -0,0 +1,40 @@
<template>
    <div style="text-align: center;color: #FFFFFF;">
    </div>
</template>

<script>
    import querystring from 'querystring'
    export default {
        name: "Signed",
        created() {
            const sso = querystring.parse(window.location.search.slice(1)).sso;
            window.console.log(sso);
            const that = this;
            this.axios.get(this.GLOBAL_API.apiUrl + "base/authentication/signIn?sso="+sso).then(function(res){
                window.console.log(res);
                if (res.data.errcode === 0){
                    that.$cookies.set('username', res.data.data.username, 60*60*24*3);
                    that.$cookies.set('token', res.data.data.token, 60*60*24*3);
                    window.location.href = that.GLOBAL_API.hostUrl;
                }else{
                    that.$notify({
                        title: '操作异常',
                        type: "warning",
                        message: "您的操作异常，导致被拒绝，无法正常登录。"
                    })
                }
            }).catch(function (err) {
                that.$notify({
                    title: '操作异常',
                    type: "warning",
                    message: "您的操作异常，导致被拒绝，无法正常登录。" + err
                })
            })
        }
    }
</script>

<style scoped>

</style>