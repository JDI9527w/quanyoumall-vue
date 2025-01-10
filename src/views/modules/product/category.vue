<template xmlns="http://www.w3.org/1999/html">
  <div>
    <el-button type="danger" @click="deleteBatch">批量删除</el-button>
    <br></br>
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick" :expand-on-click-node="false"
             node-key="catId" ref="menuTree" :show-checkbox=true>
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            添加子类
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length===0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="编辑" :visible.sync="dialogFormVisible" :closeOnClickModal="false">
      <el-form :model="newCategory">
        <el-form-item label="上级分类" v-if="dialogType=='add'">
          <el-input v-model="newCategory.pName" readonly>${}</el-input>
        </el-form-item>
        <el-form-item label="上级分类id" hidden>
          <el-input v-model="newCategory.parentCid" readonly>${}</el-input>
        </el-form-item>
        <el-form-item label="层级" hidden>
          <el-input v-model="newCategory.catLevel" readonly>${}</el-input>
        </el-form-item>
        <el-form-item label="分类名称">
          <el-input v-model="newCategory.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" v-if="newCategory.catLevel==3">
          <el-input v-model="newCategory.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      newCategory: {
        pName: '',
        catId: null,
        parentCid: '',
        catLevel: '',
        name: '',
        productUnit: '',
        showStatus: '1'
      },
      dialogType: '',
      dialogFormVisible: false,
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
    handleNodeClick(data) {
      console.log(data)
    },
    getTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/tree'),
        method: 'GET'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    append(data) {
      this.resetCategoryData()
      console.log('append', data)
      this.dialogFormVisible = true
      this.dialogType = 'add'
      this.newCategory.parentCid = data.catId
      this.newCategory.pName = data.name
      this.newCategory.catLevel = data.catLevel + 1
    },
    addCategory() {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'POST',
        data: this.$http.adornData(this.newCategory, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '操作成功!'
        })
        this.dialogFormVisible = false
        this.expandedKey = [this.newCategory.parentCid]
        this.getTree()
      })
    },
    remove(node, data) {
      let ids = [data.catId]
      this.$confirm(`确认删除【${data.name}】分类吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'DELETE',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.expandedKey = [node.parent.data.catId]
          this.getTree()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        })
      })
    },
    edit(data) {
      console.log('edithttp')
      this.resetCategoryData()
      this.dialogFormVisible = true
      this.dialogType = 'edit'
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'GET'
      }).then(({data}) => {
        this.newCategory.parentCid = data.data.parentCid
        this.newCategory.catId = data.data.catId
        this.newCategory.name = data.data.name
      })
    },
    editCategory() {
      let {catId, name} = this.newCategory
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'PUT',
        data: this.$http.adornData({catId, name}, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: '操作成功!'
        })
        this.dialogFormVisible = false
        this.expandedKey = [this.newCategory.parentCid]
        this.getTree()
      })
    },
    submitData(data) {
      if (this.dialogType === 'add') {
        this.addCategory()
      } else if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },
    resetCategoryData() {
      this.newCategory.pName = ''
      this.newCategory.catId = null
      this.newCategory.parentCid = ''
      this.newCategory.catLevel = ''
      this.newCategory.name = ''
      this.newCategory.productUnit = ''
      this.newCategory.showStatus = 1
    },
    deleteBatch() {
      let catIds = []
      let names = []
      let checkedNodes = this.$refs.menuTree.getCheckedNodes()
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId)
        names.push(checkedNodes[i].name)
      }
      this.$confirm(`确认删除【${names}】分类吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'DELETE',
          data: this.$http.adornData(catIds, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
          this.getTree()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消'
        })
      })
    }
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
