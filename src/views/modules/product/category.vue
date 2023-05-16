<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      draggable
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
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="Category">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="Icon">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="Unit">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具 js，第三方插件 js，json 文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data () {
    return {
      title: '',
      dialogType: '',
      category: { name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, icon: '', productUnit: '' },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        console.log('AAA', data.data)
        this.menus = data.data
      })
    },

    submitData () {
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },

    append (data) {
      console.log('Append Data: ', data)
      this.dialogVisible = true
      this.dialogType = 'add'
      this.title = 'Add'
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel + 1
      this.category.name = ''
      this.category.catId = null
      this.category.icon = ''
      this.productUnit = ''
      this.category.sort = ''
      this.category.showStatus = 1
    },

    addCategory () {
      console.log('Add Category Data: ', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: 'Add Successfully',
          type: 'success'
        })
        // Close dialog
        this.dialogVisible = false
        // Refresh for new menus
        this.getMenus()
        // Set default menu
        this.expandedKey = [this.category.parentCid]
      })
    },

    edit (data) {
      console.log('Edit data: ', data)
      this.dialogVisible = true
      this.dialogType = 'edit'
      this.title = 'Edit'

      // request for latest data
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        console.log('return data: ', data)
        this.category.name = data.category.name
        this.category.catId = data.category.catId
        this.category.icon = data.category.icon
        this.category.productUnit = data.category.productUnit
        this.category.parentCid = data.category.parentCid
        this.category.catLevel = data.category.catLevel
        this.category.sort = data.category.sort
        this.category.showStatus = data.category.showStatus
      })

      // this.category.name = data.name
      // this.catId = data.catId
    },

    editCategory () {
      console.log('Edit Category Data: ', this.category)
      let { catId, name, icon, productUnit } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: 'Edit Successfully',
          type: 'success'
        })
        // Close dialog
        this.dialogVisible = false
        // Refresh for new menus
        this.getMenus()
        // Set default menu
        this.expandedKey = [this.category.parentCid]
      })
    },

    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`Delete ${data.name}?`, '提示', {
        confirmButtonText: 'Yes',
        cancelButtonText: 'No',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            message: 'Deleted Successfully',
            type: 'success'
          })
          // Refresh for new menus
          this.getMenus()
          // Set default menu
          this.expandedKey = [node.parent.data.catId]
        })
      }).catch(() => {

      })
      console.log('remove: ', node, data)
    }

  },

  // 生命周期 - 创建完成（可以访问当前 this 实例）
  created () {
    this.getMenus()
  },
  // 生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted () {

  },
  beforeCreate () { }, // 生命周期 - 创建之前
  beforeMount () { }, // 生命周期 - 挂载之前
  beforeUpdate () { }, // 生命周期 - 更新之前
  updated () { }, // 生命周期 - 更新之后
  beforeDestroy () { }, // 生命周期 - 销毁之前
  destroyed () { }, // 生命周期 - 销毁完成
  activated () { } // 如果页面有 keep-alive 缓存功能,这个函数会触发
}
</script>
<style scoped>
</style>