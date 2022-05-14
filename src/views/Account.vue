<template>
  <div id="Account">
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-user-solid"></i> 用户管理
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <div class="handle-box">
        <el-form :inline="true" model="formdata"
          ><el-button
            type="danger"
            icon="el-icon-delete"
            class="handle-del mr10"
            @click="delAllSelection"
            >批量删除</el-button
          >
          <el-form-item label="新建用户：">
            <el-input v-model="formdata.username" placeholder="账号"></el-input>
          </el-form-item>
          <el-form-item>
            <el-input v-model="formdata.password" placeholder="密码"></el-input>
          </el-form-item>
          <el-form-item>
            <el-checkbox v-model="formdata.type" disabled>管理员</el-checkbox>
          </el-form-item>
          <el-button type="primary" icon="el-icon-edit" @click="handleRegister"
            >添加用户</el-button
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
        :data="tableData"
        border
        class="table"
        ref="multipleTable"
        header-cell-class-name="table-header"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          type="selection"
          width="55"
          align="center"
        ></el-table-column>
        <el-table-column
          prop="id"
          label="ID"
          width="55"
          align="center"
        ></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column label="账户类型">
          <template #default="scope"
            ><el-tag
              :type="
                scope.row.type === 1
                  ? 'success'
                  : scope.row.type === 0
                  ? 'danger'
                  : ''
              "
              >{{ scope.row.type ? "普通用户" : "管理员" }}</el-tag
            ></template
          >
        </el-table-column>
        <el-table-column label="状态" align="center">
          <template #default="scope">
            <el-tag
              :type="
                scope.row.verification === 1
                  ? 'success'
                  : scope.row.verification === 0
                  ? 'danger'
                  : ''
              "
              >{{ scope.row.verification ? "已验证" : "待验证" }}</el-tag
            >
          </template>
        </el-table-column>

        <!-- <el-table-column prop="date" label="注册时间"></el-table-column> -->
        <el-table-column label="操作" width="180" align="center">
          <template #default="scope">
            <el-button
              v-if="!scope.row.verification"
              type="text"
              icon="el-icon-check"
              class="green"
              @click="handleVerify(scope.$index, scope.row)"
              >通过</el-button
            >
            <el-button
              type="text"
              icon="el-icon-delete"
              class="red"
              @click="handleDelete(scope.$index, scope.row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- <div class="pagination">
        <el-pagination
          background
          layout="total, prev, pager, next"
          :current-page="query.pageIndex"
          :page-size="query.pageSize"
          :total="pageTotal"
          @current-change="handlePageChange"
        ></el-pagination>
      </div> -->
    </div>
  </div>
</template>

<script>
import { BACKEND } from "../utils/backend";
import request from "../utils/request";
export default {
  data() {
    return {
      // query: {
      //   pageIndex: 1,
      //   pageSize: 10,
      // },
      tableData: [],
      multipleSelection: [],
      delList: [],
      editVisible: false,
      pageTotal: 0,
      idx: -1,
      id: -1,
      currentDate: new Date(),
      formdata: {
        username: "",
        password: "",
      },
    };
  },
  created() {
    this.getData();
  },
  methods: {
    handleRegister() {
      request({
        url: `${BACKEND}/dataservice/account/register?username=${this.formdata.username}&password=${this.formdata.password}`,
        method: "post",
      })
        .then((response) => {
          if (response.success === true) {
            this.$message.success("注册成功");
            return true;
          } else {
            this.$message.error(response.message);
            return false;
          }
        })
        .then((success) => {
          if (!success) {
            this.getData();
            return;
          }
          request({
            url: `${BACKEND}/dataservice/account/verify?username=${this.formdata.username}`,
            method: "post",
          }).then((response) => {
            if (response.success) {
              this.$message.success("自动验证成功");
            } else {
              this.$message.error(response.message);
            }
            this.getData();
          });
        });
    },
    getData() {
      request({
        url: `${BACKEND}/dataservice/account/accountList`,
        method: "get",
      }).then((res) => {
        this.tableData = res.data.list;
        this.pageTotal = 1;
      });
    },
    handleVerify(index) {
      request({
        url: `${BACKEND}/dataservice/account/verify?username=${this.tableData[index].username}`,
        method: "post",
      }).then((response) => {
        if (response.success) {
          this.$message.success("验证成功");
          this.tableData[index].verification = 1;
        } else {
          this.$message.error(response.message);
        }
      });
    },
    handleDelete(index) {
      // 二次确认删除
      this.$confirm("确定要删除吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() =>
          request({
            url: `${BACKEND}/dataservice/account/delete?username=${this.tableData[index].username}`,
            method: "delete",
          }).then(() => {
            this.$message.success("删除成功");
            this.tableData.splice(index, 1);
          })
        )
        .catch(() => {});
    },
    // 多选操作
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    delAllSelection() {
      this.$confirm("确定要删除吗？", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.multipleSelection.forEach((item) =>
            request({
              url: `${BACKEND}/dataservice/account/delete?username=${item.username}`,
              method: "delete",
            })
          );
          this.$message.success(`正在处理`);
          setTimeout(() => {
            this.$message.success(`操作完成`);
            this.multipleSelection = [];
            this.getData();
          }, 500);
        })
        .catch(() => {});
    },
    // TODO:
    // 分页导航
    // handlePageChange(val) {
    //   this.$set(this.query, "pageIndex", val);
    //   this.getData();
    // },
  },
};
</script>
 
<style scoped>
.option-card {
  height: 250px;
  width: 230px;
}
#title {
  margin-top: 30px;
}

#upload {
  margin-top: 100px;
}

#main,
#body {
  position: relative;
  top: 50px;
  left: 5%;
  right: 5%;
}

.hrefButton {
  position: absolute;
  right: 10px;
  bottom: 0px;
}

.handle-box {
  margin-bottom: 20px;
}

.handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.green {
  color: #67c23a;
}
.mr10 {
  margin-right: 10px;
}
.table-td-thumb {
  display: block;
  margin: auto;
  width: 40px;
  height: 40px;
}
</style>