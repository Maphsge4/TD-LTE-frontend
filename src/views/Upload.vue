<template>
  <div id="Upload">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-data"></i> 数据管理
        </el-breadcrumb-item>
        <el-breadcrumb-item>数据导入</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <el-select
        v-model="state"
        placeholder="关系表"
        style="width: 400px"
        @change="handleChange"
      >
        <el-option
          v-for="item in tableList2"
          :key="item.label"
          :label="item.label"
          :value="item"
        >
        </el-option>
      </el-select>
      <el-upload
        v-show="this.state.label"
        class="upload"
        drag
        :action="this.urlUpload"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
        <template #tip>
          <div class="el-upload__tip">上传 xlsx 文件</div>
        </template>
      </el-upload>
    </div>
  </div>
</template>

<script>
import { BACKEND } from "../utils/backend";
import { tableList } from "../utils/table";
import { tableList2 } from "../utils/table";
export default {
  data() {
    return {
      urlUpload: "",
      state: {
        label: "",
        value: "",
        name: "",
      },
      tableList: tableList,
      tableList2:tableList2,
    };
  },
  methods: {
    handleChange() {
      this.urlUpload = `${BACKEND}/dataservice/${this.state.value}/upload`;
    },
    handleExport() {
      if (this.state.name) {
        window.location.href = `${BACKEND}/dataservice/${this.state.value}/export?filename=${this.state.name}`;
      } else {
        this.$message.error("请选择一张关系表");
      }
    },
  },
};
</script>
