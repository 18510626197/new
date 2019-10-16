@@ -0,0 +1,183 @@
<template>
    <div class="container-referee">
        <el-table
                :data="tableData"
                style="width: 100%">
            <el-table-column
                    label="用户名">
                <template slot-scope="scope">
                    <span>{{ scope.row.username }}</span>
                </template>
            </el-table-column>
            <el-table-column label="操作" width="150">
                <template slot-scope="scope">
                    <el-button
                            size="mini"
                            type="danger"
                            @click="delReferee(scope.row.username)"
                    >移除裁判</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-button type="success" class="addReferee-button" @click="dialogVisible = true">添加裁判</el-button>
        <el-dialog
                title="添加裁判"
                :visible.sync="dialogVisible"
                width="30%"
                :before-close="handleClose">
            <el-autocomplete
                    v-model="state"
                    :fetch-suggestions="querySearchAsync"
                    placeholder="输入用户名"
                    @select="handleSelect"
            >
                <template slot-scope="{ item }">
                    <div class="name">{{ item.username }}</div>
                </template>
            </el-autocomplete>
            <span slot="footer" class="dialog-footer">
                <el-button @click="dialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addReferee()">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "ManageReferee",
        data(){
            return {
                tableData:[
                ],
                dialogVisible: false,
                state: ''
            }
        },
        created(){
            this.tid = this.$route.params.tid;
            this.getReferee();
        },
        methods:{
            querySearchAsync: function(queryString, cb){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/searchUser',{
                    tid: this.tid,
                    name: queryString

                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        cb(res.data.data);
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            },
            handleSelect: function(item) {
                this.state = item.username;
                console.log(item);
            },
            getReferee: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getRefereeList',{
                    tid: this.tid,
                    name: this.state

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
            addReferee: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/addReferee',{
                    tid: this.tid,
                    name: this.state

                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.$notify({
                            type: "success",
                            title: "添加成功",
                            message: "即将为您刷新数据"
                        });
                        that.getReferee();
                        that.dialogVisible = false;
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }
                });
            },
            delReferee: function(name){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/delReferee',{
                    tid: this.tid,
                    name: name
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.$notify({
                            type: "success",
                            title: "删除成功",
                            message: "即将为您刷新数据"
                        });
                        that.getReferee();
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

<style scoped>
    .addReferee-button{
        display: block;
        width: 100%;
        margin-bottom: 20px;
        margin-top: 20px;

    }
</style>