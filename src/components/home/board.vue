<!--分页-->
<template>
    <transition enter-active-class="animated bounceInUp">
        <section class="comments">
            <el-title-header :title="$route.meta.title"></el-title-header>
            <el-confirm-cancel :show.sync="show" @confirm="confirm">确定要删除该留言么</el-confirm-cancel>

            <!--<div class="square">-->
                <!--<div>-->
                    <!--<i class="blue"></i>-->
                    <!--<span>今天</span>-->
                <!--</div>-->
                <!--<div>-->
                    <!--<i class="warn"></i>-->
                    <!--<span>举报</span>-->
                <!--</div>-->
            <!--</div>-->

            <!-- 评论表格 begin -->
            <div class="table">
                <div class="common head">
                    <!--<span>ID</span>-->
                    <span>日期</span>
                    <span>用户名</span>
                    <span>内容</span>
                    <!--<span>IP</span>-->
                    <span>操作</span>
                </div>

                <div class="common list" v-for="(item, index) in boardArr" :key="index">
                    <!--<span>{{item.messageid}}</span>-->
                    <span>{{item.messageDate | formatDate}}</span>
                    <span>{{item.messageName}}<span v-if="item.isAdmin==1" class="admin">博主</span></span>
                    <span v-html="item.messageContent"></span>
                    <!--<span>{{item.messageIp}}</span>-->
                    <span class="operation">
						<i class="open" @click="replyTrue(index)">回复</i>
						<i class="del" @click="del(index)">删除</i>
                        <i v-if="item.secret==1" class="secret" @click="secret(index)">设为私密</i>
                        <el-tag style="width: 45px;height: 37px;line-height: 18px;margin-left: 10px;" v-if="item.secret==0" type="success">私密</el-tag>
                    </span>
                </div>
            </div>

            <div class="block">
                <el-pagination
                    align="right"
                    @current-change="change"
                    :page-size="pageSize"
                    layout="prev, pager, next"
                    :total="total">
                </el-pagination>
            </div>
            <!-- 评论表格 end -->

            <!--发表留言 begin-->
            <div style="margin-top: 40px;padding: 0 5%;">
                <h2>发表留言</h2>
                <br>
                <el-input
                    style="margin-bottom: 10px;"
                    type="textarea"
                    :autosize="{ minRows: 3, maxRows: 4}"
                    placeholder="请输入内容"
                    :autofocus="focusState"
                    v-model="replyData.content">
                </el-input>
                <el-button type="primary" style="float: right;margin-right: 20px;" round @click="submit2">提交</el-button>
            </div>
            <!--发表留言 end-->
            <!-- 回复留言表格 begin-->
            <div v-if="toReply" >
                <div style="margin-top: 40px;padding: 0 5%;">
                    <h2>回复留言</h2>
                    <br>
                    <el-input
                        id="longText"
                        style="margin-bottom: 10px;"
                        type="textarea"
                        :autosize="{ minRows: 3, maxRows: 4}"
                        placeholder="请输入内容"
                        :autofocus="focusState"
                        v-model="replyData.reply">
                    </el-input>
                    <el-button type="warning" style="float: right;" @click="toReply=false" round>取消</el-button>
                    <el-button type="primary" style="float: right;margin-right: 20px;" round @click="submit">提交</el-button>
                </div>
            </div>
            <!-- 回复留言表格 end-->

        </section>
    </transition>
</template>

<script>
    export default {
        name: 'board',
        data () {
            return {
                show: false,
                cache: [],		// 已经缓存的数据。
                comments: [		// 用户评论
                    {

                    }
                ],
                total: 0,
                pageNum: 1,
                pageSize: 15,
                boardArr: [],
                toReply: false,
                replyIndex: '',
                replyData: {
                    name: '',
                    id: '',
                    reply: '',
                    url: '',
                    email: '',
                    parent: '0l',
                    pageNum: '',
                    articleid: ''
                 },
                focusState: true,
                delIndex: ''
            }
        },
        beforeCreate () {
            this.$axios.get('/board?pageNum=1').then(response => {
                this.boardArr = response.data
                this.total = response.total
            }).catch(error => {
                console.log(error)
            })
        },
        methods: {
            change (index) {
                this.pageNum = index
                this.$axios.get('/board?pageNum='+index).then(response => {
                    this.boardArr = response.data
                })
            },
            query () {
                this.$axios.get('/board?pageNum='+this.pageNum).then(response => {
                    this.boardArr = response.data
                    console.log(response.data)
                }).catch(error => {
                    console.log(error)
                })
            },
            // 确定删除
            confirm () {
                this.$axios.delete('/board?messageid='+this.boardArr[this.delIndex].messageid).then(response => {
                    if(response.code == 200) {
                        this.boardArr.splice(this.delIndex,1)
                        this.delIndex = ''
                    }
                }).catch(error => {
                        this.$message({
                            message: '删除失败,'+error,
                            type: 'error',
                            duration: 1333
                        })
                })
            },
            // 点击删除按钮, 传一个id过来
            del (index) {
                this.delIndex = index
                this.show = true;
            },
            secret (index) {
                this.$axios.get('/board/secret?messageid='+this.boardArr[index].messageid).then(response=>{
                    if(response.code == 200) {
                        this.query()
                    }
                })
            },
            reply () {
                this.$axios.post('board',this.replyData).then(response => {
                    console.log()
                }).catch(errror => {

                })
            },
            replyTrue (index) {
                this.replyIndex = index
                this.toReply = true
                this.focusState  = true
            },
            submit () {
                this.replyData.pageNum = this.pageNum,
                this.replyData.name = this.boardArr[this.replyIndex].messageName
                this.replyData.parent = this.boardArr[this.replyIndex].messageParent
                this.replyData.email = this.boardArr[this.replyIndex].messageEmail
                this.$axios.post('/board',this.replyData).then(response => {
                if(response.code = 200) {
                    this.clear()
                    this.toReply = false
                    this.replyIndex = ''
                    this.replyData.reply = ''
                    this.query()
                }
                }).catch(error => {
                    console.log(error)
                })
            },
            submit2 () {
                this.replyData.parent = 0
                this.$axios.post('/board/isay',this.replyData).then(response => {
                    if (response.code = 200) {
                        this.clear()
                        this.replyIndex = ''
                        this.query()
                    }
                })
            },
             clear () {
                this.replyData.email = ''
                this.replyData.parent = ''
                this.replyData.content = ''
                this.replyData.id = ''
                this.replyData.name = ''
                this.replyData.url = ''
                this.replyData.pageNum = ''
            }
        },
        filters: {
            formatDate: function (time) {
                let date = new Date(time);
                let y = date.getFullYear();
                let MM = date.getMonth() + 1;
                MM = MM < 10 ? ('0' + MM) : MM;
                let d = date.getDate();
                d = d < 10 ? ('0' + d) : d;
                let h = date.getHours();
                h = h < 10 ? ('0' + h) : h;
                let m = date.getMinutes();
                m = m < 10 ? ('0' + m) : m;
                let s = date.getSeconds();
                s = s < 10 ? ('0' + s) : s;
                return y + '-' + MM + '-' + d + ' ' + h + ':' + m + ':' + s;
            }
        }
    }
</script>












