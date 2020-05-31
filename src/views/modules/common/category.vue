<template>
  <div>
    <el-input placeholder="产品分类" clearable v-model="categroy" style="width: 90%"></el-input>
    <el-tree :data="menus"
             :props="defaultProps"
             node-key="catId"
             ref="menuTrees"
             :filter-node-method="filterNode"
             empty-text="正在获取数据，请稍候。。。"
             highlight-current
             @node-click="nodeClick">
    </el-tree>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        menus: [],
        categroy: '',
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
        }).then(({data}) => {
          console.log('成功获取数据', data.data)
          this.menus = data.data
        })
      },
      // data 传递的数据 node当前节点 component整个树组件
      nodeClick (data, node, component) {
        // 向父组件传递事件tree-node-click 携带任意个参数data, node, component
        this.$emit('tree-node-click', data, node, component)
      },
      // 筛选节点
      filterNode (value, data) {
        console.log(value, data)
        if (!value) return true
        return data.name.indexOf(value) !== -1 && data.catLevel === 3
      }
    },
    created () {
      this.getMenus()
    },
    watch: {
      // 根据名称筛选产品树
      categroy (val) {
        this.$refs.menuTrees.filter(val)
      }
    }
  }
</script>

<style>

</style>
