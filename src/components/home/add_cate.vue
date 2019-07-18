<!-- 添加类目，未分页 -->
<template>
    <transition enter-active-class="animated bounceInDown">
        <section class="add-tag">
            <el-title-header :title="$route.meta.title"></el-title-header>
            <el-confirm-cancel :show.sync="show" @confirm="confirm">确认删除该类目?（该操作会使对应类目的文章归类为默认类目下）</el-confirm-cancel>
            <div class="tag">
                <div v-for="(item,index) of cateArr" :key="item.cateid">
                    {{item.cateName}}
                    <i class="iconfont" @click="del(index)">&#xe629;</i>
                </div>
            </div>

            <div class="tag-title">添加类目</div>

            <div class="input">
                <div>
                    <input
                        type="text"
                        maxlength="18"
                        v-model.trim="formDate.cateName"
                        @keyup.enter="add"
                        placeholder="类目名称">
                    <input
                        type="text"
                        maxlength="18"
                        v-model.trim="formDate.cateDesc"
                        @keyup.enter="add"
                        placeholder="类目描述"
                    />
                    <input
                        type="text"
                        maxlength="18"
                        v-model.trim="formDate.cateUrl"
                        @keyup.enter="add"
                        placeholder="类目url"
                    />
                </div>
                <div class="btn" @click="add">添加</div>
            </div>
        </section>
    </transition>
</template>

<script>
    export default {
        name: 'addCate',
        data () {
            return {
                formDate: {
                    cateName: '',
                    cateDesc: '',
                    cateUrl: '',
                },
                pageNum: 1,
                show: false,
                cateArr: [],
                delid: '',
                delIndex: '',
            }
        },
        beforeCreate () {
            this.$axios.get('/category?pageNum=0').then(response => {
                this.cateArr = response.data
                console.log(this.cateArr)
            }).catch(error => {
                console.log(error)
            })
        },
        methods: {
            del (index) {
                this.show = true
                this.delid = this.cateArr[index].cateid
                this.delIndex = index
            },
            add () {
                this.$axios.post("/category",this.formDate).then(response => {
                    console.log(response)
                    this.clear()
                    this.$axios.get('/category?pageNum=0').then(response => {
                        this.cateArr = response.data
                        this.$message({
                            message:'添加成功',
                            type: 'success'
                        })
                    }).catch(error => {
                        console.log(error)
                    })
                }).catch(error => {
                    console.log(error)
                })
            },
            confirm () {
                this.$axios.delete('category?cateid='+this.delid).then(response => {
                    console.log(response)
                    this.cateArr.splice(this.delIndex,1)
                }).catch(error => {
                    console.log(error)
                })
            },
            //清除表单内容
            clear () {
               this.formDate.cateDesc = '',
               this.formDate.cateName = '',
               this.formDate.cateUrl = ''
            },
        }
    }
</script>

















