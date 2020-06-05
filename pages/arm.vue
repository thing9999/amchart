<template>
<div>
  <div style="text-align:center;font-size:14px;">
    <span style="color:#01DFD7;">시가: {{ nowPrice.open }}</span>&nbsp;
    <span style="color:red">고가: {{ nowPrice.high }}</span>&nbsp;
    <span style="color:skyblue">저가: {{ nowPrice.low }}</span>&nbsp;
    <span style="color:green">종가: {{ nowPrice.close }}</span>
  </div>
  <div class="chart_box">
    <div class="chart_screen" >
      <div ref="chartdiv" style="width:100%;height:500px;"></div>
      <div class="chart_indicator" style="position:absolute;right:5%;top:50%;color:#ffffff;">
        <span class="close_value" style="background-color:red;font-size:15px;font-weight:bold">{{ nowPrice.close }}</span>
        </div>
    </div>
  </div>
</div>
</template>
<style>
  .chart_box {
    margin-left:-15px;
    overflow:visible  !important;
  }
  .chart_screen {
    padding-right:50px !important;
    width:90%;
    overflow:visible  !important;
    float:left;
  }
  .chart_indicator{
    height:500px;    
    float:left;
  }

</style>
<script>
import ReconnectingWebSocket from 'reconnecting-websocket';

export default {
  
  methods :{
    async getData(){
      let candleData = await this.$axios.get('http://182.23.209.111:3001/test');
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
    let  ws 
    if(symbol === 'BTC-USDT'){
      ws = new ReconnectingWebSocket('ws://175.126.62.101:8282/candle');
    } else {
      ws = new ReconnectingWebSocket('ws://175.126.62.101:8382/candle');
    }
    
    
    ws.onopen = function open() {
      ws.send(JSON.stringify({action: ['candles'], instrument_id: `${symbol}`, minute: `${bong}`}));
    };
        
    let {am4core, am4charts, am4themes_animated, am4themes_dark} = this.$am4core();
    am4core.useTheme(am4themes_animated);
    // Themes end
     let chart = am4core.create(this.$refs.chartdiv, am4charts.XYChart);
    
    chart.paddingRight = 1;
    chart.paddingLeft = 50;
    chart.dateFormatter.inputDateFormat = "yyyy-MM-dd HH:mm:ss";
    
    let dateAxis = chart.xAxes.push(new am4charts.DateAxis());
    dateAxis.renderer.grid.template.location = 0;
    dateAxis.skipEmptyPeriods = true;
    let valueAxis = chart.yAxes.push(new am4charts.ValueAxis());
    valueAxis.tooltip.disabled = true;
    valueAxis.fontSize = 12;
    let series = chart.series.push(new am4charts.CandlestickSeries());
    series.dataFields.dateX = "date";
    series.dataFields.valueY = "close";
    series.dataFields.openValueY = "open";
    series.dataFields.lowValueY = "low";
    series.dataFields.highValueY = "high";
    series.columns.template.width = am4core.percent(70);
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
    let lineSeries = chart.series.push(new am4charts.LineSeries());
    lineSeries.dataFields.dateX = "date";
    lineSeries.dataFields.valueY = "close";
    // need to set on default state, as initially series is "show"
    lineSeries.defaultState.properties.visible = false;

    // hide from legend too (in case there is one)
    lineSeries.hiddenInLegend = true;
    lineSeries.fillOpacity = 0.5;
    lineSeries.strokeOpacity = 0.5;

    let scrollbarX = new am4charts.XYChartScrollbar();
    scrollbarX.series.push(lineSeries);
    chart.scrollbarX = scrollbarX;

      ws.onmessage = (event) => {
          let msg = JSON.parse(event.data)
          let chartDataContainer = []
          if(msg.Message !== "Accepted"){
            msg.message.forEach((v,i)=>{
                 let insertDate = new Date(msg.message[i].origin_time).getFullYear()+'-'+(new Date(msg.message[i].origin_time).getMonth()+1)+'-'+new Date(msg.message[i].origin_time).getDate()+' '+new Date(msg.message[i].origin_time).getHours()+':'+new Date(msg.message[i].origin_time).getMinutes()+':'+new Date(msg.message[i].origin_time).getSeconds()
                chartDataContainer.push({
                  "date": `${insertDate}`,
                  "open": `${msg.message[i].open}`,
                  "high": `${msg.message[i].high}`,
                  "low": `${msg.message[i].low}`,
                  "close": `${msg.message[i].close}`,
                  "origin_time":`${new Date(msg.message[i].origin_time).toLocaleTimeString()}`
                })
            })
            chartDataContainer.reverse()
            let mergedObjectTemp = {}
            function rpad(str, padLen, padStr) {
                if(str.indexOf('.') === -1 ){
                  return str+".00";  
                }
                if (padStr.length > padLen) {
                    // console.log("오류 : 채우고자 하는 문자열이 요청 길이보다 큽니다");
                    return str
                } 
                  str += ""; // 문자로
                  padStr += ""; // 문자로
                  while (str.length < padLen)
                      str += padStr;
                  str = str.length >= padLen ? str.substring(0, padLen) : str;
                  return str;  
            }
         
              
              if(symbol === "BTC-USDT"){
                  mergedObjectTemp = { 
                    open : rpad(chartDataContainer[chartDataContainer.length-1].open,7,'0'),
                    high : rpad(chartDataContainer[chartDataContainer.length-1].high,7,'0'),
                    low : rpad(chartDataContainer[chartDataContainer.length-1].low,7,'0'),
                    close : rpad(chartDataContainer[chartDataContainer.length-1].close ,7,'0')
                  }
              } else {
                  mergedObjectTemp = { 
                    open : rpad(chartDataContainer[chartDataContainer.length-1].open,7,'0'),
                    high : rpad(chartDataContainer[chartDataContainer.length-1].high,7,'0'),
                    low : rpad(chartDataContainer[chartDataContainer.length-1].low,7,'0'),
                    close : rpad(chartDataContainer[chartDataContainer.length-1].close ,7,'0')
                  }
              }
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
  .close_value {font-size : 14px; }
}
.hello {
  width: 100%;
  height: 500px;
}
</style>

