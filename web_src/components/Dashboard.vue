<template>
<section class="content">
<div class="qcontent">
    <div class="qcontent-header hidden-xs">服务器概览 <button type="button" class="btn btn-xs btn-default full-btn" @click.prevent="fullscreen" title="点击大屏展示"><i class="fa fa-arrows-alt"></i></button></div>
    <div class="container-fluid" :style="`height:${this.pageHeight}px;min-height:500px;`">
        <div class="col-xs-12 col-md-4">
            <div class="view-dashboard view-left">
                <div class="panel">
                    <div class="panel-body">
                        <div class="col-md-12 panel-title">
                            通道信息
                        </div>
                        <div class="col-xs-6 auth-view" style="text-align:center;">
                            <el-progress color="#337ab7" type="circle" :height="this.authViewHeight" :width="this.authViewHeight" :percentage="authData.DeviceTotal == 0 ? 0:  parseFloat((authData.DeviceOnline/authData.DeviceTotal*100).toFixed(2))"></el-progress>
                            <div class="auth-title">在线数:<span class="auth-num">{{authData.DeviceOnline}}</span><br>总设备:<span class="auth-num">{{authData.DeviceTotal}}</span></div>
                        </div>
                        <div class="col-xs-6 auth-view" style="text-align:center">
                            <el-progress color="#337ab7" type="circle" :height="this.authViewHeight" :width="this.authViewHeight" :percentage="authData.ChannelTotal == 0 ? 0:  parseFloat((authData.ChannelOnline/authData.ChannelTotal*100).toFixed(2))"></el-progress>
                            <div class="auth-title">在线数:<span class="auth-num">{{authData.ChannelOnline}}</span><br>总通道:<span class="auth-num">{{authData.ChannelTotal}}</span></div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="view-dashboard  view-left view-split">
                <div class="panel">
                    <div class="panel-body">
                        <ve-bar ref="storeChart" height="100%" :colors="chartColors" :data="storeData" :settings="storeSettings" :extend="storeExtend" legend-position="bottom" :title="{text:'存储使用', left: 'center'}"></ve-bar>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-xs-12 col-md-4">
            <div class="view-dashboard view-split-m">
                <div class="panel">
                    <div class="panel-body">
                        <ve-histogram ref="loadChart" height="100%" :colors="chartColors" :data="loadData" :settings="loadSettings" :events="loadEvents" :extend="loadExtend" :legend-visible="false" :title="{text:'负载信息', left: 'center'}"></ve-histogram>
                    </div>
                </div>
            </div>
            <div class="view-dashboard view-split">
                <div class="panel">
                    <div class="panel-body">
                        <ve-line ref="netChart" height="100%" :colors="chartColors" :data="netData" :settings="netSettings" legend-position="bottom" :legend-visible="true" :title="{text:'带宽使用(Mbps)', left: 'center'}"></ve-line>
                    </div>
                </div>
            </div>
        </div>
        <div class="col-xs-12 col-md-4">
            <div class="view-dashboard view-right view-split-m">
                <div class="panel">
                    <div class="panel-body">
                        <ve-line ref="cpuChart" height="100%" :colors="chartColors" :data="cpuData" :settings="memSettings" :legend-visible="false" :title="{text:'CPU使用', left: 'center'}"></ve-line>
                    </div>
                </div>
            </div>
            <div class="view-dashboard view-right view-split">
                <div class="panel">
                    <div class="panel-body">
                        <ve-line ref="memChart" height="100%" :colors="chartColors" :data="memData" :settings="memSettings" :legend-visible="false" :title="{text:'内存使用', left: 'center'}"></ve-line>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</section>
</template>

<script>
import Vue from "vue";
import { mapState } from "vuex";
import fullscreen from "vue-fullscreen";
Vue.use(fullscreen);

export default {
    computed: {
        ...mapState(["serverInfo", "userInfo"])
    },
    components: {},
    data() {
        return {
            chartColors: ["#337ab7", "#7FFFD4"],
            pageWidth: 0,
            pageHeight: 0,
            protocol: location.protocol,
            authViewHeight: 80,
            timer: 0,
            vods: {},
            duration: 30,
            memData: {
                columns: ["time", "use"],
                rows: []
            },
            memSettings: {
                area: true,
                xAxisType: "time",
                yAxisType: ["percent"],
                min: [0],
                max: [1],
                labelMap: {
                    use: '使用'
                },
            },
            cpuData: {
                columns: ["time", "use"],
                rows: []
            },
            vodData: {
                columns: ["type", "count"],
                rows: []
            },
            vodSettings: {
                label: {
                    show: true,
                    formatter: "{b}: {c}"
                },
                roseType: "area" //'radius'
            },
            storeData: {
                columns: ["Name", "Used", "FreeSpace"],
                rows: []
            },
            storeSettings: {
                dimension: ["Name"],
                metrics: ["Used", "FreeSpace"],
                stack: {
                    store: ["Used", "FreeSpace"]
                },
                legendName: {
                    Used: '已使用(G)',
                    FreeSpace: '剩余(G)'
                },
                labelMap: {
                    Used: '已使用(G)',
                    FreeSpace: '剩余(G)'
                }
            },
            authData: {
                ChannelOnline: 0,
                ChannelTotal: 0,
                ChannelCount: 0,
                DeviceOnline: 0,
                DeviceTotal: 0,
            },
            bandwidthData: {
                columns: ["time", "use"],
                rows: []
            },
            bandwidthSettings: {
                area: true,
                xAxisType: "time",
                labelMap: {
                    use: "使用(Mbps)"
                }
            },
            netData: {
                columns: ["time", "sent","recv"],
                rows: []
            },
            netSettings: {
                area: true,
                xAxisType: "time",
                labelMap: {
                    sent:"发送(Mbps)",
                    recv:"接收(Mbps)"
                }
            },
            loadData: {
                rows: []
            },
            loadSettings: {
                opacity: 0.8,
                dimension: ["name"],
                metrics: ["load"],
                labelMap: {
                    load: '负载',
                    name: '流媒体服务',
                    serial: '流媒体服务'
                },
            },
            loadEvents: {
                click: e => {
                    if(this.isMobile()) return;
                    var filter = "";
                    var sms = "";
                    switch(e.name) {
                        case "直播":
                            filter = "stream";
                            break;
                        case "回放":
                            filter = "playback";
                            break;
                        case "播放":
                            filter = "playing";
                            break;
                        case "录像":
                            filter = "record";
                            break;
                        case "H265":
                            filter = "hevc";
                            break;
                        case "级联":
                            filter = "cascade";
                            break;
                        default:
                            var row = this.loadData.rows[e.dataIndex];
                            if(row && row.serial) {
                                sms = row.serial;
                            }
                            break;
                    }
                    if(filter||sms) {
                        this.$emit("show-session-list", filter, sms);
                    }
                }
            },
            storeExtend: {
                series: {
                    silent: true
                }
            },
            loadExtend: {
                series: {
                    barMaxWidth: 60
                },
                xAxis: {
                    axisLabel: {
                        formatter: (name, idx) => {
                            return `${name}(${this.loadData.rows[idx]['load']})`;
                        }
                    }
                }
            }
        };
    },
    mounted() {
        this.top();
        this.timer1 = setInterval(() => {
            this.top();
        }, 2000);
        this.store();
        this.timer2 = setInterval(() => {
            this.store();
        }, 10000);
        this.timer3 = setTimeout(() => {
            this.resizeCharts();
        }, 1000);
        $(window).on('resize', this.resize);
        $(document).on('expanded.pushMenu', this.resizeCharts);
        $(document).on('collapsed.pushMenu', this.resizeCharts);
        this.fixAuthView();
    },
    created() {
        this.initHeight();
    },
    beforeDestroy() {
        if (this.timer1) {
            clearInterval(this.timer1);
            this.timer1 = 0;
        }
        if (this.timer2) {
            clearInterval(this.timer2);
            this.timer2 = 0;
        }
        if (this.timer3) {
            clearTimeout(this.timer3);
            this.timer3 = 0;
        }
        $(window).off('resize', this.resize);
        $(document).off('expanded.pushMenu', this.resizeCharts);
        $(document).off('collapsed.pushMenu', this.resizeCharts);
    },
    methods: {
        fullscreen() {
            this.$fullscreen.enter(this.$el.querySelector(".container-fluid"), {
                wrap: false,
                callback: () => {
                    this.resize();
                    this.resizeCharts();
                }
            })
        },
        top() {
            $.get("/api/v1/dashboard/top").then(result => {
                var data = result.data;
                this.memData.rows = data.memData;
                this.cpuData.rows = data.cpuData;
                this.netData.rows = data.netData;
                this.loadData.rows = data.loadData;
            });
        },
        store() {
            $.get("/api/v1/dashboard/store").then(result => {
                var data = result.data;
                this.storeData.rows = data;
            });

            $.get("/api/v1/dashboard/auth").then(result => {
                var data = result.data;
                this.authData = data;
            });
        },
        isIE() {
            if (!!window.ActiveXObject || "ActiveXObject" in window)
                return true;
            else
                return false;
        },
        initHeight() {
            this.pageWidth = window.innerWidth;
            this.pageHeight = window.innerHeight;
            if (typeof this.pageWidth != "number") {
                if (document.compatMode == "CSS1Compat") {
                    this.pageWidth = document.documentElement.clientWidth;
                    this.pageHeight = document.documentElement.clientHeight;
                } else {
                    this.pageWidth = document.body.clientWidth;
                    this.pageHeight = document.body.clientHeight;
                }
            }
            this.pageHeight = this.pageHeight - 140;
        },
        resizeCharts() {
            this.$refs["storeChart"].resize();
            this.$refs["cpuChart"].resize();
            this.$refs["memChart"].resize();
            this.$refs["loadChart"].resize();
            this.$refs["netChart"].resize();
        },
        resize() {
            this.initHeight();
            this.fixAuthView();
        },
        fixAuthView() {
            if (this.pageHeight >= 900) {
                $(".auth-view").css("padding-top", `${this.pageHeight/6}px`);
            } else if (this.pageHeight >= 600) {
                $(".auth-view").css("padding-top", `${this.pageHeight/8}px`);
            } else {
                $(".auth-view").css("padding-top", `32px`);
            }
            return
        }
    }
};
</script>

<style lang="less" scoped>
.qcontent .fullscreen {
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    background-color: #ecf0f5;
}

.qcontent .full-btn {
    margin-left: 2px;
    margin-top: -3px;
}

.qcontent .col-md-6 {
    padding-left: 0px;
    padding-right: 0px;
    height: 100%;
}

.qcontent .col-md-4 {
    padding-left: 0px;
    padding-right: 0px;
    height: 100%;
}

.qcontent .panel {
    height: 100%;
}

.qcontent .panel-body {
    height: 100%;
    padding: 15px 0;
}

.qcontent .panel-title {
    text-align: center;
    font-weight: bold;
    font-size: 18px;
}

.qcontent .container-fluid {
    padding-bottom: 20px;
    padding-left: 10px;
    padding-right: 10px;
    padding-top: 10px;
}

.qcontent {
    margin: -16px -15px;
}

.view-dashboard {
    height: 50%;
}

.view-left {
    margin-right: 5px;
}

.view-right {
    margin-left: 5px;
}

.view-split {
    margin-top: 10px;
}

.auth-vlive,
.auth-live {
    margin-top: 6%;
}

.auth-num {
    margin-top: 20%;
    font-size: 14px;
    font-weight: bold;
    color: #337ab7;
}

.auth-view {
    min-height: 200px;
    padding-top: 78px;
}

.auth-title {
    font-size: 12px;
    color: #1d3b55;
}

.auth-promt-div {
    margin-top: 10%;
}

@screen-md-min: 992px;

@media (max-width: @screen-md-min) {
    .col-md-6 {
        width: 100%;
    }
    .col-md-4 {
        width: 100%;
    }
    .container-fluid {
        height: 100% !important;
    }

    .view-dashboard {
        height: 320px;
    }

    .view-left {
        margin-right: 0;
    }

    .view-right {
        margin-left: 0;
    }

    .view-split-m,
    .view-split {
        margin-top: 10px;
    }

    .auth-vlive,
    .auth-live {
        margin-top: 0%;
    }

    .auth-num {
        margin-top: 20%;
        font-size: 14px;
        font-weight: bold;
        color: #337ab7;
    }

    .auth-view {
        height: 200px !important;
        padding-top: 78px !important;
    }

    .qcontent .container-fluid {
        padding-bottom: 10px;
    }
}
</style>
