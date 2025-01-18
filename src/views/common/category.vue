<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
             node-key="catId" ref="menuTree">
    </el-tree>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'childrenCategory',
        label: 'name'
      }
    }
  },
  computed: {},
  watch: {},
  methods: {
    getTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/tree'),
        method: 'GET'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    handleNodeClick(data) {
      this.$emit('node-clicked',data)
    },
  },
  beforeCreate() {
  }, // 生命周期-创建之前
  created() {
    this.getTree()
  }, // 生命周期-创建之后
  beforeMount() {
  }, // 生命周期-挂载之前
  mounted() {
  }, // 生命周期-挂载之后
  beforeUpdate() {
  }, // 生命周期-更新之前
  updated() {
  }, // 生命周期-更新之后
  beforeDestroy() {
  }, // 生命周期-销毁之前
  destroyed() {
  }, // 生命周期-销毁之后
  activated() {
  } // 如果页面有keep-alive缓存功能,此函数触发.
}
</script>
<style scoped>

</style>
