<template>
  <!-- 树形结构 -->
  <el-tree :data="data" :props="defaultProps" node-key="catId" ref="menuTree">
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      data: [],
      // 默认展开的分类
      expendedKey: [],
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
  },
};
</script>

<style>
</style>