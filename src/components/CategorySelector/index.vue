<template>
  <!--查询表单-->
  <el-form :inline="true" :model="cform" class="demo-form-inline">
    <!-- 一级分类 -->
    <el-form-item label="一级分类">
      <el-select
        :disabled="disabled"
        v-model="cform.category1Id"
        placeholder="请选择一级分类"
        @change="handleChange1"
      >
        <el-option
          v-for="category in category1List"
          :key="category.id"
          :label="category.name"
          :value="category.id"
        />
      </el-select>
    </el-form-item>

    <!-- 二级分类 -->
    <el-form-item label="二级分类">
      <el-select
        :disabled="disabled"
        v-model="cform.category2Id"
        placeholder="请选择二级分类"
        @change="handleChange2"
      >
        <el-option
          v-for="category in category2List"
          :key="category.id"
          :label="category.name"
          :value="category.id"
        />
      </el-select>
    </el-form-item>

    <!-- 三级分类 -->
    <el-form-item label="三级分类">
      <el-select
        :disabled="disabled"
        v-model="cform.category3Id"
        placeholder="请选择三级分类"
        @change="handleChange3"
      >
        <el-option
          v-for="category in category3List"
          :key="category.id"
          :label="category.name"
          :value="category.id"
        />
      </el-select>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  name: "CategorySelector",
  data() {
    return {
      disabled: false, // 下拉列表是否禁用
      cform: {
        category1Id: "",
        category2Id: "",
        category3Id: ""
      },
      category1List: [], // 一级分类列表
      category2List: [], // 二级分类列表
      category3List: [] // 三级分类列表
    };
  },

  mounted() {
    this.getCategory1List();
  },

  methods: {
    // 一级分类列表
    async getCategory1List() {
      const result = await this.$API.category.getCategorys1();
      this.category1List = result.data;
    },

    // 处理选中一级分类项的监听回调，获取二级分类列表
    async handleChange1(category1Id) {
      // 重置二级与三级分类
      this.cform.category2Id = "";
      this.cform.category3Id = "";
      this.category2List = [];
      this.category3List = [];
      // 分发事件，通知父元素
      this.$emit("categoryChange", { categoryId: category1Id, level: 1 });
      // 获取二级分类列表
      const result = await this.$API.category.getCategorys2(category1Id);
      this.category2List = result.data;
    },

    // 处理选中二级分类项的监听回调，获取三级分类列表
    async handleChange2(category2Id) {
      // 重置与三级分类
      this.cform.category3Id = "";
      this.category3List = [];
      // 分发事件，通知父元素
      this.$emit("categoryChange", { categoryId: category2Id, level: 2 });
      // 获取三级分类列表
      const result = await this.$API.category.getCategorys3(category2Id);
      this.category3List = result.data;
    },

    // 处理三级分类项的监听回调
    async handleChange3(category3Id) {
      // 分发事件，通知父元素
      this.$emit("categoryChange", { categoryId: category3Id, level: 3 });
    }
  }
};
</script>
