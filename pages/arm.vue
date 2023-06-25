<template>
  <div>
    <div>
      <span>시가: {{ nowPrice.open }}</span>
      <span>고가: {{ nowPrice.high }}</span>
      <span>저가: {{ nowPrice.low }}</span>
      <span>종가: {{ nowPrice.close }}</span>
    </div>
    <div>
      <div>
        <div ref="chartdiv"></div>
        <div>
          <span class="close_value">{{ nowPrice.close }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
  /* CSS styles for the chart */
</style>

<script>
import ReconnectingWebSocket from 'reconnecting-websocket';
import { create as am4coreCreate } from '@amcharts/amcharts4/core';
import * as am4charts from '@amcharts/amcharts4/charts';
import am4themes_animated from '@amcharts/amcharts4/themes/animated';

export default {
  data() {
    return {
      nowPrice: {
        open: '',
        high: '',
        low: '',
        close: '',
      },
    };
  },
  mounted() {
    // WebSocket setup
    const ws = new ReconnectingWebSocket('ws://175.126.62.101:8282/candle');
    
    ws.onopen = () => {
      ws.send(JSON.stringify({ action: ['candles'], instrument_id: `${symbol}`, minute: `${bong}` }));
    };

    ws.onmessage = (event) => {
      const msg = JSON.parse(event.data);
      if (msg.Message !== "Accepted") {
        const chartDataContainer = msg.message.map((item) => {
          const insertDate = new Date(item.origin_time);
          return {
            date: `${insertDate}`,
            open: `${item.open}`,
            high: `${item.high}`,
            low: `${item.low}`,
            close: `${item.close}`,
            origin_time: `${insertDate.toLocaleTimeString()}`,
          };
        }).reverse();

        const mergedObjectTemp = {
          open: chartDataContainer[chartDataContainer.length - 1].open,
          high: chartDataContainer[chartDataContainer.length - 1].high,
          low: chartDataContainer[chartDataContainer.length - 1].low,
          close: chartDataContainer[chartDataContainer.length - 1].close,
        };

        this.nowPrice = { ...mergedObjectTemp };
        chart.data = chartDataContainer;
      }
    };

    // amCharts setup
    const chart = am4coreCreate(this.$refs.chartdiv, am4charts.XYChart);
    // Chart configuration code

    this.chart = chart;
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose();
    }
  },
};
</script>
