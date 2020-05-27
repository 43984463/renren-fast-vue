<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" :default-expanded-keys="expandedKeys" show-checkbox node-key="catId"
             @node-click="handleNodeClick"
             :expand-on-click-node="false"
             draggable
             :allow-drop="allowDrop">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)"> 添加 </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
                     @click="() => remove(node, data)"> 删除 </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)"> 修改 </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
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
  export default {
    data () {
      return {
        menus: [],
        expandedKeys: [],
        dialogVisible: false,
        title: '',
        category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null, icon: '', productUnit: ''},
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      }
    },

    methods: {
      handleNodeClick (data) {
        console.log(data)
      },

      getMenus () {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          console.log('成功获取数据', data.data)
          this.menus = data.data
        })
      },

      append (data) {
        console.log(data)
        this.dialogVisible = true
        this.title = '增加商品信息'
        Object.assign(this.$data.category, this.$options.data().category)
        this.category.parentCid = data.catId
        this.category.catLevel = data.catLevel * 1 + 1
      },

      edit (data) {
        console.log('修改' + data)
        this.$http({
          url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          method: 'get'
        }).then(({data}) => {
          this.dialogVisible = true
          this.title = '修改商品信息'
          this.category.name = data.data.name
          this.category.catId = data.data.catId
          this.category.icon = data.data.icon
          this.category.productUnit = data.data.productUnit
          this.category.parentCid = data.data.parentCid
        })
      },

      submitData () {
        if (this.title === '增加商品信息') {
          this.addCategory()
        } else {
          this.editCategory()
        }
      },

      editCategory () {
        let {catId, name, icon, productUnit} = this.category
        this.$http({
          url: this.$http.adornUrl('/product/category/update'),
          method: 'post',
          data: this.$http.adornData({catId, name, icon, productUnit}, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '修改成功'
          })
          // 关闭弹框
          this.dialogVisible = false
          // 重新刷新菜单
          this.getMenus()
          this.expandedKeys = [this.category.parentCid]
        })
      },

      addCategory () {
        console.log(this.category)
        this.$http({
          url: this.$http.adornUrl('/product/category/save'),
          method: 'post',
          data: this.$http.adornData(this.category, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: '保存成功'
          })
          // 关闭弹框
          this.dialogVisible = false
          // 重新刷新菜单
          this.getMenus()
          this.expandedKeys = [this.category.parentCid]
        })
      },

      remove (node, data) {
        console.log(node, data)

        this.$confirm('是否删除【' + data.name + '】菜单?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = [data.catId]
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            console.log('成功删除' + data.count + '条记录')
            this.$message({
              type: 'success',
              message: `成功删除${data.count}条记录!`
            })
            this.expandedKeys = [node.parent.data.catId]
            // 重新刷新菜单
            this.getMenus()
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      }
    },

    created () {
      this.getMenus()
    }
  }
</script>

<style>

</style>
