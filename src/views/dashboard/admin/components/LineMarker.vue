<template>
  <div :id="id" :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
import resize from './mixins/resize'

import { queryDynamicAccidentsStatistics } from '@/api/statics'
// import { queryDynamicAccidentsStatistics } from '@/api/statics2'

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    id: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: 'calc(55vh - 25px)'
    }
  },
  data() {
    return {
      dataZoom_start: 0,
      dataZoom_end: 100,
      data: [],
      date: '',
      chart: null,
      // xData: ['13:00', '13:05', '13:10', '13:15', '13:20', '13:25', '13:30', '13:35', '13:40', '13:45', '13:50', '13:55', '14:00', '14:05', '14:10', '14:15', '14:20', '14:25', '14:30', '14:35', '14:40', '14:45', '14:50', '14:55'],
      // yData: {
      //   zhuiweiData: [220, 182, 191, 134, 150, 120, 11, 125, 145, 122, 165, 122, 220, 182, 191, 134, 150, 120, 110, 125, 145, 122, 165, 122],
      //   zhixingData: [120, 110, 125, 145, 122, 165, 122, 220, 0, 191, 134, 150, 12, 110, 125, 145, 122, 165, 122, 220, 182, 191, 134, 150],
      //   chaocheData: [220, 182, 125, 145, 122, 191, 134, 150, 120, 110, 165, 122, 220, 182, 125, 145, 122, 191, 134, 150, 120, 110, 165, 122],
      //   qitaDate: [12, 11, 12, 15, 22, 15, 12, 20, 0, 11, 13, 50, 1, 10, 15, 45, 22, 15, 22, 20, 82, 91, 34, 50]
      // }
      xData: [],
      yData: {
        zhuiweiData: [],
        zhixingData: [],
        chaocheData: [],
        qitaDate: []
      },
      xData2: [],
      yData2: {
        zhuiweiData: [],
        zhixingData: [],
        chaocheData: [],
        qitaDate: []
      }
    }
  },
  watch: {
    'xData2': function (val) {
      if (this.xData.length > 50) {
        // this.xData.shift() // 移除第一个元素
        // this.yData.chaocheData.shift()
        // this.yData.zhixingData.shift()
        // this.yData.zhuiweiData.shift()
        // this.yData.qitaDate.shift()
      }
      this.initChart()
      this.times()
    },
    'yData2': function (val) {
      this.initChart()
      this.times()
    }
  },
  mounted() {
    this.chart = echarts.init(document.getElementById(this.id))
    this.initChart()
    this.updateData()
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
    clearTimeout(this.times)
  },
  methods: {
    // 动态获取xData和yData
    updateData() {
      if (this.xData.length > 20 && this.xData.length < 30) {
        this.dataZoom_start = 50
      } 
      if (this.xData.length > 30) {
        this.dataZoom_start = 75
      }

      queryDynamicAccidentsStatistics().then( response => {
        if (response != '') {
          this.data = response.data
          console.log(this.data)
          // this.total = response.data.total
          // 用于显示几号的事故统计
          this.date = this.data.time.substring(0, 10)
          // console.log(this.date.slice(11))
          this.yData2.chaocheData.push( this.data.超车事故 )
          this.yData2.zhixingData.push( this.data.直行事故 )
          this.yData2.zhuiweiData.push( this.data.追尾事故 )
          this.yData2.qitaDate.push( this.data.其他事故 )
          this.xData2.push( this.data.time.substring(11))
          this.yData = this.yData2
          this.xData = this.xData2
        }
      })
    },
    // 定时器，数据发生变化时调用
    times(){
      return setTimeout(()=>{
				this.updateData()
        // console.log(this.xData, this.yData)
			}, 100 * 1000)
		},
    initChart() {
      // this.chart = echarts.init(document.getElementById(this.id))
      var option = {
        backgroundColor: '#24347c', //'#243382', //'#1d2870',
        title: {
          top: 20,
          text: this.date + ' 交通事故统计',
          textStyle: {
            fontWeight: 'normal',
            fontSize: 16,
            color: '#F1F1F3'
          },
          left: '1%'
        },
        // 鼠标选中数据指示浮窗
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            lineStyle: {
              color: 'f1ffff'
            }
          }
        },
        // 右上角图注
        legend: {
          top: 20,
          icon: 'rect',
          itemWidth: 15,
          itemHeight: 5,
          itemGap: 13,
          data: ['追尾事故', '直行事故', '超车事故', '其他事故'],
          right: '4%',
          textStyle: {
            fontSize: 12,
            color: '#F1F1F3'
          }
        },
        grid: {
          top: 100,
          left: '2%',
          right: '2%',
          bottom: 65,
          borderWidth: 0,
          containLabel: true
        },
        // x轴样式
        xAxis: [{
          type: 'category',
          boundaryGap: false,
          axisLine: {
            lineStyle: {
              color: '#f1ffff9e'
            }
          },
          splitLine: {
            show: false
          },
          axisTick: {
            show: false
          },
          splitArea: {
            show: false
          },
          axisLabel: {
            interval: 0
          },
          data: this.xData
        }],
        // y轴样式
        yAxis: [{
          type: 'value',
          name: '(%)',
          axisTick: {
            show: false
          },
          axisLine: {
            lineStyle: {
              color: '#f1ffff9e'
            }
          },
          axisLabel: {
            margin: 10,
            // interval: 0,
            textStyle: {
              fontSize: 14
            }
          },
          // 内部坐标网格线
          splitLine: {
            lineStyle: {
              color: '#6d77ac'
            }
          }
        }],
        // 图片放缩条
        dataZoom: [{
          show: true,
          height: 20,
          xAxisIndex: [
            0
          ],
          bottom: 30,
          start: this.dataZoom_start,
          end: this.dataZoom_end,
          handleIcon: 'path://M306.1,413c0,2.2-1.8,4-4,4h-59.8c-2.2,0-4-1.8-4-4V200.8c0-2.2,1.8-4,4-4h59.8c2.2,0,4,1.8,4,4V413z',
          handleSize: '110%',
          handleStyle: {
            color: '#d3dee5'

          },
          textStyle: {
            color: '#fff' },
          borderColor: '#90979c'

        }, {
          type: 'inside',
          show: true,
          height: 15,
          start: 1,
          end: 35
        }],
        // series里面放数据
        series: [{
          name: '追尾事故',
          type: 'line',
          smooth: true,
          symbol: 'circle',
          symbolSize: 5,
          showSymbol: false,
          lineStyle: {
            normal: {
              width: 1
            }
          },
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(137, 189, 27, 0.3)'
              }, {
                offset: 0.8,
                color: 'rgba(137, 189, 27, 0)'
              }], false),
              shadowColor: 'rgba(0, 0, 0, 0.1)',
              shadowBlur: 10
            }
          },
          itemStyle: {
            normal: {
              color: 'rgb(137,189,27)',
              borderColor: 'rgba(137,189,2,0.27)',
              borderWidth: 12

            }
          },
          data: this.yData.zhuiweiData
        }, {
          name: '直行事故',
          type: 'line',
          smooth: true,
          symbol: 'circle',
          symbolSize: 5,
          showSymbol: false,
          lineStyle: {
            normal: {
              width: 1
            }
          },
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(0, 136, 212, 0.3)'
              }, {
                offset: 0.8,
                color: 'rgba(0, 136, 212, 0)'
              }], false),
              shadowColor: 'rgba(0, 0, 0, 0.1)',
              shadowBlur: 10
            }
          },
          itemStyle: {
            normal: {
              color: 'rgb(0,136,212)',
              borderColor: 'rgba(0,136,212,0.2)',
              borderWidth: 12
            }
          },
          data: this.yData.zhixingData
        }, {
          name: '超车事故',
          type: 'line',
          smooth: true,
          symbol: 'circle',
          symbolSize: 5,
          showSymbol: false,
          lineStyle: {
            normal: {
              width: 1
            }
          },
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(219, 50, 51, 0.3)'
              }, {
                offset: 0.8,
                color: 'rgba(219, 50, 51, 0)'
              }], false),
              shadowColor: 'rgba(0, 0, 0, 0.1)',
              shadowBlur: 10
            }
          },
          itemStyle: {
            normal: {
              color: 'rgb(219,50,51)',
              borderColor: 'rgba(219,50,51,0.2)',
              borderWidth: 12
            }
          },
          data: this.yData.chaocheData
        }, {
          name: '其他事故',
          type: 'line',
          smooth: true,
          symbol: 'circle',
          symbolSize: 5,
          showSymbol: false,
          lineStyle: {
            normal: {
              width: 1
            }
          },
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(255, 228, 196, 0.3)'
              }, {
                offset: 0.8,
                color: 'rgba(255, 228, 196, 0)'
              }], false),
              shadowColor: 'rgba(0, 0, 0, 0.1)',
              shadowBlur: 10
            }
          },
          itemStyle: {
            normal: {
              color: 'rgb(255, 228, 196)',
              borderColor: 'rgba(255, 228, 196, 0.2)',
              borderWidth: 12
            }
          },
          data: this.yData.qitaDate
        }]
      }
      this.chart.setOption(option, true)
    }
  }
}
</script>


<!-- <style scoped>
.chart-container{
  position: relative;
  width: 100%;
  height: calc(100vh - 84px);
  color: #f1ffff91;
}
</style> -->

