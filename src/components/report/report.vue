<template>
  <el-card>
    <!-- 面包屑 -->
    <my-bread level1="数据统计" level2="数据报表"></my-bread>
    <!-- 数据报表 -->
    <div id="main" style="width:600px;height:400px;"></div>
  </el-card>
</template>
<script>
var echarts = require("echarts");
export default {
  data() {
    return {};
  },
  mounted() {
    this.useEchart();
  },
  methods: {
    async useEchart() {
      var myChart = echarts.init(document.getElementById("main"));
      //获取图表数据
      const res = await this.$http.get(`reports/type/1`);
      //console.log(res)
      let option2 = res.data.data;

      //制定图表的配置项和数据
      let option1 = {
        title: {
          text: "堆叠区域图"
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "cross",
            label: {
              backgroundColor: "#6a7985"
            }
          }
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        grid: {
          left: "3%",
          right: "4%",
          bottom: "3%",
          containLabel: true
        }
      };

      //ES6展开运算符 ...arr ...obj
      let option = { ...option1, ...option2 };
      //使用刚指定的配置项和数据显示图表
      myChart.setOption(option);
    }
  }
};
</script>
<style>
</style>