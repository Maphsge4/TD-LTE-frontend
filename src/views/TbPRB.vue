<template>
  <div id="TbPRB">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-order"></i> 业务查询
        </el-breadcrumb-item>
        <el-breadcrumb-item>tbPRB</el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container" style="min-width: 1200px">
      <el-container>
        <el-header>
          <el-form :inline="true" model="formdata">
            <!-- <el-button icon="el-icon-download" @click="handleExport"
            >导出文件</el-button
          > -->
            <el-form-item>
              <el-select
                v-model="query.sectorName"
                filterable
                placeholder="小区名称"
                @change="handleChange"
              >
                <el-option
                  v-for="item in sectorName"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                >
                </el-option>
              </el-select>
              <el-date-picker
                v-if="query.sectorName"
                v-model="query.timeRange"
                type="datetimerange"
                :shortcuts="shortcuts"
                align="right"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                :default-value="defaultTime[0]"
                :default-time="defaultTime"
                unlink-panels
              >
              </el-date-picker>
              <!-- <el-select
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
              </el-select> -->
            </el-form-item>
            <el-button
              type="primary"
              icon="el-icon-search"
              @click="handleSearch"
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
        </el-header>
        <el-container>
          <el-aside>
            <el-table
              v-show="!table.hidden"
              :data="table.data"
              border
              class="table"
              ref="multipleTable"
              header-cell-class-name="table-header"
              @selection-change="handleSelectionChange"
            >
              <el-table-column
                fixed
                prop="label"
                label="项目"
                width="110"
                align="center"
              ></el-table-column>
              <el-table-column prop="value" label="值"></el-table-column>
            </el-table>
          </el-aside>
          <el-main
            v-if="!chart.hidden"
            v-model="chart.hidden"
            @change="drawPRB"
          >
            <el-row :gutter="20">
              <el-col :span="4">
                <el-select
                  v-model="chart.index"
                  filterable
                  placeholder="PRB编号"
                  @change="drawPRB"
                >
                  <el-option
                    v-for="item in indexList"
                    :key="item"
                    :label="item"
                    :value="item"
                  >
                  </el-option>
                </el-select>
              </el-col>
              <el-col :span="20">
                <el-switch
                  style="top: 10px"
                  v-model="chart.granularity"
                  active-text="小时采样"
                  active-value="prbnew"
                  inactive-text="15分采样"
                  inactive-value="prb"
                  inactive-color="#13ce66"
                  @change="handleSearch"
                >
                </el-switch>
              </el-col>
            </el-row>
            <el-row>
              <div id="ctPRB" style="width: 800px; height: 400px"></div>
            </el-row>
          </el-main>
        </el-container>
      </el-container>
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
        sectorName: "",
        timeRange: [],
      },
      sectorName: [],
      defaultTime: [
        new Date(2020, 6, 17, 0, 0, 0),
        new Date(2020, 6, 19, 23, 59, 59),
      ],
      shortcuts: [
        {
          text: "完整时段",
          value: (() => {
            return [
              new Date(2020, 6, 17, 0, 0, 0),
              new Date(2020, 6, 19, 23, 59, 59),
            ];
          })(),
        },
      ],
      table: {
        hidden: true,
        data: [],
      },
      chart: {
        xAxis: [],
        series: [],
        index: "",
        hidden: true,
        granularity: "prb",
        start: 0,
        end: 100,
      },
    };
  },
  created() {
    this.getData();
  },
  computed: {
    seriesList() {
      let result = [];
      for (let i = 0; i < 100; i++) {
        result.push(`avginterferingnoiseonprb${("00" + i).slice(-2)}Dbmw`);
      }
      return result;
    },
    indexList() {
      let result = [];
      for (let i = 0; i < 100; i++) {
        result.push(i);
      }
      return result;
    },
  },
  methods: {
    date2time(date) {
      return `${date.toLocaleDateString()} ${date.toTimeString()}`;
    },
    handleExport() {
      // window.location.href = `${BACKEND}/dataservice/tb-cell/export`;
    },
    getData() {
      request({
        url: `${BACKEND}/dataservice/tb-prb/sectorList`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.sectorName = [];
          let len = response.data.list.length;
          for (let i = 0; i < len; i++) {
            this.sectorName.push({
              value: response.data.list[i],
              label: response.data.list[i],
            });
          }
        } else {
          this.$message.error(response.message);
        }
      });
    },
    handleChange() {
      this.chart.hidden = true;
      this.chart.index = "";
      request({
        url:
          `${BACKEND}/dataservice/tb-prb/info?sectorName=${this.query.sectorName}` +
          `&beginTime=${this.date2time(this.defaultTime[0])}` +
          `&endTime=${this.date2time(this.defaultTime[0])}`,
        method: "get",
      }).then((response) => {
        if (response.success) {
          this.table.hidden = false;
          let parts = response.data.info[0].sectorDescription.split(", ");
          this.table.data = [];
          for (let i = 0; i < parts.length; i++) {
            let pair = parts[i].split("=");
            this.table.data.push({ label: pair[0], value: pair[1] });
          }
        } else {
          this.$message.error(response.message);
        }
      });
    },
    handleSearch() {
      request({
        url:
          `${BACKEND}/dataservice/tb-${this.chart.granularity}/info?sectorName=${this.query.sectorName}` +
          `&beginTime=${this.date2time(this.query.timeRange[0])}` +
          `&endTime=${this.date2time(this.query.timeRange[1])}`,
        method: "get",
      })
        .then((response) => {
          if (response.success) {
            this.chart.series = [];
            for (let i = 0; i < this.seriesList.length; i++) {
              this.chart.series.push({
                name: this.seriesList[i],
                type: "line",
                smooth: false,
                data: [],
              });
            }
            this.chart.xAxis = [];
            for (let i = 0; i < response.data.info.length; i++) {
              this.chart.xAxis.push(response.data.info[i].startTime);
              for (let j = 0; j < this.seriesList.length; j++) {
                this.chart.series[j].data.push(
                  response.data.info[i][this.seriesList[j]]
                );
              }
            }
          } else {
            this.$message.error(response.message);
          }
        })
        .then(() => {
          this.chart.hidden = false;
          this.drawPRB();
        });
    },
    drawPRB() {
      var chart = this.$echarts.init(document.getElementById("ctPRB"));
      var option = {
        title: {
          text: `第${this.chart.index}个PRB上检测到的干扰噪声的平均值（毫瓦分贝）`,
        },
        grid: {
          height: "80%",
          bottom: 20,
        },
        tooltip: {
          trigger: "axis",
          position: function (pt) {
            return [pt[0], "10%"];
          },
        },
        toolbox: {
          feature: {
            dataZoom: {
              yAxisIndex: "none",
            },
            restore: {},
            saveAsImage: {},
          },
        },
        dataZoom: [
          {
            type: "inside",
            start: this.chart.start,
            end: this.chart.end,
          },
          {
            start: this.chart.start,
            end: this.chart.end,
          },
        ],
        // legend: {
        //   data: [this.seriesList[this.chart.index]],
        // },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: this.chart.xAxis,
        },
        yAxis: { scale: true },
        series: [this.chart.series[this.chart.index]],
      };
      chart.setOption(option);
    },
    // sectorIDSet() {
    //   this.query.sectorName = "";
    //   this.query.eNodeBID = "";
    // },
    // sectorNameSet() {
    //   this.query.sectorID = "";
    //   this.query.eNodeBID = "";
    // },
    // eNodeBIDSet() {
    //   this.query.sectorID = "";
    //   this.query.sectorName = "";
    // },
  },
};
</script>
