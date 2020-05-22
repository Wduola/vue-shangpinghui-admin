<template>
  <div>
    <!-- 三级分类 -->
    <el-card class="box-card">
      <CategorySelector @categoryChange="handleCategoryChange" />
    </el-card>

    <!-- spu列表 -->
    <el-card class="box-card" border stripe style="margin-top:20px">
      <div v-show="isShowList">
        <el-table :data="attrs" border style="width: 100%; margin-top: 20px">
          <el-table-column type="index" label="序号" width="80" align="center">
          </el-table-column>
          <el-table-column prop="attrName" label="SPU名称" width="400">
          </el-table-column>
          <el-table-column prop="attrValueList" label="SPU描述">
          </el-table-column>
          <el-table-column prop="amount2" label="操作" width="150">
            <!-- 如果列表列的内容是指定指定的结构: 使用作用域插槽 -->
            <el-table-column prop="address" label="操作">
              <template slot-scope="{ row }">
                <el-button
                  type="warning"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showUpdate(row)"
                  >修改</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteTrademark(row)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table-column>
        </el-table>
      </div>
    </el-card>

    <!-- 分页 -->
    <el-pagination
      background
      style="text-align: center; margin-top: 20px;"
      :total="total"
      :current-page="page"
      :page-sizes="[5, 10, 15, 20]"
      :page-size="limit"
      layout="prev, pager, next, jumper, ->,sizes, total"
      @current-change="getTrademarks"
      @size-change="handleSizeChange"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: "SpuList",

  data() {
    return {
      category1Id: "", // 一级分类ID
      category2Id: "", // 二级分类ID
      category3Id: "", // 三级分类ID
      attrs: [], // 所有属性的列表

      isShowList: true, // 是否显示属性列表页面   true: 列表页面, false: 添加或更新页面

      attr: {
        // 要添加或者修改的平台属性对象
        attrName: "", // 属性名
        attrValueList: [], //属性值的列表
        categoryId: "", // 3级的分类ID
        categoryLevel: 3 // 只能是3级
      },

      trademarks: [], //当前页的列表数据
      total: 0, //总数量
      page: 1, //当前页码
      limit: 5, //每页数量
      loading: false //图片加载
    };
  },

  mounted() {
    this.category1Id = 2;
    this.category2Id = 13;
    this.category3Id = 61;
    this.getAttrs();
  },

  methods: {
    // 3个级别分类发生改变时的监听回调
    handleCategoryChange({ categoryId, level }) {
      // console.log('handleCategoryChange', categoryId, level)
      if (level === 1) {
        this.category1Id = categoryId;
        // 重置2级和3级ID和属性列表
        this.category2Id = "";
        this.category3Id = "";
        this.attrs = [];
      } else if (level === 2) {
        this.category2Id = categoryId;
        // 重置3级ID和属性列表
        this.category3Id = "";
        this.attrs = [];
      } else {
        this.category3Id = categoryId;
        // 异步请求获取属性列表显示
        this.getAttrs();
      }
    },

    // 异步请求获取属性列表显示
    async getAttrs() {
      const { category1Id, category2Id, category3Id } = this;
      const result = await this.$API.attr.getList(
        category1Id,
        category2Id,
        category3Id
      );
      this.attrs = result.data;
    },

    // 更新每页数据函数
    async getTrademarks(page = 1) {
      this.page = page;
      // 发送请求前获取数据，显示loading
      this.loading = true;
      // 发异步ajax请求获取数据
      const result = await this.$API.trademark.getList(page, this.limit);
      // 数据请求完成，隐藏loading
      this.loading = false;
      if (result.code === 200) {
        // 成功，更新列表数据
        const { records, total } = result.data;
        this.trademarks = records;
        this.total = total;
      } else {
        // 失败，提示错误信息
        this.$message.error("获取品牌列表数据失败");
      }
    },

    // 每页数量改变的监听回调
    handleSizeChange(pageSize) {
      // 更新limit数据
      this.limit = pageSize;
      // 重新请求获取第一页列表显示
      this.getTrademarks();
    }
  }
};
</script>
