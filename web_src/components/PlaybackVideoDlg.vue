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
                  <LivePlayer v-if="bShow" :videoUrl="videoUrl" :waterMark="osd" :live="live" hideLiveText muted
                    :snapUrl="snapUrl" :hideBigPlayButton="!!serverInfo.HideBigPlayButton"
                    @media_info="onMediaInfo" @error="onError" @play="onPlay" @message="$message"
                    v-loading="bLoading" :loading.sync="bLoading" element-loading-text="加载中..." element-loading-background="#000">
                  </LivePlayer>
                  <div class="text-center" v-if="isDemoUser(serverInfo, userInfo) && !bOutHevcTip">
                    <br>
                    提示: 演示系统限制匿名登录播放时间, 若需测试长时间播放, 请<a target="_blank" href="//www.liveqing.com/docs/download/LiveGBS.html">下载使用</a>
                  </div>
                  <div class="text-center text-red" v-if="bOutHevcTip">
                    <br>
                    提示: 正在播放 H265 FLV 直出流, 确保浏览器版本较新, 并且开启硬件加速
                  </div>
                </div>
                <div class="modal-footer">
                  <button type="button" class="btn btn-primary" @click.prevent="scale(0.5)">0.5x</button>
                  <button type="button" class="btn btn-primary" @click.prevent="scale(1)">1x</button>
                  <button type="button" class="btn btn-primary" @click.prevent="scale(2)">2x</button>
                  <button type="button" class="btn btn-primary" @click.prevent="scale(4)">4x</button>
                  <button type="button" class="btn btn-default" data-dismiss="modal">关闭</button>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import "jquery-ui/ui/widgets/draggable";
import LivePlayer from "@liveqing/liveplayer";
import { mapState } from "vuex";

export default {
  data() {
    return {
      videoUrl: "",
      videoTitle: "",
      osd: "",
      snapUrl: "",
      serial: "",
      code: "",
      streamID: "",
      bShow: false,
      bLoading: false,
      mediaInfo: null,
      bOutHevcTip: false,
    };
  },
  props: {
    live: {
      type: Boolean,
      default: false
    },
    fade: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    ...mapState(['userInfo', 'serverInfo']),
  },
  mounted() {
    $(this.$el).find(".modal-content").draggable({
      handle: ".modal-header",
      cancel: ".modal-title span",
      addClasses: false,
      containment: "document",
      delay: 100,
      opacity: 0.5
    });
    $(this.$el).on("hide.bs.modal", () => {
      this.bShow = false;
      this.snapUrl = "";
      this.videoUrl = "";
      this.osd = "";
      this.mediaInfo = null;
      this.bOutHevcTip = false;
      if(this.streamID) {
        $.get("/api/v1/playback/stop", {
          streamid: this.streamID
        }).always(() => {
          this.streamID = "";
          this.$emit("close");
        })
      }
    }).on("show.bs.modal", () => {
      this.bShow = true;
    });
  },
  components: { LivePlayer },
  methods: {
    play(src, title, snap, serial, code, streamID, osd) {
      this.videoUrl = src || ""; // no need in next tick since player@2.6.9
      this.videoTitle = title || "";
      this.snapUrl = snap || "";
      this.serial = serial || "";
      this.code = code || "";
      this.streamID = streamID || "";
      this.osd = osd || "";
      $(this.$el).modal("show");
    },
    scale(speed = 1) {
      if(!this.streamID) return;
      $.get("/api/v1/playback/control", {
        streamid: this.streamID,
        command: "scale",
        scale: speed,
      }).then(data => {
        this.$message({
          type: "success",
          message: `${speed} 倍速设置成功`,
        });
      })
    },
    onMediaInfo(mi) {
      this.mediaInfo = mi;
    },
    onError(e) {
      if(e == 'MediaError' && this.mediaInfo && String(this.mediaInfo.videoCodec).startsWith("hvc")) {
        this.bOutHevcTip = true;
        console.log("提示: 正在播放 H265 FLV 直出流, 确保浏览器版本较新, 并且开启硬件加速");
      }
    },
    onPlay(t) {
      this.bOutHevcTip = false;
    },
  }
};
</script>

<style lang="less" scoped>
.modal-title {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
</style>
