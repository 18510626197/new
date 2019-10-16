@@ -0,0 +1,430 @@
<template>
    <div class="container-profiles">
        <el-page-header @back="goBack" content="个人主页">
        </el-page-header>
        <el-tabs v-model="activeName" @tab-click="handleClick">
            <el-tab-pane label="我报名的" name="first" v-infinite-scroll="getUserRegList" style="overflow:auto">
                <div class="event">
                    <h1>我报名的</h1>
                    <el-row :gutter="20">
                        <el-col :span="8" v-for="item in userRegList" v-bind:key="'reg-'+item.id" class="event-col">
                            <div @click="goThread(item.id)">
                                <el-card :body-style="{ padding: '0px' }" class="event-card">
                                    <el-image
                                            style="width: 100px; height: 100px"
                                            :src="GLOBAL_API.apiHost + item.banner"
                                            fit="cover" class="event-image"></el-image>
                                    <div class="event-card-body">
                                        <span class="event-card-title">{{ item.title}}</span>
                                        <div class="bottom clearfix">
                                            <span class="desc">{{ item.desc }}</span><br>
                                            <span class="time"><i class="el-icon-aim"></i> 比赛时间 {{ item.event_time }}</span>
                                        </div>
                                    </div>
                                </el-card>
                            </div>

                        </el-col>
                    </el-row>
                </div>
            </el-tab-pane>
            <el-tab-pane label="我发布的" name="second">
                <div class="event">
                    <h1>我发布的</h1>
                    <el-row :gutter="20">
                        <el-col :span="8" v-for="item in userSendList" v-bind:key="'send-'+item.id" class="event-col">
                            <div @click="goThread(item.id)">
                                <el-card :body-style="{ padding: '0px' }" class="event-card">

                                    <el-image
                                            style="width: 100px; height: 100px"
                                            :src="GLOBAL_API.apiHost + item.banner"
                                            fit="cover" class="event-image"></el-image>
                                    <div class="event-card-body">
                                        <span class="event-card-title">{{ item.title}}</span>
                                        <div class="bottom clearfix">
                                            <span class="desc">{{ item.desc }}</span><br>
                                            <span class="time"><i class="el-icon-aim"></i> 比赛时间 {{ item.event_time }}</span>
                                        </div>
                                    </div>
                                </el-card>
                            </div>

                        </el-col>
                    </el-row>
                </div>
            </el-tab-pane>
            <el-tab-pane label="参赛信息设置" name="third">
                <div class="event">
                    <h1>参赛信息设置</h1>
                    <el-button type="success" size="small" @click="eventInfoStatus = true">添加</el-button>
                    <br><br>
                    <el-table
                            :data="profilesData"
                            style="width: 100%">
                        <el-table-column
                                prop="pr_id"
                                label="ID"
                                width="180">
                        </el-table-column>
                        <el-table-column
                                prop="nickname"
                                label="参赛ID"
                                width="180">
                        </el-table-column>
                        <el-table-column
                                prop="qq"
                                label="QQ">
                        </el-table-column>
                        <el-table-column
                                prop="phone"
                                label="手机号">
                        </el-table-column>
                        <el-table-column
                                prop="wechat"
                                label="微信号">
                        </el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="scope">
                                <el-button
                                        size="mini"
                                        type="danger"
                                        @click="delProfiles(scope.row.pr_id)">删除</el-button>
                                <el-button
                                        size="mini"
                                        type="warning"
                                        @click="setProfilesDefaults(scope.row.pr_id)" v-if="profilesDefaults !== scope.row.pr_id">设为默认</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </div>
            </el-tab-pane>
        </el-tabs>
        <el-dialog title="添加报名信息" :visible.sync="eventInfoStatus" class="dialog-reg">
            <el-form :model="postData" :rules="rules" ref="postData">
                <el-form-item prop="qq">
                    <el-input v-model.number="postData.qq" autocomplete="off" placeholder="QQ账号"></el-input>
                </el-form-item>
                <el-form-item prop="wechat">
                    <el-input v-model="postData.wechat" autocomplete="off" placeholder="微信账号"></el-input>
                </el-form-item>
                <el-form-item prop="phone">
                    <el-input v-model.number="postData.phone" autocomplete="off" placeholder="电话号码"></el-input>
                </el-form-item>
                <el-form-item prop="nickname">
                    <el-input v-model="postData.nickname" autocomplete="off" placeholder="参赛ID"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="eventInfoStatus = false">取 消</el-button>
                <el-button type="primary" @click="addProfiles()">确认报名</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Profiles",
        created(){
            this.getProfiles();
            this.getUserRegList();
            this.getUserSendList();
        },
        data(){
            return{
                profilesDefaults: null,
                activeName: 'first',
                profilesData:[],
                eventInfoStatus: false,
                postData:{
                    qq: '',
                    wechat: '',
                    nickname: '',
                    phone: ''
                },
                rules:{
                    qq:[
                        { required: true, message: '请输入QQ账号', trigger: 'blur' },
                        { type: 'number', message: 'QQ账号必须为数字' },
                    ],
                    wechat:[
                        { required: true, message: '请输入微信账号', trigger: 'blur' },
                    ],
                    phone:[
                        { required: true, message: '请输入手机号码', trigger: 'blur' },
                        { type: 'number', message: '手机号码必须为数字' },
                    ],
                    nickname:[
                        { required: true, message: '请输入参赛ID', trigger: 'blur' },
                    ]
                },
                userRegList:[],
                userSendList:[],
                regPage:1,
                senPage:1,
            }
        },
        methods: {
            goBack: function () {
                this.$router.push('/');
            },
            handleClick: function(){

            },
            getProfiles: function () {
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/getProfilesEventInfo',{
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.profilesData = res.data.data.profiles;
                        that.profilesDefaults = res.data.data.defaults;
                        window.console.log(that.profilesData);
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            },
            delProfiles: function(pr_id){
                this.$confirm('此操作将会永久删除您的参赛信息, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).then(() => {
                    const that = this;
                    this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/delProfilesEventInfo',{
                        pr_id: pr_id
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
                                message: "删除成功，即将为您刷新"
                            });
                            that.getProfiles();
                        }else{
                            that.$notify({
                                type: "warning",
                                title: "网络错误",
                                message: "您的网络或者服务器出现异常，请稍后再试"
                            });
                        }



                    });

                }).catch(() => {
                    this.$message({
                        type: 'info',
                        message: '已取消删除'
                    });
                });
                //window.console.log(pr_id);

            },
            addProfiles: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/setProfilesEventInfo',{
                    qq: this.postData.qq,
                    wechat: this.postData.wechat,
                    nickname: this.postData.nickname,
                    phone: this.postData.phone
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.eventInfoStatus = false;
                        that.$notify({
                            type: "success",
                            title: "添加成功",
                            message: "添加成功，即将为您刷新"
                        });
                        that.getProfiles();
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            },
            setProfilesDefaults: function(pr_id){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/setProfilesEventInfoDefaults',{
                    pr_id: pr_id
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.$notify({
                            type: "success",
                            title: "设置成功",
                            message: "该信息设置为默认信息成功。"
                        });
                        that.getProfiles();
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
            },
            getUserRegList: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/getMyRegisterEventList',{
                        page: that.regPage
                    },
                    {
                        headers:{
                            Authorization: that.$cookies.get("token")
                        }
                    }).then(function(res){
                    window.console.log(res.data.data);
                    if (res.data.errcode === 0){
                        that.userRegList = that.userRegList.concat(res.data.data);
                        that.regPage = that.regPage+1;
                    }
                })
            },
            getUserSendList: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/profiles/getMySendEventList',{
                        page: that.sendPage
                    },
                    {
                        headers:{
                            Authorization: that.$cookies.get("token")
                        }
                    }).then(function(res){
                    window.console.log(res.data.data);
                    if (res.data.errcode === 0){
                        that.userSendList = that.userSendList.concat(res.data.data);
                        that.sendPage = that.sendPage+1;
                    }
                })
            },
            goThread: function(tid){
                this.$router.push('/Thread/' + tid);
            },
        }
    }
</script>

<style>
    .container-profiles{
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
    .el-tabs__item{
        color: #FFFFFF;
        font-size: 20px;
    }

    .event{
        color: #FFFFFF;
    }
    .event-card{
        border: 1px solid transparent;
        background: #000000;
    }
    .event-card:hover{
        border: 1px solid #409EFF;
    }
    .event-card-body{
        padding: 14px;
        text-align: center;
        margin-bottom: 8px;
        height: 120px;
    }
    .event-card-title{
        margin-top: 10px;
        font-size: 24px;
        font-weight: bold;
        color: #FFF;
    }
    .event-image {
        height:200px !important;
        width: 100% !important;
        display: block !important;
    }
    .event-col{
        margin-bottom: 20px;
    }
    .time {
        font-size: 16px;
        font-weight: bolder;
        color: #999;
        position: relative;
        top:8px;
    }
    .time > i{
        font-weight: bolder;
    }

    .bottom {
        margin-top: 13px;
        line-height: 12px;
    }

    .image {
        height:300px !important;
        width: 100% !important;
        display: block !important;
    }

    .clearfix:before,
    .clearfix:after {
        display: table;
        content: "";
    }

    .clearfix:after {
        clear: both
    }
    .desc{
        position: relative;
        top:-5px;
        color: #999999;
    }
</style>