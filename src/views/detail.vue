<template>
  <div>
    <div class="block wow slideInLeft" v-loading="loading">
      <!-- 您的位置 -->
      <location Tit1="首页" Tit2="技术博文" class="wow slideInLeft">
        <code-style-select @OnCodeStyle="getCodeStyle" />
      </location>
      <!-- <SwitchStyle /> -->
      <!-- 具体文章 -->
      <article-contents :codeStyle="codeStyle" :data="articleData" class="wow slideInLeft"></article-contents>
      <!-- 点赞/打赏 -->
      <div class="likeBox">
        <el-button type="primary" round @click="like"><i class="iconfont  My-new-icondianzan"></i>点赞（{{
            likeCount
          }}）</el-button>
        <el-button type="success" round @click="dialogVisible = true"><i class="iconfont My-new-icondashang"></i>打赏</el-button>
      </div>
      <!-- 弹框 -->
      <el-dialog :visible.sync="dialogVisible" width="30%" custom-class="tankuang">
      </el-dialog>
      <!-- 评论 -->
      <comment :articleId="id"></comment>
    </div>
  </div>
</template>

<script>

import comment from "@/components/comment/Comment.vue";
import location from "../components/Location/location.vue";
import ArticleContents from "../components/ArticleContents/ArticleContents.vue";
import CodeStyleSelect from '@/components/CodeStyleSelect/CodeStyleSelect.vue'
import Cookie from "js-cookie";
import SwitchStyle from "@/components/switchStyle/SwitchStyle.vue";
import { getArticleDetail } from '@/network/article.js'
export default {
  components: {
    comment,
    location,
    ArticleContents,
    SwitchStyle,
    CodeStyleSelect
  },
  // 同一组件路由动态变化时被调用 /detail/168 -> /detail/204
  beforeRouteUpdate(to, from, next) {
    this.loading = true;
    this.id = to.params.id;
    this.GetArticleDetail(to.params.id);
    next();
  },
  data() {
    return {
      loading: true,
      id: this.$route.params.id,
      articleData: {},

      likeCount: "",

      dialogVisible: false,

      nickname: "",

      codeStyle: this.$store.state.codeStyle
    };
  },
  methods: {
    getCodeStyle(newVal) {
      this.codeStyle = newVal
    },
    GetArticleDetail(id) {
      getArticleDetail({ article_id: id }).then(({ data }) => {
        this.articleData = data;
        this.likeCount = data.like_count;
        this.loading = false;
      });
    },
    // 点赞
    like() {
      // 判断是否登录了
      if (Cookie.get("token")) {
        this.$http
          .post("/api/article/like", {
            article_id: this.$route.params.id,
          })
          .then((res) => {
            let message = {};
            if (res.data.code === 200) {
              this.likeCount = res.data.count.like_count;
              message = {
                message: res.data.msg,
                type: "success",
              }
            } else {
              message = {
                message: res.data.msg,
                type: "warning",
              }
            }
            this.$message(message);
          });
      } else {
        this.$message({
          message: "请登录后进行操作",
          type: "warning",
        });
        this.$router.push({ name: "login" });
      }
    },
  },
  created() {
    this.GetArticleDetail(this.id);
  },
  mounted() {
    this.$animation();
  },
};
</script>

<style lang="scss" scoped>
.head {
  margin-top: 20px;
}
.likeBox {
  display: flex;
  justify-content: center;
  margin: 25px 0;
  .el-button {
    margin: 0 10px;
  }
  i {
    margin-right: 8px;
  }
}

.wx {
  width: 150px;
}
</style>
