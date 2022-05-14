<template>
  <div id="TbCellKPI">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-s-order"></i> 业务查询
        </el-breadcrumb-item>
        <el-breadcrumb-item>tbCellKPI</el-breadcrumb-item>
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
            @change="drawCellKPI"
          >
            <el-row>
              <el-select
                v-model="chart.index"
                placeholder="属性"
                @change="drawCellKPI"
              >
                <el-option
                  v-for="item in series"
                  :key="item.label"
                  :label="item.label"
                  :value="item.index"
                >
                </el-option>
              </el-select>
            </el-row>
            <el-row>
              <div id="ctCellKPI" style="width: 800px; height: 400px"></div>
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
        index: 0,
        hidden: true,
        start: 0,
        end: 100,
      },
      series: [
        { value: "rrcComplete", label: "RRC连接建立完成次数 (无)", index: 0 },
        {
          value: "rrcRequest",
          label: "RRC连接请求次数（包括重发） (无)",
          index: 1,
        },
        { value: "rrcSuccessPer", label: "RRC建立成功率qf (%)", index: 2 },
        { value: "erabSuccess", label: "E-RAB建立成功总次数 (无)", index: 3 },
        { value: "erabTry", label: "E-RAB建立尝试总次数 (无)", index: 4 },
        { value: "erabSuccessPer", label: "E-RAB建立成功率2 (%)", index: 5 },
        {
          value: "enodebTringErabException",
          label: "eNodeB触发的E-RAB异常释放总次数 (无)",
          index: 6,
        },
        {
          value: "sectorSwitchOutErabException",
          label: "小区切换出E-RAB异常释放总次数 (无)",
          index: 7,
        },
        { value: "erabOfflinePer", label: "E-RAB掉线率(新) (%)", index: 8 },
        { value: "wirelessOnlinePer", label: "无线接通率ay (%)", index: 9 },
        {
          value: "enodebS1ResetUeContextRelease",
          label: "eNodeB发起的S1 RESET导致的UE Context释放次数 (无)",
          index: 10,
        },
        {
          value: "ueContextException",
          label: "UE Context异常释放次数 (无)",
          index: 11,
        },
        {
          value: "ueContextSuccess",
          label: "UE Context建立成功总次数 (无)",
          index: 12,
        },
        { value: "wirelessOfflinePer", label: "无线掉线率 (%)", index: 13 },
        {
          value: "enodebInDiffFreqSwitchOutSuccess",
          label: "eNodeB内异频切换出成功次数 (无)",
          index: 14,
        },
        {
          value: "enodebInDiffFreqSwitchOutTry",
          label: "eNodeB内异频切换出尝试次数 (无)",
          index: 15,
        },
        {
          value: "enodebInSameFreqSwitchOutSuccess",
          label: "eNodeB内同频切换出成功次数 (无)",
          index: 16,
        },
        {
          value: "enodebInSameFreqSwitchOutTry",
          label: "eNodeB内同频切换出尝试次数 (无)",
          index: 17,
        },
        {
          value: "enodebBetweenDiffFreqSwitchOutSuccess",
          label: "eNodeB间异频切换出成功次数 (无)",
          index: 18,
        },
        {
          value: "enodebBetweenDiffFreqSwitchOutTry",
          label: "eNodeB间异频切换出尝试次数 (无)",
          index: 19,
        },
        {
          value: "enodebBetweenSameFreqSwitchOutSuccess",
          label: "eNodeB间同频切换出成功次数 (无)",
          index: 20,
        },
        {
          value: "enodebBetweenSameFreqSwitchOutTry",
          label: "eNodeB间同频切换出尝试次数 (无)",
          index: 21,
        },
        {
          value: "enbInSwitchSuccessPer",
          label: "eNB内切换成功率 (%)",
          index: 22,
        },
        {
          value: "enbBetweenSwitchSuccessPer",
          label: "eNB间切换成功率 (%)",
          index: 23,
        },
        {
          value: "sameFreqSwitchSuccessPer",
          label: "同频切换成功率zsp (%)",
          index: 24,
        },
        {
          value: "diffFreqSwitchSuccessPer",
          label: "异频切换成功率zsp (%)",
          index: 25,
        },
        { value: "switchSuccessPer", label: "切换成功率 (%)", index: 26 },
        {
          value: "sectorPdcpLayerRecvUploadThroughputBit",
          label: "小区PDCP层所接收到的上行数据的总吞吐量 (比特)",
          index: 27,
        },
        {
          value: "sectorPdcpLayerSendDnloadThroughputBit",
          label: "小区PDCP层所发送的下行数据的总吞吐量 (比特)",
          index: 28,
        },
        { value: "rrcReconnect", label: "RRC重建请求次数 (无)", index: 29 },
        { value: "rrcReconnectPer", label: "RRC连接重建比率 (%)", index: 30 },
        {
          value: "throughRebuildBackEnodebBeteeenSameFreqSwitchOutSuccess",
          label: "通过重建回源小区的eNodeB间同频切换出执行成功次数 (无)",
          index: 31,
        },
        {
          value: "throughRebuildBackEnodebBeteeenDiffFreqSwitchOutSuccess",
          label: "通过重建回源小区的eNodeB间异频切换出执行成功次数 (无)",
          index: 32,
        },
        {
          value: "throughRebuildBackEnodebInSameFreqSwitchOutSuccess",
          label: "通过重建回源小区的eNodeB内同频切换出执行成功次数 (无)",
          index: 33,
        },
        {
          value: "throughRebuildBackEnodebInDiffFreqSwitchOutSuccess",
          label: "通过重建回源小区的eNodeB内异频切换出执行成功次数 (无)",
          index: 34,
        },
        {
          value: "enbInSwitchOutSuccess",
          label: "eNB内切换出成功次数 (次)",
          index: 35,
        },
        {
          value: "enbInSwitchOutTry",
          label: "eNB内切换出请求次数 (次)",
          index: 36,
        },
      ],
    };
  },
  created() {
    this.getData();
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
        url: `${BACKEND}/dataservice/tb-cell-kpi/sectorList`,
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
          `${BACKEND}/dataservice/tb-cell-kpi/info?sectorName=${this.query.sectorName}` +
          `&beginTime=${this.date2time(this.defaultTime[0])}` +
          `&endTime=${this.date2time(this.defaultTime[1])}`,
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
          `${BACKEND}/dataservice/tb-cell-kpi/info?sectorName=${this.query.sectorName}` +
          `&beginTime=${this.date2time(this.query.timeRange[0])}` +
          `&endTime=${this.date2time(this.query.timeRange[1])}`,
        method: "get",
      })
        .then((response) => {
          if (response.success) {
            this.chart.series = [];
            for (let i = 0; i < this.series.length; i++) {
              this.chart.series.push({
                name: this.series[i].value,
                type: "line",
                smooth: false,
                data: [],
              });
            }
            this.chart.xAxis = [];
            for (let i = 0; i < response.data.info.length; i++) {
              this.chart.xAxis.push(response.data.info[i].startTime);
              for (let j = 0; j < this.series.length; j++) {
                this.chart.series[j].data.push(
                  response.data.info[i][this.series[j].value]
                );
              }
            }
          } else {
            this.$message.error(response.message);
          }
        })
        .then(() => {
          this.chart.hidden = false;
          this.drawCellKPI();
        });
    },
    drawCellKPI() {
      var chart = this.$echarts.init(document.getElementById("ctCellKPI"));
      var option = {
        title: {
          text: this.series[this.chart.index].label,
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
        //   data: [this.seriesList[this.series[this.chart.key][1]]],
        // },
        xAxis: {
          type: "category",
          boundaryGap: false,
          data: this.chart.xAxis,
        },
        yAxis: { scale: true },
        series: [this.chart.series[this.chart.index]],
      };
      console.log(option);
      chart.setOption(option);
    },
  },
};
</script>