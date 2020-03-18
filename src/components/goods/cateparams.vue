<template>
  <el-card>
    <!-- 1.面包屑 -->
    <my-bread level1="商品管理" level2="分类参数"></my-bread>
    <!-- 提示 el-alert -->
    <el-alert class="alert" title="只允许为第三季分类设置参数" type="error" center show-icon></el-alert>
    <!-- 级联选择器 -->
    <el-form class="formas" label-position="left" label-width="80px">
      <el-form-item label="商品分类">
        <el-cascader
          :show-all-levels="false"
          clearable
          expand-trigger="hover"
          :options="options"
          v-model="selectedOptions"
          :props="defaultProp"
          @change="handleChange"
        ></el-cascader>
      </el-form-item>
    </el-form>
    <el-tabs v-model="active" @tab-click="handleClick">
      <el-tab-pane label="动态参数" name="1">
        <!-- 按钮 -->
        <el-button type="danger">设置动态参数</el-button>
        <!-- 表格 -->
        <el-table :data="arrDyparams" style="width: 100%">
          <el-table-column type="expand" label="#">
            <template slot-scope="scope">
              <!-- el-tag -->
              <el-tag
                :key="tag"
                v-for="tag in scope.row.attr_vals"
                closable
                :disable-transitions="false"
                @close="handleClose(scope.row.attr_vals,scope.row.attr_id,scope.row.attr_name,tag)"
              >{{tag}}</el-tag>
              <el-input
                class="input-new-tag"
                v-if="inputVisible"
                v-model="inputValue"
                ref="saveTagInput"
                size="small"
                @keyup.enter.native="handleInputConfirm(scope.row.attr_vals,scope.row.attr_id,scope.row.attr_name)"
                @blur="handleInputConfirm(scope.row.attr_vals)"
              ></el-input>
              <el-button v-else class="button-new-tag" size="small" @click="showInput">+ New Tag</el-button>
            </template>
          </el-table-column>
          <el-table-column label="属性名称" prop="attr_name"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                plain
                type="primary"
                icon="el-icon-edit"
                circle
                @click="showEditDyCateDia(scope.row)"
              ></el-button>
              <el-button
                size="mini"
                plain
                type="danger"
                icon="el-icon-delete"
                circle
                @click="showDeleDyCateMsgBox(scope.row.id)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
      <el-tab-pane label="静态参数" name="2">
        <!-- 按钮 -->
        <el-button type="danger">设置静态参数</el-button>
        <!-- 表格 -->
        <el-table :data="arrStaticparams" style="width: 100%">
          <el-table-column type="index" label="#">            
          </el-table-column>
          <el-table-column label="属性名称" prop="attr_name"></el-table-column>
          <el-table-column label="属性值" prop="attr_vals"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button
                size="mini"
                plain
                type="primary"
                icon="el-icon-edit"
                circle
                @click="showEditStaticCateDia(scope.row)"
              ></el-button>
              <el-button
                size="mini"
                plain
                type="danger"
                icon="el-icon-delete"
                circle
                @click="showDeleStaticCateMsgBox(scope.row.id)"
              ></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-tab-pane>
    </el-tabs>
  </el-card>
</template>
<script>
export default {
  data() {
    return {
      //级联选择器绑定的数据
      options: [],
      selectedOptions: [],
      defaultProp: {
        label: "cat_name",
        value: "cat_id",
        children: "children"
      },
      arrDyparams: [],
      arrStaticparams:[],
      active: 1,
      inputVisible: false,
      inputValue: ""
    };
  },
  created() {
    this.getGoodCate();
  },
  methods: {
    //级联选择器@change触发的方法
    async handleChange() {
      if (this.selectedOptions.length === 3) {
        //获取动态参数数据
        const res = await this.$http.get(
          `categories/${this.selectedOptions[2]}/attributes?sel=many`
        );

        this.arrDyparams = res.data.data;
        this.arrDyparams.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.trim().split(",");
        });
        console.log(this.arrDyparams);
      }
    },
    //获取三级分类数据
    async getGoodCate() {
      const res = await this.$http.get(`categories?type=3`);
      //console.log(res)
      this.options = res.data.data;
    },
    //tab切换时
    async handleClick() {
      if (this.selectedOptions.length === 3) {
        //获取静态参数数据
        const res = await this.$http.get(`categories/${this.selectedOptions[2]}/attributes?sel=only`)
        //console.log(res)
        this.arrStaticparams = res.data.data
      }
    },
    //点击×按钮
    async handleClose(attr_vals,attr_id,attr_name,tag) {
      attr_vals.splice(attr_vals.indexOf(tag), 1);
      //发送请求
      //categories/:id/attributes/:attrId    put请求
      //put 请求体{attr_name:?,attr_sel:?,attr_vals:?}
      //id -> 分类id
      //attrId -> 参数id
      let putData={
          attr_name:attr_name,
          attr_sel:'many',
          attr_vals:attr_vals.join(',')
      }
      const res = await this.$http.put(`categories/${this.selectedOptions[2]}/attributes/${attr_id}`,putData)
      console.log(res)  
    },
    //点击newTag+按钮
    showInput() {
      this.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    //回车键or失去焦点
    async handleInputConfirm(attr_vals,attr_id,attr_name) {
      let inputValue = this.inputValue;
      if (inputValue) {
        attr_vals.push(inputValue);
        //发送请求
        let putData={
          attr_name:attr_name,
          attr_sel:'many',
          attr_vals:attr_vals.join(',')
        }
        const res = await this.$http.put(`categories/${this.selectedOptions[2]}/attributes/${attr_id}`,putData)
        console.log(res)  
      }
      this.inputVisible = false
      this.inputValue = ""
    }
  }
};
</script>
<style>
.alert {
  margin-top: 20px;
}
.formas {
  margin-top: 20px;
}
.el-tag + .el-tag {
  margin-left: 10px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: 90px;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>