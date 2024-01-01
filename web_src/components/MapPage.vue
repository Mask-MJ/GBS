<template>
<div id="wrapper" v-loading="loading">
  <div id="map">
    <div class="form-group has-feedback search" v-show="bShowSearch">
        <input type="text" name="q" class="form-control" v-model.trim="q" autocomplete="off" placeholder="搜索" @mousedown.stop @touchstart.stop @dblclick.stop @contextmenu.stop>
        <span class="glyphicon glyphicon-search form-control-feedback text-gray"></span>
    </div>
  </div>
  <SimpleVideoDlg ref="videoDlg" id="video-dlg" :talk="talk" :protocol="protocol" :ptz="ptz" :token="token"
    :fluent="fluent" :stretch="stretch" :autoplay="autoplay" :controls="controls" :snap="snap" :custom="custom" :muted="muted">
  </SimpleVideoDlg>
  <el-dialog :title="msgDlgTitle" :visible.sync="bMsgDlg" :modal-append-to-body="false" :lock-scroll="false" width="450px" top="30vh">
    <span v-html="msgDlgText"></span>
    <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="copyMsgText" size="small">一键复制</el-button>
    </span>
  </el-dialog>
</div>
</template>

<script>
import "font-awesome/css/font-awesome.css"
import "bootstrap/dist/css/bootstrap.css"
import "admin-lte/dist/css/skins/_all-skins.css"
import "assets/styles/AdminLTE-custom.less"
import "assets/styles/custom.less"

import "bootstrap/dist/js/bootstrap.js"
import "leaflet/dist/leaflet.css"
import "@penggy/leaflet.fullscreen/Control.FullScreen.css"
import "@penggy/leaflet-contextmenu/dist/leaflet.contextmenu.css"
import "leaflet"
import "leaflet-polylinedecorator"
import "@penggy/leaflet.fullscreen"
import "@penggy/leaflet-contextmenu/dist/leaflet.contextmenu.js"

import { mapState, mapActions } from "vuex"
import Vue from "vue"

import ElementUI from "element-ui"
import 'assets/styles/element-custom.scss'
Vue.use(ElementUI);

Vue.prototype.$updateQueryString = (uri, key, value) => {
  var re = new RegExp("([?&])" + key + "=.*?(&|$)", "i");
  var separator = uri.indexOf('?') !== -1 ? "&" : "?";
  if (uri.match(re)) {
    return uri.replace(re, '$1' + key + "=" + value + '$2');
  } else {
    return uri + separator + key + "=" + value;
  }
}
Vue.prototype.$getQueryString = (name, defVal = "") => {
  var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
  var r = window.location.search.substr(1).match(reg);
  if (r != null) {
    return decodeURIComponent(r[2]);
  }
  return defVal;
}
Vue.prototype.$iframe = (link, w, h) => {
  var _link = Vue.prototype.$updateQueryString(link, "aspect", "fullscreen")
  return `<iframe src="${_link}" width="${w}" height="${h}" allowfullscreen allow="autoplay; fullscreen"></iframe>`
}
Vue.prototype.isMobile = () => {
  return videojs.browser.IS_IOS || videojs.browser.IS_ANDROID;
}
Vue.prototype.isIE = () => {
  return !!videojs.browser.IE_VERSION;
}
Vue.prototype.flvSupported = () => {
  return videojs.browser.IE_VERSION || (flvjs.getFeatureList() && flvjs.getFeatureList().mseLiveFlvPlayback);
}
Vue.prototype.rtcSupported = () => {
    return !!window.RTCPeerConnection;
}
Vue.prototype.canTalk = () => {
  return location.protocol.indexOf("https") == 0 || location.hostname === 'localhost' || location.hostname === '127.0.0.1';
}
Vue.prototype.hasAnyRole = (serverInfo, userInfo, ...roles) => {
    // toggle share on, no need login
    // if (serverInfo && serverInfo.APIAuth === false && !userInfo) {
    if (!userInfo) {
        return true;
    }
    var userRoles = [];
    if (userInfo) {
        userRoles = userInfo.Roles || [];
    }
    var checked = false;
    for(var role of (roles||[])) {
        if (!role || userRoles.some(ur => (ur == role || ur == '超级管理员'))) {
            checked = true;
            break;
        }
    }
    return checked;
}
Vue.prototype.isDemoUser = (serverInfo, userInfo) => {
  if (serverInfo && userInfo && userInfo.Name === serverInfo.DemoUser) return true;
  return false;
}

import SimpleVideoDlg from "components/SimpleVideoDlg.vue";
import $ from "jquery"
import _ from "lodash"
$.ajaxSetup({ cache: false });

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
      serial: "",
      code: "",
      protocol: "",
      fluent: true,
      stretch: false,
      autoplay: true,
      controls: true,
      snap: undefined,
      custom: true,
      muted: true,
      ptz: true,
      talk: false,
      trace: false,
      search: true,
      bShowSearch: false,
      loading: false,
      token: "",
      otherParams: "",
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
      channel: null,
      traceLine: null,
      traceLineDec: null,
      msgDlgTitle: "提示",
      msgDlgText: "",
      bMsgDlg: false,
    };
  },
  beforeDestroy() {
    if(this.map) {
      this.map.remove();
      this.map = null;
    }
  },
  created(){
    this.muted = this.$getQueryString("muted", "yes") == "yes";
    this.autoplay = this.$getQueryString("autoplay", "yes") == "yes";
    this.controls = this.$getQueryString("controls", "yes") == "yes";
    var snap = this.$getQueryString("snap", "");
    if(snap) {
      this.snap = (snap == "yes");
    }
    this.custom = this.$getQueryString("custom", "yes") == "yes";
    this.fluent = this.$getQueryString("fluent", "yes") == "yes";
    this.stretch = this.$getQueryString("stretch", "no") == "yes";
    this.token = this.$getQueryString("token","");
  },
  async mounted() {
    var serverInfo = await this.getServerInfo();
    if (serverInfo) {
      document.title = serverInfo.LogoText || "LiveGBS";
    }
    await this.getUserInfo({
      token: this.token,
    });
    this.serial = this.$getQueryString("serial", "");
    this.code = this.$getQueryString("code", "");
    this.protocol = this.$getQueryString("protocol", serverInfo.PreferStreamFmt||"");
    this.ptz = this.$getQueryString("ptz", "yes") == "yes";
    this.talk = this.$getQueryString("talk", "no") == "yes";
    this.trace = this.$getQueryString("trace", "no") == "yes";
    this.search = this.$getQueryString("search", "yes") == "yes";
    this.otherParams = this.getOtherParams(["serial", "code", "protocol", "ptz", "talk", "trace", "search", "autoplay", "controls", "snap", "custom", "fluent", "stretch", "muted"]);
    $(document).ajaxError((evt, xhr, opts, ex) => {
      if (xhr.status == 401) {
        if (this.serverInfo.DemoUser) {
          location.href = `/login?r=${encodeURIComponent(location.href)}`;
        } else {
          console.log("登录认证过期");
        }
        return false;
      } else if (xhr.status) {
        let msg = xhr.responseText || "网络请求失败";
        if (xhr.status == 404) {
          msg = "请求服务不存在或已停止";
        } else if (xhr.status == 504) {
          msg = "Gateway Timeout";
        }
        try {
          msg = JSON.parse(msg)
        } catch (error) {}
        console.log(msg);
      } else if(xhr) {
        console.log(`ajax error: ${xhr.status} ${xhr.responseText}`);
      }
    })

    var mapInfo = serverInfo.MapInfo || {};
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
    this.bShowSearch = this.search && this.zoom >= this.markerZoom;
    this.map.on('move', e => {
      if(this.map.getZoom() < this.markerZoom) {
        this.bShowSearch = false;
        if(this.channels.length > 0) {
          this.channels = [];
          this.refreshChannelMarkers();
        }
        return
      }
      this.bShowSearch = this.search;
      this.doDelaySearch();
    }).whenReady(() => {
      if(this.map.getZoom() >= this.markerZoom) {
        this.getChannels();
      }
      this.getChannelInfo();
    });
  }, //mounted
  computed: {
    ...mapState(["serverInfo", "userInfo"]),
  },
  watch: {
    q: function(newVal, oldVal) {
        this.doDelaySearch();
    }
  },
  methods: {
    ...mapActions(["getServerInfo", "getUserInfo"]),
    getOtherParams(without) {
      var url = location.search;
      var theRequest = "";
      if (url.indexOf("?") != -1) {
        var str = url.substr(1);
        var strs = str.split("&");
        for (var i = 0; i < strs.length; i++) {
          if (without.indexOf(strs[i].split("=")[0]) == -1) {
            if (theRequest == "") {
              theRequest = strs[i];
            } else {
              theRequest += "&" + strs[i];
            }
          }
        }
      }
      return theRequest;
    },
    doDelaySearch: _.debounce(function(){
      this.getChannels();
    }, 800),
    getChannelInfo() {
      if(!this.serial || !this.code) return
      $.get("/api/v1/device/channelinfo", {
        serial: this.serial,
        code: this.code,
        token: this.token,
      }).then(channel => {
        this.channel = channel;
        var lat = channel.CustomLatitude || channel.Latitude;
        var lng = channel.CustomLongitude || channel.Longitude;
        if(!this.trace) {
          this.map && this.map.setView([lat, lng], this.maxZoom);
          return
        }
        $.get("/api/v1/device/positionlog", {
          serial: this.serial,
          code: this.code,
          token: this.token,
          sort: "CreatedAt",
          order: "asc",
        }).then(ret => {
          var latlngs = [];
          for(var log of ret.LogList) {
            latlngs.push([log.Latitude, log.Longitude])
          }
          if(latlngs.length > 0) {
            this.traceLine = L.polyline(latlngs, {weight: 8, color: 'red'});
            this.traceLineDec = L.polylineDecorator(this.traceLine, {
              patterns: [{
                offset: 50,
                repeat: 50,
                symbol: L.Symbol.arrowHead({
                  pixelSize: 5,
                  headAngle: 75,
                  polygon: false,
                  pathOptions: {
                    stroke: true,
                    weight: 2,
                    color: 'white',
                  }
                })
              }]
            });
          }
        }).always(() => {
          this.map && this.map.setView([lat, lng], this.maxZoom);
        })
      })
    },
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
        token: this.token,
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
            if(this.serial == marker.Serial && this.code == marker.Code && this.map.getZoom() == this.maxZoom) {
              if(this.traceLine) this.traceLine.addTo(this.map);
              if(this.traceLineDec) this.traceLineDec.addTo(this.map);
            }
            return true;
          }
        }
        if(this.serial == marker.Serial && this.code == marker.Code) {
          if(this.map.getZoom() < this.maxZoom) {
            if(this.traceLine) this.traceLine.remove();
            if(this.traceLineDec) this.traceLineDec.remove();
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
        if(this.serial == channel.DeviceID && this.code == channel.ID) {
          m.setIcon(channel.Status == "ON" ? camera_focus_on : camera_focus_off);
          // if(m.PTZType === 1) {
          //   m.setIcon(channel.Status == "ON" ? dome_focus_on : dome_focus_off);
          // }
          m.setZIndexOffset(999);
          if(this.traceLine) this.traceLine.addTo(this.map);
          if(this.traceLineDec) this.traceLineDec.addTo(this.map);
        }
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
          token: this.token,
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
};
</script>

<style lang="less">
html,body {
  height: 100%;
  margin: 0;
  padding: 0;
}
</style>
<style lang="less" scoped>
#wrapper {
  width: 100%;
  height: 100%;

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
