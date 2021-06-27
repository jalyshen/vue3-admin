<template>
  <el-card class="introduce">
    <div>
      <el-form :model="rdemoForm" ref="dataRef">
        <el-form-item required label="每天最高温度" prop="firstGroup">
          <el-input
            style="width: 350px"
            v-model="rdemoForm.firstGroup"
            placeholder="请输入，逗号分割。如：10, 11, 13, 11, 12, 12, 9"
          ></el-input>
        </el-form-item>
        <el-form-item required label="每天最低温度" prop="secondGroup">
          <el-input
            style="width: 350px"
            v-model="rdemoForm.secondGroup"
            placeholder="请输入，逗号分割。如：1, -2, 2, 5, 3, 2, 0"
          ></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitData()"
            >提交给R计算均值</el-button
          >
        </el-form-item>
      </el-form>
    </div>
    <div id="zoom"></div>
  </el-card>
</template>

<script>
import {
  reactive,
  ref,
  toRefs,
  onMounted,
  //onBeforeUnmount,
  //getCurrentInstance,
  onUnmounted,
} from "vue";
import axios from "@/utils/axios";

// 用于绘图的Echart对象
var rDemoChart = null;

var highTemperature = function (highTemps) {  
  return {
    name: "最高气温",
    type: "line",
    data: highTemps, //[10, 11, 13, 11, 12, 12, 9],    
  };
};

var lowTemperature = function (lowTemps) {  
  return {
    name: "最低气温",
    type: "line",
    data: lowTemps, //[1, -2, 2, 5, 3, 2, 0],    
  };
};

var averageTemperature = function (averageTemp) {  
  return {
    name: "平均气温",
    type: "line",
    data: averageTemp, //[10, 11, 13, 11, 12, 12, 9],    
  };
};

export default {
  name: "RDemo",
  setup() {
    const dataRef = ref(null);
    const state = reactive({
      // 用户输入两组数据，然后计算每对数据的均值，然后画出均值线
      // 每组数据，用逗号分割
      rdemoForm: {
        firstGroup: "", // 第一组数据
        secondGroup: "", // 第二组数据
      },
      means: [], // R返回的均值
    });
    // 指定图标的配置项和数据
    var option;
    option = {
      title: {
        text: "未来一周气温变化",
      },
      tooltip: {
        trigger: "axis",
      },
      legend: {
        data: ["最高气温", "平均气温", "最低气温"],
      },
      toolbox: {
        show: true,
        feature: {
          dataZoom: {
            yAxisIndex: "none",
          },
          dataView: { readOnly: false },
          magicType: { type: ["line", "bar"] },
          restore: {},
          saveAsImage: {},
        },
      },
      xAxis: {
        type: "category",
        boundaryGap: false,
        data: ["周一", "周二", "周三", "周四", "周五", "周六", "周日"],
      },
      yAxis: {
        type: "value",
        axisLabel: {
          formatter: "{value} °C",
        },
      },
      series: [],
    };
    onMounted(() => {
      if (window.echarts) {
        // 基于准备好的dom，初始化echarts实例
        rDemoChart = window.echarts.init(document.getElementById("zoom"));
      }
    });
    onUnmounted(() => {
      rDemoChart.dispose();
    });

    // 提交
    const submitData = () => {
      let httpOption = axios.post;
      let params = {
        firstGroupData: state.rdemoForm.firstGroup,
        secondGroupData: state.rdemoForm.secondGroup,
      };

      console.log(params);

      // 数据提交给后段
      httpOption('/calculateMeans', params)
        .then(function (response) {
          console.log("提交成功");
          // 使用刚指定的配置项和数据显示图表。
          rDemoChart = window.echarts.init(document.getElementById("zoom"));
          option.series = [averageTemperature(response.meansReuslt), 
              highTemperature(response.highTemp), 
              lowTemperature(response.lowTemp)];
          rDemoChart.setOption(option,true);
        })
        .catch(function (err) {
          // console.log(err);
          // 使用刚指定的配置项和数据显示图表。
          rDemoChart = window.echarts.init(document.getElementById("zoom"));
          option.series = [averageTemperature(err.meansReuslt), 
              highTemperature(err.highTemp), 
              lowTemperature(err.lowTemp)];
          rDemoChart.setOption(option,true);
        });
    };
    return {
      ...toRefs(state),
      dataRef,
      submitData,
    };
  },
};
</script>

<style scoped>
.introduce .order {
  display: flex;
  margin-bottom: 50px;
}
.introduce .order .order-item {
  flex: 1;
  margin-right: 20px;
}
.introduce .order .order-item:last-child {
  margin-right: 0;
}
#zoom {
  min-height: 400px;
}
</style>