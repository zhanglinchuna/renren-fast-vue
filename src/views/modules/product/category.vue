<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKeys"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">添加</el-button>
          <el-button type="text" size="mini" @click="() => edit(node, data)">修改</el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >删除</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="菜单名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标地址">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData(dialogType)">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      dialogTitle: '',
      dialogType: '',
      menus: [],
      expandedKeys: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      deleteMenuName: '',
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: '',
        productUnit: '',
        productCount: 0
      },
      dialogFormVisible: false
    }
  },
  methods: {
    getDataList () {
      this.dataListLoading = true
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        this.menus = data.data
      })
    },
    handleNodeClick (data) {},
    append (data) {
      this.dialogType = 'add'
      this.dialogTitle = '新增商品分类'
      this.dialogFormVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    edit (node, data) {
      if (!node.checked) {
        this.$message.error({
          message: '请选中要修改的菜单！',
          duration: '2000'
        })
        return
      }
      this.dialogType = 'edit'
      this.dialogTitle = '修改商品分类'
      this.dialogFormVisible = true

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        this.category.name = data.category.name
        this.category.catId = data.category.catId
        this.category.icon = data.category.icon
        this.category.productUnit = data.category.productUnit
        this.category.parentCid = data.category.parentCid
      })
    },
    submitData (dialogType) {
      // eslint-disable-next-line eqeqeq
      if (this.dialogType == 'add') {
        this.addCategory()
        // eslint-disable-next-line eqeqeq
      } else if (this.dialogType == 'edit') {
        this.editCategory()
      }
    },
    editCategory () {
      var { name, catId, icon, productUnit } = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ name, catId, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: `成功修改菜单`,
          type: 'success',
          duration: '2000'
        })
        this.dialogFormVisible = false
        this.getDataList()
        this.expandedKeys = [this.category.parentCid]
      })
    },
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          message: `成功添加菜单`,
          type: 'success',
          duration: '2000'
        })
        this.dialogFormVisible = false
        this.getDataList()
        this.expandedKeys = [this.category.parentCid]
      })
    },

    remove (node, data) {
      var ids = [data.catId]
      this.deleteMenuName = data.name
      if (!node.checked) {
        this.$message.error({
          message: '请选中要删除的菜单！',
          duration: '2000'
        })
        return
      }
      this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: `成功删除【${this.deleteMenuName}】菜单`,
              type: 'success',
              duration: '2000'
            })
            this.getDataList()
            this.expandedKeys = [node.parent.data.catId]
          })
        })
        .catch(() => {
          console.log('已取消')
        })
    },
    cancel () {
      this.category = {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: '',
        productUnit: '',
        productCount: 0
      }
    }
  },
  created () {
    this.getDataList()
  },
  watch: {
    dialogFormVisible: function (nval, oval) {
      if (oval) {
        this.category = {
          name: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0,
          catId: null,
          icon: '',
          productUnit: '',
          productCount: 0
        }
      }
    }
  }
}
</script>

<style>
</style>
