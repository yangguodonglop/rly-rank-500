<template>
  <div id="uhome">
    <el-container>
      <el-header class="header">
        <headers></headers>
      </el-header>

      <div class="search-header">
        <div class="search-button">
          <!--          <el-input size="small" v-model="input" placeholder="请输入内容" class="rank-input"></el-input>-->
          <!--          <el-button style="margin-left: 10px;" type="success" size="small">查询</el-button>-->
          <el-select @change="selectChange" v-model="value" placeholder="请选择查询条件">
            <el-option
              v-for="item in options"
              :key="item.value"

              :label="item.label"
              :value="item.value">
            </el-option>
          </el-select>
          <el-date-picker
            style="margin-left: 20px;"
            v-model="value1"
            type="daterange"
            range-separator="至"
            start-placeholder="开始日期"
            @change="dateChange"
            end-placeholder="结束日期">
          </el-date-picker>
          <el-button style="margin-left: 10px;" @click="toSort()" type="danger" size="small">{{ sortText }}</el-button>
          <el-button style="margin-left: 10px;" @click="queryInfo()" type="success" size="small">查询</el-button>
          <el-button style="margin-left: 10px;" @click="resetInfo()" type="primary" size="small">重置</el-button>
        </div>
      </div>
      <div class="song-rank">
        <div class="left">
          <div class="rank-nav">
            <img src="@/assets/rank.jpg" width="40" height="40" alt="">
            <div class="rank-text">
              <div>飙升榜</div>
              <div>刚刚更新</div>
            </div>
          </div>
        </div>
        <div class="right">
          <div class="right-top">
            <img src="@/assets/rank.jpg" width="100" height="100" alt="">
            <div class="rank-text" style="padding-left: 10px;color: #999999">
              <div>飙升榜</div>
              <div>最近更新：{{ nowDate }}（刚刚更新）</div>
              <!--              <div>评论数：9999999</div>-->
            </div>
          </div>
          <div class="rank-ul">歌曲列表</div>
          <div class="right-bottom">
            <el-table
              :data="tableData"
              style="width: 100%">
              <el-table-column
                prop="SongData.name"
                label="歌曲名"
                width="280">
              </el-table-column>
              <el-table-column
                prop="SongData.firstRecTimes"
                label="首次上榜时间"
                width="180">
              </el-table-column>
              <el-table-column
                label="当日排名">
                <template slot-scope="scope">
                  <div style="color: #218dfa;">{{ scope.row.SongData.rank }}名</div>
                </template>
              </el-table-column>
              <el-table-column
                width="150"
                label="较昨日排名比较">
                <template slot-scope="scope">
                  <div style="color: green" v-if="scope.row.Fluctuate<0">上升{{ scope.row.Fluctuate * -1 }}名</div>
                  <div style="color: red" v-else>下降{{ scope.row.Fluctuate }}名</div>

                </template>
              </el-table-column>
              <el-table-column label="操作" fixed="right">
                <template slot-scope="scope">
                  <div class="optionDiv" style="white-space: nowrap; display: inline-block">
                    <el-button type="text" size="small" @click="checkDetail(scope.row)"> 查看</el-button>
                    <el-button style="margin-left: 20px;" type="text" size="small" @click="toListen(scope.row)"> 试听
                    </el-button>
                  </div>
                </template>
              </el-table-column>
            </el-table>
            <div style="float: right;">
              <el-pagination
                layout="prev, pager, next"
                :current-page="this.curPage"
                @current-change="currentChange"
                :total="songTotal">
              </el-pagination>
            </div>
          </div>
        </div>
      </div>
      <el-dialog
        title="歌曲详情"
        :visible.sync="dialogVisible"
        width="50%"
        :before-close="handleClose">
        <div style="line-height: 35px;color: #f53b11;font-size: 16px;text-align: center;">歌曲名:{{ songName }}</div>
        <div ref="chart" style="width:100%;height: 400px;"></div>
        <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
          <!--    <el-button type="primary" @click="dialogVisible = false">确 定</el-button>-->
  </span>
      </el-dialog>
    </el-container>
  </div>
</template>

<script>
import Headers from "./header/Headers.vue";
import Footers from "./footer/Footers.vue";
import Find from "views/uhome/container/find/Find.vue";
import {rlyTop500, rlyTop500Detail} from 'network/home'
import * as echarts from 'echarts'
import fmtDate from "common/js/Date.js";

export default {
  name: "rank",
  components: {Find, Headers, Footers},
  data() {
    return {
      input: '',
      curPage: 0,
      curPageActive: 0,
      songTotal: 0,
      dialogVisible: false,
      value1: '',
      nowDate: 0,
      startDate: 0,
      endDate: 0,
      songName: '',
      options: [{
        value: '1',
        label: '当日'
      }, {
        value: '2',
        label: '5日'
      }, {
        value: '3',
        label: '10日'
      }, {
        value: '4',
        label: '20日'
      }],
      sortType: true,
      sortText: '倒序',

      value: '',
      tableData: []
    }
  },
  mounted() {
    const date = new Date()
    const beforeDate = new Date(date.getTime() - 24 * 60 * 60 * 1000)
    this.nowDate = this.dateFmt1(date)
    this.startDate = this.dateFmt(beforeDate)
    this.endDate = this.dateFmt(date)
    this.queryInfo()

  },
  methods: {
    toSort() {
      this.sortType = !this.sortType
      this.sortType ? this.sortText = '倒序' : this.sortText = '正序'
      this.curPage = 0
      this.queryInfo()

    },
    //试听
    toListen(val) {
      const url = val['SongData']['playURL']
      window.open(url, '_blank')

    },
    //充值
    resetInfo() {
      const date = new Date()
      const beforeDate = new Date(date.getTime() - 24 * 60 * 60 * 1000)
      this.startDate = this.dateFmt(beforeDate)
      this.endDate = this.dateFmt(date)
      this.curPage = 0
      this.curPageActive = 0
      this.value = ''
      this.value1 = ''
      this.sortType = true
      this.queryInfo()
    },
    //时间转化
    dateFmt(date) {
      let d = new Date(date);
      return fmtDate(d, "yyyyMMdd");
    },
    dateFmt2(date) {
      let d = new Date(date);
      return fmtDate(d, "yyyy-MM-dd");
    },
    dateFmt1(date) {
      let d = new Date(date);
      return fmtDate(d, "yyyy年MM月dd日");
    },
    queryInfo() {
      const param =
        {
          "startDate": parseInt(this.startDate),
          "endDate": parseInt(this.endDate),
          'sort': this.sortType ? 0 : 1,
          "curPage": this.curPageActive,
          "pageSize": 10
        }

      rlyTop500(param).then((res) => {
        if (res.data) {
          this.songTotal = res.data.count
          this.tableData = []
          this.tableData = res.data.sort
          this.tableData.forEach((item) => {
            console.log(new Date(item['SongData']['firstRecTime'] * 1000))
            const tempTime = (item['SongData']['firstRecTime'] * 1000)
            item['SongData']['firstRecTimes'] = item['SongData']['firstRecTime'] ? this.dateFmt2(tempTime) : '暂无上榜时间'
            item.SongData.rank = item.SongData.rank + 1
            // item.Fluctuate = (item.Fluctuate) * -1
          })
        } else {
          this.tableData = []
          this.songTotal = 0
        }

      })
    },
    handleClose(done) {
      this.dialogVisible = false
      // this.$confirm('确认关闭？')
      //   .then(_ => {
      //     done();
      //   })
      //   .catch(_ => {
      //   });
    },
    checkDetail(val) {
      this.songName = val['SongData']['name']

      this.dialogVisible = true
      // this.initCharts()
      // setTimeout(this.initCharts)
      let newPromise = new Promise((resolve) => {
        resolve()
      })
      newPromise.then(() => {
        this.initCharts(val)
      })

    },
    currentChange(val) {
      this.curPage = val
      this.curPageActive = val - 1
      this.queryInfo()

    },
    selectChange(val) {
      const date = new Date()
      this.endDate = this.dateFmt(date)

      switch (val) {
        case '1':
          const beforeDate1 = new Date(date.getTime() - 24 * 60 * 60 * 1000)
          this.startDate = this.dateFmt(beforeDate1)
          break;
        case '2':
          const beforeDate5 = new Date(date.getTime() - 5 * 24 * 60 * 60 * 1000)
          this.startDate = this.dateFmt(beforeDate5)
          break;
        case '3':
          const beforeDate10 = new Date(date.getTime() - 10 * 24 * 60 * 60 * 1000)
          this.startDate = this.dateFmt(beforeDate10)
          break;
        case '4':
          const beforeDate20 = new Date(date.getTime() - 20 * 24 * 60 * 60 * 1000)
          this.startDate = this.dateFmt(beforeDate20)
          break;
      }
    },
    dateChange(val) {
      this.startDate = this.dateFmt(
        val[0],
      )
      this.endDate = this.dateFmt(
        val[1],
      )

    },
    initCharts(val) {
      const param = {
        "songName": val['SongData']['name'],
        "singer": val['SongData']['singer']
      }
      rlyTop500Detail(param).then((res) => {
        const xArr = []
        const yArr = []
        if (res['data']) {
          const tempArray = res['data']['date']
          tempArray.forEach((item) => {
            xArr.push(item['date_'])
            yArr.push(item['rank'] + 1)
          })
        }
        let myChart = this.$echarts.init(this.$refs.chart);
        // 绘制图表
        myChart.setOption(
          {
            backgroundColor: "#080b30",
            title: {
              text: "排名变化表",
              textStyle: {
                align: "center",
                color: "#fff",
                fontSize: 16,
              },
              top: "2%",
              left: "center",
            },
            tooltip: {

              show: true,
              trigger: "axis",
              showContent: true,
              axisPointer: {
                lineStyle: {
                  color: {
                    type: "linear",
                    x: 0,
                    y: 0,
                    x2: 0,
                    y2: 1,
                    colorStops: [
                      {
                        offset: 0,
                        color: "rgba(0, 255, 233,0)",
                      },
                      {
                        offset: 0.5,
                        color: "rgba(255, 255, 255,1)",
                      },
                      {
                        offset: 1,
                        color: "rgba(0, 255, 233,0)",
                      },
                    ],
                    global: false,
                  },
                },
              },
            },
            toolbox: {
              feature: {
                dataZoom: {
                  yAxisIndex: 'none'
                },
                restore: {},
                saveAsImage: {}
              }
            },
            grid: {
              top: "15%",
              left: "5%",
              right: "5%",
              bottom: "15%",
              // containLabel: true
            },
            xAxis: [
              {
                type: "category",
                axisLine: {
                  show: true,
                },
                splitArea: {
                  // show: true,
                  color: "#f00",
                  lineStyle: {
                    color: "#f00",
                  },
                },
                axisLabel: {
                  color: "#fff",
                },
                splitLine: {
                  show: false,
                },
                boundaryGap: false,
                data: xArr
                // data: ["03-01", "03-02", "03-03", "03-04", "03-05", "03-06", "03-07", "03-08", "03-09", "03-10", "03-11", "03-12", "03-13", "03-14", "03-15", "03-16", "03-17", "03-18", "03-19", "03-20"],
              },
            ],
            dataZoom: [
              {
                type: 'inside',
                start: 0,
                end: 20
              },
              {
                start: 0,
                end: 20
              },
            ],

            yAxis: [
              {
                type: "value",
                min: 0,
                // max: 140,
                splitNumber: 4,
                splitLine: {
                  show: true,
                  lineStyle: {
                    color: "rgba(255,255,255,0.1)",
                  },
                },
                axisLine: {
                  show: false,
                },
                axisLabel: {
                  show: false,
                  margin: 20,
                  textStyle: {
                    color: "#d1e6eb",
                  },
                },
                axisTick: {
                  show: false,
                },
              },
            ],
            series: [
              {
                name: "当天排名",
                type: "line",
                // smooth: true, //是否平滑
                showAllSymbol: true,
                // symbol: 'image://./static/images/guang-circle.png',
                symbol: "circle",
                symbolSize: 5,
                lineStyle: {
                  normal: {
                    color: "#00ca95",
                    shadowColor: "rgba(0, 0, 0, .3)",
                    shadowBlur: 0,
                    shadowOffsetY: 2,
                    shadowOffsetX: 2,
                  },
                },
                label: {
                  show: true,
                  position: "top",
                  textStyle: {
                    color: "#00ca95",
                  },
                },

                itemStyle: {
                  color: "#00ca95",
                  borderColor: "#fff",
                  borderWidth: 3,
                  shadowColor: "rgba(0, 0, 0, .3)",
                  shadowBlur: 0,
                  shadowOffsetY: 2,
                  shadowOffsetX: 2,
                },
                areaStyle: {
                  normal: {
                    color: new echarts.graphic.LinearGradient(
                      0,
                      0,
                      0,
                      1,
                      [
                        {
                          offset: 0,
                          color: "rgba(0,202,149,0.3)",
                        },
                        {
                          offset: 1,
                          color: "rgba(0,202,149,0)",
                        },
                      ],
                      false
                    ),
                    shadowColor: "rgba(0,202,149, 0.9)",
                    shadowBlur: 20,
                  },
                },
                // data: [1, 2, 3, 4, 5, 6, 57, 8, 9, 10, 60, 12, 13, 14, 15, 16, 17, 18, 19, 20],
                data: yArr
              },
            ],
          }
        );

      })

    }
  }
};
</script>

<style scoped>
.header {
  background-color: #242424;
  border-bottom: 1px solid #000;
}

.search-header {
  width: 100%;
  height: 60px;
  background: #999999;
  background-color: #C20C0C;
  border-bottom: 1px solid #a40011;
  display: flex;
  align-items: center;
}

.search-button {
  width: 1200px;
  margin: 0 auto;
}

.song-rank {
  width: 1200px;
  height: 805px;
  margin: 0 auto;
  display: flex;
}

.left {
  width: 240px;
  height: 805px;
  border: 1px solid #ccc;
  border-top: none;
}

.right {
  margin-left: 40px;
  height: 805px;
  width: 800px;
  border: 1px solid #ccc;
  border-top: none;
}

.rank-nav {
  width: 240px;
  height: 60px;
  background: #e6e6e6;
  font-size: 12px;
  color: #999999;
  display: flex;
  align-items: center;

}

.right-top {
  display: flex;
  align-items: center;
  height: 150px;


}

img {
  margin-left: 20px;
}

.rank-text {
  margin-left: 10px;
}

.rank-input {
  width: 200px;
  height: 32px;
  border-radius: 50px;

}

.rank-ul {
  height: 40px;
  line-height: 40px;
  border-bottom: 1px solid #ccc;
  padding-left: 20px;
}

/deep/ .el-table .el-table__cell {
  padding: 3px 0px;
}

/deep/ .el-dialog__body {
  padding: 0px 20px;
}

.el-main {
  padding: 0;
}

.el-footer {
  padding: 0;
}
</style>
