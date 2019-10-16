@@ -0,0 +1,53 @@
<template>
    <div class="container-manage">
        <el-page-header @back="goBack" content="赛事管理">
        </el-page-header>
        <el-tabs v-model="activeName" @tab-click="handleClick">
            <el-tab-pane label="报名列表" name="list">
                <ManageUserList></ManageUserList>
            </el-tab-pane>
            <el-tab-pane label="裁判管理" name="reference">
                <ManageReferee></ManageReferee>
            </el-tab-pane>
            <el-tab-pane label="赛事统计" name="analyze">

            </el-tab-pane>
        </el-tabs>
    </div>
</template>

<script>
    import ManageUserList from './ManageUserList'
    import ManageReferee from './ManageReferee'
    export default {
        name: "Manage",
        data(){
            return {
                activeName: 'list'
            }
        },
        methods: {
            goBack: function () {
                this.$router.go(-1);
            },
        },
        components:{
            ManageUserList,
            ManageReferee
        }
    }
</script>

<style>
    .container-manage{
        width: 80%;
        margin: 10px auto 0;
        min-width: 1000px;
    }
    .el-page-header__left{
        color: #FFFFFF;
    }
    .el-page-header__content{
        color: #FFFFFF;
    }
</style>