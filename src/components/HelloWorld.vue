<template>
  <view class="content">
    <!-- <image class="logo" src="/static/logo.png"></image> -->
    <view class="text-area">
      <text class="title">{{title}}</text>
    </view>
    <view class="block">
      <view class="flex">
        websocket地址:<input class="input" type="text" v-model="wsurl">
      </view>
      <button type="default" @click="initWebSocket">连接</button>
      <button type="default" @click="websocketclose">断开连接</button>
    </view>

    <view>
      <text>歌曲: {{musicName}}</text>
      <br>
      <text>歌手: {{singerName}}</text>
      <br>
      <text>歌词: {{lyric}}</text>
    </view>
    <view>
      <button type="default" @click="action('playOrPause')">播放/暂停</button>
      <button type="default" @click="action('playPre')">播放上一曲</button>
      <button type="default" @click="action('playNext')">播放下一曲</button>
      <button type="default" @click="action('volumeUp')">音量 +</button>
      <button type="default" @click="action('volumeDown')">音量 -</button>
    </view>
    <view class="block">
      <view class="flex">
        歌单ID: <input class="input" type="text" v-model="id" placeholder="/playlist?id=">
      </view>
      <button type="default" @click="action('add2playList')">添加到指定列表</button>
      <button type="default" @click="action('like')">添加收藏</button>
      <button type="default" @click="action('dislike')">取消收藏</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      title: 'Hello',
      websock: null, // ws对象
      wsurl: 'ws://192.168.2.214:23333', // wsServer地址
      musicName: '', // 歌名
      singerName: '', // 歌手名
      lyric: '', // 歌词
      id: '', // 歌单ID 通过分享url中获取
    }
  },
  created() {
    // this.initWebSocket();
  },
  onLoad() {

  },
  unmounted: function () {
    this.websock.websocketclose() //离开路由之后断开websocket连接
  },
  methods: {
    initWebSocket() { //初始化weosocket
      // this.websocketclose();
      console.log('initWebSocket')
      const wsuri = this.wsurl;
      this.websock = new WebSocket(wsuri);
      this.websock.onmessage = this.websocketonmessage;
      this.websock.onopen = this.websocketonopen;
      this.websock.onerror = this.websocketonerror;
      this.websock.onclose = this.websocketclose;
      console.log(this.websock);
    },
    websocketonopen() { //连接建立之后执行send方法发送数据
      let actions = {
        "test": "12345"
      };
      this.websocketsend(JSON.stringify(actions));
    },
    websocketonerror() { //连接建立失败重连
      this.initWebSocket();
    },
    websocketonmessage(e) { //数据接收
      const redata = JSON.parse(e.data);
      console.log('接受数据: ', redata);
      if (redata.action === 'music') {
        this.musicName = redata.musicName;
        this.singerName = redata.singerName;
      } else if (redata.action === 'lrc') {
        this.lyric = redata.content;
      }
    },
    websocketsend(Data) { //数据发送
      this.websock.send(Data);
    },
    websocketclose(e) { //关闭
      console.log('断开连接', e);
      if (this.websock != null) {
        this.websock.close();
      }
    },
    action(ins) {
      console.log("指令: ", ins);
      // 一般指令通用封装
      let actions = {};
      actions['action'] = ins;
      // 添加到指定歌单列表时的列表ID
      if (ins === 'add2playList') {
        actions['playListId'] = this.id
      }
      console.log("actions: ", actions);
      this.websocketsend(JSON.stringify(actions));
    }
  }
}
</script>

<style>
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.logo {
  height: 200px;
  width: 200px;
  margin-top: 200px;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 50px;
}

.text-area {
  display: flex;
  justify-content: center;
}

.title {
  font-size: 36px;
  color: #8f8f94;
}

.flex {
  display: flex;
}

.input {
  border: 1px solid #007AFF;
  border-radius: 3px;
  padding-left: 5px;
  margin-left: 5px;
  vertical-align: middle;
}

.block {
  margin-top: 20px;
}
</style>
