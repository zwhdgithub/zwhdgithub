<template>
  <div>
    <!-- Header 区域 -->
    <van-nav-bar fixed title="文章详情" left-arrow @click-left="$router.back()" />

    <!-- 文章信息区域 -->
    <div class="article-container">
      <!-- 文章标题 -->
      <h1 class="art-title">{{artObj.title}}</h1>

      <!-- 用户信息 -->
      <van-cell center :title="artObj.aut_name" :label="artObj.pubdate">
        <template #icon>
          <img :src="artObj.arraut_photo" alt class="avatar" />
        </template>
        <template #default>
          <div>
            <van-button
              type="info"
              size="mini"
              @click="followedFn(false)"
              v-if="artObj.is_followed"
            >已关注</van-button>
            <van-button
              icon="plus"
              type="info"
              size="mini"
              plain
              @click="followedFn(true)"
              v-else
            >关注</van-button>
          </div>
        </template>
      </van-cell>

      <!-- 分割线 -->
      <van-divider></van-divider>

      <!-- 文章内容 -->
      <div class="art-content" v-html="artObj.content"></div>

      <!-- 分割线 -->
      <van-divider>End</van-divider>

      <!-- 点赞 -->
      <div class="like-box">
        <van-button
          icon="good-job"
          type="danger"
          size="small"
          @click="loveFn(-1)"
          v-if="artObj.attitude===1"
        >已点赞</van-button>
        <van-button icon="good-job-o" type="danger" plain size="small" @click="loveFn(1)" v-else>点赞</van-button>
      </div>
    </div>
    <!-- 文章评论区 -->
    <CommentList></CommentList>
  </div>
</template>

<script>
import CommentList from './CommentList.vue'
import {
  followedUserAPI,
  unFollowedUserAPI,
  articleDetailAPI,
  articleLikeAPI,
  articleDisLikeAPI
} from '@/api'

export default {
  // eslint-disable-next-line key-spacing
  name: 'ArticleDetail',
  data () {
    return {
      artObj: {} // 文章对象
    }
  },
  components: {
    CommentList
  },
  async created () {
    const { data: res } = await articleDetailAPI({
      id: this.$route.query.art_id
    })
    console.log(res)
    this.artObj = res.data
  },
  methods: {
    async followedFn (flag) {
      if (flag === true) {
        this.artObj.is_followed = true
        await followedUserAPI({
          target: this.artObj.aut_id
        })
      } else {
        this.artObj.is_followed = false
        await unFollowedUserAPI({
          // 文档中写了body
          uid: this.artObj.aut_id
        })
      }
    },
    async loveFn (num) {
      if (num === 1) {
        this.artObj.attitude = 1
        await articleLikeAPI({
          target: this.$route.query.art_id
        })
      } else {
        this.artObj.attitude = -1
        await articleDisLikeAPI({
          // 文档中写了路径参数
          artId: this.$route.query.art_id
        })
      }
    }
  }
}
</script>

<style scoped lang="less">
.article-container {
  padding: 10px;
  margin-top: 46px;
}
.art-title {
  font-size: 16px;
  font-weight: bold;
  margin: 10px 0;
}

.art-content {
  font-size: 12px;
  line-height: 24px;
  width: 100%;
  overflow-x: scroll;
  word-break: break-all;
  /deep/ img {
    width: 100%;
  }
  /deep/ pre {
    white-space: pre-wrap;
    word-wrap: break-word;
  }
}

.van-cell {
  padding: 5px 0;
  &::after {
    display: none;
  }
}

.avatar {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background-color: #f8f8f8;
  margin-right: 5px;
  border: none;
}

.like-box {
  display: flex;
  justify-content: center;
}
</style>
