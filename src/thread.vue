@@ -0,0 +1,643 @@
<template>
    <div class="container-thread" id="thread" v-if="isLoaded">
        <el-page-header @back="goBack" content="赛事详情">
        </el-page-header>
        <div class="base">
            <div class="base-img">
                <el-image
                        :src="GLOBAL_API.apiHost + eventData.event.banner"
                        fit="cover" class="base-background"></el-image>
            </div>

            <div class="base-card">
                <div class="base-card-title">
                    <h1>{{ eventData.event.title }}</h1>
                </div>
                <div class="base-card-location">
                    <p><i class="el-icon-location-information"></i>赛事地点 | {{ eventData.event.site}}</p>
                </div>
                <div class="base-card-regnum">
                    <p><i class="el-icon-user"></i> {{eventData.event.register_num}}/<span v-if="eventData.event.register_limit === 0">∞</span><span v-else>{{eventData.event.register_limit}}</span></p>
                </div>

                <div class="base-card-time">
                    <el-row :gutter="10">
                        <el-col :span="12">
                            <span class="el-icon-remove-outline">报名截止</span><br>
                            <span>{{eventData.event.register_deadline}}</span>
                        </el-col>
                        <el-col :span="12">
                            <span class="el-icon-video-play">比赛时间</span><br>
                            <span>{{eventData.event.event_time}}</span>
                        </el-col>
                    </el-row>
                </div>
            </div>

            <div class="options">
                <el-button type="success" @click="regEvent" class="reg-button" icon="el-icon-s-check" v-if="allowReg">{{regButton}}</el-button>
            </div>

            <div class="other">
                <el-row :gutter="10">
                    <el-col :span="12" class="other-font">
                        <p class="el-icon-brush">赛事类型</p>
                        <p>{{eventData.event.game_mode}}</p>
                    </el-col>
                    <el-col :span="12" class="other-font">
                        <p class="el-icon-video-play">赛事直播</p>
                        <p v-if="eventData.event.event_live === 0">有</p>
                        <p v-if="eventData.event.event_live === 1">没有</p>
                    </el-col>
                </el-row>
            </div>
            <!-- TODO 切换Tab，增加一下直播、赛事比分的模块 -->
            <div class="describe">
                <el-divider class="dividers"></el-divider>
                <h1 class="describe-title"><i class="el-icon-data-board"></i> 赛事介绍</h1>
                <div class="base-card-user">
                    <p>{{eventData.event.author.username}}</p>
                </div>
                <div id="competition-body" style="width:100%;overflow:hidden;" v-html="eventData.content">

                </div>
            </div>
        </div>

        <div class="tab">
            <el-menu class="vertical-tab" :collapse="true">
                <div class="vertab-tips">
                    <p><i class="el-icon-info"></i> 状态</p>
                    <p>{{tipsAccept}}</p>
                    <p>{{tipsPublic}}</p>
                </div>
                <el-menu-item index="1" @click="changeOpenOrClose">
                    <i :class="iconAccept"></i>
                    <span slot="title">{{fontAccept}}</span>
                </el-menu-item>
                <el-menu-item index="2" @click="changePrivateOrPublic">
                    <i :class="iconPublic"></i>
                    <span slot="title">{{fontPublic}}</span>
                </el-menu-item>
                <el-menu-item index="3" @click="goToManage(tid)">
                    <i class="el-icon-star-off"></i>
                    <span slot="title">管理赛事</span>
                </el-menu-item>
            </el-menu>
        </div>
        <el-dialog title="报名信息填写" :visible.sync="regFormVisible" class="dialog-reg">
            <el-form :model="postData" :rules="rules" ref="postData">

                <el-select v-model="postData.profiles" placeholder="请选择" style="width: 100%;">
                    <el-option
                            v-for="(item,index) in profilesData"
                            :key="item.pr_id"
                            :label="'参赛ID: ' + item.nickname + ' 微信: ' + item.wechat + ' 手机: ' + item.phone + ' QQ: '+ item.qq"
                            :value="item.pr_id" v-if="index === 0" style="background: #000000">
                        <span style="float: left;width: 140px;color: #FFFFFF">参赛ID: <span style="color: #606266">{{ item.nickname }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">微信: <span style="color: #606266">{{ item.wechat }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">手机: <span style="color: #606266">{{ item.phone }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">QQ: <span style="color: #606266">{{ item.qq }}</span></span>
                    </el-option>
                    <el-option
                            v-for="(item,index) in profilesData"
                            :key="item.pr_id"
                            :label="'参赛ID: ' + item.nickname + ' 微信: ' + item.wechat + ' 手机: ' + item.phone + ' QQ: '+ item.qq"
                            :value="item.pr_id" v-if="index !== 0" style="background: #000000">
                        <span style="float: left;width: 140px;color: #FFFFFF">参赛ID: <span style="color: #606266">{{ item.nickname }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">微信: <span style="color: #606266">{{ item.wechat }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">手机: <span style="color: #606266">{{ item.phone }}</span></span>
                        <span style="float: left;width: 140px;color: #FFFFFF">QQ: <span style="color: #606266">{{ item.qq }}</span></span>
                    </el-option>
                </el-select>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="regFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="registerEvent">确认报名</el-button>
            </div>
        </el-dialog>
    </div>
</template>

<script>
    /* eslint-disable */
    import marked from 'marked'
    export default {
        name: "Thread",
        created(){
            let tid = this.$route.params.tid;
            this.getThread(tid);
            this.getProfiles();
            this.isexistReg();
        },
        data(){
            return {
                header: {
                    Authorization: this.$cookies.get('token')
                },
                iconAccept: 'el-icon-circle-close',
                fontAccept: '关闭比赛',
                tipsAccept: '赛事开启',
                iconPublic: 'el-icon-lock',
                fontPublic: '设置私密',
                tipsPublic: '公开赛事',
                regButton: '报名该比赛',
                regStatus: false,
                tid: undefined,
                isLoaded: false,
                regFormVisible: false,
                allowReg: false,
                eventData:{
                    event: null,
                    content: null
                },
                profilesData: [],
                postData:{
                    ydk: '',
                    profiles: ''
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
                }
            }
        },
        methods:{
            goBack: function(){
                this.$router.push("/");
            },
            regEvent: function () {
                this.regFormVisible = true
            },
            getThread: function(tid){
                window.console.log(tid);
                this.tid = tid;
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getEvent',{
                    tid: tid
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.data.base.status === 1){
                        that.eventData.event = res.data.data.base;
                        that.eventData.content = marked(res.data.data.content , { sanitize: true });
                        that.isAllowReg();
                        that.eventIDToName();
                        that.changeFontAndIcon();
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "不存在该比赛",
                            message: "您访问的这个比赛不存在"
                        });
                        that.$router.push("/");
                    }



                });
                this.isLoaded = true;
            },
            isAllowReg: function(){
                const that = this;
                let event_accept = this.eventData.event.accept;
                let event_status = this.eventData.event.reg_status;

                window.console.log(event_status);

                if(event_accept === 1 && event_status === 1){
                    that.allowReg = true;
                }


            },
            eventIDToName: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/modeIDToName',{
                    modeid: this.eventData.event.game_mode
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.eventData.event.game_mode = res.data.data.name;
                    }else{
                        that.$notify({
                            type: "warning",
                            title: "网络错误",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        });
                    }



                });
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
                        that.postData.profiles = res.data.data.profiles[0].pr_id;
                    }



                });
            },
            registerEvent: function(){
                const that = this;
                window.console.log(this.regStatus);
                if(this.allowReg === true ){
                    this.regFormVisible = false;
                    this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/registerEvent',{
                        tid: this.tid,
                        profiles: this.postData.profiles
                    },{
                        headers:{
                            Authorization: that.$cookies.get("token")
                        }
                    }).then(function(res){
                        window.console.log(res.data);
                        if(res.data.errcode === 0){
                            that.$message({
                                type: "success",
                                message: "恭喜您，已经成功的报名该赛事"
                            });
                        }else{
                            that.$notify({
                                type: "warning",
                                title: "网络错误",
                                message: "您的网络或者服务器出现异常，请稍后再试"
                            });
                        }
                    });
                }else{
                    that.$message({
                        type: 'warning',
                        message: '您不能报名该比赛！'
                    })
                }

            },
            isexistReg: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getRegisterStatus',{
                    tid: this.tid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    if(res.data.errcode === 0){
                        that.regStatus = false;
                    }else if (res.data.errcode === 1){
                        that.regStatus = true;
                        that.regButton = "重新提交报名信息";
                    }
                });
            },
            setPublic: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/setPublicEvent',{
                    tid: this.tid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    if(res.data.errcode === 0){
                        that.$message({
                            'type': "success",
                            'message': "设置比赛状态成功"
                        });
                        that.iconPublic = 'el-icon-lock';
                        that.fontPublic = '设置私密';
                        that.tipsPublic = '公开赛事';
                        that.eventData.event.public = 1;
                    }else{
                        that.$message({
                            'type': "warning",
                            'message': "设置比赛状态失败"
                        })
                    }
                });
            },
            setPrivate: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/setPrivateEvent',{
                    tid: this.tid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    if(res.data.errcode === 0){
                        that.$message({
                            'type': "success",
                            'message': "设置比赛状态成功"
                        });
                        that.iconPublic = 'el-icon-unlock';
                        that.fontPublic = '设置公开';
                        that.tipsPublic = '私密赛事';
                        that.eventData.event.public = 0;
                    }else{
                        that.$message({
                            'type': "warning",
                            'message': "设置比赛状态失败"
                        })
                    }
                });
            },
            setOpen: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/setOpenEvent',{
                    tid: this.tid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    if(res.data.errcode === 0){
                        that.$message({
                            'type': "success",
                            'message': "设置比赛状态成功"
                        });
                        that.iconAccept = 'el-icon-circle-close';
                        that.fontAccept = '关闭比赛';
                        that.tipsAccept = '赛事开放';
                        that.eventData.event.accept = 1;
                    }else{
                        that.$message({
                            'type': "warning",
                            'message': "设置比赛状态失败"
                        })
                    }
                });
            },
            setClose: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/setCloseEvent',{
                    tid: this.tid
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    if(res.data.errcode === 0){
                        that.$message({
                            'type': "success",
                            'message': "设置比赛状态成功"
                        });
                        that.iconAccept = 'el-icon-circle-check';
                        that.fontAccept = '开放比赛';
                        that.tipsAccept = '赛事关闭';
                        that.eventData.event.accept = 0;
                    }else{
                        that.$message({
                            'type': "warning",
                            'message': "设置比赛状态失败"
                        })
                    }
                });
            },
            changeOpenOrClose: function(){
                const status = this.eventData.event.accept;

                const that = this;
                if(status === 1){
                    that.setClose();
                }else{
                    that.setOpen();
                }
            },
            changePrivateOrPublic: function(){
                const status = this.eventData.event.public;

                const that = this;
                if(status === 1){
                    that.setPrivate();
                }else{
                    that.setPublic();
                }
            },
            changeFontAndIcon: function(){
                const isAccept = this.eventData.event.accept;
                const isPublic = this.eventData.event.public;

                const that = this;
                if (isAccept === 1){
                    that.iconAccept = 'el-icon-circle-close';
                    that.fontAccept = '关闭比赛';
                    that.tipsAccept = '赛事开放';
                }else{
                    that.iconAccept = 'el-icon-circle-check';
                    that.fontAccept = '开放比赛';
                    that.tipsAccept = '赛事关闭';
                }

                if (isPublic === 1){
                    that.iconPublic = 'el-icon-lock';
                    that.fontPublic = '设置私密';
                    that.tipsPublic = '公开赛事';
                }else{
                    that.iconPublic = 'el-icon-unlock';
                    that.fontPublic = '设置公开';
                    that.tipsPublic = '私密赛事';
                }
            },
            goToManage: function(tid){
                this.$router.push("/Manage/" + tid);
            }


        }
    }
</script>

<style>
    .container-thread{
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
    .base{
        margin-top: 10px;
        border-radius: 5px !important;
        width: 100%;
        height:400px;
        position: relative;
    }
    .base-img{
        filter: blur(4px);
        background-color: #000000;
    }
    .base-background{
        width: 100%;
        height: 400px;
        border-radius: 5px !important;
        filter: alpha(opacity=20);
        -moz-opacity:0.2;
        opacity:0.2;

    }
    .base-card{
        position: absolute;
        top:0;
        width: 100%;

    }
    .base-card-title{
        text-align: center;
        font-size: 28px;
        color: #FFFFFF;
        position: relative;
        top:-20px;
    }
    .base-card-location{
        text-align: center;
        font-weight: bold;
        color: #FFFFFF;
        position: relative;
        top:-50px;
    }
    .base-card-location > p > i {
        font-weight: bold;

    }
    .base-card-regnum{
        text-align: center;
        font-weight: bold;
        font-size: 70px;
        color: #FFFFFF;
        position: relative;
        top:-80px;
    }
    .base-card-user{
        text-align: center;
        font-weight: bold;
        font-size: 20px;
        color: #FFFFFF;
        position: relative;
        top:-20px;
    }
    .base-card-time{
        text-align: center;
        font-weight: bold;
        font-size: 30px;
        color: #FFFFFF;
        position: relative;
        top:-90px;
    }
    .options{
        position: relative;
        top:20px;
    }
    .admin-options{
        position: relative;
        top:25px;
        margin-bottom: 20px;
    }
    .reg-button{
        display: block;
        width: 100%;
        font-size: 25px;
    }
    .other{
        position: relative;
        top:40px;
    }
    .other-font{
        font-size: 25px;
        color: #FFFFFF;
        text-align: center;
    }
    .other-font > p{
        margin: 5px;
    }
    .other-admin-font{
        text-align: center;
    }
    .other-admin-font > .el-button{
        font-size: 18px;
        color: #FFFFFF;
        text-align: center;
    }
    .other-admin-font > p{
        margin: 5px;
    }
    .describe{
        position: relative;
        top:40px;
        color: #FFFFFF;
    }
    .describe-title{
        text-align: center;
        font-size: 40px;
    }
    .describe-title > i {
        font-weight: bold;
    }
    .tab{
        position: fixed;
        top:190px;
        right:0;
    }
    .vertical-tab{
        background: #303133;
        border-right: 0 !important;
    }
    .dialog-reg > div{
        background: #121417;
        border: 1px solid #999999;
        border-radius: 5px;
        color: #FFFFFF;
    }
    .dialog-reg .el-dialog__title{
        color: #FFFFFF;
    }
    .dialog-reg .el-form-item__label{
        color: #FFFFFF;
    }
    .dialog-reg .el-input input{
        background: #303133;
        color: #FFFFFF;
        border: 1px solid #121417;
    }
    .upload-reg{
        margin: 0 auto;
        text-align: center;
    }
    .upload-reg .el-upload-dragger{
        background-color: #303133 !important;
    }
    .el-upload__tip{
        color: cornflowerblue;
    }
    .vertab-tips{
        color: #FFFFFF;
        font-size: 10px;
        text-align: center;
    }
</style>