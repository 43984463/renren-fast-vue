<template>
  <div>
    <el-tree :data="menus" :props="defaultProps" :default-expanded-keys="expandedKeys" show-checkbox node-key="catId"
             @node-click="handleNodeClick"
             :expand-on-click-node="false"
             draggable
             :allow-drop="allowDrop"
             @node-drop="handleDrop">
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
        updateNodes: [],
        maxLevel: 0,
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
        // data.catLevel * 1作用是把data.catLevel变成数字类型
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

      allowDrop (draggingNode, dropNode, type) {
        // 被拖动的当前节点以及所在的父节点总层数不能大于3
        console.log('AllowDrop', draggingNode, dropNode, type)
        // 设置被拖动节点的最大子节点深度
        this.countNodeLevel(draggingNode.data)
        // deep 被拖动节点以及子节点的总共深度
        let deep = this.maxLevel - draggingNode.data.catLevel + 1
        if (type === 'inner') {
          return (deep + dropNode.level) <= 3
        } else {
          return (deep + dropNode.parent.level) <= 3
        }
      },

      handleDrop (draggingNode, dropNode, dropType, ev) {
        console.log('handleDrop', draggingNode, dropNode, dropType)
        // 1、 当前节点的最新的父节点id
        let pCid = 0
        // 拖拽后新的节点的兄弟节点
        let siblings = null
        if (dropType === 'inner') {
          pCid = dropNode.data.catId
          siblings = dropNode.childNodes
        } else {
          // dropType === 'before' || dropType === 'after'
          // 有可能拖动到根节点, 根节点的.parent.data变成根节点数组,根节点的.parent.data.catId变成undefined
          pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
          siblings = dropNode.parent.childNodes
        }
        // 2、 当前拖拽节点的最新顺序
        for (let i = 0; i < siblings.length; i++) {
          // 如果遍历的是当前拖拽的节点，需要添加它的父Id，然后传入后台进行修改
          // 此处所以属性的key对应后端CategoryEntity的属性名称
          if (siblings[i].data.catId === draggingNode.data.catId) {
            this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid})
          } else {
            this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
          }
        }

        // 3、 当前拖拽节点的最新层级
      },
      countNodeLevel (node) {
        // 找到所有子节点，求出最大子节点的level
        if (node.children != null && node.children.length > 0) {
          for (let i = 0; i < node.children.length; i++) {
            if (node.children[i].catLevel > this.maxLevel) {
              this.maxLevel = node.children[i].catLevel
            }
            this.countNodeLevel(node.children[i])
          }
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
