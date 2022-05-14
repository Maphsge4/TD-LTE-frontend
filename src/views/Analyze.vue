<template>
  <div id="Export">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-data"></i> 数据分析
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <el-form :inline="true">
        <el-form-item>
          <span>主邻小区C2I干扰分析</span>
        </el-form-item>
        <el-form-item>
          <el-input v-model="query.min1" placeholder="记录数量下限"> </el-input>
        </el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="handleSearch1"
          >查询信息</el-button
        >
        <el-button
          type="info"
          icon="el-icon-data-board"
          class="button"
          :disabled="state.ready < 1"
          @click="getData1"
          >显示</el-button
        >
      </el-form>
      <el-form :inline="true" :disabled="state.ready < 1">
        <el-form-item>
          <span>重叠覆盖干扰小区三元组分析</span>
        </el-form-item>
        <el-form-item>
          <el-input v-model="query.min2" placeholder="概率下限(%)"> </el-input>
        </el-form-item>
        <el-button type="primary" icon="el-icon-search" @click="handleSearch2"
          >查询信息</el-button
        >
        <el-button
          type="info"
          icon="el-icon-data-board"
          class="button"
          :disabled="state.ready < 2"
          @click="getData2"
          >显示</el-button
        >
      </el-form>
           <el-table
        v-show="state.show===1"
        :data="data.table1"
        border
        height="1000"
      >
        <el-table-column
          prop="scell"
          label="主小区"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="ncell"
          label="邻小区"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="c2iMean"
          label="平均值"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="std"
          label="标准差"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="prbABS6"
          label="绝对值小于6的概率"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="prC2I9"
          label="小于9的概率"
          align="center"
        ></el-table-column></el-table>
      <el-table
        v-show="state.show===2"
        :data="data.table2"
        border
        height="1000"
      >
        <el-table-column
          prop="sector1"
          label="小区A"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="sector2"
          label="小区B"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="sector3"
          label="小区C"
          align="center"
        ></el-table-column></el-table>
    </div>
  </div>
</template>

<script>
import { BACKEND } from "../utils/backend";
import request from "../utils/request";
export default {
  data() {
    return {
      query: {
        min1: "",
        min2: "",
      },
      state: {
        ready: 0,
        show: 0,
      },
      data: {
        table1: [],
        table2: [],
      },
    };
  },
  methods: {
    handleSearch1() {
      this.state.ready = 0;
      this.state.show = 0;
      request({
        url: `${BACKEND}/dataservice/tb-mro-data/analayzeC2I?min=${this.query.min1}`,
        method: "post",
        timeout: 30000,
      }).then((response) => {
        if (response.success) {
          this.$message.success(response.message);
          this.state.ready = 1;
        } else {
          this.$message.error(response.message);
        }
      });
    },
    getData1() {
      request({
        url: `${BACKEND}/dataservice/tb-c2-inew/show`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.$message.success(response.message);
          this.data.table1 = response.data.result;
          this.state.show = 1;
        } else {
          this.$message.error(response.message);
        }
      });
    },
    handleSearch2() {
      this.state.ready = 1;
      this.state.show &= 1;
      request({
        url: `${BACKEND}/dataservice/tb-c2-inew/analyzeTriple?min=${this.query.min2}`,
        method: "post",
        timeout: 60000,
      }).then((response) => {
        if (response.success) {
          this.$message.success(response.message);
          this.state.ready = 2;
        } else {
          this.$message.error(response.message);
        }
      });
    },
    getData2() {
      request({
        url: `${BACKEND}/dataservice/tb-c2-i3/show`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.$message.success(response.message);
          this.data.table2 = response.data.result;
          this.state.show = 2;
        } else {
          this.$message.error(response.message);
        }
      });
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
