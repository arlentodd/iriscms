<template>
  <div class="me-view-body" v-title :data-title="title">
    <el-container class="me-view-container" style="width: 980px;">
      <el-main>
        <div class="me-view-card">
          <h1 class="me-view-title">{{article.title}}</h1>
          <div class="me-view-author">
            <a class="">
              <img class="me-view-picture" :src="article.author.avatar"></img>
            </a>
            <div class="me-view-info">
              <span>{{article.author.nickname}}</span>
              <div class="me-view-meta">
                <span>{{article.createTime | format}}</span>
                <span>阅读   {{article.viewNum}}</span>
                <span>评论   {{article.commentNum}}</span>
              </div>
            </div>
            <el-button
              v-if="this.article.author.id == this.$store.state.id"
              @click="editArticle()"
              style="position: absolute;left: 60%;"
              size="mini"
              type="success"
              round
              icon="el-icon-edit">编辑</el-button>
          </div>
          <div class="me-view-content">
            <markdown-editor :editor=article.editor></markdown-editor>
          </div>

          <el-card shadow="hover" style="
    padding: 3px;
    background-color: #ecf8ff;
    border-radius: 4px;
    border-left: 5px solid #50bfff;
    font-size: 13px;
">
            <b style="font-size: 14px; margin-bottom: 8px;">资源地址: <a :href="this.article.source_url" target="_blank">{{this.article.source_url}}</a></b>
            <br/>
            <br/>
            获取下载密码:<br/>
            &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[打开微信]->[扫描<span style="color: orangered;cursor: pointer" @click="dialogTableVisible=true">二维码</span>]->[关注公众号]
            <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;输入 <i style="color: orangered">{{this.article.id}}</i> 获取分享码 如果取消关注本公众号，再次关注即可再次享受服务！
          </el-card>

          <el-dialog title="公众号" :visible.sync="dialogTableVisible" width="395px">
              <img src="../../../static/qrcode/wechat.jpg" style="width: 350px" />
          </el-dialog>

          <div class="me-view-tag">
            标签：
            <!--<el-tag v-for="t in article.tags" :key="t.id" class="me-view-tag-item" size="mini" type="success">{{t.tagname}}</el-tag>-->
            <el-button @click="tagOrCategory('tag', t)" size="mini" type="primary" v-for="t in article.tags" :key="t" round plain>{{t}}</el-button>
          </div>

          <div class="me-view-tag">
            文章分类：
            <el-button @click="tagOrCategory('category', article.category.id)" size="mini" type="primary" round plain>我是分类标题</el-button>
          </div>

          <div class="me-view-comment">
            <div class="me-view-comment-write">
              <el-row :gutter="20">
                <el-col :span="2">
                  <a class="">
                    <img class="me-view-picture" :src="avatar"></img>
                  </a>
                </el-col>
                <el-col :span="21">
                  <el-input
                    type="textarea"
                    :autosize="{ minRows: 2}"
                    placeholder="你的评论..."
                    class="me-view-comment-text"
                    v-model="comment.content"
                    resize="none">
                  </el-input>
                </el-col>
              </el-row>

              <el-row :gutter="20">
                <el-col :span="3" :offset="21">
                  <el-button type="text"  @click="publishComment()" style="margin-top: 5px;">评论</el-button>
                </el-col>
              </el-row>
            </div>

            <div class="me-view-comment-title">
              <span>{{article.commentNum}} 条评论</span>
            </div>

            <commment-item
              v-for="(c,index) in comments"
              :comment="c"
              :articleId="article.id"
              :index="index"
              :rootCommentCounts="comments.length"
              @commentCountsIncrement="commentCountsIncrement"
              :key="c.id">
            </commment-item>
          </div>
        </div>
      </el-main>

    </el-container>
  </div>
</template>

<script>
  import MarkdownEditor from '@/components/markdown/MarkdownEditor'
  import CommmentItem from '@/components/comment/CommentItem'
  import {viewArticle} from '@/api/article'
  import {getCommentsByArticle, publishComment} from '@/api/comment'

  import default_avatar from '@/assets/img/default_avatar.png'

  export default {
    name: 'BlogView',
    watch: {
      '$route': 'getArticle'
    },
    data() {
      return {
        dialogTableVisible: false,
        article: {
          id: '',
          title: '',
          commentNum: 0,
          viewNum: 0,
          summary: '',
          createTime: '',
          author: {},
          tags: [],
          category:{},
          editor: {
            value: '',
            toolbarsFlag: false,
            subfield: false,
            defaultOpen: 'preview'
          }
        },
        comments: [],
        comment: {
          article: {},
          content: ''
        }
      }
    },
    components: {
      'markdown-editor': MarkdownEditor,
      CommmentItem
    },
    computed: {
      avatar() {
        let avatar = this.$store.state.avatar
        if (avatar.length > 0) {
          return avatar
        }
        return default_avatar
      },
      title() {
        return `${this.article.title} - 文章 - ` + window.title + ' - ' + window.keywords + ' - ' + window.description
      }
    },
    created() {
      this.getArticle()
    },
    methods: {
      tagOrCategory(type, id) {
        this.$router.push({path: `/${type}/${id}`})
      },
      editArticle() {
        this.$router.push({path: `/write/${this.article.id}`})
      },
      getArticle() {
        let that = this
        viewArticle(that.$route.params.id).then(data => {
          data.data.tags = data.data.tags.split(',')
          Object.assign(that.article, data.data)
          that.article.editor.value = data.data.content
          // that.getCommentsByArticle()
        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '文章加载失败', showClose: true})
          }
        })
      },
      publishComment() {
        let that = this
        if (!that.comment.content) {
          return;
        }
        that.comment.article.id = that.article.id

        publishComment(that.comment).then(data => {
          that.$message({type: 'success', message: '评论成功', showClose: true})
          that.comments.unshift(data.data)
          that.commentCountsIncrement()
          that.comment.content = ''
        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '评论失败', showClose: true})
          }
        })
      },
      getCommentsByArticle() {
        let that = this
        getCommentsByArticle(that.article.id).then(data => {
          that.comments = data.data
        }).catch(error => {
          if (error !== 'error') {
            that.$message({type: 'error', message: '评论加载失败', showClose: true})
          }
        })
      },
      commentCountsIncrement() {
        this.article.commentCounts += 1
      }
    },
    //组件内的守卫 调整body的背景色
    beforeRouteEnter(to, from, next) {
      window.document.body.style.backgroundColor = '#fff';
      next();
    },
    beforeRouteLeave(to, from, next) {
      window.document.body.style.backgroundColor = '#f5f5f5';
      next();
    }
  }
</script>

<style>
  .me-view-body {
    margin: 100px auto 140px;
  }

  .me-view-container {
    width: 700px;
  }

  .el-main {
    overflow: hidden;
  }

  .me-view-title {
    font-size: 34px;
    font-weight: 700;
    line-height: 1.3;
  }

  .me-view-author {
    /*margin: 30px 0;*/
    margin-top: 30px;
    vertical-align: middle;
  }

  .me-view-picture {
    width: 40px;
    height: 40px;
    border: 1px solid #ddd;
    border-radius: 50%;
    vertical-align: middle;
    background-color: #5fb878;
  }

  .me-view-info {
    display: inline-block;
    vertical-align: middle;
    margin-left: 8px;
  }

  .me-view-meta {
    font-size: 12px;
    color: #969696;
  }

  .me-view-end {
    margin-top: 20px;
    padding-right: 25px;
  }

  .me-view-tag {
    margin-top: 20px;
    padding-left: 6px;
    border-left: 4px solid #c5cac3;
  }

  .me-view-tag-item {
    margin: 0 4px;
  }

  .me-view-comment {
    margin-top: 60px;
  }

  .me-view-comment-title {
    font-weight: 600;
    border-bottom: 1px solid #f0f0f0;
    padding-bottom: 20px;
  }

  .me-view-comment-write {
    margin-top: 20px;
  }

  .me-view-comment-text {
    font-size: 16px;
  }

  .v-show-content {
    padding: 8px 25px 15px 0px !important;
  }

  .v-note-wrapper .v-note-panel {
    box-shadow: none !important;
  }

  .v-note-wrapper .v-note-panel .v-note-show .v-show-content, .v-note-wrapper .v-note-panel .v-note-show .v-show-content-html {
    background: #fff !important;
  }


</style>
