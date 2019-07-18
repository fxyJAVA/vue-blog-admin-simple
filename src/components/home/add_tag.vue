<!-- 添加分类,未分页-->
<template>
	<transition enter-active-class="animated bounceInDown">
		<section class="add-tag">
			<el-title-header :title="$route.meta.title"></el-title-header>
            <el-confirm-cancel :show.sync="show" @confirm="confirm">确认删除该类目?（该操作会使文章不再有标签标注）</el-confirm-cancel>
			<div class="tag">
				<div v-for="(item,index) of tagArr">
                    {{item.tagName}}
					<i class="iconfont" @click="del(index)">&#xe629;</i>
				</div>
			</div>

			<div class="tag-title">添加标签</div>

			<div class="input">
				<div>
					<input
						type="text"
						maxlength="18"
						v-model.trim="formData.tagName"
						@keyup.enter="add"
						placeholder="标签名称">
                    <input
                        type="text"
                        maxlength="18"
                        v-model.trim="formData.tagUrl"
                        @keyup.enter="add"
                        placeholder="标签url"
                    />
				</div>
				<div class="btn" @click="add">添加</div>
			</div>
		</section>
	</transition>
</template>

<script>
export default {
	name: 'addTag',
	data () {
		return {
		    formData: {
                tagName: '',
                tagUrl: '',
            },
            show: false,
            tagArr: [],
            delTagid: '',
            delIndex: '',
		}
	},
    beforeCreate () {
	    this.$axios.get('/tag').then(response => {
	        this.tagArr = response.data
            console.log(this.tagArr)
        }).catch(error => {
            console.log(error)
        })
    },
	methods: {
		del (index) {
		    this.show = true
            this.delTagid = this.tagArr[index].tagid
            this.delIndex = index
		},
		add () {
		    this.$axios.post('/tag',this.formData).then(response => {
		        console.log(response)
                this.clear()
                this.$axios.get('/tag').then(response => {
                    this.tagArr = response.data
                    console.log(this.tagArr)
                }).catch(error => {
                    console.log(error)
                })
            }).catch(error => {
                console.log(error)
            })
		},
        confirm () {
            this.$axios.delete('tag?tagid='+this.delTagid).then(response => {
                console.log(response)
                this.tagArr.splice(this.delIndex,1)
            })
        },
        //清除表单内容
        clear () {
		    this.formData.tagName = '',
            this.formData.tagUrl = ''
        }
	}
}
</script>

















