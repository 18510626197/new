@@ -0,0 +1,280 @@
<template>
    <div class="container">
        <div class="carousel">
            <el-carousel height="455px">
                <el-carousel-item>
                    <img src="https://event.ygobbs.com/Uploads/competition/2019-02-03/5c56be2adc5d3.png">
                </el-carousel-item>
                <el-carousel-item>
                    <img src="https://event.ygobbs.com/Uploads/competition/2019-02-03/5c56bedf648b0.png">
                </el-carousel-item>
            </el-carousel>
        </div>
        <div class="user-data">
            <div class="fast-use">
                <el-button type="success" icon="el-icon-edit-outline" class="fast-button" @click="goNewThread">发布赛事</el-button>
                <el-button type="warning" icon="el-icon-coordinate" class="fast-button" @click="goProfiles">报名状态</el-button>
            </div>
            <!-- 推荐赛事 -->
            <div class="recommend">
                <h1><i class="el-icon-s-flag"></i> 推荐赛事</h1>
                <el-row :gutter="10">
                    <el-col :span="12" v-for="(item,index) in eventRecommend" v-bind:key="index">
                        <div @click="goThread(item.id)">
                            <el-card :body-style="{ padding: '0px' }" class="recommend-card">
                                <el-image
                                        style="width: 100px; height: 100px"
                                        :src="GLOBAL_API.apiHost + item.banner"
                                        fit="cover" class="image">
                                    <div slot="placeholder" class="image-slot">
                                        加载中<span class="dot">...</span>
                                    </div>
                                </el-image>
                                <div class="recommend-card-body">
                                    <span class="recommend-card-title">{{ item.title }}</span>
                                    <div class="bottom clearfix">
                                        <span class="desc">{{ item.describe }}</span><br>
                                        <span class="time"><i class="el-icon-aim"></i> 比赛时间 {{ item.event_time }}</span>
                                    </div>
                                </div>
                            </el-card>
                        </div>

                    </el-col>
                </el-row>
            </div>
            <!-- 用户赛事 -->
            <div class="event">
                <h1><i class="el-icon-s-opportunity"></i> 新鲜赛事</h1>
                <el-row :gutter="20" v-infinite-scroll="loadMore" v-loading="loading">
                    <el-col :span="8" v-for="(item,index) in eventData" v-bind:key="index" class="event-col" >
                        <div @click="goThread(item.id)">
                            <el-card :body-style="{ padding: '0px' }" class="event-card">
                                <el-image
                                        :src="GLOBAL_API.apiHost + item.banner"
                                        fit="cover" class="event-image">
                                    <div slot="placeholder" class="image-slot">
                                        加载中<span class="dot">...</span>
                                    </div>
                                </el-image>
                                <div class="event-card-body">
                                    <span class="event-card-title">{{ item.title }}</span>
                                    <div class="bottom clearfix">
                                        <span class="desc">{{ item.describe }}</span><br>
                                        <span class="time"><i class="el-icon-aim"></i> 比赛时间 {{ item.event_time }}</span>
                                    </div>
                                </div>
                            </el-card>
                        </div>

                    </el-col>
                </el-row>
            </div>
        </div>

    </div>
</template>

<script>
    let page = 1;
    export default {
        name: "Home",
        data(){
            return {
                eventRecommend:[],
                eventData:[],
                page: 1,
                loading: false,
            }
        },
        beforeDestroy(){
            page = 1;
        },
        destroyed(){

        },
        created(){
            this.getRecommend();
            //this.getEvent();
        },
        methods:{
            getRecommend: function(){
                const that = this;
                this.axios.get(this.GLOBAL_API.apiUrl + 'base/thread/getRecommend').then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 0){
                        that.eventRecommend = res.data.data;
                    }else{
                        that.$notify({
                            type: "error",
                            title: "获取失败",
                            message: "服务器出现异常，请联系管理员处理"
                        })
                    }
                }).catch(function(err){
                    that.$notify({
                        type: "error",
                        title: "获取失败",
                        message: "服务器出现异常，请联系管理员处理" + err.toString()
                    })
                })
            },
            getEvent: function(){
                const that = this;
                this.axios.post(this.GLOBAL_API.apiUrl + 'base/thread/getList',{
                    page: page
                }).then(function(res){
                    window.console.log(res.data);
                    if(res.data.errcode === 1){
                        that.$message({
                            type: "warning",
                            message: "·。· 已经达到最底端 QAQ"
                        });
                        that.loading = false;
                    }else if(res.data.errcode === 0){
                        that.eventData = that.eventData.concat(res.data.data);
                        page += 1;
                        that.loading = false;
                    }else{
                        that.$notify({
                            type: "error",
                            title: "获取失败",
                            message: "服务器出现异常，请联系管理员处理"
                        })
                    }
                }).catch(function(err){
                    that.$notify({
                        type: "error",
                        title: "获取失败",
                        message: "服务器出现异常，请联系管理员处理" + err.toString()
                    })
                })
            },
            goThread: function(tid){
                this.$router.push('/Thread/' + tid);

            },
            loadMore: function () {
                this.loading = true;
                this.getEvent();
            },
            goNewThread: function(){
                this.$router.push("/NewThread/");
            },
            goProfiles: function () {
                this.$router.push("/Profiles/");
            }
        }
    }
</script>

<style scoped>
    .user-data{
        width: 80%;
        margin: 30px auto 0;
    }
    .fast-use{
        text-align: center;
    }

    .fast-button{
        font-size:16px;
        font-weight: bold;
    }
    .desc{
        position: relative;
        top:-5px;
        color: #999999;
    }
    .recommend{
        color: #FFFFFF;
    }
    .recommend-card{
        border: 1px solid transparent;
        background: #000000;
    }
    .recommend-card:hover{
        border: 1px solid #409EFF;
    }
    .recommend-card-body{
        padding: 14px;
        text-align: center;
        margin-bottom: 8px;
    }
    .recommend-card-title{
        margin-top: 10px;
        font-size: 24px;
        font-weight: bold;
        color: #FFF;
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
        height: 100px;
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
        height: 100%;
        margin-bottom: 20px;
    }
    .image-slot{
        text-align: center;
        line-height: 300px;
        color: #FFFFFF;
    }

</style>