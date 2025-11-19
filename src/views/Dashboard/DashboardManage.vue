<template>
  <div class="dashboard-container">
    <!-- 1. 统计卡片区域 -->
    <div class="stats-card-wrapper">
      <div class="statistics-card-group">
        <!-- 新增客户卡片 -->
        <StatisticsCard title="新增客户" :value="statisticsData.newCustomerCount" :change="statisticsData.customerChange" icon="User" color="#4285f4" />
        <!-- 新增线索卡片 -->
        <StatisticsCard title="新增线索" :value="statisticsData.newLeadCount" :change="statisticsData.leadChange" icon="Search" color="#00b42a" />
        <!-- 新增合同卡片 -->
        <StatisticsCard title="新增合同" :value="statisticsData.newContractCount" :change="statisticsData.contractChange" icon="Document" color="#ff7d00" />
        <!-- 合同总金额卡片 -->
        <StatisticsCard
          title="合同总金额"
          :value="formatCurrency(statisticsData.contractAmount)"
          :change="statisticsData.amountChange"
          icon="Money"
          color="#f53f3f"
          :isAmount="true"
        />
        <!-- 今日审核通过卡片 -->
        <StatisticsCard title="今日审核通过" :value="statisticsData.todayApprovedCount" icon="Check" color="#00b42a" :change="0" />
        <!-- 今日审核拒绝卡片 -->
        <StatisticsCard title="今日审核拒绝" :value="statisticsData.todayRejectedCount" icon="Close" color="#f53f3f" :change="0" />
      </div>
    </div>

    <!-- 2. 图表区域 -->
    <div class="chart-wrapper">
      <el-card class="full-height-card">
        <template #header>近7日数据趋势</template>
        <TrendChart :chartData="trendData" />
      </el-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { DashboardApi, DashboardStatistics, DashboardTrend } from '@/api/modules/dashboard'
import StatisticsCard from './components/StatisticsCard.vue'
import TrendChart from './components/TrendChart.vue'

// 初始化统计数据（带完整默认值）
const statisticsData = ref<DashboardStatistics>({
  newCustomerCount: 0,
  customerChange: 0,
  newLeadCount: 0,
  leadChange: 0,
  newContractCount: 0,
  contractChange: 0,
  contractAmount: 0,
  amountChange: 0,
  todayApprovedCount: 0,
  todayRejectedCount: 0
})

// 初始化趋势数据
const trendData = ref<DashboardTrend>({
  dates: [],
  customerData: [],
  leadData: [],
  contractData: []
})

/**
 * 格式化金额显示
 */
const formatCurrency = (value: number) => `¥${value.toFixed(2)}`

/**
 * 获取仪表盘统计数据
 */
const fetchStatisticsData = async () => {
  try {
    const res = await DashboardApi.getDashboardData()

    // 数据完整性校验
    if (!res?.data || !res.data.statistics || !res.data.trend) {
      console.error('数据格式不完整，缺少statistics或trend字段')
      return
    }

    // 处理趋势数据（确保数组长度一致）
    const { trend } = res.data
    const requiredArrays = [trend.dates, trend.customerData, trend.leadData, trend.contractData]
    if (requiredArrays.some((arr) => !Array.isArray(arr))) {
      console.error('trend下的核心字段必须为数组')
    } else {
      const maxLength = Math.min(...requiredArrays.map((arr) => arr.length))
      trendData.value = {
        dates: trend.dates.slice(0, maxLength),
        customerData: trend.customerData.slice(0, maxLength),
        leadData: trend.leadData.slice(0, maxLength),
        contractData: trend.contractData.slice(0, maxLength)
      }
    }

    // 处理统计数据（补充默认值）
    statisticsData.value = {
      ...res.data.statistics,
      todayApprovedCount: res.data.statistics.todayApprovedCount || 0,
      todayRejectedCount: res.data.statistics.todayRejectedCount || 0
    }
  } catch (error) {
    console.error('获取统计数据失败:', error)
    // 异常时重置为默认值
    statisticsData.value = {
      newCustomerCount: 0,
      customerChange: 0,
      newLeadCount: 0,
      leadChange: 0,
      newContractCount: 0,
      contractChange: 0,
      contractAmount: 0,
      amountChange: 0,
      todayApprovedCount: 0,
      todayRejectedCount: 0
    }
    trendData.value = { dates: [], customerData: [], leadData: [], contractData: [] }
  }
}

// 组件挂载时获取数据
onMounted(() => {
  fetchStatisticsData()
})
</script>

<style scoped>
.dashboard-container {
  display: flex;
  flex-direction: column;
  min-height: calc(100vh - 60px); /* 适配顶部导航高度 */
  padding: 20px;
  box-sizing: border-box;
  gap: 24px;
}

/* 统计卡片区域样式 */
.stats-card-wrapper {
  height: 160px;
}

.statistics-card-group {
  display: flex;
  gap: 20px;
  height: 100%;
  align-items: stretch;
  flex-wrap: wrap; /* 支持响应式换行 */
}

/* 图表区域样式 */
.chart-wrapper {
  flex: 1;
  min-height: 350px; /* 确保图表有足够高度 */
}

.full-height-card {
  height: 100%;
  display: flex;
  flex-direction: column;
}

.full-height-card .el-card__body {
  flex: 1;
  padding: 16px;
  box-sizing: border-box;
  margin: 0;
}
</style>
