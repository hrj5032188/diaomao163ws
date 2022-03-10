<template>
    <view class="content">
        <a-modal v-model:visible="wsDialogVisible" title="填写叼毛 websocket 地址" :keyboard="false" :maskClosable="false" :closable="false">
            <a-input v-model:value="wsurl" placeholder="192.168.0.1:23333">
                <template #prefix>
                    <android-outlined style="color: rgba(0, 0, 0, 0.45)" />
                </template>
                <template #suffix>
                    <a-tooltip title="ip:port形式">
                        <info-circle-outlined style="color: rgba(0, 0, 0, 0.45)" />
                    </a-tooltip>
                </template>
            </a-input>
            <div style="margin-top: 8px; color: #666; font-size: 14px">
                帮助：请先保证手机和当前网页在同一局域网，手机ip在设置-状态信息-ip可以看到，端口号改为实际的
            </div>
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
                <template #title>收藏</template>
                <heart-two-tone two-tone-color="#eb2f96" @click="action('like')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>取消收藏</template>
                <frown-outlined @click="action('dislike')" class="action" />
            </a-tooltip>
            <a-tooltip>
                <template #title>添加到指定歌单</template>
                <folder-add-outlined @click="onClickAddPlayList" class="action" />
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
    </view>
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
} from "@ant-design/icons-vue";
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
    },
    data() {
        return {
            websock: null, // ws对象
            wsurl: "", // wsServer地址
            musicName: "", // 歌名
            singerName: "", // 歌手名
            lyric: "", // 歌词
            playListId: "", // 歌单ID 通过分享url中获取
            wsDialogVisible: true,
            wsLoading: false,
            playState: -1,
            playListDialogVisible: false,
        };
    },
    created() {
        this.wsurl = localStorage.getItem("wsurl") || "";
        this.playListId = localStorage.getItem("playListId") || "";
        this.initWebSocket();
    },
    onLoad() {},
    unmounted: function () {
        this.websock.websocketclose(); //离开路由之后断开websocket连接
    },
    methods: {
        initWebSocket() {
            let wsuri = this.wsurl || "";
            if (wsuri.indexOf("ws://") < 0) {
                wsuri = `ws://${wsuri}`;
            }
            const urlReg =
                /^ws:\/\/(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5]):\d+$/;
            if (!urlReg.test(wsuri)) {
                message.error("websocket地址填写不合法！");
                return;
            }
            localStorage.setItem("wsurl", wsuri);
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
        },
        websocketonerror() {
            // this.initWebSocket();
            this.wsDialogVisible = true;
            this.wsLoading = false;
            message.error("websocket连接失败，请检查ip:port");
        },
        websocketonmessage(e) {
            const redata = JSON.parse(e.data);
            console.log("接受数据: ", redata);
            if (redata.action === "music") {
                this.musicName = redata.musicName;
                this.singerName = redata.singerName;
                this.playState = redata.state || -1;
            } else if (redata.action === "lrc") {
                this.lyric = redata.content;
            }
        },
        websocketsend(Data) {
            this.websock.send(Data);
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
