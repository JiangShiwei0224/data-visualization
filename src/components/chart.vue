<template>
  <div class="container">
    <infoPrompt class="prompt"></infoPrompt>
    <searchInput class="search"></searchInput>
    <div id="myChart" :style="{width: '800px', height: '550px'}"></div>
  </div>
</template>

<script>
import searchInput from './searchInput.vue'      //引入搜索框组件
import infoPrompt from './infoPrompt.vue'      //引入搜索框组件
import $ from 'jquery'         //引入jQuery

export default {
  name : 'chart',
  components: {
    searchInput,
    infoPrompt
  },
  data () {
    return {
      data: {},
      url: __dirname+'./static/datas/test.json',
    }
  },
  computed: {
  },
  mounted(){
    this.getDatas(this.url)         //获取要渲染的数据
  },
  methods: {
    getDatas(url){
      var _this= this;
      this.$axios.get(url)      //异步获取数据并存储到组件的data中      
                  .then(function (res){
                    _this.data = res.data.charts
                    _this.drawchart()             //渲染图表 
                  })
    },
    drawchart(){
      let dataName = [];              //定义一个数组，用来存储数据中的name
      let enableName = [];             //定义一个数组，用于存放ignored为'ENABLE'的name
      let disableName = [];              //定义一个数组，用于存放ignored为'DISABLE'的name
      let enableValue = [];             //定义一个数组，存放所有gains，当ignored为'ENABLE'时，正常存储，当ignored为'DISABLE'时，存储gains为0
      let onlyEnableValue = [];         //定义一个数组，仅存放ignored为'ENABLE'的gains
      let disableValue = [];            //定义一个数组，存放所有gains，当ignored为'DISABLE'时，正常存储，当ignored为'ENABLE'时，存储gains为0
      let onlyDisableValue = [];        //定义一个数组，仅存放ignored为'DISABLE'的gains
      let eFlag = true                  //标记ignored为'ENABLE'的数据是否显示，“可控”图例是否被点击
      let dFlag = true                  //标记ignored为'DISABLE'的数据是否显示，“不可控”图例是否被点击

      this.data.forEach(element => {         //遍历数据，将数据分别存入满足条件的数组中
        dataName.push(element.name)
        element.ignored === 'ENABLE'?(function(){
          enableName.push(element.name);
          onlyEnableValue.push(element.gains);
          enableValue.push(element.gains);
          disableValue.push(0)
        })():(function(){
          disableName.push(element.name);          
          onlyDisableValue.push(element.gains);
          enableValue.push(0);
          disableValue.push(element.gains)
        })()
      });
      
      let myChart = this.$echarts.init(document.getElementById('myChart'))     //创建一个echarts实例
      var option = {               //echarts图表配置项
        legend: {                  //图例
          data: ['可控', '不可控']
        },
        tooltip:{                  //图例的配置
          trigger: 'item',
        },
        dataZoom:{                 //数据区域缩放组件
          left: "92%",
          start: 0,
          end: 100,
          filterMode: 'empty',     //数据过滤
          yAxisIndex: [0]
        },
        xAxis: {
          type: 'value'            //数据
        },
        yAxis: {
          type: 'category',        //类目
          triggerEvent: "true",
          data: dataName
        },
        series: [
          {
            name: '可控',
            type: 'bar',
            stack: '总量',
            color: '#174775',
            data: enableValue
          },
          {
            name: '不可控',
            type: 'bar',
            stack: '总量',
            color: '#c2c2c2',
            data: disableValue
          },
          {
            type: "line",
            name: "当前选择",
            markLine: {         //标线，初始化后标示最后一行
              silent: true,
              data: [{
                yAxis: dataName[0]
              }]
            }
          }
        ]
      }
      myChart.setOption(option)       //根据配置项渲染图表
      
      this.makingMove(myChart,this.data)
      this.searchClick(myChart,this.data)
      this.attributeChange(myChart,this.data,enableValue,disableValue,eFlag,dFlag)
      this.legendChange(myChart,eFlag,dFlag,dataName,enableName,disableName,onlyEnableValue,onlyDisableValue,enableValue,disableValue)
    },
    makingMove(myChart,data){
      myChart.on('click',function (params){
      data.forEach(element => {
          element.name == params.name?console.log(element.id):''         //打印id
        });
        var option = {
          series:[{
              name : "当前选择",
              markLine: {
              data: [{yAxis: ''+params.name}]                         //重新配置图表数据，重新渲染图表
              }
            }
          ]}
        myChart.setOption(option)
      })
    },
    searchClick(myChart,data){
      $('.autocomplete-suggestions').off('click').on('click',function (e){     //点击事件搜索功能 
        let reg = /<strong>/g;
        let reg1 = /<\/strong>/g;
        let target = e.target||e.srcElement
        let str = $(target).html();
        let eValue=[] ;
        let dValue=[] ;
        let name = [];
        let id;
        
        str = str.replace(reg, '');
        str = str.replace(reg1, '');
        
        data.forEach(element => {
            if (element.name == str) {
                id = element.id;
                console.log(id);
                if (element.ignored == "ENABLE") {
                    eValue.push(element.gains);
                    dValue=[0]   
                }else{
                    dValue.push(element.gains);   
                    eValue=[0]
                }
                name.push(str)
            }
        });
        myChart.setOption({
          xAxis: {
            type: 'value'
          },
          yAxis: {
            type: 'category',
            triggerEvent: "true",
            data: name
          },
          series: [{
              data: eValue
          },
          {
              data: dValue
          },
          {
              markLine: {
              data: [{yAxis: name}]
              }
          }]
        })
      })
    },
    attributeChange(myChart,data,enableValue,disableValue,eFlag,dFlag){
      myChart.on('click',function (params){           //改变可控不可控属性
        data.forEach(element => {
          if(element.name == params.value){             //点击后交换ignored的值
            element.ignored == 'ENABLE'?element.ignored = 'DISABLE':element.ignored = 'ENABLE'
            enableValue[(element.id % 2017111300 - 1)] = disableValue[(element.id % 2017111300 - 1)] - enableValue[(element.id % 2017111300 - 1)]
            disableValue[(element.id % 2017111300 - 1)] = disableValue[(element.id % 2017111300 - 1)] - enableValue[(element.id % 2017111300 - 1)]
            enableValue[(element.id % 2017111300 - 1)] = disableValue[(element.id % 2017111300 - 1)] + enableValue[(element.id % 2017111300 - 1)]
            console.log(element.id)
            console.log(element.ignored)
          }
        });
        if(eFlag == true&&dFlag == true){
          var option = {
            series: [{
              data: enableValue
            },                                  //可控不可控数据均显示时，重新配置图表数据，重新渲染图表
            {
              data: disableValue
            }
          ]}
          myChart.setOption(option)
        }
      })
    },
    legendChange(myChart,eFlag,dFlag,dataName,enableName,disableName,onlyEnableValue,onlyDisableValue,enableValue,disableValue){
      myChart.on('legendselectchanged',function (params){             //监听图例的改变
        if(params.name == '可控'){
          if(eFlag){
            eFlag = false
            var option = {
              yAxis: {
                data: disableName
              },
              series: [{
                data: []
              },
              {
                data: onlyDisableValue
              },
              {
                markLine: {
                  data: [{yAxis: 'cabbage'}]
                }
              }]
            }
            myChart.setOption(option)
          }else{
            eFlag = true
            if(dFlag == true){
              var option = {
                yAxis: {
                  data: dataName
                },
                series: [
                {
                  data: enableValue
                },
                {
                  data: disableValue
                } ]
              }
              myChart.setOption(option)
            }else{
              var option = {
                yAxis: {
                  data: enableName
                },
                series: [{
                  data: onlyEnableValue
                },
                {
                  data: []
                },
                {
                  markLine: {
                    data: [{yAxis: 'broccoli'}]
                  }
                }]
              }
              myChart.setOption(option)
            }
          }
        }
        if(params.name == '不可控'){
          if(dFlag){
            dFlag = false
            var option = {
              yAxis: {
                data: enableName
              },
              series: [{
                data: onlyEnableValue
              },
              {
                data: []
              },
              {
                markLine: {
                  data: [{
                    yAxis: 'broccoli'
                  }]
                }
              }]
            }
            myChart.setOption(option)
          }else{
            dFlag = true
            if(eFlag == true){
              var option = {
                yAxis: {
                  data: dataName
                },
                series: [
                {
                  data: enableValue
                },
                {
                  data: disableValue
                }]
              }
              myChart.setOption(option)
            }else{
              var option = {
                yAxis: {
                  data: disableName
                },
                series: [{
                  data: ''
                },
                {
                  data: onlyDisableValue
                },
                {
                  markLine: {
                    data: [{yAxis: 'cabbage'}]
                  }
                }]
              }
              myChart.setOption(option)
            }
          }
        }
        if (eFlag == false&&dFlag == false) {
          var option = {
              yAxis: {
                data: ['无数据']
              },
              series: [
              {
                type: "line",
                name: "当前选择",
                markLine: { 
                  data: []
                }
              }]
            }
            myChart.setOption(option)
        }
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .container{
    width: 800px;
    height: 600px;
    position: relative;
  }
  .search{
    position: absolute;
    right: 0;
    z-index: 50;
  }
</style>
