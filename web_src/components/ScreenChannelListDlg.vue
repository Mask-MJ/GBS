<template>
  <div
    :class="['modal', { fade: fade }]"
    data-backdrop="static"
    data-disable="false"
    data-keyboard="true"
    tabindex="-1"
  >
    <div :class="['modal-dialog', size]">
      <div class="modal-content">
        <div class="modal-header">
          <button
            type="button"
            class="close"
            data-dismiss="modal"
            aria-label="Close"
          >
            <span aria-hidden="true">&times;</span>
          </button>
          <h4 class="modal-title text-center text-primary">
            <span>{{ title }}</span>
          </h4>
        </div>
        <div class="modal-body">
          <el-tabs v-model="activeName">
            <el-tab-pane label="选择通道" name="1">
              <div class="form-inline" autocomplete="off" spellcheck="false">
                <div class="form-group form-group-sm">
                  <label>搜索</label>
                  <input
                    type="text"
                    class="form-control"
                    placeholder="关键字"
                    v-model.trim="q"
                    @keydown.enter.prevent
                    ref="q"
                  />
                </div>
                <span class="hidden-xs">&nbsp;&nbsp;</span>
                <div class="form-group form-group-sm">
                  <label>在线状态</label>
                  <select class="form-control" v-model.trim="online">
                    <option value="">全部</option>
                    <option value="true">在线</option>
                    <option value="false">离线</option>
                  </select>
                </div>
              </div>
              <br />
              <el-table
                :data="channels"
                stripe
                @sort-change="sortChange"
                :max-height="500"
                @row-click="rowClick"
                @row-dblclick="rowDblclick"
                :row-style="rowStyle"
                ref="channelTable"
                v-loading="loading"
                element-loading-text="加载中..."
              >
                <el-table-column
                  prop="DeviceID"
                  label="设备国标编号"
                  min-width="200"
                  show-overflow-tooltip
                  sortable="custom"
                ></el-table-column>
                <el-table-column
                  prop="ID"
                  label="通道国标编号"
                  min-width="200"
                  show-overflow-tooltip
                  sortable="custom"
                ></el-table-column>
                <!-- <el-table-column prop="DeviceName" label="设备名称" min-width="120" :formatter="formatDeviceName" show-overflow-tooltip></el-table-column> -->
                <el-table-column
                  prop="Name"
                  label="通道名称"
                  min-width="160"
                  :formatter="formatName"
                  show-overflow-tooltip
                ></el-table-column>
                <el-table-column min-width="100" label="快照">
                  <template slot-scope="props">
                    <span v-if="isDir(props.row)">
                      <i class="fa fa-sitemap"></i>
                    </span>
                    <el-popover
                      v-else
                      :open-delay="1000"
                      :close-delay="10"
                      placement="left"
                      :title="props.row.ID"
                      width="400"
                      trigger="hover"
                    >
                      <img
                        onerror="this.src='/images/default_snap.png';"
                        style="width:100%;height:100%;"
                        :src="props.row.SnapURL"
                      />
                      <img
                        onerror="this.src='/images/default_snap.png';"
                        style="height:30px;width:50px;"
                        slot="reference"
                        :src="props.row.SnapURL"
                      />
                    </el-popover>
                  </template>
                </el-table-column>
                <el-table-column prop="Status" label="在线状态" min-width="100">
                  <template slot-scope="props">
                    <span v-if="isDir(props.row)">-</span>
                    <span
                      v-else-if="
                        (props.row.CustomStatus || props.row.Status) == 'ON'
                      "
                      :class="{
                        'text-success': props.row.Status == 'ON',
                        'text-orange': !!props.row.CustomStatus,
                      }"
                      >在线</span
                    >
                    <span
                      v-else
                      :class="{ 'text-orange': !!props.row.CustomStatus }"
                      >离线</span
                    >
                  </template>
                </el-table-column>
                <!-- <el-table-column prop="SubCount" label="子节点数" min-width="100" v-if="hasAnyRole(serverInfo, userInfo, '管理员')" sortable="custom"></el-table-column> -->
                <el-table-column
                  prop="Manufacturer"
                  label="厂家"
                  min-width="120"
                  :formatter="formatManufacturer"
                  show-overflow-tooltip
                ></el-table-column>
              </el-table>
              <el-pagination
                v-if="total > 0"
                layout="total,prev,pager,next"
                :pager-count="isMobile() ? 3 : 5"
                class="pull-right"
                :total="total"
                :page-size.sync="pageSize"
                :current-page.sync="currentPage"
              ></el-pagination>
              <div class="clearfix"></div
            ></el-tab-pane>
            <el-tab-pane label="选择设备树" name="2">
              <el-tree
                ref="devTree"
                id="dev-tree"
                node-key="key"
                :props="treeProps"
                :load="treeLoad"
                lazy
                @node-click="treeNodeClick"
              >
                <span class="custom-tree-node" slot-scope="{ node, data }">
                  <span
                    :class="{
                      'text-green': treeLeaf(data) && data.status === 'ON',
                    }"
                  >
                    <i
                      :class="[
                        'fa',
                        {
                          'fa-sitemap': !treeLeaf(data),
                          'fa-camera': treeLeaf(data),
                          'fa-spinner': node.playing,
                          'fa-spin': node.playing,
                          'fa-play': node.play,
                        },
                      ]"
                    ></i>
                    <span
                      class="ellipsis"
                      style="user-select: none"
                      :title="node.label"
                      >{{ node.label }}</span
                    >
                  </span>
                </span>
              </el-tree>
            </el-tab-pane>
          </el-tabs>
        </div>
        <!-- <div class="modal-footer">
        <button type="button" class="btn btn-default" data-dismiss="modal">取消</button>
      </div> -->
      </div>
      <!-- /.modal-content -->
    </div>
    <!-- /.modal-dialog -->
  </div>
</template>

<script>
import _ from "lodash";
import $ from "jquery";
import "jquery-ui/ui/widgets/draggable";
import { mapState } from "vuex";

export default {
  props: {
    title: {
      default: "",
    },
    size: {
      type: String,
      default: "modal-lgg",
    },
    fade: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      q: "",
      online: "true",
      total: 0,
      pageSize: 10,
      currentPage: 1,
      sort: "",
      order: "",
      loading: false,
      channels: [],
      index: 0,
      activeName: "1",
      treeProps: {
        label: (data, node) => {
          node.serial = data.serial;
          node.code = data.code;
          var label = data.customName || data.name || data.code || data.id;
          if (
            data.custom &&
            !data.code &&
            !this.hasAllChannel(this.serverInfo, this.userInfo)
          ) {
            return label;
          }
          if (!this.treeLeaf(data) && data.subCountDevice > 0) {
            label += ` [${data.onlineSubCount}/${data.subCountDevice}]`;
          }
          return label;
        },
        isLeaf: (data, node) => {
          return this.treeLeaf(data);
        },
        disabled: (data, node) => {
          return this.treeLeaf(data) && data.status != "ON";
        },
      },
    };
  },
  computed: {
    ...mapState(["userInfo", "serverInfo"]),
  },
  watch: {
    q: function(newVal, oldVal) {
      this.doDelaySearch();
    },
    online: function(newVal, oldVal) {
      this.doSearch();
    },
    currentPage: function(newVal, oldVal) {
      this.doSearch(newVal);
    },
    pageSize: function(newVal, oldVal) {
      this.doSearch();
    },
  },
  mounted() {
    $(this.$el)
      .find(".modal-content")
      .draggable({
        handle: ".modal-header",
        cancel: ".modal-title span",
        addClasses: false,
        containment: "document",
        delay: 100,
        opacity: 0.5,
      });
    $(this.$el)
      .on("shown.bs.modal", () => {
        this.$emit("show");
      })
      .on("hidden.bs.modal", () => {
        this.errors.clear();
        // this.reset();
        this.$emit("hide");
      });
  },
  methods: {
    sortChange(data) {
      this.sort = data.prop;
      this.order = data.order == "ascending" ? "asc" : "desc";
      this.getChannels();
    },
    doSearch(page = 1) {
      this.currentPage = page;
      this.getChannels();
    },
    doDelaySearch: _.debounce(function() {
      this.doSearch();
    }, 800),
    formatName(row, col, cell) {
      var devName = row.DeviceCustomName || row.DeviceName || "";
      var chName = row.CustomName || row.Name || "";
      if (devName && devName != chName) {
        if (!chName) {
          chName = devName;
        } else {
          chName = `${chName}@${devName}`;
        }
      }
      return chName || "-";
    },
    formatDeviceName(row, col, cell) {
      return row.DeviceCustomName || row.DeviceName || "-";
    },
    formatChannelName(row, col, cell) {
      return row.CustomName || row.Name || "-";
    },
    formatManufacturer(row, col, cell) {
      if (cell) return cell;
      return "-";
    },
    getChannels() {
      this.loading = true;
      $.get("/api/v1/device/channellist", {
        q: this.q,
        start: (this.currentPage - 1) * this.pageSize,
        limit: this.pageSize,
        channel_type: "device",
        online: this.online,
        sort: this.sort,
        order: this.order,
      })
        .then((ret) => {
          this.total = ret.ChannelCount;
          this.channels = ret.ChannelList || [];
        })
        .always(() => {
          this.$nextTick(() => {
            this.loading = false;
          });
        });
    },
    rowClick(row, event, column) {
      if ((row.CustomStatus || row.Status) !== "ON") return;
      if (this.serverInfo.StrictChannelParental && row.Parental == 1) return;
      if (this.serverInfo.StrictChannelCode && row.ID.length != 20) return;
      console.log("selected", this.index, row);
      this.$emit("selected", this.index, row);
      this.hide();
    },
    rowDblclick(row, event, column) {
      // if((row.CustomStatus || row.Status) !== "ON") return;
      if (this.serverInfo.StrictChannelParental && row.Parental == 1) return;
      if (this.serverInfo.StrictChannelCode && row.ID.length != 20) return;
      this.$emit("selected", this.index, row);
      this.hide();
    },
    rowStyle({ row, rowIndex }) {
      if ((row.CustomStatus || row.Status) === "ON") {
        if (this.serverInfo.StrictChannelParental && row.Parental == 1)
          return "";
        if (this.serverInfo.StrictChannelCode && row.ID.length != 20) return "";
        return "cursor:pointer";
      }
      return "";
    },
    isDir(row) {
      // return row && (row.SubCount > 0 || row.Parental == 1);
      if (row) {
        if (row.SubCount > 0) return true;
        if (this.serverInfo.StrictChannelParental && row.Parental == 1)
          return true;
        if (this.serverInfo.StrictChannelCode && row.ID.length <= 10)
          return true;
        if (row.Parental == 1 && row.Manufacturer == "LiveQing") return true;
        // if (row.ID.length == 20 && (row.ID.substring(10, 13) == "216" || row.ID.substring(10, 13) == "215" || row.ID.substring(10, 13) == "200")) {
        if (
          row.ID.length == 20 &&
          (row.ID.substring(10, 13) == "216" ||
            row.ID.substring(10, 13) == "215")
        ) {
          return true;
        }
      }
      return false;
    },
    reset() {
      this.index = 0;
      this.channels = [];
      this.q = "";
      this.online = "true";
      this.currentPage = 1;
      this.pageSize = 10;
    },
    show(index) {
      this.index = index;
      $(this.$el).modal("show");
      this.getChannels();
    },
    hide() {
      $(this.$el).modal("hide");
    },
    treeLoad(node, resolve) {
      var serial = (node.data || {}).serial || "";
      var pcode = (node.data || {}).code || "";
      this.treeLoading = true;
      $.get("/api/v1/device/channeltree", {
        serial: serial,
        pcode: pcode,
      })
        .then((ret) => {
          resolve(
            (ret || []).map((v, idx, arr) => {
              if (!node.level && !this.devLevelFilter) {
                if (arr.length > 100) {
                  this.devLevelFilter = true;
                } else if (v.subCount > 1000) {
                  this.devLevelFilter = true;
                }
              }
              return Object.assign(v, {
                key: v.id,
              });
            })
          );
          if (this.q && !node.matched) {
            this.$refs["devTree"].filter(this.q);
          }
        })
        .always(() => {
          this.$nextTick(() => {
            this.treeLoading = false;
          });
          if (
            this.q &&
            !this.q.endsWith(" ") &&
            this.q.split(/\s+/).length === node.level &&
            this.devLevelFilter
          ) {
            this.q += " ";
            this.$refs["q"].focus();
          }
        });
    },
    treeLeaf(data) {
      if (
        data &&
        data.subCount == 0 &&
        data.serial &&
        data.code &&
        !data.custom
      ) {
        if (this.serverInfo.StrictChannelParental && data.parental)
          return false;
        if (this.serverInfo.StrictChannelCode && data.code.length != 20)
          return false;
        if (!data.parental || data.manufacturer != "LiveQing") {
          // if (data.code.length != 20 || (data.code.substring(10, 13) != "216" && data.code.substring(10, 13) != "215" && data.code.substring(10, 13) != "200")) {
          if (
            data.code.length != 20 ||
            (data.code.substring(10, 13) != "216" &&
              data.code.substring(10, 13) != "215")
          ) {
            return true;
          }
        }
      }
      return false;
    },
    treeNodeClick(data, node, c) {
      this.contextMenuNodeData = null;
      this.contextMenuNode = null;
      if (this.treeLeaf(data) && node && !node.playing) {
        if (node.timer) {
          clearTimeout(node.timer);
        }
        if (!node.clickCnt || node.clickCnt > 1) {
          node.clickCnt = 1;
        } else {
          node.clickCnt++;
        }
        node.timer = setTimeout(() => {
          node.clickCnt = 0;
          node.timer = 0;
        }, 500);
        console.log("selected", this.index, data);
        if (data.status === "ON" || node.clickCnt == 2) {
          const [DeviceID, ID] = data.id.split(":");
          this.$emit("selected", this.index, {
            DeviceID: DeviceID,
            ID: ID,
            PTZType: data.ptzType,
            CustomName: data.customName,
            Name: data.name,
          });
          this.hide();
        }
      }
    },
  },
};
</script>

<style lang="less" scoped>
.modal-content {
  overflow: hidden;
}

@media screen and(min-width: 992px) {
  .modal-dialog.modal-lgg {
    width: 90%;
  }
}

@media screen and(min-width: 1200px) {
  .modal-dialog.modal-lgg {
    width: 1200px;
  }
}
</style>
