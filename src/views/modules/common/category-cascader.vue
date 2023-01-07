<template>
  <div>
    <el-cascader
      filterable
      clearable
      placeholder="试试搜索：手机"
      v-model="paths"
      :options="categorys"
      :props="setting"
    ></el-cascader>
  </div>
</template>

<script>
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  //接受父组件传来的值
  props: {
    catelogPath: {
      type: Array,
      default () {
        return []
      }
    }
  },
  data () {
    //这里存放数据
    return {
      setting: {
        value: 'catId',
        label: 'name',
        children: 'children'
      },
      categorys: [],
      paths: this.catelogPath
    }
  },
  watch: {
    catelogPath (v) {
      this.paths = this.catelogPath
    },
    paths (v) {
      this.$emit('update:catelogPath', v)
      //还可以使用pubsub-js进行传值
      this.PubSub.publish('catPath', v)
    }
  },
  //方法集合
  methods: {
    getCategorys () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.categorys = data.data
      })
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created () {
    this.getCategorys()
  }
}
</script>
