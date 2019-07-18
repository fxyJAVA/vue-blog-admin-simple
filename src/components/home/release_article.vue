<!-- 发布文章 -->
<template>
    <transition enter-active-class="animated bounceInDown">
        <section class="release-article">
            <el-title-header :title="$route.meta.title"></el-title-header>

            <!--<div class="radio">-->
            <!--<div v-for="n of 8" @click="index = n" :class="{active: index == n}">JavaScript</div>-->
            <!--</div>-->
            <el-form>
                <el-form-item label="标题">
                    <el-input v-model="formData.title"/>
                </el-form-item>
                <el-form-item label="摘要">
                    <el-input type="textarea" v-model="formData.summary"/>
                </el-form-item>

                <el-select
                    v-model="tempCateid"
                    filterable placeholder="所属类目"
                    @change="chooseTag">
                    <el-option
                        v-for="item in tagArr"
                        :key="item.tagid"
                        :label="item.tagName"
                        :value="item.tagid">
                    </el-option>
                </el-select>
                &nbsp;&nbsp;&nbsp;
                <el-select v-model="formData.tagid" multiple placeholder="所属标签">
                    <el-option
                        v-for="item in cateArr"
                        :key="item.cateid"
                        :label="item.cateName"
                        :value="item.cateid">
                    </el-option>
                </el-select>

                <el-upload
                    style="margin-top: 20px;"
                    action="#"
                    :multiple="false"
                    :limit="1"
                    :on-change="image2base64"
                    list-type="picture-card"
                    :auto-upload="false">
                    <i slot="default" class="el-icon-plus"></i>
                    <div slot="file" slot-scope="{file}">
                        <img
                            class="el-upload-list__item-thumbnail"
                            :src="file.url" alt=""
                        >
                        <span class="el-upload-list__item-actions">
                        <span
                            class="el-upload-list__item-preview"
                            @click="handlePictureCardPreview(file)">
                          <i class="el-icon-zoom-in"></i>
                        </span>
                        <span
                            v-if="!disabled"
                            class="el-upload-list__item-delete"
                            @click="handleDownload(file)">
                          <i class="el-icon-download"></i>
                        </span>
                        <span
                            v-if="!disabled"
                            class="el-upload-list__item-delete"
                            @click="handleRemove(file)">
                          <i class="el-icon-delete"></i>
                        </span>
                      </span>
                    </div>
                </el-upload>
                <el-dialog>
                    <img width="100%" :src="dialogImageUrl" alt="">
                </el-dialog>
                <div>
                    <br>
                    <span style="display: block;font-size: 20px">正文</span>
                    <markdown-editor v-model="formData.contentMd" :configs="configs"></markdown-editor>
                </div>
            </el-form>
            <div class="release-btn" @click="submit">发布文章</div>
        </section>
    </transition>
</template>

<script>
    import markd from 'marked'
    export default {
        name: 'releaseArticle',
        data() {
            return {
                tempCateid:'',
                flag: true,
                index: -1,
                imageFile: '',
                tagArr: [],
                cateArr: [],
                tagList: [{
                    value: '选项1',
                    label: '黄金糕'
                }, {
                    value: '选项2',
                    label: '双皮奶'
                }, {
                    value: '选项3',
                    label: '蚵仔煎'
                }, {
                    value: '选项4',
                    label: '龙须面'
                }, {
                    value: '选项5',
                    label: '北京烤鸭'
                }],
                dialogImageUrl: ''  ,
                formData:{
                  title: '',
                  image: '',
                  contentMd: '',
                  contentHtml: '',
                  summary: '',
                  cateid: [],
                  tagid: [],
                },
                cateList: [{
                    value: '选项1',
                    label: '黄金糕'
                }, {
                    value: '选项2',
                    label: '双皮奶'
                }, {
                    value: '选项3',
                    label: '蚵仔煎'
                }, {
                    value: '选项4',
                    label: '龙须面'
                }, {
                    value: '选项5',
                    label: '北京烤鸭'
                }],
                content: '',
                configs: {
                    status: false,
                    spellChecker: false,
                    renderingConfig: {
                        codeSyntaxHighlighting: true,
                        highlightingTheme: 'atom-one-light'
                    }
                }
            }
        },
        beforeCreate () {
            this.$axios('/tag').then(response => {
                this.tagArr = response.data
                console.log(this.tagArr)
            }).catch(error =>{
                console.log(error)
            })
            this.$axios.get('/category?pageNum=0').then(response => {
                this.cateArr = response.data
                console.log(this.cateArr)
            }).catch(error => {
                console.log(error)
            })
        },

        previewRender: function(plainText) {
            return customMarkdownParser(plainText) // 返回HTML自定义解析器
        },
        methods: {
            submit () {
                this.formData.image = this.temp3
                console.log(markd(this.formData.contentMd))
                console.log(this.formData.image)
                this.$axios.post('/article',this.formData).then(response=>{
                    console.log(response)
                }).catch(error=>{
                    console.log(error)
                })
            },
            chooseTag() {
                this.formData.cateid = []
                this.formData.cateid.push(this.tempCateid)
                console.log(this.formData.cateid)
            },
            image2base64 (file) {
                let self = this
                let reader = new FileReader()
                reader.readAsDataURL(file.raw)
                reader.onload = function(e) {
                    self.temp3 = this.result
                }
            }
        }
    }
</script>














