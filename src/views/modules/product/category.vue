/* eslint-disable standard/object-curly-even-spacing */
<!--  -->
<template>
  <div class=''>
    <el-switch v-model="draggable" active-text="开启拖拽" inactive-text="关闭拖拽"></el-switch>
    <el-button v-if="draggable" @click="batchSave">批量保存</el-button>
    <el-button type="danger" @click="batchDelete">批量删除</el-button>
    <el-tree 
      :data="menus" 
      node-key="catId"
      :props="defaultProps" 
      :expand-on-click-node="false" 
      :show-checkbox="true" 
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="handerDrop"
      ref="categoryTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }"> 
      <!-- node是节点标签，data是节点数据 -->
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">Append</el-button>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => edit(data)">Edit</el-button>
          <el-button v-if="node.childNodes.length == 0" type="text" size="mini" @click="() => remove(node, data)">Delete</el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false">
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
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data () {
    return {
      category: {
        catId: null,
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: '',
        productUnit: ''
      },
      draggable: false,
      dialogTitle: '',
      dialogType: '',
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      maxLevel: 0,
      updateNodes: [],
      pCid: []
    }
  },
  methods: {
    getCategories () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        this.menus = data.data
      })
    },
    handleNodeClick (data) {
    },
    append (data) {
      // console.log('添加分类: ', data)
      this.dialogType = 'add'
      this.dialogTitle = '添加分类'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      // 恢复默认
      this.category.catId = null
      this.category.name = null
      this.category.showStatus = 1
      this.category.sort = 0
      this.category.icon = ''
      this.category.productUnit = ''
    },
    edit (data) {
      // console.log('修改分类: ', data)
      this.dialogType = 'edit'
      this.dialogTitle = '修改分类'
      this.dialogVisible = true
      // 发送请求获取当前节点的最新数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        // 旧数据回显
        this.category.catId = data.data.catId
        this.category.name = data.data.name
        this.category.parentCid = data.data.parentCid
        this.category.catLevel = data.data.catLevel
        this.category.showStatus = data.data.showStatus
        this.category.sort = data.data.sort
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
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
    // 添加三级分类
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '分类保存成功',
          duration: 1000
        })
        this.dialogVisible = false
        this.getCategories()
        this.expandedKey = [this.category.parentCid]
      })
    },
    // 编辑三级分类
    editCategory () {
      // 解构出修改部门
      var { catId, name, icon, productUnit } = this.category
      var data = { catId, name, icon, productUnit }
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '分类修改成功',
          duration: 1000
        })
        this.dialogVisible = false
        this.getCategories()
        this.expandedKey = [this.category.parentCid]
      })
    },
    // 删除三级分类
    remove (node, data) {
      var ids = [data.catId]
      this.$confirm(`是否删除 [${data.name}] 分类?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          this.$message({
            type: 'success',
            message: '分类删除成功',
            duration: 1000
          })
          // 刷新分类
          this.getCategories()
          // 设置默认展开
          this.expandedKey = [this.category.parentCid]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除分类',
          duration: 1000
        })
      })
    },
    /**
     * 节点拖拽放置判定
     * @param draggingNode 拖动节点
     * @param dropNode     目标节点
     * @param type
     * inner插入至目标节点
     * prev放置在目标节点前
     * next放置在目标节点后
     */
    allowDrop (draggingNode, dropNode, type) {
      // console.log('allowDrop ', draggingNode, dropNode, type)
      this.countNodeLevel(draggingNode)
      // console.log('maxLevel: ', this.maxLevel)
      // 深度 = 最大层级 - 当前层级 + 1
      let depth = Math.abs(this.maxLevel - draggingNode.level + 1)
      if (type === 'inner') { // 插入
        // 被拖动的节点深度+目标节点层级 <= 3
        return depth + dropNode.level <= 3
      } else { // 放置
        // 被拖动的节点深度+父节点层级 <= 3
        return depth + dropNode.parent.level <= 3
      }
    },
    // 计算maxLevel
    countNodeLevel (node) {
      // 找到所有子节点，求出最大层级Level
      if (node.childNodes !== null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level
          }
          this.countNodeLevel(node.childNodes[i])
        }
      }
    },
    // 拖拽节点成功后对节点重新赋值
    handerDrop (draggingNode, dropNode, type, ev) {
      // 1. 当前节点最新的父节点id和兄弟节点
      let pCid = 0
      let siblings = null
      if (type === 'before' || type === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      this.pCid.push(pCid)
      // 2. 当前拖拽节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        // 遍历到的节点是当前正在拖拽的节点
        if (siblings[i].data.catId === draggingNode.data.catId) {
          // 3. 当前节点层级变化，更新当前节点和子节点的层级
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            catLevel = siblings[i].level
            // 修改子节点的层级
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel })
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i })
        }
      }
      // console.log('', this.updateNodes)
    },
    // 更新子节点的层级
    updateChildNodeLevel (node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var data = node.childNodes[i].data
          this.updateNodes.push({ catId: data.catId, catLevel: node.childNodes[i].level })
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    // 批量保存数据
    batchSave () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({ data }) => {
        this.$message({
          type: 'success',
          message: '分类顺序修改成功',
          duration: 1000
        })
        this.getCategories()
        this.expandedKey = this.pCid
        this.updateNodes = []
        this.maxLevel = 0
      })
    },
    // 批量删除数据
    batchDelete () {
      let catIds = []
      let checkedNodes = this.$refs.categoryTree.getCheckedNodes()
      console.log('Batch delete: ', checkedNodes)
      for (let i = 0; i < checkedNodes.length; i++) {
        catIds.push(checkedNodes[i].catId)
      }
      this.$confirm(`是否批量删除 [${catIds}] 分类?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(catIds, false)
        }).then(({ data }) => {
          this.$message({
            type: 'success',
            message: '分类批量删除成功',
            duration: 1000
          })
          this.getCategories()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消批量删除分类',
          duration: 1000
        })
      })
    }
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期 - 创建完成（可以访问当前this实例）
  created () {
    this.getCategories()
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {

  },
  beforeCreate () {}, // 生命周期 - 创建之前
  beforeMount () {}, // 生命周期 - 挂载之前
  beforeUpdate () {}, // 生命周期 - 更新之前
  updated () {}, // 生命周期 - 更新之后
  beforeDestroy () {}, // 生命周期 - 销毁之前
  destroyed () {}, // 生命周期 - 销毁完成
  activated () {} // 如果页面有keep-alive缓存功能，这个函数会触发
}
</script>

<style scoped>
/* @import url(); 引入公共css类 */

</style>