<template>
<div>
  <div style="text-align:center;font-size:16px;">
    <span style="color:#01DFD7;font-size:14px;">시가: {{ nowPrice.open }}</span>&nbsp;
    <span style="color:red;font-size:14px;">고가: {{ nowPrice.high }}</span>&nbsp;
    <span style="color:skyblue;font-size:14px;">저가: {{ nowPrice.low}}</span>&nbsp;
    <span style="color:green;font-size:14px;">종가: {{ nowPrice.close }}</span>
  </div>
  <div class="chart_box" style="overflow:hidden">
    <div class="chart_screen" style="color:#000000;float:left;overflow:hidden">
      <div ref="chartdiv" style="width:85%;height:500px;margin:0px;float:left"></div>
    </div>
    <div class="chart_indicator" style="position:absolute;right:5%;top:50%;color:#ffffff;"><span class="close_value" style="background-color:red;font-size:14px;">{{ nowPrice.close }}</span></div>
  </div>
</div>
</template>
<style>
  .chart_screen {
    width:100%;
    float:left;
  }
  .chart_indicator{
    height:500px;    
    float:left;
  }

</style>
<script>
// import WebSocket from 'ws';
// const WebSocket = require('ws');
import ReconnectingWebSocket from 'reconnecting-websocket';

export default {
  
  methods :{
    async getData(){
      // let candleData = await this.$axios.get('http://182.23.209.111:3001/test');
      return candleData
    },
    showPrice(){
      
    }
  },
  data : ()=>({  
          nowPrice : {
              open : '',
              high : '',
              low : '',
              close : '',
          },
          chartData : []
        
  }),
  async mounted() {
    let symbol = this.$route.query.symbol
    let bong = this.$route.query.bong
    const ws = new ReconnectingWebSocket('ws://175.126.62.101:8382/candle');
    // const ws = new WebSocket('ws://141.164.55.165:8888')
    ws.onopen = function open() {
      
      ws.send(JSON.stringify({action: ['candles'], instrument_id: `${symbol}`, minute: `${bong}`}));
    };
        
    // ws.send(JSON.stringify({action: ['candles'], instrument_id: 'BTC-USDT', minute: 1}));

    let {am4core, am4charts, am4themes_animated, am4themes_dark} = this.$am4core();

    function am4themes_myTheme(target) {
      if (target instanceof am4core.InterfaceColorSet) {
        target.setFor("text", am4core.color("#ffffff"));
        
      }
    }

    am4core.useTheme(am4themes_myTheme);
    // Themes end
     let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
    chart.background.fill = '#202028'
    chart.paddingRight = 10;
    chart.paddingLeft = 50;
    
    // chart.dateFormatter.inputDateFormat = "yyyy-MM-dd";
    
    let dateAxis = chart.xAxes.push(new am4charts.DateAxis());
    dateAxis.renderer.grid.template.location = 1;
    dateAxis.skipEmptyPeriods = true;
    let valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
    valueAxis.tooltip.disabled = true;
    // dateAxis.renderer.grid.template.stroke = "#000000";
    // valueAxis.renderer.grid.template.stroke = "#000000";
    let series = chart.series.push(new am4charts.CandlestickSeries());
    valueAxis.fontSize = 12;
    series.dataFields.dateX = "date";
    series.dataFields.valueY = "close";
    series.dataFields.openValueY = "open";
    series.dataFields.lowValueY = "low";
    series.dataFields.highValueY = "high";
    
    series.tooltipText = "{origin_time}\n시가:{openValueY.value}\n고가:{lowValueY.value}\n저가:{highValueY.value}\n종가:{valueY.value}";

    // important!
    // candlestick series colors are set in states. 
    series.riseFromOpenState.properties.fill = am4core.color("#f5365c");
    series.dropFromOpenState.properties.fill = am4core.color("#5e72e4");
    series.riseFromOpenState.properties.stroke = am4core.color("#f5365c");
    series.dropFromOpenState.properties.stroke = am4core.color("#5e72e4");
    
    // series.riseFromPreviousState.properties.fillOpacity = 1;
    // series.dropFromPreviousState.properties.fillOpacity = 0;

    chart.cursor = new am4charts.XYCursor();

    // a separate series for scrollbar
    // let lineSeries = chart.series.push(new am4charts.LineSeries());
    // lineSeries.dataFields.dateX = "date";
    // lineSeries.dataFields.valueY = "close";
    // need to set on default state, as initially series is "show"
    // lineSeries.defaultState.properties.visible = false;
    // hide from legend too (in case there is one)
    // lineSeries.hiddenInLegend = true;
    // lineSeries.fillOpacity = 0.5;
    // lineSeries.strokeOpacity = 0.5;

    // let scrollbarX = new am4charts.XYChartScrollbar();
    // scrollbarX.series.push(lineSeries);
    // chart.scrollbarX = scrollbarX;

      ws.onmessage = (event) => {
          let msg = JSON.parse(event.data)
          let chartDataContainer = []
          if(msg.Message !== "Accepted"){
            msg.message.forEach((v,i)=>{
                chartDataContainer.push({
                  "date": parseInt(msg.message[i].time),
                  "open": `${msg.message[i].open}`,
                  "high": `${msg.message[i].high}`,
                  "low": `${msg.message[i].low}`,
                  "close": `${msg.message[i].close}`,
                  "origin_time":`${new Date(msg.message[i].origin_time).toLocaleTimeString()}`
                })
            })
            chartDataContainer.reverse()
            let mergedObjectTemp = {}
            
            mergedObjectTemp = { open : chartDataContainer[chartDataContainer.length-1].open,
              high : chartDataContainer[chartDataContainer.length-1].high,
              low : chartDataContainer[chartDataContainer.length-1].low,
              close : chartDataContainer[chartDataContainer.length-1].close }
            this.nowPrice = Object.assign({},this.nowPrice,mergedObjectTemp)
            
            chart.data = chartDataContainer
          }
      }
      

      //  ws.onmessage = function (event) {
      //     let candleData = JSON.parse(event.data)
      //     let candleJson = []
      //     candleJson = JSON.parse(candleData.data).reverse()
      //     console.log(candleJson)
      //     let chartDataContainer = []
      //     candleJson.forEach((v,i)=>{      
      //       chartDataContainer.push({
      //         "date": `${new Date(candleJson[i][0]).getTime() / 1000}`,
      //         "open": `${candleJson[i][1]}`,
      //         "high": `${candleJson[i][2]}`,
      //         "low": `${candleJson[i][3]}`,
      //         "close": `${candleJson[i][4]}`
      //       })
      //       chart.data = chartDataContainer
      //       console.log(chart.data)
      //     })
          
      // }
     
    
    },
    beforeDestroy() {
      if (this.chart) {
        this.chart.dispose();
      }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@media screen and (max-width: 768px) { 
  .close_value {font-size : 16px !important; }
  .close_text { display : none;}
}

.hello {
  width: 100%;
  height: 500px;
}
</style>

