@@ -0,0 +1,194 @@

<template>
    <div class="components-list">
        <!--
        <el-card shadow="hover" class="download-card">
            自定义文件下载名 <br>
            MyCard账号：{mycardid}，参赛ID：{eventid}，QQ账号：{qq}，微信账号：{wechat}，手机号：{phone};<br>
            范例：{eventid}+{qq}, 则输出的文件名为帕尼+1173440278<br><br>
            <el-input v-model="postData.format" placeholder="请输入内容"></el-input>
            <br><br>
            <el-button type="success">保存</el-button>
            <el-button type="danger">清空</el-button>
        </el-card>
        <br>
        <el-button type="success" class="download-button">批量下载卡组文件</el-button>
        <br>-->
        <el-table
                :data="tableData"
                style="width: 100%">
            <el-table-column
                    label="用户名"
                    width="140">
                <template slot-scope="scope">
                    <span>{{ scope.row.username }}</span>
                </template>
            </el-table-column>
            <el-table-column
                    label="参赛ID"
                    width="180">
                <template slot-scope="scope">
                    <span>{{ scope.row.base_info.nickname }}</span>
                </template>
            </el-table-column>
            <el-table-column
                    label="手机号"
                    width="120">
                <template slot-scope="scope">
                    <span>{{ scope.row.base_info.phone }}</span>
                </template>
            </el-table-column>
            <el-table-column
                    label="QQ"
                    width="120">
                <template slot-scope="scope">
                    <span>{{ scope.row.base_info.qq }}</span>
                </template>
            </el-table-column>
            <el-table-column
                    label="微信号"
                    width="180">
                <template slot-scope="scope">
                    <span>{{ scope.row.base_info.wechat }}</span>
                </template>
            </el-table-column>
            <el-table-column
                    label="报名时间"
                    width="180">
                <template slot-scope="scope">
                    <span>{{ scope.row.update_time }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作">
                <template slot-scope="scope">
                    <!--<el-button
                            size="mini"
                            >下载卡组</el-button>-->
                    <el-button
                            size="mini"
                            type="danger"
                            @click="delRegister(scope.row.id)"
                    >移除选手</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>

<script>

    export default {
        name: "ManageUserList",
        data(){
            return {
                tid: undefined,
                tableData:[
                ],
                postData:{
                    format: '{mycardid}+{qq}'
                }
            }
        },
        created(){
            this.tid = this.$route.params.tid;
            this.getUserList();
        },
        methods:{
            getUserList: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getRegisterList',{
                    tid: this.tid

                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.tableData = res.data.data;
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            },
            delRegister: function(rid){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/delRegister',{
                    tid: this.tid,
                    registerID: rid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.$notify({
                            type: "success",
                            title: "剔除成功",
                            message: "即将为您刷新数据"
                        });
                        that.getUserList();
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            }
        }
    }
</script>

<style>
    .el-table{
        background-color: #303133 !important;
        border-radius: 5px;
        overflow: hidden;
    }
    .el-table__header{
        background: #303133 !important;
    }
    .el-table__header-wrapper{
        background-color: #303133 !important;
    }
    thead, tr, th{
        background-color: #303133 !important;
        border-bottom: 1px solid #999999 !important;

    }
    th{
        color: #999999;
    }
    tr{
        color: #FFFFFF;
    }
    .el-table--enable-row-hover .el-table__body tr:hover>td{
        background-color: #212e3e !important;
    }
    .el-table__body tr>td{
        border-bottom: 1px solid #999999 !important;
    }
    .download-card{
        background: #303133 !important;
        border: 1px solid transparent;
        color: #FFFFFF;
    }
    .download-button{
        width: 100%;
        display: block;
    }
</style>