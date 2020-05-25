<template>
  <el-tree :data="menus" :props="defaultProps" :default-expanded-keys="expandedKeys" show-checkbox node-key="catId" @node-click="handleNodeClick"
           :expand-on-click-node="false">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)"> 添加 </el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini"
                     @click="() => remove(node, data)"> 删除 </el-button>
        </span>
      </span>
  </el-tree>
</template>

<script>
  export default {
    data() {
      return {
        menus: [],
        expandedKeys: [],
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
    },

    methods: {
      handleNodeClick(data) {
        console.log(data);
      },

      getMenus() {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data}) => {
          console.log("成功获取数据", data.data);
          this.menus = data.data;
        })
      },

      append(data) {
        console.log(data);
      },

      remove(node, data) {
        console.log(node, data);

        this.$confirm('是否删除【'+data.name+'】菜单?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            console.log("成功删除"+data.count+"条记录");
            this.$message({
              type: 'success',
              message: `成功删除${data.count}条记录!`
            });
            this.expandedKeys = [node.parent.data.catId];
            // 重新刷新菜单
            this.getMenus();
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });

      },
    },

    created() {
      this.getMenus();
    }
  };
</script>

<style>

</style>
