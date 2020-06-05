<template>
    <div class="frame_gaurd">
    
    <table class='gameinfo_table'>
      <thead>
      <tr>
        <th class='  fw1' colspan='2'>매도현황</th>
        <th class='  fw1' colspan='2'>매수현황</th>
        <th class='  fw1'>회원 매도 승</th>
        <th class='  fw1'>회원 매도 패</th>
        <th class='  fw1'>회원 매수 승</th>
        <th class='  fw1'>회원 매수 패</th>
        <th class=' ' rowspan='2'>예상 지급 수수료</th>
        <th class=' ' rowspan='2'>예상 수익 합계</th>
      </tr>
      <tr>
        <th class=' '>거래 포인트</th>
        <th class=' '>비율%</th>
        <th class=' '>거래 포인트</th>
        <th class=' '>비율%</th>
        <th class=' '>(본사 수익 기준)</th>
        <th class=' '>(본사 수익 기준)</th>
        <th class=' '>(본사 수익 기준)</th>
        <th class=' '>(본사 수익 기준)</th>
        
      </tr>
      </thead>
      <tbody>
      <tr>
        <td class=' '>
        {{  realTimeData.sellprice | currency }}
        </td>
        <td class=' '>
        {{ parseInt(realTimeData.sellprice / (realTimeData.sellprice + realTimeData.buyprice) * 100) | removeNan  }}
        </td>
        <td class=' '>
        {{ realTimeData.buyprice | currency}}
        </td>
        <td class=' '>
        {{ parseInt(realTimeData.buyprice / (realTimeData.sellprice + realTimeData.buyprice) * 100) | removeNan }}
        </td>


        <td class=' '>
        {{ realTimeData.sellWin | currency}}
        </td>
        <td class=' '>
        {{ realTimeData.selllose | currency}}
        </td>
        <td class=' '>
         {{ realTimeData.buyWin | currency}}
        </td>
        <td class=' '>
         {{ realTimeData.buyloss |currency }}
        </td>
        <td class=' '>
        매도 : {{ realTimeData.fee_loss | currency }}<br>
        매수 : {{ realTimeData.fee_win | currency }}
        </td>
        <td class=' '>
        매도 : {{ realTimeData.buyimport | currency }}<br>
        매수 : {{ realTimeData.sellimport | currency }}
        </td>
      </tr>
      </tbody>
    </table>
    
    </div>
  </template>
  <style scoped>
    
    th
    {
      background-color:#36a2f5 !important;
      padding:5px;
      font-size:13px;
      color:#ffffff;
      border-right:1px solid #dee2e6;
      border-bottom:1px solid #dee2e6;
    }
    td 
    {
      padding: 15px;
      text-align: right;
      border-right:1px solid #dee2e6;
      border-bottom:1px solid #dee2e6;
    }
    .gameinfo_table{
      
      width:100%;
      height:100px;
      border-top:1px solid #dee2e6;
      border-left:1px solid #dee2e6;
    }
    table
    {
      border-spacing: 0px;
    }
  </style>
  
  <script>
    // symbol ==> 심볼
    // sell_price  ==> 매도금액
    // buy_price ==> 매수금액
    // fee_win ==> 유저기준승 수수료
    // fee_loss ==> 유저기준패 수수료
    // t_num ==> 거래회차
    export default {
      data : function(){  
        return {
            
            intervalid1:'',
            realTimeData : {
              symbol:'',
              sell_price:0,
              buy_price:0,
              fee_win:0,
              fee_loss:0,
              t_num:'',
            }
        }
    },
    filters : {
      removeNan(nan){
        if(!nan){
          return 0;
        }
      },
      currency(val) {
        if (!val) return '0'
        if (typeof val === 'number') val = val.toString()
        return val.replace(/(\d)(?=(\d{3})+(?:\.\d+)?$)/g, "$1,")
      }
    },
    computed:{
        changes : {
            get : function(){
                return this.pgvalue;
            },
            set : function(v){
                this.pgvalue =  v;
            }
        }
    },
    mounted : async function(){
      console.log(this.$route.params)
        this.todo()
        
    },
    beforeDestroy () {
       clearInterval(this.intervalid1)
    },
    methods : {
        todo : function(){ 
          
            this.intervalid1 = setInterval(async () => {
                let { data } = await this.$axios.get(`http://182.23.209.111:3001/api/realtime/game?site=${this.$route.query.site}&bunbong=${this.$route.query.bunbong}`)   
                this.realTimeData = Object.assign({},this.realTimeData,data)
                console.log(JSON.stringify(this.realTimeData))
            }, 1000);
        }
      }
    }
  </script>
  