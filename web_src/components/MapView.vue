<template>
<div id="wrapper" v-loading="loading" :style="`height:${this.pageHeight}px;min-height:500px;`">
  <div id="map">
    <div class="form-group has-feedback search" v-show="bShowSearch">
        <input type="text" name="q" class="form-control" v-model.trim="q" autocomplete="off" placeholder="搜索" @mousedown.stop @touchstart.stop @dblclick.stop @contextmenu.stop>
        <span class="glyphicon glyphicon-search form-control-feedback text-gray"></span>
    </div>
  </div>
  <SimpleVideoDlg ref="videoDlg" id="video-dlg" ptz talk></SimpleVideoDlg>
  <el-dialog :title="msgDlgTitle" :visible.sync="bMsgDlg" :modal-append-to-body="false" :lock-scroll="false" width="450px" top="30vh">
    <span v-html="msgDlgText"></span>
    <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="copyMsgText" size="small">一键复制</el-button>
    </span>
  </el-dialog>
</div>
</template>

<script>
import { mapState } from "vuex"
import SimpleVideoDlg from "components/SimpleVideoDlg.vue"
import $ from "jquery"
import _ from "lodash"

var camera_on = L.icon({
  iconUrl: "/images/camera-on.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var camera_off = L.icon({
  iconUrl: "/images/camera-off.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var camera_focus_on = L.icon({
  iconUrl: "/images/camera-red-on.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var camera_focus_off = L.icon({
  iconUrl: "/images/camera-red-off.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var dome_on = L.icon({
  iconUrl: "/images/dome-on.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var dome_off = L.icon({
  iconUrl: "/images/dome-off.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var dome_focus_on = L.icon({
  iconUrl: "/images/dome-red-on.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var dome_focus_off = L.icon({
  iconUrl: "/images/dome-red-off.png",
  iconSize: [30, 30],
  iconAnchor: [10, 30],
})
var defaultCenter = [31.82, 117.22];
var defaultZoom = 8;
var defaultMinZoom = 8;
var defaultMaxZoom = 9;
var defaultMarkerZoom = 9;
var defaultAttribution = "";
var defaultAttributionPrefix = "<a href='//www.liveqing.com' target='_blank'>LiveQing</a>";
export default {
    components: { SimpleVideoDlg },
    data() {
        return {
            q: "",
            pageHeight: 0,
            loading: false,
            bShowSearch: false,
            map: null,
            center: defaultCenter,
            zoom: defaultZoom,
            minZoom: defaultMinZoom,
            maxZoom: defaultMaxZoom,
            markerZoom: defaultMarkerZoom,
            attribution: defaultAttribution,
            attributionPrefix: defaultAttributionPrefix,
            channels: [],
            channelMarkers: [],
            msgDlgTitle: "提示",
            msgDlgText: "",
            bMsgDlg: false,
        }
    },
    beforeDestroy() {
        if(this.map) {
            this.map.remove();
            this.map = null;
        }
        $(window).off('resize', this.initHeight);
    },
    created() {
        this.initHeight();
    },
    mounted() {
        var mapInfo = this.serverInfo.MapInfo || {};
        if(mapInfo.Center && mapInfo.Center.length >= 2) {
            this.center = [mapInfo.Center[1], mapInfo.Center[0]];
        }
        this.zoom = mapInfo.Zoom || defaultZoom;
        this.minZoom = mapInfo.MinZoom || defaultMinZoom;
        this.maxZoom = mapInfo.MaxZoom || defaultMaxZoom;
        this.markerZoom = mapInfo.MarkerZoom || defaultMarkerZoom;
        this.attribution = mapInfo.Attribution || defaultAttribution;
        this.attributionPrefix = mapInfo.AttributionPrefix || defaultAttributionPrefix;
        this.map = L.map("map", {
            fullscreenControl: true,
            fullscreenControlOptions: {
                fullscreenElement: this.$el,
            },
            contextmenu: true,
            contextmenuItems: [{
                text: "显示坐标",
                callback: this.showCoordinates,
            }],
            center: this.center,
            zoom: this.zoom,
            attributionControl: !!this.attribution,
        });
        L.tileLayer("/map/{z}/{x}/{y}.png", {
            minZoom: this.minZoom,
            maxZoom: this.maxZoom,
            attribution: this.attribution,
        }).addTo(this.map);
        if (this.map.attributionControl && this.attributionPrefix) {
            this.map.attributionControl.setPrefix(this.attributionPrefix);
        }
        this.bShowSearch = this.zoom >= this.markerZoom;
        this.map.on('move', e => {
            if(this.map.getZoom() < this.markerZoom) {
                this.bShowSearch = false;
                if(this.channels.length > 0) {
                    this.channels = [];
                    this.refreshChannelMarkers();
                }
                return
            }
            this.bShowSearch = true;
            this.doDelaySearch();
        }).whenReady(() => {
            if(this.map.getZoom() >= this.markerZoom) {
                this.getChannels();
            }
        })
        $(window).on('resize', this.initHeight);
    },
    computed: {
        ...mapState(["serverInfo", "userInfo"]),
    },
    watch: {
        q: function(newVal, oldVal) {
            this.doDelaySearch();
        }
    },
    methods: {
        doDelaySearch: _.debounce(function(){
            this.getChannels();
        }, 800),
        getChannels() {
            if(!this.map) return;
            if(this.map.getZoom() < this.markerZoom) {
                if(this.channels.length > 0) {
                    this.channels = [];
                    this.refreshChannelMarkers();
                }
                return
            }
            $.get("/api/v1/device/channellist", {
                channel_type: "device",
                bounds: this.map.getBounds().toBBoxString(),
                q: this.q,
            }).then(ret => {
                this.channels = ret.ChannelList || [];
                this.refreshChannelMarkers();
            })
        },
        refreshChannelMarkers() {
            if(!this.map) return;
            var channelMap = this.channels.reduce((pval, channel) => {
                var key = `${channel.DeviceID}_${channel.ID}`;
                pval[key] = channel;
                return pval;
            }, {})
            this.channelMarkers = this.channelMarkers.filter(marker => {
                var channel = channelMap[marker.ID];
                if(channel) {
                    var lat = channel.CustomLatitude||channel.Latitude;
                    var lng = channel.CustomLongitude||channel.Longitude;
                    if(marker.Lat == lat && marker.Lng == lng && marker.Status == channel.Status) {
                        delete channelMap[marker.ID];
                        return true;
                    }
                }
                marker.remove();
                return false;
            })
            for(var key in channelMap) {
                var channel = channelMap[key];
                var lat = channel.CustomLatitude||channel.Latitude;
                var lng = channel.CustomLongitude||channel.Longitude;
                var m = L.marker([lat, lng], {
                    icon: channel.Status == "ON" ? camera_on : camera_off,
                    title: `${channel.CustomName || channel.Name} | ${channel.Status == "ON" ? "在线" : "离线"}`,
                    contextmenu: true,
                    contextmenuItems: [{
                        text: "设备信息",
                        callback: this.showChannelInfo,
                    }],
                });
                m.ID = key;
                m.Lat = lat;
                m.Lng = lng;
                m.Serial = channel.DeviceID;
                m.Code = channel.ID;
                m.Status = channel.Status;
                m.ChannelInfo = channel;
                m.Title = channel.CustomName || channel.Name || channel.Code;
                m.PTZType = channel.CustomPTZType || channel.PTZType || 0;
                // if(m.PTZType === 1) {
                //   m.setIcon(channel.Status == "ON" ? dome_on : dome_off);
                // }
                m.SnapURL = channel.SnapURL || "";
                if(m.SnapURL) {
                    m.bindTooltip(`<img src="${m.SnapURL}" width="288" height="162" onerror="this.src='/images/default_snap.png';"/>`, {
                        direction: 'top',
                        offset: [0, -20],
                        opacity: 1,
                        className: "tooltip-snap",
                        sticky: true,
                    })
                }
                if(channel.Status == "ON") {
                    m.on('click', this.onChannelMarkerClick);
                } else {
                    m.on('dblclick', this.onChannelMarkerClick);
                }
                m.addTo(this.map);
                this.channelMarkers.push(m);
            }
        },
        onChannelMarkerClick(e) {
            var marker = e.sourceTarget;
            if(!marker) return;
            var channel = marker.ChannelInfo;
            if(!channel) return;
            this.loading = true;
            $.ajax({
                method: "GET",
                url: "/api/v1/stream/start",
                global: false,
                data: {
                    serial: channel.DeviceID,
                    code: channel.ID,
                }
            }).then(streamInfo => {
                this.$refs["videoDlg"].play(channel.DeviceID, channel.ID, streamInfo);
            }).fail(xhr => {
                var msg = "加载视频失败";
                if(channel.Status != "ON") {
                    msg = "设备离线";
                } else if(xhr) {
                    msg = `${xhr.status} ${xhr.responseText}`;
                }
                this.$refs["videoDlg"].play(channel.DeviceID, channel.ID, {
                    ChannelName: channel.CustomName || channel.Name,
                    Message: msg,
                })
            }).always(() => {
                this.loading = false;
            })
        },
        initHeight() {
            this.pageHeight = window.innerHeight;
            if (typeof this.pageWidth != "number") {
                if (document.compatMode == "CSS1Compat") {
                    this.pageHeight = document.documentElement.clientHeight;
                } else {
                    this.pageHeight = document.body.clientHeight;
                }
            }
            this.pageHeight = this.pageHeight - 91;
        },
        showCoordinates(e) {
            this.showMsgDlg("坐标(经,纬)", `${parseFloat(e.latlng.lng.toFixed(5))},${parseFloat(e.latlng.lat.toFixed(5))}`);
        },
        showChannelInfo(e) {
            var m = e.relatedTarget;
            var msg =  `编号: ${m.ID}<br>`;
            msg += `名称: ${m.Title}<br>`;
            msg += `经纬: ${parseFloat(m.Lng.toFixed(5))},${parseFloat(m.Lat.toFixed(5))}`;
            this.showMsgDlg("设备信息", msg);
        },
        showMsgDlg(title, text) {
            this.msgDlgTitle = title;
            this.msgDlgText = text;
            this.bMsgDlg = true;
        },
        copyMsgText() {
            this.copy(this.msgDlgText.replaceAll("<br>", "\r\n"));
            this.bMsgDlg = false;
        },
        centerHere(e) {
            this.map.panTo(e.latlng);
        },
        async copy(text) {
            try {
                let textArea = document.createElement("textarea");
                textArea.value = text;
                textArea.style.position = "absolute";
                textArea.style.opacity = 0;
                textArea.style.left = "-999999px";
                textArea.style.top = "-999999px";
                this.$el.appendChild(textArea);
                textArea.focus();
                textArea.select();
                await new Promise((res, rej) => {
                    document.execCommand('copy') ? res() : rej();
                    textArea.remove();
                });
                this.$message.success('复制成功！');
            } catch (err) {
                try {
                    await navigator.clipboard.writeText(text);
                    this.$message.success('复制成功！！');
                } catch (errr) {
                    console.log(err, errr);
                    this.$message.error('复制失败', err, errr);
                }
            }
        },
    }, // methods
}
</script>

<style lang="less" scoped>
#wrapper {
  width: 100%;

  #map {
    width: 100%;
    height: 100%;

    .form-group.search {
        position: absolute;
        z-index: 1000;
        top: 15px;
        right: 15px;
        width: 200px;
    }
  }
}
</style>
