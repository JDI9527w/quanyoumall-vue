<template>
  <el-dialog
    :title="!dataForm.attrGroupId ? '新增' : '修改'"
    :close-on-click-modal="false"
    @closed="cleanData"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()"
             label-width="80px">
      <el-form-item label="组名" prop="attrGroupName">
        <el-input v-model="dataForm.attrGroupName" placeholder="组名"></el-input>
      </el-form-item>
      <el-form-item label="排序" prop="sort">
        <el-input v-model="dataForm.sort" placeholder="排序"></el-input>
      </el-form-item>
      <el-form-item label="描述" prop="descript">
        <el-input v-model="dataForm.descript" placeholder="描述"></el-input>
      </el-form-item>
      <el-form-item label="组图标" prop="icon">
        <el-input v-model="dataForm.icon" placeholder="组图标"></el-input>
      </el-form-item>
      <el-form-item label="所属分类" prop="catelogId">
        <div class="block">
          <span class="demonstration"></span>
          <el-cascader
            v-model="dataForm.catelogPath"
            :options="categories"
            :props="props"
            filterable
            placeholder="试试搜索：手机"></el-cascader>
        </div>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
export default {
  data() {
    return {
      props: {
        value: "catId",
        label: "name",
        children: "childrenCategory"
      },
      categories: [],
      visible: false,
      dataForm: {
        attrGroupId: 0,
        attrGroupName: '',
        sort: '',
        descript: '',
        icon: '',
        catelogPath: [],
        catelogId: 0
      },
      dataRule: {
        attrGroupName: [
          {required: true, message: '组名不能为空', trigger: 'blur'}
        ],
        sort: [
          {required: true, message: '排序不能为空', trigger: 'blur'}
        ],
        descript: [
          {required: true, message: '描述不能为空', trigger: 'blur'}
        ],
        icon: [
          // {required: true, message: '组图标不能为空', trigger: 'blur'}
        ],
        catelogId: [
          {required: true, message: '所属分类不能为空', trigger: 'blur'}
        ]
      }
    }
  },
  methods: {
    getTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/tree'),
        method: 'GET'
      }).then(({data}) => {
        this.categories = data.data
      })
    },
    init(id) {
      this.dataForm.attrGroupId = id || 0
      this.visible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].resetFields()
        if (this.dataForm.attrGroupId) {
          this.$http({
            url: this.$http.adornUrl(`/product/attrgroup/info/${this.dataForm.attrGroupId}`),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.dataForm.attrGroupName = data.attrGroup.attrGroupName
              this.dataForm.sort = data.attrGroup.sort
              this.dataForm.descript = data.attrGroup.descript
              this.dataForm.icon = data.attrGroup.icon
              this.dataForm.catelogPath = data.attrGroup.catelogPath
            }
          })
        }
      })
    },
    // 表单提交
    dataFormSubmit() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.$http({
            url: this.$http.adornUrl(`/product/attrgroup/${!this.dataForm.attrGroupId ? 'save' : 'update'}`),
            method: 'post',
            data: this.$http.adornData({
              'attrGroupId': this.dataForm.attrGroupId || undefined,
              'attrGroupName': this.dataForm.attrGroupName,
              'sort': this.dataForm.sort,
              'descript': this.dataForm.descript,
              'icon': this.dataForm.icon,
              'catelogId': this.dataForm.catelogPath[this.dataForm.catelogPath.length-1]
            })
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.visible = false
                  this.$emit('refreshDataList')
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }
      })
    },
    cleanData(){
      this.dataForm.catelogPath = []
    },
    find(array, id) {
      let stack = []
      let going = true
      let walker = (array, id) => {
        array.forEach(item => {
          if (!going) return
          stack.push(item['catId'])
          if (item['catId'] === id) {
            going = false
          } else if (item['childrenCategory']) {
            walker(item['childrenCategory'], id)
          } else {
            stack.pop()
          }
        })
        if (going) stack.pop()
      }
      walker(array, id)
      return stack
    },
  },
  created() {
    this.getTree()
  }
}
</script>
