<template>
    <div :class="{ content: !darkMode, contentBlack: darkMode }">
        <a-modal v-model:visible="wsDialogVisible" title="填写叼毛控制码" :keyboard="false" :maskClosable="false" :closable="false">
            <a-input v-model:value="wsChannelCode" placeholder="">
                <template #prefix>
                    <android-outlined style="color: rgba(0, 0, 0, 0.45)" />
                </template>
                <template #suffix>
                    <a-tooltip title="控制码在叼毛设置打开远程开关可见">
                        <info-circle-outlined style="color: rgba(0, 0, 0, 0.45)" />
                    </a-tooltip>
                </template>
            </a-input>
            <div style="margin-top: 8px; color: #666; font-size: 14px"></div>
            <template #footer>
                <a-button key="submit" type="primary" :loading="wsLoading" @click="initWebSocket">确定</a-button>
            </template>
        </a-modal>
        <a-modal v-model:visible="playListDialogVisible" title="填写歌单id" @ok="action('add2playList')">
            <a-input v-model:value="playListId" placeholder="">
                <template #prefix>
                    <android-outlined style="color: rgba(0, 0, 0, 0.45)" />
                </template>
                <template #suffix>
                    <a-tooltip title="数字">
                        <info-circle-outlined style="color: rgba(0, 0, 0, 0.45)" />
                    </a-tooltip>
                </template>
            </a-input>
            <div style="margin-top: 8px; color: #666; font-size: 14px">帮助：歌单 id 获取需要分享歌单链接</div>
        </a-modal>
        <view>
            <div class="music-title">{{ musicName }}</div>
            <div class="music-singler">{{ singerName }}</div>
            <div class="mucic-lyric">{{ lyric }}</div>
        </view>
        <a-row type="flex" justify="center" align="middle">
            <a-tooltip>
                <template #title>减小音量</template>
                <minus-outlined @click="action('volumeDown')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>播放上一曲</template>
                <step-backward-outlined @click="action('playPre')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>播放/暂停</template>
                <pause-outlined @click="action('playOrPause')" class="action action-pause" v-if="playState === 3" />
                <caret-right-outlined @click="action('playOrPause')" class="action action-play" v-else />
            </a-tooltip>

            <a-tooltip>
                <template #title>播放下一曲</template>
                <step-forward-outlined @click="action('playNext')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>增大音量</template>
                <plus-outlined @click="action('volumeUp')" class="action" />
            </a-tooltip>
        </a-row>
        <a-row type="flex" justify="center" align="middle">
            <a-tooltip>
                <template #title>取消收藏</template>
                <frown-outlined @click="action('dislike')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>收藏</template>
                <heart-two-tone two-tone-color="#eb2f96" @click="action('like')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>添加到指定歌单</template>
                <folder-add-outlined @click="onClickAddPlayList" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>切换夜间模式</template>
                <bulb-outlined @click="() => (darkMode = !darkMode)" class="action" />
            </a-tooltip>
        </a-row>
        <!-- <div>
            <button type="default" @click="action('playOrPause')">播放/暂停</button>
            <button type="default" @click="action('playPre')">播放上一曲</button>
            <button type="default" @click="action('playNext')">播放下一曲</button>
            <button type="default" @click="action('volumeUp')">音量 +</button>
            <button type="default" @click="action('volumeDown')">音量 -</button>
        </div> -->
        <!-- <view class="block">
            <view class="flex"> 歌单ID: <input class="input" type="text" v-model="id" placeholder="/playlist?id=" /> </view>
            <button type="default" @click="action('add2playList')">添加到指定列表</button>
            <button type="default" @click="action('like')">添加收藏</button>
            <button type="default" @click="action('dislike')">取消收藏</button>
        </view> -->
    </div>
</template>

<script>
import { message } from "ant-design-vue";
import {
    AndroidOutlined,
    InfoCircleOutlined,
    StepBackwardOutlined,
    StepForwardOutlined,
    PauseOutlined,
    CaretRightOutlined,
    MinusOutlined,
    PlusOutlined,
    FrownOutlined,
    FolderAddOutlined,
    HeartTwoTone,
    BulbOutlined,
} from "@ant-design/icons-vue";
import { v4 as uuidv4 } from "uuid";
export default {
    components: {
        AndroidOutlined,
        InfoCircleOutlined,
        StepBackwardOutlined,
        StepForwardOutlined,
        PauseOutlined,
        CaretRightOutlined,
        MinusOutlined,
        PlusOutlined,
        FrownOutlined,
        FolderAddOutlined,
        HeartTwoTone,
        BulbOutlined,
    },
    data() {
        return {
            websock: null, // ws对象
            wsChannelCode: "",
            musicName: "", // 歌名
            singerName: "", // 歌手名
            lyric: "", // 歌词
            playListId: "", // 歌单ID 通过分享url中获取
            wsDialogVisible: true,
            wsLoading: false,
            playState: -1,
            playListDialogVisible: false,
            darkMode: false,
        };
    },
    created() {
        this.wsChannelCode = localStorage.getItem("wsChannelCode") || "";
        this.playListId = localStorage.getItem("playListId") || "";
        if (this.wsChannelCode.length > 0) {
            this.initWebSocket();
        }
    },
    onLoad() {},
    unmounted: function () {
        this.websocketclose(); //离开路由之后断开websocket连接
    },
    methods: {
        initWebSocket() {
            let wsuri = "wss://hack.chat/chat-ws";
            localStorage.setItem("wsChannelCode", this.wsChannelCode);
            console.log("initWebSocket", wsuri);
            this.wsLoading = true;
            this.websock = new WebSocket(wsuri);
            this.websock.onmessage = this.websocketonmessage;
            this.websock.onopen = this.websocketonopen;
            this.websock.onerror = this.websocketonerror;
            this.websock.onclose = this.websocketclose;
            console.log(this.websock);
        },
        websocketonopen() {
            this.wsDialogVisible = this.wsLoading = false;
            let channelName = this.wsChannelCode;
            console.log("channelName", channelName);
            if (channelName !== "diaomao") channelName = "diaomao163_" + channelName;
            const nickName = "diaomao_web_" + uuidv4().slice(0, 6);
            this.websock.send('{"cmd":"join","channel":"' + channelName + '","nick":"' + nickName + '"}');
        },
        websocketonerror() {
            // this.initWebSocket();
            this.wsDialogVisible = true;
            this.wsLoading = false;
            message.error("websocket连接失败，请重试");
        },
        websocketonmessage(e) {
            const rawData = JSON.parse(e.data);
            console.log("接受数据: ", rawData);
            const cmd = rawData.cmd;
            if (cmd != "chat") return;
            const text = rawData.text;
            try {
                const redata = JSON.parse(text);
                if (redata.action === "music") {
                    if (this.musicName !== redata.musicName) {
                        this.musicName = redata.musicName;
                        this.singerName = redata.singerName;
                        this.lyric = "";
                    }
                    this.playState = redata.state || 3;
                } else if (redata.action === "lrc") {
                    this.lyric = redata.content;
                }
            } catch (e) {
                console.log(e);
            }
        },
        websocketsend(data) {
            const msg = {
                cmd: "chat",
                text: data,
            };
            this.websock.send(JSON.stringify(msg));
        },
        websocketclose(e) {
            console.log("断开连接", e);
            if (this.websock != null) {
                this.websock.close();
            }
            message.error("websocket已关闭，请重新连接");
            this.wsDialogVisible = true;
            this.wsLoading = false;
        },
        onClickAddPlayList() {
            this.playListDialogVisible = true;
        },
        action(ins) {
            console.log("指令: ", ins);
            // 一般指令通用封装
            let actions = {};
            actions["action"] = ins;
            // 添加到指定歌单列表时的列表ID
            if (ins === "add2playList") {
                actions["playListId"] = this.playListId;
                if (!this.playListId) {
                    message.error("歌单 id 不能为空");
                    return;
                }
                localStorage.setItem("playListId", this.playListId);
                this.playListDialogVisible = false;
            }
            console.log("actions: ", actions);
            this.websocketsend(JSON.stringify(actions));
            message.success("操作成功");
        },
    },
};
</script>

<style>
.content {
    padding-top: 60px;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
    /* align-items: center; */
    /* justify-content: center; */
    /* background-color: rgb(209, 22, 22); */
}
.contentBlack {
    padding-top: 60px;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
    background-color: #26363e;
    display: flex;
    flex-direction: column;
    /* align-items: center; */
    /* justify-content: center; */

    /* div{
        color: #c9d1d9;
    } */
}
.contentBlack div {
    color: #f0f6fc;
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
    margin-bottom: 32px;
}

.flex {
    display: flex;
}

.input {
    border: 1px solid #007aff;
    border-radius: 3px;
    padding-left: 5px;
    margin-left: 5px;
    vertical-align: middle;
}

.block {
    margin-top: 20px;
}
.music-title {
    font-size: 24px;
    font-weight: 600;
}
.music-singler {
    font-size: 16px;
}
.mucic-lyric {
    margin: 16px 0;
    font-size: 36px;
}
.acion {
    font-size: 24px;
}
.action {
    font-size: 32px;
    margin: 2px 4px;
    padding: 8px;
}
.action-pause,
.action-play {
    font-size: 56px;
}
</style>
