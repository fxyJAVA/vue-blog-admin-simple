<template>
	<transition enter-active-class="animated bounceInUp">
		<section class="comments">
			<el-title-header :title="$route.meta.title"></el-title-header>
			<el-confirm-cancel :show.sync="show" @confirm="confirm">确定要删除吗</el-confirm-cancel>

			<div class="square">
				<div>
					<i class="blue"></i>
					<span>今天</span>
				</div>
				<div>
					<i class="warn"></i>
					<span>举报</span>
				</div>
			</div>

			<!-- 评论表格 begin -->
			<div class="table">
				<div class="common head">
					<span>日期</span>
					<span>用户名</span>
					<span>内容</span>
					<span>IP</span>
                    <span>评论文章</span>
					<span>操作</span>
				</div>

				<div class="common list" v-for="(item, index) in comments" :key="index">
					<span>{{item.commentDate | formatDate}}</span>
					<span>{{item.commentName}}<span v-if="item.isAdmin==1" class="admin">博主</span></span>
					<span v-html="item.commentContent"></span>
					<span>{{item.commentIp}}</span>
                    <span>{{item.articleName}}</span>
					<span class="operation">
						<i class="open" target="_blank" @click="reply(index)">回复</i>
						<i class="del" @click="del2(index)">删除</i>
                        <i v-if="item.secret==0" class="secret" @click="secret(index)">设为私密</i>
                        <el-tag style="width: 45px;height: 37px;line-height: 18px;margin-left: 10px;" v-if="item.secret==1" type="success">私密</el-tag>
					</span>
				</div>
			</div>
			<!-- 评论表格 end -->

            <!--分页 begin-->
            <div class="block">
                <el-pagination
                    align="right"
                    @current-change="change"
                    :page-size="pageSize"
                    layout="prev, pager, next"
                    :total="total">
                </el-pagination>
            </div>
            <!--分页 end-->

            <!-- 回复评论表格 begin-->
            <div v-if="toReply" >
                <div style="margin-top: 40px;padding: 0 5%;">
                    <h2>回复评论</h2>
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
            <!-- 回复评论表格 end-->
		</section>
	</transition>
</template>

<script>
export default {
	name: 'comments',
	data () {
		return {
			show: false,
			cache: [],		// 已经缓存的数据。
			comments: [],		// 用户评论
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
            total: 0,
            pageSize: 15,
            replyIndex: -1,
            toReply: false,
            focusState: false,
            delIndex: '',
            pageNum: 0
		}
	},
    beforeCreate() {
	    this.$axios.get('/comment?pageNum=1').then(response => {
            this.total = response.total
            this.comments = response.data
	        console.log(this.comments)
        }).catch(error => {

            })
    },
	methods: {
		// 确定删除
		confirm () {
            this.$axios.delete('/comment?commentid='+this.comments[this.delIndex].commentid).then(response => {
                if(response.code == 200) {
                    this.comments.splice(this.delIndex,1)
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
        secret (index) {
            this.$axios.get('/comment/secret?commentid='+this.comments[index].commentid).then(response=>{
                if(response.code == 200) {
                    this.query()
                }
            }).catch(error => {
                console.log(error)
            })
        },
        del2 (index) {
            this.delIndex = index
            this.show = true
        },
        //分页查询
        change (index) {
            this.pageNum = index
            this.$axios.get('/comment?pageNum='+index).then(response => {
                this.comments = response.data
            })
        },
		// 点击删除按钮, 传一个id过来
		del (id) {
			this.show = true;
		},
        reply (index) {
            this.toReply = true
            this.focusState = true
            this.replyIndex = index
        },
        query () {
            this.$axios.get('/comment?pageNum='+this.pageNum).then(response => {
                this.comments = response.data
            }).catch(error => {
                console.log(error)
            })
        },
        submit () {
            this.replyData.pageNum = this.pageNum,
            this.replyData.name = this.comments[this.replyIndex].commentName
            this.replyData.parent = this.comments[this.replyIndex].commentParent
            this.replyData.email = this.comments[this.replyIndex].commentEmail
            this.replyData.articleid = this.comments[this.replyIndex].articleid
            this.$axios.post('/comment',this.replyData).then(response => {
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












