<template>
  <!-- 树形结构 -->
  <el-tree :data="data" :props="defaultProps" node-key="catId" ref="menuTree" @node-click="nodeClick">
  </el-tree>
</template>

<script>
export default {
  data () {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },

  created () {
    this.getMenus()
  },
  methods: {
    // 获取到分类菜单树
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({data}) => {
        this.data = data.data
      })
    },

    nodeClick(data,node){
        this.$emit("tree-node-click",data,node);
    }
  },
}
</script>

<style>
</style>
