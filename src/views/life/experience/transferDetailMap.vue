<template>
  <!-- 刷新对话框 -->
  <el-dialog :title="title" v-model="open" width="900px" append-to-body>
    <div class="app-container">
      <el-form :model="queryParams" ref="queryRef" :inline="true">
        <el-form-item label="地图类型" prop="field">
          <el-select
            v-model="queryParams.field"
            placeholder="地图类型"
            style="width: 115px"
            @change="handleQuery">
            <el-option
              v-for="dict in fieldOptions"
              :key="dict.id"
              :label="dict.text"
              :value="dict.id" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" icon="refresh" @click="handleQuery" v-hasPermi="['life:experience:transferDetailMap']">刷新</el-button>
          <el-button type="danger" icon="Close" @click="open=false" >关闭</el-button>
        </el-form-item>
      </el-form>
      
      <!--图表数据-->
      <div ref="statChart" :style="{height:height,margin:0 }" />

    </div>
  </el-dialog>

</template>

<script setup name="ExperienceTransferDetailMap">
  import { getExperienceTransferDetailMap } from "@/api/life/experience";
  import * as echarts from 'echarts';
  import '@/components/echarts/map/china.js'
  import { createMapChart, createChinaTransferMapChartOption } from "@/utils/mulanbay_echartsMap";

  const { proxy } = getCurrentInstance();
  //图形实例
  const statChart = ref(null);
  //echarts实例
  let statChartIns = ref(null);
  //是否初始化
  let chartInited = ref(false);
  const height = ref((document.body.clientHeight - 250).toString() + 'px');
  const fieldOptions = ref([]);

  //可执行时间段
  const title = ref('地图统计');
  const open = ref(false);
  const formRef = ref();

  const data = reactive({
    queryParams: {
      field: 'CITY',
    },
    rules: {}
  });

  const { queryParams, rules } = toRefs(data);

  // 定义 success 事件，用于操作成功后的回调
  const emit = defineEmits(['success']);

  /** 打开弹窗 */
  const showData = async (expId) => {
    open.value = true;
    queryParams.value.expId = expId;
    proxy.$nextTick(() => {
      if (!chartInited.value) {
        statChartIns = echarts.init(statChart.value, "macarons");
        chartInited.value = true;
      }
      handleQuery();
    });
  }

  // 提供 open 方法，用于打开弹窗
  defineExpose({ showData });

  /** 初始化下拉树结构 */
  function initOptions() {
    proxy.getEnumDict('MapField', 'FIELD', false).then(response => {
      fieldOptions.value = response;
    });
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

  // 表单重置
  function resetForm() {
    queryParams.value = {

    };
    proxy.resetForm("queryRef");
  }

  /** 图表 */
  function initChart() {
    proxy.$modal.loading("正在加载数据，请稍候！");
    getExperienceTransferDetailMap(queryParams.value).then(response => {
      proxy.$modal.closeLoading();
      //组装chart数据
      let option = createChinaTransferMapChartOption(response, statChartIns);
      createMapChart(option, statChartIns);
    });
  }

  /** 初始化 **/
  onMounted(() => {
    initOptions();
  })
</script>