<template>
  <div>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <!-- 树形结构 -->
    <el-tree
      :data="data"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expendedKey"
      ref="menuTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            添加分类
          </el-button>
          <el-button
            v-if="node.childNodes == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除分类
          </el-button>
          <el-button type="text" size="mini" @click="() => update(data)">
            修改分类
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      // 默认展开的分类
      expendedKey: [],
      // 消息框显示
      dialogVisible: false,
      // 消息类型
      dialogType: null, //add、update
      // 提示类型
      title: null,
      maxLevel: 0,
      updateNodes: [],
      // 分类信息
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productCount: 0,
        icon: "",
        productUnit: "",
      },
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  created() {
    this.getMenus();
  },
  methods: {
    // 获取到分类菜单树
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.data = data.data;
      });
    },

    //添加分类按钮
    append(data) {
      this.dialogVisible = true;
      // 指定对话框类型为add
      this.dialogType = "add";
      // 添加提示信息
      this.title = "添加分类";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      // 清空数据
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },

    // 修改分类按钮
    update(data) {
      this.dialogVisible = true;
      // 指定消息框类型
      this.dialogType = "update";
      // 修改提示信息
      this.title = "修改分类";
      // 发送请求获取数据，回显分类信息
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category.catId = data.data.catId;
        this.category.parentCid = data.data.parentCid;
        this.category.name = data.data.name;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
      });
    },

    // 根据消息框类型调用对应的方法
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "update") {
        this.updateCategory();
      }
    },

    // 修改分类消息框
    updateCategory() {
      // 解构数据
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then(({ data }) => {
        // 关闭对话框
        this.dialogVisible = false;
        this.$message({
          type: "success",
          message: "菜单修改成功",
        });
        // 刷新分类树
        this.getMenus();
        // 默认展开的菜单
        this.expendedKey = [this.category.parentCid];
      });
    },

    // 删除分类按钮
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】分类?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          // 发送请求
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            // 刷新菜单
            this.getMenus();
            // 默认展开的菜单
            this.expendedKey = [node.parent.data.catId];
          });

          this.$message({
            type: "success",
            message: "分类删除成功!",
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },

    // 添加分类消息框
    addCategory() {
      this.dialogVisible = false;
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      })
        .then(({ data }) => {
          // 刷新分类树
          this.getMenus();
          // 默认展开的菜单
          this.expendedKey = [this.category.parentCid];

          this.$message({
            type: "success",
            message: "分类保存完成",
          });
        })
        .catch(() => {
          this.$message({
            type: "warning",
            message: "分类保存失败",
          });
        });
    },

    // 批量删除
    batchDelete() {
      this.$confirm(`是否批量删除分类?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          // 获取到批量选择的分类id
          let catIds = [];
          let categorys = this.$refs.menuTree.getCheckedNodes();
          for (let i = 0; i < categorys.length; i++) {
            catIds.push(categorys[i].catId);
          }

          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(catIds, false),
          }).then(({ data }) => {
            this.$message({
              message: "分类批量删除成功",
              type: "success",
            });
        
          // 刷新分类树
          this.getMenus(); 
          });
        })
        .catch(() => {
          this.$message({
              message: "分类批量删除失败",
              type: "error",
            });
        });

      let catIds = [];
      let categorys = this.$refs.menuTree.getCheckedNodes();
      for (let i = 0; i < categorys.length; i++) {
        catIds.push(categorys[i].catId);
      }
    },

    // // 判断分类是否可以拖拽
    // allowDrop(draggingNode, dropNode, type) {
    //   // 被拖动的当前节点以及所在的父节点的总层数不能大于3

    //   // 被拖动的当前节点层数
    //   this.countNodeLevel(draggingNode.data);
    //   // 当前正在拖动的节点+父节点所在深度不大于3即可
    //   let deep = this.maxLevel - draggingNode.data.catLevel + 1;
    //   if (type == "inner") {
    //     return deep + dropNode.level <= 3;
    //   } else {
    //     return deep + dropNode.parent.level <= 3;
    //   }
    // },

    // // 求出拖拽节点的最大深度
    // countNodeLevel(node) {
    //   // 找到所有子节点，求出最大深度
    //   if (node.children != null && node.children.length > 0) {
    //     for (let i = 0; i < node.children.length; i++) {
    //       if (node.children[i].catLevel > this.maxLevel) {
    //         this.maxLevel = node.children[i].catLevel;
    //       }
    //       // 递归
    //       this.countNodeLevel(node.children[i]);
    //     }
    //   }
    // },

    // handleDrop(draggingNode, dropNode, dropType, ev) {
    //   console.log("handleDrop: ", draggingNode, dropNode, dropType);
    //   // 当前节点最新的父节点
    //   let pCid = 0;
    //   let siblings = null;
    //   if (dropType == "before" || dropType == "after") {
    //     pCid = dropNode.parent.data.catId;
    //     siblings = dropNode.parent.childNodes;
    //   } else {
    //     pCid = dropNode.data.catId;
    //     siblings = dropNode.childNodes;
    //   }
    //   // 当前节点的最新顺序
    //   for (let i = 0; i < siblings.length; i++) {
    //     if(siblings[i].data.catId == draggingNode.data.catId){
    //       if(siblings[i].data.catLevel != draggingNode.data.catId){

    //       }
    //       this.updateNodes.push({ catId: siblings[i].data.catId, sort: i ,parentCid:pCid});
    //     }else{
    //       this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
    //     }

    //   }
    //   // 当前节点的最新层级
    // },
  },
};
</script>


<style>
</style>