<template>
  <div id="Export">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-data"></i> 数据管理
        </el-breadcrumb-item>
        <el-breadcrumb-item>数据导出</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <el-select v-model="state" placeholder="关系表" style="width: 400px">
        <el-option
          v-for="item in tableList"
          :key="item.label"
          :label="item.label"
          :value="item"
        >
        </el-option>
      </el-select>
      <el-button type="success" @click="handleExport"
        >下载<i class="el-icon-download el-icon--right"></i
      ></el-button>
    </div>
  </div>
</template>

<script>
import { BACKEND } from "../utils/backend";
import { tableList } from "../utils/table";
export default {
  data() {
    return {
      state: {
        label: "",
        value: "",
        name: "",
      },
      tableList: tableList,
    };
  },
  methods: {
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