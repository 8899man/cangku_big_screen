<template>
  <div class="warehouse flex-box">
  <fs-boom :visible="boomShow" :number="boomNum" @show="showBoom" @hide="hideBoom" pre="出货量"></fs-boom>
    <div class="logo-wrapper">
      <img src="../assets/tm_logo.png" />
    </div>
    <div class="flex-one left-area">
      <div class="spec-block-one">
        <p class="time">数据更新时间:{{updateTime}}</p>
        <div class="line-one"></div>
        <div class="line-two"></div>
      </div>
      <div class="spec-block-two flex-box">
        <img src="../assets/warehouse_pic.png" />
        <p class="desc">各发货仓实时进度</p>
        <div class="line-one"></div>
        <div class="line-two"></div>
      </div>
      <div class="main-block">
        <div class="list-block one">
          <p class="title">发货英雄榜</p>
          <div class="line"></div>
          <ul class="list">
            <li v-for="item,index in sendOrder" class="item flex-box" :class="{good: index <= 2}">
              <p class="order">{{index + 1}}</p>
              <p class="flex-one">{{item.name}}</p>
              <p class="flex-one">{{item.num}}</p>
            </li>
          </ul>
        </div>
        <div class="list-block two">
          <p class="title">找货英雄榜</p>
          <div class="line"></div>
          <ul class="list">
            <li v-for="item,index in findOrder" class="item flex-box" :class="{good: index <= 2}">
              <p class="order">{{index + 1}}</p>
              <p class="flex-one">{{item.name}}</p>
              <p class="flex-one">{{item.num}}</p>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div class="center-area">
      <div class="number-block">
        <vue-countup class="number" :start="saleData.start" :end="saleData.end"></vue-countup>
        <div class="line"></div>
        <p class="desc">今日出货总量</p>
      </div>
      <div class="all-show-block flex-box">
        <div class="each-block flex-one" v-for="item in warehouseData">
          <p class="t-number">{{item.target}}</p>
          <fs-fill :target="item.target" :now="item.now" :height="560"></fs-fill>
          <div class="warehouse-name">
            <span>{{item.name}}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="flex-one right-area">
      <div class="spec-block flex-box">
        <img src="../assets/warehouse_logo_new.png" />
        <p class="title">天马云仓实时发货数据</p>
        <div class="line-one"></div>
        <div class="line-two"></div>
      </div>
      <div class="main-block">
        <div class="table-line-block each-block">
          <p class="title">完成发货速率(1小时)</p>
          <div class="line"></div>
          <div class="line-table">
            <vue-echarts :options="polar" :auto-resize="true"></vue-echarts>
          </div>
        </div>
        <div class="piv-order-block each-block">
          <p class="title">顾客发货地域分布(省份)</p>
          <div class="line"></div>
          <div class="area-block">
            <div class="each-area" v-for="item,index in provinceData" v-if="index <= 5">
              <div class="desc flex-box">
                <span class="name">{{item.name}}</span>
                <span class="num">{{item.percent + '%'}}</span>
              </div>
              <fs-lineprogress :percent="index === 0 ? 100 : item.percent/provinceData[0].percent*100"></fs-lineprogress>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import jsonp from '@/utils/jsonp'
  import * as moment from 'moment'
  import VueEcharts from 'vue-echarts'
  import VueCountup from '@/comment/vue-countup'
  import FsFill from '@/comment/fs-fill'
  import FsLineprogress from '@/comment/fs-lineprogress'
  import FsBoom from '@/comment/fs-boom'

export default {
  name: 'warehouse',
  data () {
    return {
      boomNum: '',
      boomShow: false,
      showBigAni: globalData.warehouseBigAni,
      updateTime: '0000-00-00 00:00:00',
      saleData: {
        start: 0,
        end: 0
      },
      provinceData: [],
      warehouseData: [],
      sendOrder: [],
      findOrder: [],
      polar: {
        tooltip: {
          trigger: 'axis'
        },
        grid: {
          left: '0',
          top: '6',
          width: '400px',
          height: '230px',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          data: ['07', '08', '09', '10', '11', '12', '13'],
          boundaryGap: false,
          axisLine: {
            lineStyle: {
              color: '#4e7eff'
            }
          },
          splitLine: {
            lineStyle: {
              color: 'rgba(78, 126, 255, 0.5)'
            }
          },
          nameTextStyle: {
            color: 'rgb(255,255,255)'
          }
        },
        yAxis: {
          type: 'value',
          nameTextStyle: {
            color: 'rgb(255,255,255)'
          },
          axisLine: {
            show: false,
            lineStyle: {
              color: '#4e7eff'
            }
          },
          splitLine: {
            lineStyle: {
              color: 'rgba(78, 126, 255, 0.5)'
            }
          }
        },
        series: [
          {
            name: '完成发货速率',
            type: 'line',
            stack: '总量',
            lineStyle: {
              normal: {
                color: '#ffd702'
              }
            },
            data: []
          }
        ]
      }
    }
  },
  created() {
    this.getWarehouseData()
    setInterval(() => {
      this.getWarehouseData()
    }, globalData.duration)
  },
 mounted() {

 },
  methods: {
    getWarehouseData() {
      jsonp(globalData.api, {}, {param: 'jsonpCallback', prefix: 'jp'}).then((res) => {
        if (res.success) {
          this.updateTime = res.results.updateTime
          this.saleData.end = res.results.saledData
          this.isShowBigAni(this.saleData.end)
          this.provinceData = res.results.provinceData
          this.warehouseData = res.results.warehouseData
          this.sendOrder = res.results.sendOrder
          this.findOrder = res.results.findOrder
          let newArr = []
          let newArr2 = []
          res.results.radioData.forEach((item) => {
            newArr.push(+item.num)
            newArr2.push(item.hour)
          })
          this.polar.series[0].data = newArr
          this.polar.xAxis.data = newArr2
        }
      })
    },
    showBoom() {
      this.boomShow = true
    },
    hideBoom() {
      this.boomShow = false
    },
    isShowBigAni(now) {
        this.showBigAni.forEach((item, index) => {
          let flag = now >= item.val && (this.showBigAni[index + 1] ? now <= this.showBigAni[index + 1].val : true) && item.flag
          if (flag) {
            let textNum = this.toTrand(item.val)
            this.boomNum = textNum
            item.flag = false
          }
        })
      },
    toTrand(num) {
        return (num || 0).toString().replace(/(\d)(?=(?:\d{3})+$)/g, '$1,')
    }
  },
  components: {
    VueEcharts,
    VueCountup,
    FsFill,
    FsLineprogress,
    FsBoom
  }
}
</script>

<style  lang="less">

.warehouse {
  padding-top: 127px;
  width: 100%;
  height: 100%;
  background-image: url(../assets/bg-img.jpg);
  .left-area {
    .spec-block-one {
      margin-bottom: 65px;
      .line-one {
        position: absolute;
        top: 36px;
        left: 96px;
        width: 330px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::before {
          content: '';
          display: inline-block;
          position: absolute;
          left:0;
          top: 0px;
          width: 114px;
          height: 1px;
          background-color: #ff4e00
        }
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          top: -2px;
          right: 0;
          width: 5px;
          height: 5px;
          background-color: #ff4e00
        }
      }
      .line-two {
        position: absolute;
        top: 52px;
        left: 0;
        width: 75px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          right: -24px;
          top: -8px;
          width: 26px;
          height: 1px;
          background-color: rgba(78, 126, 255, 0.80);
          transform: rotateZ(141deg);
        }
      }
    }
    .spec-block-two {
      align-items: center;
      padding-left: 100px;
      margin-bottom: 80px;
      .desc {
        margin-left: 6px;
        position: relative;
        top: -15px;
      }
      img {
        width: 170px;
      }
      .line-one {
        position: absolute;
        top: 64px;
        left: 270px;
        width: 150px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::before{
          content: '';
          display: inline-block;
          position: absolute;
          top: -2px;
          left: 0;
          width: 5px;
          height: 5px;
          background-color: #ff4e00
        }
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          right: -109px;
          top: 57px;
          width: 140px;
          height: 1px;
          background-color: rgba(78, 126, 255, 0.8);
          transform: rotateZ(55deg);
        }
      }
      .line-two {
        position: absolute;
        top: 179px;
        left: 500px;
        width: 100px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          top: -2px;
          width: 5px;
          height: 5px;
          background-color: #ff4e00
        }
        &::after {
          right: 0;
        }

      }
    }
    .spec-block-one,.spec-block-two {
      position: relative;
    }
    .time {
      padding-left: 100px;
    }
    .time, .desc {
      font-size: 20px;
      color: #fff;
      font-weight: 700;
    }
    .main-block {
      position: relative;
      .list-block {
        position: absolute;
        padding: 16px;
        width: 200px;
        height: 464px;
        background-image: url('../assets/find-order.png');
        background-size: contain;
        background-repeat: no-repeat;
        &.one {
          left: 100px;
        }
        &.two {
          left: 320px;
          top:50px;;
        }
        .title {
          font-size: 20px;
          font-weight: 700;
          color: #ff4e00;
          margin-bottom: 10px;;

        }
        .list {
          margin-top: 20px;
          color: #fff;
          .item {
            padding: 6px 0;
            align-items: center;
            border-bottom: 1px solid #ff4e00;
            font-size: 16px;
            &.good {
              border-bottom: 1px solid #ffd702;
              .order {
                color: #ffd702;
              }
            }
            .order {
              width: 30px;
              flex: 0 0 30px;
              color: #ff4e00;
            }

          }
        }
        .line {
          position: relative;
          width: 100%;
          height: 1px;
          background-color: rgba(78, 126, 255, 0.80);
          &::before,&::after {
            content: '';
            display: inline-block;
            position: absolute;
            top: -2px;
            width: 5px;
            height: 5px;
            background-color: #ff4e00
          }
          &::before {
            left: 0;
          }
          &::after {
            right: 0;
          }
        }
      }
    }
  }
  .center-area {
    width: 700px;
    flex: 0 0 700px;
    text-align: center;
    .number-block {
      margin: 40px auto 0 auto;
      width: 85%;
      .number {
        color: #ff4e00;
        font-size: 90px;
        font-family: initial;
        font-weight: 700;
        letter-spacing: 4px;
      }
      .line {
        margin: 4px 0;
        position: relative;
        width: 100%;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::before,&::after {
          content: '';
          display: inline-block;
          position: absolute;
          top: -2px;
          width: 5px;
          height: 5px;
          background-color: #ff4e00
        }
        &::before {
          left: 0;
        }
        &::after {
          right: 0;
        }
      }
      .desc {
        color: #ff4e00;
        font-size: 24px;
        font-weight: 700;
      }
    }
    .all-show-block {
      margin-top: 50px;
      text-align: center;
      align-items: flex-end;
      .each-block {

        .t-number {
          margin-bottom: 10px;
          font-size: 24px;
          font-weight: 700;
          color: #01ff8a;
        }
        .warehouse-name {
          margin: 10px auto 0 auto;
          width: 120px;
          height: 34px;
          line-height: 34px;
          font-size: 20px;
          font-weight: 700;
          color: rgba(255, 78, 0, 0.9);
          background-image: url('../assets/warehouse_text_bg.png');
          background-size: cover;
        }
      }
    }
  }
  .right-area {
    .spec-block {
      position: relative;
      top: -20px;
      margin-left: 70px;
      margin-bottom: 30px;
      align-items: baseline;
      font-size: 24px;
      font-weight: 700;
      color: #fff;
      img {
        width: 120px;
        position: relative;
        right: -290px;
        top: 40px;
      }
      .title {
        position: relative;
        left: -128px;
        margin-left: 10px;
      }
      .line-one {
        position: absolute;
        top: 135px;
        left: -13px;
        width: 280px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.8);
        &::before {
          content: '';
          display: inline-block;
          position: absolute;
          left:0;
          top: 0px;
          width: 114px;
          height: 1px;
          background-color: #ff4e00
        }
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          top: -2px;
          left: 0;
          width: 5px;
          height: 5px;
          background-color: #ff4e00
        }
      }
      .line-two {
        position: absolute;
        top: 155px;
        left: 290px;
        width: 400px;
        height: 1px;
        background-color: rgba(78, 126, 255, 0.80);
        &::after {
          content: '';
          display: inline-block;
          position: absolute;
          left: -26px;
          top: -10px;
          width: 30px;
          height: 1px;
          background-color: rgba(78, 126, 255, 0.8);
          transform: rotateZ(40deg);
        }
      }
    }
    .main-block {
      margin-left: 60px;
      .each-block {
        padding: 16px;
        width: 440px;
        height: 318px;
        background-image: url('../assets/right-two.png');
        background-size: contain;
        background-repeat: no-repeat;
        .title {
          font-size: 20px;
          font-weight: 700;
          color: #fff;
          margin-bottom: 10px;

        }
        .line {
          position: relative;
          width: 200px;
          height: 1px;
          background-color: rgba(78, 126, 255, 0.80);
          &::before,&::after {
            content: '';
            display: inline-block;
            position: absolute;
            top: -2px;
            width: 5px;
            height: 5px;
            background-color: #ff4e00;
          }
          &::before {
            left: 0;
          }
          &::after {
            right: 0;
          }
        }
      }
      .table-line-block {
        margin-bottom: 30px;
        .line-table {
          margin-top: 20px;
          width: 100%;
          height:250px;
        }

      }
      .piv-order-block {
        .line {
          width: 230px;
        }
        .area-block {
          margin-top: 12px;
          .each-area {
            padding: 10px 0 0 0;
          }
          .desc {
            margin-bottom: 6px;
            justify-content: space-between;
            color: #fff;
            .name {

            }
            .num {

            }
          }
        }
      }
    }

  }
}

</style>
