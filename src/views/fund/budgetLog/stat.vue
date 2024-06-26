<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true">
      <el-form-item label="选择预算" prop="budgetKey">
        <el-tree-select
          v-model="queryParams.budgetKey"
          style="width: 240px"
          clearable
          :data="budgetOptions"
          :props="{ value: 'id', label: 'text', children: 'children' }"
          value-key="id"
          placeholder="选择预算"
          :check-strictly="true" />
      </el-form-item>
      <el-form-item label="流水日期" style="width: 308px">
        <el-date-picker
          v-model="dateRange"
          unlink-panels
          value-format="YYYY-MM-DD"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
          :shortcuts="datePickerOptions"></el-date-picker>
      </el-form-item>
      <el-form-item label="消费预测" prop="predict">
        <el-switch v-model="queryParams.predict"></el-switch>
      </el-form-item>
      <el-form-item label="包含突发消费" prop="needOutBurst">
        <el-switch v-model="queryParams.needOutBurst"></el-switch>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="TrendCharts" @click="handleQuery" v-hasPermi="['fund:budgetLog:stat']">统计</el-button>
        <el-button icon="refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!--图表数据-->
    <div ref="statChart" :style="{height:height,margin:0 }" />

  </div>
</template>

<script setup name="BudgetLogStat">
  import { getBudgetLogStat } from "@/api/fund/budgetLog";
  import { getBudgetTree } from "@/api/fund/budget";
  import { getBudgetStat } from "@/api/fund/budgetSnapshot";
  import * as echarts from 'echarts';
  import { createChart, createMixLineBarChartOption } from "@/utils/mulanbay_echarts";

  const { proxy } = getCurrentInstance();
  //图形实例
  const statChart = ref(null);
  //echarts实例
  let statChartIns = ref(null);
  const height = ref((document.body.clientHeight - 240).toString() + 'px');

  const budgetOptions = ref([]);

  //日期范围快速选择
  const datePickerOptions = ref(proxy.datePickerOptions);
  const dateRange = ref(proxy.getYearDateRange(0));

  const data = reactive({
    queryParams: {
      type: 'MONTHLY',
      needOutBurst: true,
      predict: false,
      budgetKey: "p3"
    }
  });

  const { queryParams } = toRefs(data);

  /** 下拉框加载 */
  function loadOptions() {
    getBudgetTree(false, false).then(
      response => {
        budgetOptions.value = response;
      }
    );
  }

  /** 搜索按钮操作 */
  function handleQuery() {
    initChart();
  }

  /** 重置按钮操作 */
  function resetQuery() {
    proxy.resetForm("queryRef");
    initChart();
  }

  function initChart() {
    let budgetKey = queryParams.value.budgetKey.toString();
    if (budgetKey.substr(0, 1) == 'p') {
      initPeriodChart();
    } else {
      initBudgetChart();
    }
  }

  function initPeriodChart() {
    proxy.$modal.loading("正在加载数据，请稍候！");
    getBudgetLogStat(proxy.addDateRange(queryParams.value, dateRange.value)).then(
      response => {
        proxy.$modal.closeLoading();
        //组装chart数据
        response.serieTypes = ['line', 'line'];
        response.smooth = true;
        let option = createMixLineBarChartOption(response);;
        createChart(option, statChartIns);
      }
    );
  }

  function initBudgetChart() {
    proxy.$modal.loading("正在加载数据，请稍候！");
    const paraData = {
      budgetId: queryParams.value.budgetKey
    };
    getBudgetStat(proxy.addDateRange(paraData, dateRange.value)).then(
      response => {
        proxy.$modal.closeLoading();
        //组装chart数据
        response.smooth = true;
        response.serieTypes = ['line', 'bar'];
        response.s2Size = 2;
        let option = createMixLineBarChartOption(response);;
        createChart(option, statChartIns);
      }
    );
  }

  /** 初始化 **/
  onMounted(() => {
    statChartIns = echarts.init(statChart.value, "macarons");
    initChart();
    loadOptions();
  })
</script>