<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      @node-click="handleNodeClick"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expendedKey"
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
            Update
          </el-button>

          <el-button
            v-if="node.childNodes == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog title="新增商品" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="商品名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory(category)"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <el-dialog title="更新商品" :visible.sync="editDialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="商品名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateCategory(category)"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    return {
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      editDialogVisible: false,
      dialogVisible: false,
      menus: [],
      expendedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    handleNodeClick(data) {},
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据。。。", data.data);
        this.menus = data.data;
      });
    },
    append(node, data) {
      this.dialogVisible = true;
      console.log("更新：" + node);
    },
    edit(data) {
      console.log(data);

      //todo 这样回显的不是最新数据，应该每次都从服务器去拉去最新的数据
      //todo 注意更新完成之后将category置为初始值
      this.category = { ...data };
      console.log(this.category);
      this.editDialogVisible = true;
    },
    remove(node, data) {
      console.log("删除" + data);
      let ids = [data.catId];

      this.$confirm(`是否删除【${data.name}】菜单`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            this.$message({
              message: "删除成功！",
              type: "success",
            });
            this.getMenus();
            this.expendedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {
          console.log("取消删除");
        });
    },

    addCategory(param) {
      console.log("正在添加数据");
      console.log(param);
      //1.发送表单添加商品
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(param, false),
      }).then(({ data }) => {
        this.$message({
          message: "添加成功！",
          type: "success",
        });
        //2.关闭弹框
        this.dialogVisible = false;
        this.getMenus();
        this.expendedKey = [param.parentCid];
      });
    },
    updateCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        //2.关闭弹框
        this.editDialogVisible = false;
        this.getMenus();
        this.expendedKey = [this.category.parentCid];
      });
    },
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
</style>