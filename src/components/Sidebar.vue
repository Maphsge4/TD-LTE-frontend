<template>
  <div class="sidebar">
    <el-menu
      class="sidebar-el-menu"
      :default-active="onRoutes"
      :collapse="collapse"
      background-color="#324157"
      text-color="#bfcbd9"
      active-text-color="#20a0ff"
      unique-opened
      router
    >
      <template v-for="item in items">
        <template v-if="item.subs">
          <el-submenu :index="item.index" :key="item.index">
            <template #title>
              <i :class="item.icon"></i>
              <span>{{ item.title }}</span>
            </template>
            <template v-for="subItem in item.subs">
              <el-submenu
                v-if="subItem.subs"
                :index="subItem.index"
                :key="subItem.index"
              >
                <template #title>{{ subItem.title }}</template>
                <el-menu-item
                  v-for="(threeItem, i) in subItem.subs"
                  :key="i"
                  :index="threeItem.index"
                  >{{ threeItem.title }}</el-menu-item
                >
              </el-submenu>
              <el-menu-item
                v-else
                :index="subItem.index"
                :key="subItem.index"
                >{{ subItem.title }}</el-menu-item
              >
            </template>
          </el-submenu>
        </template>
        <template v-else>
          <el-menu-item :index="item.index" :key="item.index">
            <i :class="item.icon"></i>
            <template #title>{{ item.title }}</template>
          </el-menu-item>
        </template>
      </template>
    </el-menu>
  </div>
</template>

<script>
// import bus from "../common/bus";
export default {
  data() {
    return {
      items: [
        // {
        //   icon: "el-icon-lx-home",
        //   index: "dashboard",
        //   title: "系统首页",
        // },
        // {
        //   icon: "el-icon-lx-cascades",
        //   index: "table",
        //   title: "基础表格",
        // },
        // {
        //   icon: "el-icon-lx-copy",
        //   index: "tabs",
        //   title: "tab选项卡",
        // },
        // {
        //   icon: "el-icon-lx-calendar",
        //   index: "3",
        //   title: "表单相关",
        //   subs: [
        //     {
        //       index: "form",
        //       title: "基本表单",
        //     },
        //     //         {
        //     //             index: "3-2",
        //     //             title: "三级菜单",
        //     //             subs: [
        //     //                 {
        //     //                     index: "editor",
        //     //                     title: "富文本编辑器"
        //     //                 },
        //     //                 {
        //     //                     index: "markdown",
        //     //                     title: "markdown编辑器"
        //     //                 }
        //     //             ]
        //     //         },
        //     {
        //       index: "upload",
        //       title: "文件上传",
        //     },
        //   ],
        // },
        // {
        //   icon: "el-icon-lx-emoji",
        //   index: "icon",
        //   title: "自定义图标",
        // },
        // {
        //   icon: "el-icon-pie-chart",
        //   index: "charts",
        //   title: "schart图表",
        // },
        {
          icon: "el-icon-user-solid",
          index: "account",
          title: "用户管理",
        },
        {
          icon: "el-icon-office-building",
          index: "system",
          title: "系统管理",
        },
        {
          icon: "el-icon-data-analysis",
          index: "analyze",
          title: "数据分析",
        },
        {
          icon: "el-icon-s-data",
          index: "data",
          title: "数据管理",
          subs: [
            {
              index: "upload",
              title: "数据导入",
            },
            {
              index: "export",
              title: "数据导出",
            },
          ],
        },
        {
          icon: "el-icon-s-order",
          index: "business",
          title: "业务查询",
          subs: [
            {
              index: "tbcell",
              title: "小区/基站工参表 tbCell",
            },
            {
              index: "tbcellkpi",
              title: "优化区17-19日KPI统计表 tbKPI",
            },
            {
              index: "tbprb",
              title: "优化区17-19日PRB干扰 tbPRB",
            },
            // {
            //   index: "",
            //   title: "",
            // },
          ],
        },
        // {
        //   icon: "el-icon-lx-warn",
        //   index: "7",
        //   title: "错误处理",
        //   subs: [
        //     {
        //       index: "permission",
        //       title: "权限测试",
        //     },
        //     {
        //       index: "404",
        //       title: "404页面",
        //     },
        //   ],
        // },
      ],
    };
  },
  computed: {
    onRoutes() {
      return this.$route.path.replace("/", "");
    },
    collapse() {
      return this.$store.state.collapse;
    },
  },
};
</script>

<style scoped>
.sidebar {
  display: block;
  position: absolute;
  left: 0;
  top: 70px;
  bottom: 0;
  overflow-y: scroll;
}
.sidebar::-webkit-scrollbar {
  width: 0;
}
.sidebar-el-menu:not(.el-menu--collapse) {
  width: 250px;
}
.sidebar > ul {
  height: 100%;
}
</style>
