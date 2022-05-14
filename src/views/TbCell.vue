<template>
  <div id="TbCell">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-order"></i> 业务查询
        </el-breadcrumb-item>
        <el-breadcrumb-item>tbCell</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-form :inline="true" model="formdata">
          <!-- <el-button icon="el-icon-download" @click="handleExport"
            >导出文件</el-button
          > -->
          <el-form-item>
            <el-select v-model="select.type" clearable placeholder="查询依据">
              <el-option label="小区ID" :value="1"></el-option>
              <el-option label="小区名称" :value="2"></el-option>
              <el-option label="小区所属基站标识" :value="3"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-input
              v-model="query.sectorID"
              v-if="select.type === 1"
              placeholder="小区ID"
              @change="sectorIDSet"
            >
            </el-input>
            <el-select
              v-model="query.sectorName"
              v-else-if="select.type === 2"
              filterable
              placeholder="小区名称"
              @change="sectorNameSet"
            >
              <el-option
                v-for="item in sectorName"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
            <el-select
              v-model="query.eNodeBID"
              v-else-if="select.type === 3"
              filterable
              placeholder="小区所属基站标识"
              @change="eNodeBIDSet"
            >
              <el-option
                v-for="item in eNodeBList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-button type="primary" icon="el-icon-search" @click="handleSearch"
            >查询信息</el-button
          >
          <el-button
            type="info"
            icon="el-icon-refresh"
            class="button"
            @click="getData"
            >刷新</el-button
          >
        </el-form>
      </div>
      <el-table
        v-show="!table.hidden"
        :data="table.data"
        border
        height="900"
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          prop="city"
          label="地区名称"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="sectorId"
          label="小区ID"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="sectorName"
          label="小区名称"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="enodebid"
          label="小区所属基站标识"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="enodebName"
          label="基站名称"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="earfcn"
          label="小区频点编号"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="pci"
          label="物理小区标识"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="pss"
          label="主同步信号标识"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="sss"
          label="辅同步信号标识"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="tac"
          label="跟踪区编码"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="vendor"
          label="设备厂家"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="longitude"
          label="基站经度"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="latitude"
          label="基站纬度"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="style"
          label="基站类型"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="azimuth"
          label="天线方位角"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="height"
          label="天线高度"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="electtilt"
          label="天线电下倾角"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="mechtilt"
          label="天线机械下倾角"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="totletilt"
          label="总下倾角"
          align="center"
        ></el-table-column>
      </el-table>
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
        sectorID: "",
        sectorName: "",
        eNodeBID: "",
      },
      select: { type: "" },
      table: { data: [], hidden: true },
      eNodeBList: [],
      sectorName: [],
    };
  },
  created() {
    this.getData();
  },
  methods: {
    handleExport() {
      window.location.href = `${BACKEND}/dataservice/tb-cell/export`;
    },
    getData() {
      request({
        url: `${BACKEND}/dataservice/tb-cell/eNodeBList`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.eNodeBList = [];
          let len = response.data.eNodeBList.length;
          for (let i = 0; i < len; i++) {
            let pair = response.data.eNodeBList[i].split(":");
            this.eNodeBList.push({ value: pair[0], label: pair[1] });
          }
        } else {
          this.$message.error(response.message);
        }
      });

      request({
        url: `${BACKEND}/dataservice/tb-cell/sectorNameList`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.sectorName = [];
          let len = response.data.sectorName.length;
          for (let i = 0; i < len; i++) {
            this.sectorName.push({
              value: response.data.sectorName[i],
              label: response.data.sectorName[i],
            });
          }
        } else {
          this.$message.error(response.message);
        }
      });
    },
    handleSearch() {
      if (this.select.type === 1) {
        this.sectorIDSet();
      } else if (this.select.type === 2) {
        this.sectorNameSet();
      } else if (this.select.type === 3) {
        this.eNodeBIDSet();
      }
      request({
        url: `${BACKEND}/dataservice/tb-cell/sectorInfo?sectorId=${this.query.sectorID}&sectorName=${this.query.sectorName}&eNodeBID=${this.query.eNodeBID}`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.table.hidden = false;
          this.table.data = response.data.sectorInfo;
        } else {
          this.$message.error(response.message);
        }
      });
    },
    sectorIDSet() {
      this.query.sectorName = "";
      this.query.eNodeBID = "";
    },
    sectorNameSet() {
      this.query.sectorID = "";
      this.query.eNodeBID = "";
    },
    eNodeBIDSet() {
      this.query.sectorID = "";
      this.query.sectorName = "";
    },
  },
};
</script>