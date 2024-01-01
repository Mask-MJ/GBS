<template>
  <el-dialog
    class="ScreenHistoryDlg"
    title="回放录像"
    :visible="visible"
    top="3vh"
    width="50%"
  >
    <div class="box box-primary records">
      <div class="box-body">
        <div class="form-inline" autocomplete="off" spellcheck="false">
          <div class="form-group">
            <span>&nbsp;&nbsp;</span>
            <button
              type="button"
              class="btn btn-default btn-sm"
              @click.prevent="showTimeRangeDlg"
              v-if="!isDemoUser(serverInfo, userInfo) && videos.length > 0"
              v-loading.fullscreen.lock="bDownloading"
              element-loading-text="合成中..."
            >
              <i class="fa fa-download"></i> 下载
            </button>
          </div>
          <div class="form-group-line">
            <h4 class="text-primary text-center">
              云端录像({{ name }})-时间轴视图
            </h4>
            <div class="input-group input-group-sm">
              <CloudRecordDatePicker
                class="form-control"
                :day="cloundData.day"
                @update:day="updateDay"
                ref="datePicker"
                :serial="cloundData.serial"
                :code="cloundData.code"
              ></CloudRecordDatePicker>
              <div class="input-group-btn">
                <button
                  type="button"
                  class="btn btn-sm btn-default"
                  @click.prevent="showCloudRecordDatePicker"
                >
                  <i class="fa fa-calendar"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
        <br />
        <div class="clearfix"></div>
        <LivePlayer
          :videoUrl="videoUrl"
          :waterMark="osd"
          muted
          :hideBigPlayButton="!!serverInfo.HideBigPlayButton"
          :currentTime="currentTime"
          @media_info="onMediaInfo"
          @error="onVideoError"
          @play="onVideoPlay"
          @ended="onVideoEnd"
          @timeupdate="onVideoTimeUpdate"
          v-loading="loading"
          :loading.sync="loading"
          element-loading-text="加载中..."
          element-loading-background="#000"
        >
        </LivePlayer>
        <div class="text-center text-red" v-if="bOutHevcTip">
          <br />
          提示: 正在播放 H265 FLV 直出流, 确保浏览器版本较新, 并且开启硬件加速
        </div>
        <br />
        <CloudRecordTimeRule
          :videos="videos"
          @timeChange="onTimeChange"
          ref="timeRule"
          v-loading="loadingRecords"
          :minutesPerUnit="serverInfo.MinutesPerUnit || 5"
        ></CloudRecordTimeRule>
        <br />
      </div>
      <TimeRangeDlg
        ref="timeRangeDlg"
        format="HHmmss"
        @submit="onTimeRangeChange"
      ></TimeRangeDlg>
    </div>

    <span slot="footer" class="dialog-footer">
      <el-button @click="handleShowHistoryCard">取 消</el-button>
      <el-button type="primary" @click="handleShowHistoryCard">确 定</el-button>
    </span>
  </el-dialog>
</template>

<script>
import moment from "moment";
import CloudRecordDatePicker from "components/CloudRecordDatePicker.vue";
import CloudRecordTimeRule from "components/CloudRecordTimeRule.vue";
import TimeRangeDlg from "components/TimeRangeDlg.vue";
import LivePlayer from "@liveqing/liveplayer";
import { mapState } from "vuex";

export default {
  components: {
    CloudRecordDatePicker,
    LivePlayer,
    CloudRecordTimeRule,
    TimeRangeDlg,
  },
  emits: ["handleShowHistoryCard"],
  props: {
    visible: { type: Boolean, default: false },
    activeHistoryCardData: { type: Object, default: () => ({}) },
  },
  data() {
    return {
      videos: [],
      videoUrl: "",
      currentTime: 0,
      bActive: false,
      loading: false,
      loadingRecords: false,
      name: "",
      osd: "",
      mediaInfo: null,
      bOutHevcTip: false,
      bDownloading: false,
    };
  },
  computed: {
    ...mapState(["userInfo", "serverInfo"]),
    videoUrls() {
      return this.videos.map((val, index, array) => {
        return (val || {}).hls;
      });
    },
    cloundData() {
      return {
        serial: this.activeHistoryCardData.serial,
        code: this.activeHistoryCardData.code,
        // 获取当前日期 格式为 YYYYMMDD
        date: moment().format("YYYYMMDD"),
      };
    },
  },
  methods: {
    handleShowHistoryCard() {
      this.$emit("handleShowHistoryCard");
    },
    updateDay(day) {
      this.$router.replace(
        `/cloudrecord/timeview/${this.cloundData.serial}/${this.cloundData.code}/${day}`
      );
    },
    showCloudRecordDatePicker() {
      $(this.$refs["datePicker"].$el).focus();
    },
    updateVideos() {
      this.loadingRecords = true;
      $.get("/api/v1/cloudrecord/querydaily", {
        serial: this.cloundData.serial,
        code: this.cloundData.code,
        period: this.cloundData.day,
      })
        .then((data) => {
          this.name =
            data.name || `${this.cloundData.serial}:${this.cloundData.code}`;
          this.osd = data.osd || "";
          this.videos = data.list;
        })
        .always(() => {
          this.$nextTick(() => {
            this.loadingRecords = false;
          });
        });
    },
    onTimeChange(video) {
      this.videoUrl = (video || {}).hls || "";
      this.currentTime = (video || {}).currentTime || 0;
    },
    onMediaInfo(mi) {
      this.mediaInfo = mi;
    },
    onVideoError(e) {
      if (
        e == "MediaError" &&
        this.mediaInfo &&
        String(this.mediaInfo.videoCodec).startsWith("hvc")
      ) {
        this.bOutHevcTip = true;
        console.log(
          "提示: 正在播放 H265 FLV 直出流, 确保浏览器版本较新, 并且开启硬件加速"
        );
        return;
      }
    },
    onVideoPlay(t) {
      this.bOutHevcTip = false;
    },
    onVideoEnd() {
      var idx = this.videoUrls.indexOf(this.videoUrl);
      if (idx >= 0 && idx < this.videos.length - 1) {
        this.mediaInfo = null;
        this.bOutHevcTip = false;
        var nextVideo = this.videos[idx + 1];
        if (!nextVideo) return;
        var startTime = moment(nextVideo.startAt, "YYYYMMDDHHmmss");
        var n = startTime.diff(startTime.clone().startOf("day"), "minutes");
        // this.$refs["timeRule"].clickMinute(n);
      }
    },
    onVideoTimeUpdate(currentTime) {
      if (currentTime < 1) return;
      var idx = this.videoUrls.indexOf(this.videoUrl);
      if (idx >= 0 && idx < this.videos.length) {
        var video = this.videos[idx];
        var startTime = moment(video.startAt, "YYYYMMDDHHmmss");
        startTime.add(currentTime, "seconds");
        var n = startTime.diff(startTime.clone().startOf("day"), "minutes");
        // this.$refs["timeRule"].clickMinute(n, false);
      }
    },
    showTimeRangeDlg() {
      var start = moment()
        .second(0)
        .subtract(30, "minutes");
      var end = moment().second(0);
      if (!start.isSame(end, "day")) {
        start = end.clone().startOf("day");
      }
      var dlg = this.$refs["timeRangeDlg"];
      dlg.setDefault([start.toDate(), end.toDate()]);
      dlg.show();
    },
    onTimeRangeChange(v) {
      if (v && v.length == 2) {
        var start = `${this.cloundData.day}${v[0]}`;
        var end = `${this.cloundData.day}${v[1]}`;
        var t1 = moment(start, "YYYYMMDDHHmmss");
        var t2 = moment(end, "YYYYMMDDHHmmss");
        if (t1.isBefore(t2)) {
          var found = false;
          for (
            var i = t1.clone();
            i.isSameOrBefore(t2);
            i = i.add(1, "minute")
          ) {
            var secs = i.unix();
            secs = secs - (secs % ((this.serverInfo.MinutesPerUnit || 5) * 60)); // floor to unit
            var itext = moment.unix(secs).format("HH:mm");
            // if (this.$refs["timeRule"].activeMinutes[itext]) {
            //   found = true;
            //   break;
            // }
          }
          if (!found) {
            this.$message({
              type: "error",
              message: `${t1.format("HH:mm:ss")} - ${t2.format(
                "HH:mm:ss"
              )} 没有录像`,
            });
            return;
          }
          this.$refs["timeRangeDlg"].hide();
          this.bDownloading = true;
          var url = `/api/v1/cloudrecord/video/download/${this.cloundData.serial}/${this.cloundData.code}/${start}/${end}/video.mp4`;
          $.get(url)
            .then((ret) => {
              window.location.href = ret;
            })
            .always(() => {
              this.bDownloading = false;
            });
          return;
        }
      }
      this.$message({
        type: "error",
        message: `无效的时间段`,
      });
    },
  },
  mounted() {
    let mmt = moment();
    let n = mmt.diff(mmt.clone().startOf("day"), "minutes");
    n -= 10;
    if (n < 0) n = 0;
    // this.$refs.timeRule.clickMinute(n);
    this.updateVideos();
  },
  watch: {
    visible(val) {
      if (val) {
        console.log(this.cloundData);
      }
    },
  },
};
</script>

<style lang="less" scoped>
.box {
  margin-bottom: 0px;
}
.form-group-line {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
<style lang="less">
.ScreenHistoryDlg {
  .el-dialog__body {
    padding: 5px 10px;
  }
}
</style>
