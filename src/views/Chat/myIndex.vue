<template>
  <div class="container">
    <!-- 固定导航 -->
    <van-nav-bar fixed left-arrow @click-left="$router.back()" title="小思同学"></van-nav-bar>

    <!-- 聊天主体区域 -->
    <div class="chat-list">
      <div v-for="(obj,index) in list" :key="index">
        <!-- 左侧是机器人小思 -->
        <div class="chat-item left" v-if="obj.name!=='me'">
          <van-image fit="cover" round src="https://img.yzcdn.cn/vant/cat.jpeg" />
          <div class="chat-pao">{{obj.msg}}</div>
        </div>

        <!-- 右侧是当前用户 -->
        <div class="chat-item right" v-else>
          <div class="chat-pao">{{obj.msg}}</div>
          <van-image fit="cover" round :src="$store.state.userPhoto" />
        </div>
      </div>
    </div>

    <!-- 对话区域 -->
    <div class="reply-container van-hairline--top">
      <van-field v-model="word" placeholder="说点什么...">
        <template #button>
          <span style="font-size:12px;" @click="sendFn">提交</span>
        </template>
      </van-field>
    </div>
  </div>
</template>

<script>
// 导入 socket.io-client 包
import { io } from 'socket.io-client'
// 定义变量，存储 websocket 实例

export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: 'Chat',
  data () {
    return {
      word: '', // 输入框的内容
      // 所有的聊天消息
      list: [
        // 只根据 name 属性，即可判断出这个消息应该渲染到左侧还是右侧
        { name: 'xs', msg: 'hi，你好！我是小思' },
        { name: 'me', msg: '我是编程小王子' }
      ],
      socket: null
    }
  },
  created () {
    // 创建客户端 websocket 的实例,io建立socket连接
    this.socket = io('http://toutiao.itheima.net', {
      query: {
        token: this.token
      },
      transports: ['websocket']
    })
    // 监听是否连接成功，成功了就执行后面：Event：connect
    this.socket.on('connect', () => {
      console.log('连接建立成功')
    })
    // 接收到消息事件
    this.socket.on('message', (data) => {
      console.log(data)
      this.list.push({
        name: 'xs',
        msg: data.msg
      })
    })
  },
  // 组件销毁之前，关闭连接
  beforeDestroy () {
    this.socket.close()
    // 将实例化对象置为空
    this.socket = null
  },
  methods: {
    sendFn () {
      // 判断内容是否为空
      if (!this.word) return
      // 把消息发送给 websocket 服务器
      this.socket.emit('message', {
        msg: this.word,
        timestamp: new Date().getTime()
      })
      // 添加聊天消息到 list 列表中
      this.list.push({
        name: 'me',
        msg: this.word
      })
      // 清空文本框的内容
      this.word = ''
    }
  },
  watch: {
    list: {
      handler () {
        this.$nextTick(() => {
          const chatItem = document.querySelectorAll('.chat-item')
          const lastItem = chatItem[chatItem.length - 1]
          lastItem.scrollIntoView({
            behavior: 'smooth' // 动画平滑
          })
        })
      }
    }
  }
}
</script>

<style lang="less" scoped>
.container {
  height: 100%;
  width: 100%;
  position: absolute;
  left: 0;
  top: 0;
  box-sizing: border-box;
  background: #fafafa;
  padding: 46px 0 50px 0;
  .chat-list {
    height: 100%;
    overflow-y: scroll;
    .chat-item {
      padding: 10px;
      .van-image {
        vertical-align: top;
        width: 40px;
        height: 40px;
      }
      .chat-pao {
        vertical-align: top;
        display: inline-block;
        min-width: 40px;
        max-width: 70%;
        min-height: 40px;
        line-height: 38px;
        border: 0.5px solid #c2d9ea;
        border-radius: 4px;
        position: relative;
        padding: 0 10px;
        background-color: #e0effb;
        word-break: break-all;
        font-size: 14px;
        color: #333;
        &::before {
          content: '';
          width: 10px;
          height: 10px;
          position: absolute;
          top: 12px;
          border-top: 0.5px solid #c2d9ea;
          border-right: 0.5px solid #c2d9ea;
          background: #e0effb;
        }
      }
    }
  }
}
.chat-item.right {
  text-align: right;
  .chat-pao {
    margin-left: 0;
    margin-right: 15px;
    &::before {
      right: -6px;
      transform: rotate(45deg);
    }
  }
}
.chat-item.left {
  text-align: left;
  .chat-pao {
    margin-left: 15px;
    margin-right: 0;
    &::before {
      left: -5px;
      transform: rotate(-135deg);
    }
  }
}
.reply-container {
  position: fixed;
  left: 0;
  bottom: 0;
  height: 44px;
  width: 100%;
  background: #f5f5f5;
  z-index: 9999;
}
</style>
