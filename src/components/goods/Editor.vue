<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-alert
        title="编辑商品信息"
        :closable="false"
        center
        show-icon
        type="info"
      >
      </el-alert>
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品信息"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <el-form
        ref="editorFormRef"
        :model="goodlist"
        :rules="editorFormRules"
        label-width="80px"
        label-position="top"
      >
        <el-tabs
          v-model="activeIndex"
          :before-leave="beforTabLeave"
          :tab-position="'left'"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="goodlist.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="goodlist.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input
                v-model="goodlist.goods_weight"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input
                v-model="goodlist.goods_number"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品分类：" prop="goods_cat">
              <el-cascader
                :options="catelist"
                :props="cateProps"
                clearable
                v-model="goodlist.goods_cat"
                @change="CateChanged"
              >
              </el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTabelData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  :label="cb"
                  border
                  v-for="(cb, i) in item.attr_vals"
                  :key="i"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTabelData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              action="http://127.0.0.1:8888/api/private/v1/upload"
              :headers="headerobj"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              :on-success="handleSuccess"
              list-type="picture"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="goodlist.goods_introduce"></quill-editor>
            <el-button type="primary" class="btnAdd" @click="editor"
              >修改商品</el-button
            >
            <el-button type="primary" class="btnAdd" @click="Gohome"
              >退出</el-button
            >
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg" />
    </el-dialog>
  </div>
</template>

<script>
import _ from "lodash";

export default {
  data() {
    return {
      previewPath: "",
      previewVisible: false,
      activeIndex: "0",
      catelist: [],
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10,
      },
      total: 0,
      editorFormRules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" },
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" },
        ],
        goods_wight: [
          { required: true, message: "请输入商品重量", trigger: "blur" },
        ],
        goods_number: [
          { required: true, message: "请输入商品数量", trigger: "blur" },
        ],
        goods_cat: [
          { required: true, message: "请选择商品分类", trigger: "blur" },
        ],
      },
      goodlist: {
        // goods_name: "",
        // goods_price: 0,
        // goods_weight: 0,
        // goods_number: 0,
        // goods_cat: [],
        // pics: [],
        // goods_introduce: "",
        // attrs: [],
      },
        uId: "",
      manyTabelData: [],
      onlyTabelData: [],
      cateProps: {
        expandTrigger: "hover",
        value: "cat_id",
        label: "cat_name",
        children: "children",
      },
      headerobj: {
        Authorization: window.sessionStorage.getItem("token"),
      },
    };
  },
  methods: {
    async getCateList() {
      const { data } = await this.$http.get("categories");
      if (data.meta.status !== 200) {
        return this.$message.error(data.meta.msg);
      }
      this.catelist = data.data;
    },
    async getGoodList() {
      let urlId = window.location.href;
      const idArr = urlId.split("&");
      this.uId = idArr[1];
      const { data } = await this.$http.get("goods/" + idArr[1]);
      if (data.meta.status !== 200) {
        return this.$message.error(data.meta.msg);
      }
      this.goodlist = data.data;
    },
    beforTabLeave(activeName, oldActiveName) {
      if (oldActiveName === "0" && this.goodlist.goods_cat.length !== 3) {
        this.$message.error("请先选择分类！");
        return false;
      }
    },
    async tabClicked() {
      if (this.activeIndex === "1") {
        const { data } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: "many",
            },
          }
        );
        if (data.meta.status !== 200) {
          return this.$message.error(data.meta.msg);
        }
        data.data.forEach((item) => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(" ");
        });
        this.manyTabelData = data.data;
      } else if (this.activeIndex === "2") {
        const { data } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: "only",
            },
          }
        );
        if (data.meta.status !== 200) {
          return this.$message.error(data.meta.msg);
        }
        this.onlyTabelData = data.data;
      }
    },
    CateChanged() {
      if (this.goodlist.goods_cat.length !== 3) {
        this.goodlist.goods_cat = [];
        this.$message.error("请选择三级分类");
      }
    },
    handlePreview(file) {
      this.previewPath = file.response.data.url;
      this.previewVisible = true;
    },
    handleRemove(fire) {
      const picPath = fire.response.data.tmp_path;
      const i = this.goodlist.pics.findIndex((x) => x.pic === picPath);
      this.goodlist.pics.splice(i, 1);
    },
    handleSuccess(response) {
      const picInfo = { pic: response.data.tmp_path };
      this.goodlist.pics.push(picInfo);
      // console.log(this.addForm)
    },
    editor() {
      this.$refs.editorFormRef.validate(async (v) => {
        if (!v) return this.$message.err("请填写必填项!");
        const form = _.cloneDeep(this.goodlist);
        form.goods_cat = form.goods_cat.join(",");
        this.manyTabelData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(" "),
          };
          this.goodlist.attrs.push(newInfo);
        });
        this.onlyTabelData.forEach((item) => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals,
          };
          this.goodlist.attrs.push(newInfo);
        });
        form.attrs = this.goodlist.attrs;
        form.goods_name=this.goodlist.goods_name.toString()
        console.log(this.goodlist);

        console.log(form.goods_name)
        const { data } = await this.$http.put("goods/"+this.uId,form);
        // console.log(data)
        if (data.meta.status !== 200) {
          return this.$message.error(data.meta.msg);
        }
        this.$message.success("修改商品成功！");
        this.$router.push("/goods");
      });
    },
        Gohome() {
      this.$router.push('/goods')
    }
  },
  created() {
    this.getCateList();
    this.getGoodList();
  },
  computed: {
    cateId() {
      if (this.goodlist.goods_cat.length === 3) {
        return this.goodlist.goods_cat[2];
      }
      return null;
    },
  },
};
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}

.previewImg {
  width: 100%;
}

.btnAdd {
  margin-top: 15px;
}
</style>
