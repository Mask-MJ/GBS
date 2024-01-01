<template>
  <div>
    <div
      class="screen-favorites-box-hidden"
      v-if="showFavoritesCard"
      @click="handleShowFavoritesCard"
    >
      &lt;&lt; 收藏夹
    </div>
    <div class="screen-favorites-box-show" v-else>
      <div class="title" @click="handleShowFavoritesCard">>> 收藏夹</div>
      <el-tree
        :data="list"
        node-key="id"
        :render-content="renderContent"
        :expand-on-click-node="false"
        draggable
        :allow-drag="allowDrag"
        :allow-drop="allowDrop"
      ></el-tree>
    </div>
  </div>
</template>

<script>
export default {
  emits: ["handleShowFavoritesCard"],
  props: {
    showFavoritesCard: { type: Boolean, default: false },
  },
  data() {
    return {
      list: [],
      createInput: "",
    };
  },
  mounted() {
    this.getFavoritesList();
  },
  methods: {
    handleShowFavoritesCard() {
      this.$emit("handleShowFavoritesCard");
    },
    // 获取收藏夹列表
    getFavoritesList() {
      const token = localStorage.getItem("livegbs_v2_token");
      $.ajax({
        method: "GET",
        url: "/v2/user/favorites",
        beforeSend: function(xhr) {
          xhr.setRequestHeader("Authorization", "Bearer " + token);
        },
      }).then((data) => {
        console.log("favorites", data);
        // this.list[1] = data.lastVisited;
        this.list = [
          { id: "1", label: "自定义收藏夹", children: data.folders },
          { id: "2", label: "自动保存", children: [{ label: "上次浏览记录" }] },
          { id: "3", label: "收藏摄像机", children: data.favorites },
        ];
      });
    },
    append(data) {
      const title = data.type === "folder" ? "摄像机" : "文件夹";
      const h = this.$createElement;
      this.$msgbox({
        message: h("p", null, [
          h("span", null, `请输入${title}名称: `),
          h("input", {
            style: "width: 250px",
            on: {
              input: (v) => {
                this.createInput = v;
              },
            },
          }),
        ]),
        showCancelButton: true,
        confirmButtonText: "确定",
        cancelButtonText: "取消",
      })
        .then(() => {
          const newChild = {
            id: "1111",
            label: this.createInput.target.value,
            type: "folder",
            children: [],
          };
          this.createInput.target.value = "";
          if (!data.children) {
            this.$set(data, "children", []);
          }
          data.children.push(newChild);
          this.$message({ type: "success", message: "添加成功!" });
        })
        .catch(() => {
          this.createInput.target.value = "";
          this.$message({ type: "info", message: "已取消添加" });
        });
    },
    remove(node, data) {
      this.$confirm(`此操作将删除<${data.label}>, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          const parent = node.parent;
          const children = parent.data.children || parent.data;
          const index = children.findIndex((d) => d.id === data.id);
          children.splice(index, 1);
          this.$message({ type: "success", message: "删除成功!" });
        })
        .catch(() => {
          this.$message({ type: "info", message: "已取消删除" });
        });
    },
    renderContent(h, { node, data, store }) {
      if (node.data.label === "自定义收藏夹") {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
            <span>
              <el-button
                size="mini"
                type="text"
                on-click={() => this.append(data)}
              >
                +
              </el-button>
            </span>
          </span>
        );
      } else if (
        ["自动保存", "收藏摄像机", "上次浏览记录"].includes(node.data.label)
      ) {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
          </span>
        );
      } else {
        return (
          <span class="custom-tree-node">
            <span>{node.label}</span>
            <el-button
              size="mini"
              type="text"
              on-click={() => this.remove(node, data)}
            >
              -
            </el-button>
          </span>
        );
      }
    },
    allowDrag(draggingNode) {
      const noDraggable = [
        "自定义收藏夹",
        "自动保存",
        "收藏摄像机",
        "上次浏览记录",
      ];
      if (noDraggable.includes(draggingNode.data.label)) {
        return false;
      } else {
        return true;
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      if (
        draggingNode.data.type === "folder" &&
        (type === "next" || type === "prev")
      ) {
        return true;
      } else if (dropNode.data.type === "folder" && type === "inner") {
        return draggingNode.data.type !== "folder";
      } else if (
        dropNode.data.type === "camera" &&
        (type === "next" || type === "prev")
      ) {
        return true;
      } else {
        return false;
      }
    },
  },
};
</script>

<style lang="less" scoped>
.screen-favorites-box-hidden {
  position: absolute;
  right: 0;
  top: 50px;
  width: 25px;
  padding: 10px 5px;
  color: #fff;
  background-color: #4c9fb6;
  cursor: pointer;
}

.screen-favorites-box-show {
  position: absolute;
  right: 0;
  top: 0;
  z-index: 999;
  width: 200px;
  padding-top: 50px;
  cursor: pointer;
  background-color: #fff;
  height: 100%;
  border-left: 1px solid #ebeef5;
  .title {
    text-align: center;
    padding: 10px;
    color: #fff;
    background-color: #4c9fb6;
  }
}
</style>
<style lang="less">
.el-tree-node__content {
  width: 100%;
}
.custom-tree-node {
  width: 100%;
  padding-right: 10px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}
</style>
