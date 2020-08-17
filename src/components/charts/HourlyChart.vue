<template>
  <v-chart :options="hourlyChartOptions" />
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator';

import ECharts from 'vue-echarts';
import 'echarts/lib/chart/bar';
import 'echarts/lib/chart/line';
import 'echarts/lib/component/title';
import 'echarts/lib/component/tooltip';
import 'echarts/lib/component/legend';

import { Trade } from '@/store/types';
import { timestampHour } from '@/shared/formatters';

// Define Column labels here to avoid typos
const CHART_PROFIT = 'Profit';
const CHART_TRADE_COUNT = 'Trade Count';

@Component({
  components: {
    'v-chart': ECharts,
  },
})
export default class HourlyChart extends Vue {
  @Prop({ required: true }) trades!: Array<Trade>;

  get hourlyData() {
    const res = new Array(24);
    for (let i = 0; i < 24; i += 1) {
      res[i] = { hour: i, hourDesc: `${i}h`, profit: 0, count: 0 };
    }

    for (let i = 0, len = this.trades.length; i < len; i += 1) {
      const trade = this.trades[i];
      if (trade.close_timestamp) {
        const hour = timestampHour(trade.close_timestamp);

        res[hour].profit += trade.close_profit;
        res[hour].count += 1;
      }
    }
    return res;
  }

  get hourlyChartOptions() {
    console.log(this.hourlyData);
    return {
      title: {
        text: 'Hourly Profit',
        show: true,
      },
      dataset: {
        dimensions: ['hourDesc', 'profit', 'count'],
        source: this.hourlyData,
      },
      tooltip: {
        trigger: 'axis',
        axisPointer: {
          type: 'line',
          label: {
            backgroundColor: '#6a7985',
          },
        },
      },
      legend: {
        data: [CHART_PROFIT, CHART_TRADE_COUNT],
      },
      xAxis: {
        type: 'category',
      },
      yAxis: [
        {
          type: 'value',
          name: CHART_PROFIT,
          splitLine: {
            show: false,
          },
          nameRotate: 90,
          nameLocation: 'middle',
          nameGap: 30,
        },
        {
          type: 'value',
          name: CHART_TRADE_COUNT,
          nameRotate: 90,
          nameLocation: 'middle',
          nameGap: 30,
        },
      ],
      series: [
        {
          type: 'line',
          name: CHART_PROFIT,
          color: 'black',
        },
        {
          type: 'bar',
          name: CHART_TRADE_COUNT,
          color: 'rgba(150,150,150,0.3)',
          yAxisIndex: 1,
        },
      ],
    };
  }
}
</script>

<style scoped></style>