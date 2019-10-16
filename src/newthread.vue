@@ -0,0 +1,507 @@
<template>
    <div class="container-newthread">
        <el-page-header @back="goBack" content="发布赛事">
        </el-page-header>
        <el-steps :active="active" finish-status="success" align-center class="step">
            <el-step title="上传封面图" icon="el-icon-picture"></el-step>
            <el-step title="填写信息" icon="el-icon-edit"></el-step>
            <el-step title="填写描述" icon="el-icon-document"></el-step>
            <el-step title="设置权限" icon="el-icon-lock"></el-step>
        </el-steps>
        <div class="newthread-card">
            <div class="upload-banner" v-if="active === 0">
                <el-upload
                        class="avatar-uploader"
                        :action="GLOBAL_API.apiUrl + 'base/uploads/imgUploader'"
                        :headers="header"
                        :show-file-list="false"
                        :on-success="handleAvatarSuccess"
                        :before-upload="beforeAvatarUpload">

                    <el-image
                            v-if="imageUrl" :src="imageUrl"
                            fit="cover" class="avatar"></el-image>
                    <i v-else class="el-icon-picture-outline-round avatar-uploader-icon">点击这里/拖拽到此处选择封面图</i>
                </el-upload>
            </div>
            <div class="write-base" v-if="active === 1">
                <el-form :model="eventData" :rules="rules" ref="eventData" class="eventForm">
                    <el-form-item prop="title">
                        <el-input v-model="eventData.title" placeholder="赛事名称"></el-input>
                    </el-form-item>
                    <el-form-item required :gutter="120">
                        <el-col :span="24">
                            <el-date-picker prop="playDate"
                                            v-model="eventData.playDate"
                                            type="datetime"
                                            placeholder="选择比赛开始日期时间"
                                            style="width: 100%;">
                            </el-date-picker>
                        </el-col>

                    </el-form-item>
                    <el-form-item required :gutter="120">
                        <el-col :span="24">
                            <el-date-picker prop="deadlineDate"
                                            v-model="eventData.deadlineDate"
                                            type="datetime"
                                            placeholder="选择报名截止日期时间"
                                            style="width: 100%;">
                            </el-date-picker>
                        </el-col>
                    </el-form-item>
                    <el-form-item prop="describe">
                        <el-input v-model="eventData.describe" placeholder="赛事简介"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-row :gutter="120">
                            <el-col :span="12">
                                <el-form-item prop="location">
                                    <el-input v-model="eventData.location" placeholder="比赛地点"></el-input>
                                </el-form-item>
                            </el-col>
                            <el-col :span="12">
                                <el-form-item prop="maxPlayer">
                                    <el-input v-model.number="eventData.maxPlayer" placeholder="参赛人数上限(不限制请填0)"></el-input>
                                </el-form-item>
                            </el-col>
                        </el-row>
                    </el-form-item>
                    <el-row>
                        <el-col :span="8">
                            <el-form-item prop="gameMode">
                                <el-select v-model="eventData.gameMode" placeholder="请选择比赛模式">
                                    <el-option v-for="item in eventMode" v-bind:key="item.id" :label="item.mode_name" :value="item.id"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-col>
                        <el-col :span="8">
                            <span style="color: #FFFFFF">报名需要 </span>
                            <el-checkbox-group v-model="eventData.qq" class="checkbox">
                                <el-checkbox label="qq" key="1">QQ</el-checkbox>
                            </el-checkbox-group>
                            <el-checkbox-group v-model="eventData.wechat" class="checkbox">
                                <el-checkbox label="wechat" key="1">微信</el-checkbox>
                            </el-checkbox-group>
                            <el-checkbox-group v-model="eventData.phone" class="checkbox">
                                <el-checkbox label="wechat" key="1">手机号</el-checkbox>
                            </el-checkbox-group>
                            <el-checkbox-group v-model="eventData.nickname" class="checkbox">
                                <el-checkbox label="wechat" key="wechat">参赛ID</el-checkbox>
                            </el-checkbox-group>
                        </el-col>
                        <el-col :span="8">
                            <el-form-item class="reset-button">
                                <el-button @click="resetForm('eventData')" type="danger">重置</el-button>
                            </el-form-item>
                        </el-col>
                    </el-row>
                </el-form>
            </div>
            <div class="content" v-if="active === 2">
                <mavon-editor v-model="eventData.content"/>
            </div>
            <div class="private" v-if="active === 3">

                <el-switch
                        class="private-switch"
                        v-model="eventData.public"
                        active-text="公开赛事"
                        inactive-text="私密赛事">
                </el-switch>
                <br>

                <el-switch
                        class="private-switch"
                        v-model="eventData.accept"
                        active-text="开放报名"
                        inactive-text="关闭报名">
                </el-switch>
                <br>
                <el-switch
                        class="private-switch"
                        v-model="eventData.evnet_live"
                        active-text="开启直播"
                        inactive-text="关闭直播">
                </el-switch>

            </div>
            <div class="content" v-if="active === 4">
                <h1>正在发布，请稍后</h1>
            </div>
        </div>
        <el-button v-if="showButton" type="success" class="newthread-button" @click="next">下一步</el-button>
    </div>
</template>

<script>
    export default {

        name: "NewThread",
        created(){
            this.header.Authorization = this.$cookies.get('token');
            this.getMode();
            window.console.log(this.header);
        },
        data(){
            return {
                header: {
                    Authorization: this.$cookies.get('token')
                },
                eventMode:[],
                showButton: true,
                imageUrl: '',
                active: 0,
                eventData:{
                    title: '',
                    gameMode: '',
                    playDate: '',
                    deadlineDate: '',
                    location: '',
                    evnet_live: true,
                    maxPlayer: null,
                    qq: false,
                    wechat: false,
                    phone: false,
                    nickname: false,
                    describe: '',
                    public: true,
                    accept: true,
                    banner: ''
                },
                rules:{
                    title:[
                        { required: true, message: '请输入赛事名称', trigger: 'blur' },
                        { min: 3, max: 22, message: '长度在 3 到 22 个字符', trigger: 'blur' }
                    ],
                    gameMode:[
                        { required: true, message: '请选择一个比赛模式', trigger: 'change' }
                    ],
                    playDate:[
                        { type: 'datetime', required: true, message: '请选择日期', trigger: 'change' }
                    ],
                    deadlineDate:[
                        { type: 'datetime', required: true, message: '请选择日期', trigger: 'change' }
                    ],
                    location:[
                        { required: true, message: '请输入比赛地点', trigger: 'blur' },
                    ],
                    maxPlayer:[
                        { required: true, message: '请输入最大参赛人数', trigger: 'blur' },
                        { type: 'number', required: true, message: '请输入数字'}
                    ],
                    content:[
                        { required: true, message: '请输入主要介绍内容', trigger: 'blur' },
                    ],
                    describe:[
                        { required: true, message: '请输入简介内容', trigger: 'blur' },
                    ],

                }
            }
        },
        methods:{
            postThread: function(){
                let publics , accept, qq, phone, wechat, nickname = 0;
                if(this.eventData.public === true){
                    publics = 1;
                }
                if(this.eventData.accept === true){
                    accept = 1;
                }

                if(this.eventData.qq === true){
                    qq = 1;
                }

                if(this.eventData.phone === true){
                    phone = 1;
                }

                if(this.eventData.wechat === true){
                    wechat = 1;
                }

                if(this.eventData.nickname === true){
                    nickname = 1;
                }
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + "base/thread/create",{
                    title: this.eventData.title,
                    site: this.eventData.location,
                    banner: this.eventData.banner,
                    describe: this.eventData.describe,
                    use_qq: qq,
                    use_wechat: wechat,
                    use_phone: phone,
                    use_nickname: nickname,
                    evnet_live: this.eventData.evnet_live,
                    game_mode: this.eventData.gameMode,
                    public: publics,
                    accepts: accept,
                    register_limit: this.eventData.maxPlayer,
                    content: this.eventData.content,
                    event_time: this.eventData.playDate,
                    register_deadline: this.eventData.deadlineDate
                },{
                    headers:{
                        Authorization: that.$cookies.get("token")
                    }
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.$notify({
                            type: "success",
                            title: "发布成功",
                            message: "发布赛事成功，即将为您跳转至该赛事。"
                        });
                        that.$router.push("/Thread/" + res.data.data.tid);
                    }
                })
            },
            goBack: function(){
                this.$router.push('/');
            },
            handleAvatarSuccess: function(res){
                window.console.log(this.GLOBAL_API.apiHost + "img/" + res.data.src);
                this.imageUrl = this.GLOBAL_API.apiHost + "img/" + res.data.src;
                this.eventData.banner = "img/" + res.data.src;
            },
            beforeAvatarUpload: function(file){
                const isJPG = file.type === 'image/jpeg';
                const isLt2M = file.size / 1024 / 1024 < 2;

                if (!isJPG) {
                    this.$message.error('上传头像图片只能是 JPG 格式!');
                }
                if (!isLt2M) {
                    this.$message.error('上传头像图片大小不能超过 2MB!');
                }
                return isJPG && isLt2M;
            },
            next: function(){
                if (this.active === 0){
                    if (this.imageUrl === ""){
                        this.$message({
                            message: "请上传图片后方可点击下一步",
                            type: "warning"
                        })
                    }else{
                        this.doNext();
                    }
                }else if (this.active === 1){
                    if (this.eventData.title !== "" && this.eventData.site !== "" && this.eventData.describe !== "" &&
                        this.eventData.gameMode !== "" && this.eventData.maxPlayer !== "" && this.eventData.deadlineDate != "" &&
                        this.eventData.playDate !== "" &&
                        (this.eventData.qq === true || this.eventData.wechat === true || this.eventData.phone === true || this.eventData.nickname === true)){
                        this.doNext();
                    }else{
                        this.$message({
                            message: "请填全信息后才能继续",
                            type: "warning"
                        })
                    }
                }else if (this.active === 2){
                    if (this.eventData.content !== ""){
                        this.doNext();
                    }else{
                        this.$message({
                            message: "请填全信息后才能继续",
                            type: "warning"
                        })
                    }
                }else if (this.active === 3){
                    this.doNext();
                }else{
                    this.doNext();
                }

            },
            doNext: function(){
                if (this.active++ >= 3) {
                    this.postThread();
                    this.showButton = false;
                }
            },
            resetForm: function(formName) {
                this.$refs[formName].resetFields();
            },
            getMode: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getEventMode',{
                }).then(function(res){
                    window.console.log(res.data);
                    if (res.data.errcode === 0){
                        that.eventMode = res.data.data;
                    }else{
                        that.$notify({
                            title: "网络异常",
                            type: "warning",
                            message: "您的网络或者服务器出现异常，请稍后再试"
                        })
                    }
                });
            }
        }
    }
</script>

<style>
    .container-newthread{
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
    .upload-banner{
        width: 100%;
        margin-top: 20px;
    }
    .write-base{
        width: 100%;
        margin-top: 20px;
    }
    .content{
        width: 100%;
        margin-top: 20px;
        border-radius: 5px;
        overflow: hidden;
    }
    .private{
        width: 100%;
        margin-top: 30px;
        text-align: center;
    }
    .avatar-uploader{
        width: 100% !important;
        height: 600px !important;
    }
    .avatar-uploader .el-upload {
        width: 100% !important;
        height: 600px !important;
        border: 1px dashed #FFFFFF;
        border-radius: 6px;
        cursor: pointer;
        position: relative;
        overflow: hidden;
    }
    .avatar-uploader .el-upload:hover {
        border-color: #409EFF;
    }
    .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 100%;
        height: 600px;
        line-height: 600px;
        text-align: center;
    }
    .avatar {
        width: 100%;
        height: 600px;
        display: block;
    }
    .step{
        margin-top: 20px;
    }
    .el-step__icon{
        background: #121417 !important;

    }
    .el-step__head{
        color: #FFFFFF;
    }
    .el-step__head.is-finish{
        color: #67C23A !important;
    }
    .el-step__title.is-finish{
        color: #67C23A !important;
    }
    .el-step__head.is-process{
        color: #999999;
    }
    .el-step__title.is-process{
        color: #999999;
    }
    .newthread-button{
        width: 100%;
        display: block;
        margin-top: 20px;
        margin-bottom: 20px;
    }
    .eventForm .el-form-item__label{
        color: #FFFFFF;
    }
    .eventForm .el-input input{
        background: #303133;
        color: #FFFFFF;
        border: 1px solid #121417;
    }
    .reset-button{
        text-align: right;
    }
    .el-picker-panel, .el-time-panel{
        background: #121417 !important;
    }
    .el-picker-panel__body{
        background-color: #121417 !important;
    }
    .el-time-spinner__item.active{
        color: #FFFFFF !important;
    }
    .el-time-panel__btn{
        color: #999999;
    }
    .el-select-dropdown{
        background-color: #121417;
    }
    .selected{
        background: #303133 !important;
    }
    .hover{
        background: #8c939d !important;
    }
    .markdown-body{
        border-radius: 5px !important;
        background: #303133 !important;
    }
    .v-note-op,.shadow{
        background:  #303133 !important;
        color: #999999 !important;
    }
    .auto-textarea-input{
        background: #303133 !important;
        color: #FFFFFF !important;
    }
    .v-note-show,.v-show-content{
        background: #303133 !important;
        color: #FFFFFF !important;
    }
    .private-switch{
        display: block;
        font-size: 20px !important;
        text-align: center;
    }
    .el-switch__label{
        color: #FFFFFF;
    }
    .el-switch__label > span{
        font-size: 20px;
    }
    .checkbox{
        display: inline;
        margin-right: 10px;
    }
    .el-input--small .el-input__inner{
        background: #303133 !important;
        color: #FFFFFF !important;
    }
    .el-picker-panel__footer{
        background-color: #303133 !important;
    }
</style>