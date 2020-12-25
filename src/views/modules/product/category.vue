<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-checked-keys="expandedKey"
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
    <el-dialog title="添加分类" :visible.sync="dialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
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
      //表单的值
      category: { name: "", parentCid: 0, catLevel: 0, showStatus: 1, sort: 0 },
      //是否可见
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取菜单数据....", data.data);
        this.menus = data.data;
      });
    },

     //添加三级分类
    addCategory() {
      //点击添加分类,  关闭表单
      this.dialogVisible = false;
      console.log("提交的表单数据:  ", this.category);

      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        //添加成功后的操作
        this.$message({
          type: "success",
          message: "分类添加成功!",
        });
        this.dialogVisible = false;
        this.getMenus();
        this.expandedkey = [this.category.parentCid];
        this.category.name = "";
      });
    },

    append(data) {
      //点击 append 弹出添加分类的对话框
      this.dialogVisible = true;
      //父级的id
      this.category.parentCid = data.catId;
      //分类的层级, 当前层级, 加一
      this.category.catLevel = data.catLevel * 1 + 1;
      //console.log("append", data);
    },

    remove(node, data) {
      var ids = [data.catId];
      //删除弹窗实现
      this.$confirm(`是否删除 [ ${data.name} ] 菜单?`, "提示", {
        confirmButtonText: "确定", // 点击确定, 触发then
        cancelButtonText: "取消", // 点击取消 触发 catch ,由于取消不需要任何的操作, 但删除catch会报错, 因此保留空方法
        type: "warning",
      })
        .then(() => {
          var idArr = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(idArr, false),
          }).then(({ data }) => {
            this.$message({
              type: "success",
              message: "删除成功!",
            });
            //刷新出新的菜单
            this.getMenus();
            //设置需要默认展开的菜单
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});
      console.log("remove", node, data);
    },
  },
  //计算属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合

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
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>