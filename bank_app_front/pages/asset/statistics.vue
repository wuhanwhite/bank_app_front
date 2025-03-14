<template>
	<view class="container">
		<view class="header">
			<text class="title">资产统计</text>
			<view class="period-selector">
				<text 
					v-for="(item, index) in periods" 
					:key="index" 
					:class="['period-item', { active: currentPeriod === item.value }]"
					@click="currentPeriod = item.value"
				>
					{{ item.label }}
				</text>
			</view>
		</view>
		
		<view class="overview-card">
			<view class="overview-header">
				<text class="overview-title">资产概览</text>
				<text class="overview-date">{{ formatDate(new Date()) }}</text>
			</view>
			
			<view class="total-assets">
				<text class="total-label">总资产 (元)</text>
				<text class="total-amount">{{ totalAssets.toFixed(2) }}</text>
			</view>
			
			<view class="chart-container">
				<!-- 这里可以使用echarts等图表库展示资产分布饼图 -->
				<view class="chart-placeholder">
					<view class="pie-chart">
						<view v-for="(item, index) in assetDistribution" :key="index" 
							class="pie-segment" 
							:style="{
								'background-color': item.color,
								'transform': `rotate(${getRotation(index)}deg)`,
								'clip-path': `polygon(50% 50%, 50% 0%, ${getCoordinates(item.percentage)} 0%)`
							}"
						></view>
					</view>
				</view>
				
				<view class="chart-legend">
					<view v-for="(item, index) in assetDistribution" :key="index" class="legend-item">
						<view class="legend-color" :style="{ 'background-color': item.color }"></view>
						<text class="legend-label">{{ item.name }}</text>
						<text class="legend-value">{{ item.amount.toFixed(2) }}</text>
						<text class="legend-percent">{{ item.percentage.toFixed(2) }}%</text>
					</view>
				</view>
			</view>
		</view>
		
		<view class="ranking-card">
			<view class="card-header">
				<text class="card-title">银行资产排行</text>
			</view>
			
			<view class="ranking-list">
				<view v-for="(bank, index) in bankRanking" :key="index" class="ranking-item">
					<view class="ranking-index">{{ index + 1 }}</view>
					<view class="bank-info">
						<image class="bank-logo" :src="bank.logo || '/static/images/bank-default.png'" mode="aspectFit"></image>
						<text class="bank-name">{{ bank.name }}</text>
					</view>
					<view class="bank-amount">{{ bank.amount.toFixed(2) }}</view>
				</view>
			</view>
		</view>
		
		<view class="trend-card">
			<view class="card-header">
				<text class="card-title">资产趋势</text>
				<view class="trend-selector">
					<text 
						v-for="(item, index) in trendTypes" 
						:key="index" 
						:class="['trend-item', { active: currentTrendType === item.value }]"
						@click="currentTrendType = item.value"
					>
						{{ item.label }}
					</text>
				</view>
			</view>
			
			<view class="trend-chart">
				<!-- 这里可以使用echarts等图表库展示资产趋势折线图 -->
				<view class="chart-placeholder trend-placeholder">
					<view class="trend-line">
						<view v-for="(point, index) in trendData" :key="index" class="trend-point" 
							:style="{
								'left': `${index * (100 / (trendData.length - 1))}%`,
								'bottom': `${(point.value / maxTrendValue) * 80}%`
							}"
						>
							<view class="point-dot"></view>
							<view class="point-value">{{ point.value.toFixed(0) }}</view>
						</view>
					</view>
					<view class="trend-x-axis">
						<text v-for="(point, index) in trendData" :key="index" class="x-label"
							:style="{ 'left': `${index * (100 / (trendData.length - 1))}%` }"
						>
							{{ point.label }}
						</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref, computed } from 'vue';

// 时间周期选择
const periods = [
  { label: '周', value: 'week' },
  { label: '月', value: 'month' },
  { label: '季', value: 'quarter' },
  { label: '年', value: 'year' }
];
const currentPeriod = ref('month');

// 趋势类型选择
const trendTypes = [
  { label: '总资产', value: 'total' },
  { label: '储蓄卡', value: 'debit' },
  { label: '信用卡', value: 'credit' }
];
const currentTrendType = ref('total');

// 模拟资产数据
const assetData = ref({
  total: 35000,
  distribution: [
    { name: '中国银行', amount: 15000, color: '#3c9cff' },
    { name: '工商银行', amount: 8000, color: '#ff9500' },
    { name: '建设银行', amount: 12000, color: '#34c759' }
  ],
  trend: {
    total: [
      { label: '1月', value: 30000 },
      { label: '2月', value: 32000 },
      { label: '3月', value: 31000 },
      { label: '4月', value: 33000 },
      { label: '5月', value: 35000 },
      { label: '6月', value: 34000 }
    ],
    debit: [
      { label: '1月', value: 25000 },
      { label: '2月', value: 26000 },
      { label: '3月', value: 25000 },
      { label: '4月', value: 27000 },
      { label: '5月', value: 28000 },
      { label: '6月', value: 27000 }
    ],
    credit: [
      { label: '1月', value: 5000 },
      { label: '2月', value: 6000 },
      { label: '3月', value: 6000 },
      { label: '4月', value: 6000 },
      { label: '5月', value: 7000 },
      { label: '6月', value: 7000 }
    ]
  }
});

// 计算总资产
const totalAssets = computed(() => {
  return assetData.value.total;
});

// 计算资产分布百分比
const assetDistribution = computed(() => {
  return assetData.value.distribution.map(item => {
    return {
      ...item,
      percentage: (item.amount / assetData.value.total) * 100
    };
  });
});

// 银行资产排行
const bankRanking = computed(() => {
  return [...assetData.value.distribution].sort((a, b) => b.amount - a.amount);
});

// 获取当前趋势数据
const trendData = computed(() => {
  return assetData.value.trend[currentTrendType.value];
});

// 获取趋势数据最大值
const maxTrendValue = computed(() => {
  return Math.max(...trendData.value.map(item => item.value)) * 1.2;
});

// 格式化日期
const formatDate = (date) => {
  return `${date.getFullYear()}年${date.getMonth() + 1}月${date.getDate()}日`;
};

// 计算饼图旋转角度
const getRotation = (index) => {
  let rotation = 0;
  for (let i = 0; i < index; i++) {
    rotation += assetDistribution.value[i].percentage * 3.6;
  }
  return rotation;
};

// 计算饼图坐标
const getCoordinates = (percentage) => {
  const angle = percentage * 3.6;
  if (angle <= 90) {
    return `${50 + 50 * Math.tan(angle * Math.PI / 180)}% 0%`;
  } else if (angle <= 180) {
    return `100% ${50 - 50 * Math.tan((180 - angle) * Math.PI / 180)}%`;
  } else if (angle <= 270) {
    return `${50 - 50 * Math.tan((angle - 180) * Math.PI / 180)}% 100%`;
  } else {
    return `0% ${50 + 50 * Math.tan((360 - angle) * Math.PI / 180)}%`;
  }
};
</script>

<style>
.container {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #f8f8f8;
  padding: 30rpx;
}

.header {
  display: flex;
  flex-direction: column;
  margin-bottom: 30rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 20rpx;
}

.period-selector {
  display: flex;
  background-color: #f0f0f0;
  border-radius: 30rpx;
  overflow: hidden;
  align-self: flex-start;
}

.period-item {
  padding: 15rpx 30rpx;
  font-size: 28rpx;
  color: #666;
}

.period-item.active {
  background-color: #3c9cff;
  color: #fff;
}

.overview-card, .ranking-card, .trend-card {
  background-color: #fff;
  border-radius: 20rpx;
  padding: 30rpx;
  margin-bottom: 30rpx;
  box-shadow: 0 5rpx 15rpx rgba(0, 0, 0, 0.05);
}

.overview-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20rpx;
}

.overview-title, .card-title {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
}

.overview-date {
  font-size: 24rpx;
  color: #999;
}

.total-assets {
  margin-bottom: 30rpx;
}

.total-label {
  font-size: 28rpx;
  color: #666;
  display: block;
  margin-bottom: 10rpx;
}

.total-amount {
  font-size: 48rpx;
  font-weight: bold;
  color: #333;
  display: block;
}

.chart-container {
  display: flex;
  flex-direction: column;
}

.chart-placeholder {
  height: 400rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.pie-chart {
  width: 300rpx;
  height: 300rpx;
  border-radius: 50%;
  position: relative;
  overflow: hidden;
}

.pie-segment {
  position: absolute;
  width: 100%;
  height: 100%;
  transform-origin: 50% 50%;
}

.chart-legend {
  display: flex;
  flex-direction: column;
  margin-top: 30rpx;
}

.legend-item {
  display: flex;
  align-items: center;
  margin-bottom: 20rpx;
}

.legend-color {
  width: 20rpx;
  height: 20rpx;
  border-radius: 10rpx;
  margin-right: 15rpx;
}

.legend-label {
  font-size: 28rpx;
  color: #333;
  flex: 1;
}

.legend-value {
  font-size: 28rpx;
  color: #333;
  font-weight: bold;
  margin-right: 20rpx;
}

.legend-percent {
  font-size: 24rpx;
  color: #999;
  width: 80rpx;
  text-align: right;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30rpx;
}

.ranking-list {
  display: flex;
  flex-direction: column;
}

.ranking-item {
  display: flex;
  align-items: center;
  padding: 20rpx 0;
  border-bottom: 1px solid #f5f5f5;
}

.ranking-item:last-child {
  border-bottom: none;
}

.ranking-index {
  width: 60rpx;
  height: 60rpx;
  border-radius: 30rpx;
  background-color: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28rpx;
  color: #666;
  margin-right: 20rpx;
}

.ranking-item:nth-child(1) .ranking-index {
  background-color: #ffcc00;
  color: #fff;
}

.ranking-item:nth-child(2) .ranking-index {
  background-color: #c0c0c0;
  color: #fff;
}

.ranking-item:nth-child(3) .ranking-index {
  background-color: #cd7f32;
  color: #fff;
}

.bank-info {
  display: flex;
  align-items: center;
  flex: 1;
}

.bank-logo {
  width: 60rpx;
  height: 60rpx;
  margin-right: 20rpx;
}

.bank-name {
  font-size: 28rpx;
  color: #333;
}

.bank-amount {
  font-size: 32rpx;
  color: #333;
  font-weight: bold;
}

.trend-selector {
  display: flex;
  background-color: #f0f0f0;
  border-radius: 30rpx;
  overflow: hidden;
}

.trend-item {
  padding: 10rpx 20rpx;
  font-size: 24rpx;
  color: #666;
}

.trend-item.active {
  background-color: #3c9cff;
  color: #fff;
}

.trend-placeholder {
  position: relative;
  padding: 30rpx 0;
}

.trend-line {
  position: relative;
  height: 300rpx;
  width: 100%;
}

.trend-point {
  position: absolute;
  transform: translate(-50%, 50%);
}

.point-dot {
  width: 16rpx;
  height: 16rpx;
  border-radius: 8rpx;
  background-color: #3c9cff;
  margin-bottom: 10rpx;
}

.point-value {
  font-size: 24rpx;
  color: #666;
  transform: translateX(-50%);
}

.trend-x-axis {
  display: flex;
  position: relative;
  margin-top: 20rpx;
  height: 40rpx;
}

.x-label {
  position: absolute;
  font-size: 24rpx;
  color: #999;
  transform: translateX(-50%);
}
</style>
