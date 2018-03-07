<template>
  <div class="list">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 按钮组, 带搜索框 -->
    <section class="list_btns">
      <el-button plain size="mini" icon="el-icon-plus" @click="add">新增</el-button>
      <el-button plain size="mini" icon="el-icon-check" @click="all">全选</el-button>
      <el-button plain size="mini" icon="el-icon-close" @click="del">删除</el-button>

      <div class="list_btns_right">
        <el-input placeholder="请输入商品名称" prefix-icon="el-icon-search" label-width="200px" size="mini" v-model="apiQuery.searchvalue" @blur="search">
        </el-input>
      </div>
    </section>

    <!-- 大表格 -->
    <!-- data属性用来配置表格数据  -->
    <el-table @selection-change="change" ref="multipleTable" :data="tableData3" style="width: 100%">

      <!-- type为selection, 即多选框 -->
      <el-table-column type="selection" width="55"></el-table-column>

      <!-- label用来设置当前列的表头 -->
      <!-- 里面的template用来自定义表格中的内容与数据, 相比较prop属性的方式, 更加灵活, 可以对数据进行标签包裹 -->
      <el-table-column label="标题">
        <template slot-scope="scope">
          <el-tooltip class="item" effect="dark" content="Right Center 提示文字" placement="right">
            <router-link style="color: #666;" :to="{path:`/admin/goods/detail/${scope.row.id}` }">{{ scope.row.title }}</router-link>
            <div slot="content">
              <img :src="scope.row.imgurl" alt="商品图片" style="width:150px;">
            </div>
            <el-button>右边</el-button>
          </el-tooltip>

        </template>
      </el-table-column>

      <!-- 当前列要展示对象中的那个字段的值, 就配置prop属性为那个字段名 -->
      <el-table-column prop="categoryname" label="所属类别" width="120"></el-table-column>

      <!-- 当前列要展示对象中的那个字段的值, 就配置prop属性为那个字段名 -->
      <el-table-column prop="stock_quantity" label="库存" width="120" show-overflow-tooltip></el-table-column>

      <el-table-column prop="market_price" label="市场价" width="120" show-overflow-tooltip></el-table-column>

      <el-table-column prop="sell_price" label="销售价" width="120" show-overflow-tooltip></el-table-column>

      <el-table-column label="属性" width="120" show-overflow-tooltip>
        <!-- 注意template要加slot-scope属性 -->
        <template slot-scope="scope">
          <!-- 三元表达式 -->
          <span :class="['el-icon-picture-outline',scope.row.is_slide == 1?'active':'']"></span>
          <span :class="['el-icon-upload2',scope.row.is_slide == 1?'':'active']"></span>
          <span :class="['el-icon-star-off',scope.row.is_slide == 1?'active':'']"></span>
        </template>
      </el-table-column>

      <el-table-column label="操作" width="120" show-overflow-tooltip>
        <template slot-scope="scope">
          <router-link style="color: #666;" :to="{ path:`/admin/goods/detail/:id` }">修改</router-link>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <!-- total用来设定数据总数，current-page用来设定当前页，page-size用来设定当前每页数量 -->

    <!-- <span class="demonstration">完整功能</span> -->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="apiQuery.pageIndex" :page-sizes="[2, 4, 6, 8]" :page-size="apiQuery.pageSize" layout="total, sizes, prev, pager, next, jumper" :total="apiQuery.total">
    </el-pagination>

  </div>
</template>

<script>
export default {
  data() {
    return {
      // 搜索
      apiQuery: {
        pageIndex: 1,
        pageSize: 2,
        searchvalue: "",
        total: 0
      },

      // 被选中的商品数据
      selectedGoodsList: [],

      // 表格数据
      tableData3: [
        {
          date: "2016-05-03",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        },
        {
          date: "2016-05-03",
          name: "王小虎",
          address: "上海市普陀区金沙江路 1518 弄"
        }
      ]
    };
  },

  //编写供自己使用的方法
  methods: {
    // 搜索
    search() {
      this.getGoodsData();
    },

    // 获取商品列表数据
    getGoodsData() {
      // 这个接口需要pageIndex指定页, pageSize指定每页数量, searchvalue用于商品搜索
      let api = `${this.$api.gsList}?pageIndex=${
        this.apiQuery.pageIndex
      }&pageSize=${this.apiQuery.pageSize}&searchvalue=${
        this.apiQuery.searchvalue
      }`;
      this.$http.get(api).then(res => {
        if (res.data.status == 0) {
          this.tableData3 = res.data.message; // 把请求回来的数据覆盖原data数量, 表格就会自动刷新

          //后端接口返回的数量总量赋值给分页组件使用
          this.apiQuery.total = res.data.toalcount;
        }
      });
    },

    // 监听多选框状态的变化, 参数可以拿到被选的商品数据
    change(selection) {
      console.log(selection);
      this.selectedGoodsList = selection;
    },

    // 删除按钮
    del() {
      let delIDS = this.selectedGoodsList.map(v => v.id); // 提取所有被选商品的id
      this.$http.get(this.$api.gsDel + delIDS).then(res => {
        // 删除成功后重新获取数据进行渲染
        if (res.data.status == 0) {
          this.getGoodsData();
        }
      });
      // this.$alert('请问删除吗？')
    },

    all() {
      document.querySelector(".el-checkbox__inner").click();
    },
    add(){
        
        this.$router.push({path:`/admin/goods/detail/id`});
    },

    handleSizeChange(size) {
      //接收到新的页面，赋值给data里的数量，分页组件就会刷新视频
      this.apiQuery.pageSize = size;
      this.getGoodsData();
      // console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(page) {
      this.apiQuery.pageIndex = page;
      this.getGoodsData();
      // console.log(`当前页: ${val}`);
    }
  },

  // 页面一上来就自动调用接口获取表格数据进行展示
  created() {
    this.getGoodsData();
  }
};
</script>

<style scoped lang="less">
.list {
  // 按钮组
  &_btns {
    margin-top: 20px;
    margin-bottom: 20px;

    &_right {
      float: right;
      width: 200px;
    }
  }

  //添加icon点亮的样式
  [class^="el-icon"].active {
    color: rgb(44, 93, 255);
    font-weight: 600;
  }
}
</style>
