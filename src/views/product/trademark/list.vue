<template>
  <div>
    <!-- 按钮 -->
    <el-button
      type="primary"
      icon="el-icon-plus"
      @click="showAdd"
      style="margin-bottom:20px"
    >
      添加商品
    </el-button>

    <!-- 表各 -->
    <el-table v-loading="loading" :data="trademarks" border stripe>
      <!-- 序号列 -->
      <el-table-column label="序号" type="index" width="80" align="center">
      </el-table-column>
      <el-table-column prop="tmName" label="品牌名称"> </el-table-column>
      <el-table-column label="姓名">
        <template slot-scope="{ row }">
          <img :src="row.logoUrl" alt="" style="width: 100px; height: 60px;" />
        </template>
      </el-table-column>
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
    </el-table>

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

    <!-- 弹窗 -->
    <el-dialog
      :title="form.id ? '更新品牌' : '添加品牌'"
      :visible.sync="isShowDialog"
    >
      <el-form :model="form" style="width: 80%" :rules="rules" ref="tmForm">
        <el-form-item
          label="品牌名称"
          :label-width="formLabelWidth"
          prop="tmName"
        >
          <el-input
            v-model="form.tmName"
            autocomplete="off"
            placeholder="请输入品牌名称"
          ></el-input>
        </el-form-item>
        <el-form-item
          label="品牌LOGO"
          :label-width="formLabelWidth"
          prop="logoUrl"
        >
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleLogoSuccess"
            :before-upload="beforeLogoUpload"
          >
            <img v-if="form.logoUrl" :src="form.logoUrl" class="avatar" />
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            <div class="el-upload__tip" slot="tip">
              只能上传jpg/png文件，且不超过500kb
            </div>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="isShowDialog = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTrademark">
          确 定
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "TrademarkList",
  data() {
    return {
      trademarks: [], //当前页的列表数据
      total: 0, //总数量
      page: 1, //当前页码
      limit: 5, //每页数量
      loading: false, //图片加载

      isShowDialog: false, //是否显示弹窗
      form: {
        tmName: "", //品牌名称
        logoUrl: "" //品牌logo地址
      },
      formLabelWidth: "100px",

      // dialog表单验证
      rules: {
        tmName: [
          { required: true, message: "请输入品牌名称", trigger: "change" }, //值发生改变时触发
          // { min: 2, max: 10, message: "长度在 2 到 10 个字符", trigger: "blur" } //失去焦点触发
          { validator: this.validateTmName, trigger: "blur" } //validator校验器: 用于校验的回调函数
        ],
        logoUrl: [{ required: true, message: "请添加品牌LOGO图片" }]
      }
    };
  },
  mounted() {
    // 测试定义的接口函数
    // this.$API.trademark.getById(1);
    this.getTrademarks();
  },
  methods: {
    // 当上传图片成功时调用
    handleLogoSuccess(res, file) {
      // 得到返回的logo的url
      const logoUrl = res.data;
      this.form.logoUrl = logoUrl;
    },

    // 指定在准备发送上传图片请求前的回调函数, 如果返回值为false, 不会发出请求
    //   作用: 限制上传图片的类型与大小
    //   要求: 只能上传jpg/png文件，且不超过500kb
    beforeLogoUpload(file) {
      const isJPGOrPng =
        ["image/jpeg", "image/png", "image/jpg"].indexOf(file.type) >= 0;
      const isLt500KB = file.size / 1024 < 500;
      if (!isJPGOrPng) {
        this.$message.error("上传头像图片只能是 JPG/PNG.JPG 格式!");
      }
      if (!isLt500KB) {
        this.$message.error("上传头像图片大小不能超过 500kb!");
      }
      return isJPGOrPng && isLt500KB;
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
    },

    // 校验品牌名称的自定义校验函数
    validateTmName(rule, value, callback) {
      if (value.length < 2 || value.length > 10) {
        callback(new Error("长度在 2 到 10 个字符"));
      } else {
        callback();
      }
    },

    // 添加或更新品牌 校验品牌
    addOrUpdateTrademark() {
      // 先进行表单验证
      this.$refs.tmForm.validate(async valid => {
        if (valid) {
          // 取得请求需要的数据
          const trademark = this.form;
          let result;
          // 判断trademark中有id 更新请求 否则添加
          if (trademark.id) {
            result = await this.$API.trademark.update(trademark);
          } else {
            result = await this.$API.trademark.add(trademark);
          }

          if (result.code === 200) {
            // 成功，重新显示新的品牌列表，隐藏dialog 提示
            this.$message.success(`${trademark.id ? "更新" : "添加"}品牌成功`);
            this.isShowDialog = false;
            this.getTrademarks(trademark.id ? this.page : 1);
            // 清除数据
            this.form = {
              tmName: "",
              logoUrl: ""
            };
          } else {
            // 失败，提示错误信息
            this.$message.error(`${trademark.id ? "更新" : "添加"} 品牌失败`);
          }
        } else {
          //没通过，什么也不做（可以不写else）
        }
      });
    },

    // 显示添加弹窗
    showAdd() {
      // 清除上一次的数据
      this.form = {
        tmName: "",
        logoUrl: ""
      };
      this.isShowDialog = true;
    },

    // 修改品牌信息
    showUpdate(trademark) {
      // 将当前的form数据保存到trademark中
      this.form = { ...trademark }; //form和trademark指向不再是同一个对象   对象的浅拷贝
      // 显示需要修改的dialog
      this.isShowDialog = true;
    },

    // 删除指定的品牌
    deleteTrademark(trademark) {
      this.$confirm(`确定删除 ${trademark.tmName} 吗?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(async () => {
          // 点击确定
          // 发送删除品牌的请求
          const result = await this.$API.trademark.remove(trademark.id);
          // 如果成功了
          if (result.code === 200) {
            this.$message.success("删除品牌成功!");
            const page =
              this.trademarks.length === 1 ? this.page - 1 : this.page;
            this.getTrademarks(page);
          } else {
            // 如果失败了
            this.$message.error("删除品牌失败!");
          }
        })
        .catch(() => {
          // 点击取消
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
