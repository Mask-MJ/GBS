<template>
    <div :class="['modal', { fade: fade }]" data-keyboard="true" data-backdrop="static" tabindex="-1">
        <div class="modal-dialog modal-lg">
            <div class="modal-content">
                <div class="modal-header">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
					    <span aria-hidden="true">&times;</span>
					</button>
                    <h4 class="modal-title text-primary text-center"><span>{{videoTitle}}</span></h4>
                </div>
                <div class="modal-body">
                    <LivePlayer v-if="bShow" :videoUrl="videoUrl" :waterMark="osd" :snapUrl="snapUrl" :live="live"
                        @message="$message" :hideBigPlayButton="!!serverInfo.HideBigPlayButton"
                        v-loading="bLoading" :loading.sync="bLoading" element-loading-text="加载中..." element-loading-background="#000">
                    </LivePlayer>
                    <div class="text-center" v-if="isDemoUser(serverInfo, userInfo)">
                        <br>
                        提示: 演示系统限制匿名登录播放时间, 若需测试长时间播放, 请<a target="_blank" href="//www.liveqing.com/docs/download/LiveGBS.html">下载使用</a>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import 'jquery-ui/ui/widgets/draggable'
import LivePlayer from '@liveqing/liveplayer'
import { mapState } from "vuex"

export default {
    data() {
        return {
            videoUrl: "",
            videoTitle: "",
            osd: "",
            snapUrl: "",
            bShow: false,
            bLoading: false
        }
    },
    props: {
        live : {
            type: Boolean,
            default: false
        },
        fade: {
            type: Boolean,
            default: false
        }
    },
    mounted() {
        $(this.$el).find('.modal-content').draggable({
            handle: '.modal-header',
            cancel: ".modal-title span",
            addClasses: false,
            containment: 'document',
            delay: 100,
            opacity: 0.5
        });
        $(this.$el).on("hide.bs.modal", () => {
            this.bShow = false;
        }).on("show.bs.modal", () => {
            this.bShow = true;
        })
    },
    components: { LivePlayer },
    computed: {
        ...mapState(['userInfo', 'serverInfo']),
    },
    methods: {
        play(src, title, snap, osd) {
            this.videoUrl = src || "";
            this.videoTitle = title || "";
            this.snapUrl = snap || "";
            this.osd = osd || "";

            $(this.$el).modal("show");
        }
    }
}
</script>

<style lang="less" scoped>
.modal-title {
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
</style>
