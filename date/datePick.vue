<template lang="html">
  <div>
    <div class="date-pick">
      <div class="date-tit">
        <div class="pre" @click.stop="preMonth"></div>
        <div class="month"><span>{{ year }}</span>年<span>{{ month }}</span>月</div>
        <div class="next" @click.stop="nextMonth"></div>
      </div>
      <ul class="date-week">
        <li class="week-item">日</li>
        <li class="week-item">一</li>
        <li class="week-item">二</li>
        <li class="week-item">三</li>
        <li class="week-item">四</li>
        <li class="week-item">五</li>
        <li class="week-item">六</li>
      </ul>
      <div class="date-num">
        <div
          class="item"
          v-for="(item , index) in tempDateArr"
          :key="index"
          @click.stop="dateClick(item)"
          :class="{
            is_today : item.today ,
            is_selected : item.is_selected && !item.is_disabled && normsDays < 7,
            is_disabled : item.is_disabled,
            is_start : item.timeStamp == selstart && resourceNorms.length == 1 && normsDays >=7 && !item.is_null ,
            is_center : item.timeStamp > selstart && item.timeStamp < selend && normsDays >=7 && !item.is_disabled,
            is_end : item.timeStamp == selend && normsDays >=7
          }"
        >
          <div class="date-box">
            <span class="date-day">{{ item.day }}</span>
          </div>
          <div class="date-price" v-if="item.price">
            <span v-if="item.price>0">¥{{ item.price }}</span>
            <span v-if="item.price=='不可预定'" class="not-order"></span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// import { twoPoint } from 'src/filters/index'
export default {
  data() {
    return {
      tempDateArr: [], // 存放界面显示用的日期数组
      miniBookDateArr: [], // 覆盖最小起订天数的所有日历日期数组
      year: 2018, // 年份
      month: 1, // 月份
      miniYear: 2018, // 年份
      miniMonth: 1, // 月份
      bstop: true, // 开关
      selstart: 0, // 选择的开始日期的时间戳
      selend: 0, // 选择的结束日期的时间戳
      selectedDate: [], // 选中的日期数组
      price: '', // 活动的价格  或者是  周期大于天的价格  每天都为同一个价格
      MondayPrice: '', // 周一价格
      TuesdayPrice: '',
      WednesdayPrice: '',
      ThursdayPrice: '',
      FridayPrice: '',
      SaturdayPrice: '',
      WeekdayPrice: '', // 周天价格
      MonCostPrice: '', // 周一成本价价格
      TuesCostPrice: '',
      WednesCostPrice: '',
      ThursCostPrice: '',
      FriCostPrice: '',
      SaturCostPrice: '',
      WeekCostPrice: '', // 周天成本价价格
      MonOriginalPrice: '', // 周一市场价格
      TuesOriginalPrice: '',
      WednesOriginalPrice: '',
      ThursOriginalPrice: '',
      FriOriginalPrice: '',
      SaturOriginalPrice: '',
      WeekOriginalPrice: '', // 周天市场价格
      notdayPrice: '', // 非天价格
      notdayCostPrice: '', // 非天成本价
      notdayOriginalPrice: '', // 非天市场价

      monId: '', // 规格id
      tueId: '',
      wedId: '',
      thuId: '',
      friId: '',
      satId: '',
      sunId: '',
      notdayId: '', // 规格非天的规格id

      monDe: '', // 周一的押金
      tueDe: '',
      wedDe: '',
      thuDe: '',
      friDe: '',
      satDe: '',
      sunDe: '',
      notdayDeposit: '' // 规格非天每天的押金
    }
  },
  props: {
    resourceNorms: {
      type: Array,
      default: function() {
        return []
      }, // 规格 价格数组
      required: true
    },
    selectDate: {
      type: Array,
      default: function() {
        return []
      }, // 已选规格 价格数组
      required: true
    },
    rate: {
      default: 0, // 补贴率
      type: Number,
      required: true
    },
    normsDays: {
      default: 1, // 时间规格天数 一年365年
      type: Number,
      required: true
    },
    advanceDay: {
      default: 0, // 提前预定天数
      type: Number,
      required: true
    },
    miniBookingDays: {
      type: Number,
      default: 1, // 起订天数
      required: true
    },
    calendars: {
      type: Array,
      default: function() {
        return []
      }, // 不可预定天数组
      required: true
    }
  },
  // filters: { twoPoint },
  created() {
    // 场地每天的价格
    if (this.resourceNorms.length >= 1) {
      if (this.normsDays < 7) {
      // 规格为天 每天的价格
        for (const norm of this.resourceNorms) {
        // mon the  wed  thu  fri  sat  sun
          if (norm.lease_term_type.name === 'Mon') {
            this.MondayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.MonCostPrice = +norm.cost_price / 100
            this.MonOriginalPrice = +norm.original_price / 100
            this.monId = norm.id
            this.monDe = +norm.deposit
          } else if (norm.lease_term_type.name === 'Tue') {
            this.TuesdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.tueId = norm.id
            this.tueDe = +norm.deposit
            this.TuesCostPrice = +norm.cost_price / 100
            this.TuesOriginalPrice = +norm.original_price / 100
          } else if (norm.lease_term_type.name === 'Wed') {
            this.WednesdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.wedId = norm.id
            this.wedDe = +norm.deposit
            this.WednesCostPrice = +norm.cost_price / 100
            this.WednesOriginalPrice = +norm.original_price / 100
          } else if (norm.lease_term_type.name === 'Thu') {
            this.ThursdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.thuId = norm.id
            this.thuDe = +norm.deposit
            this.ThursCostPrice = +norm.cost_price / 100
            this.ThursOriginalPrice = +norm.original_price / 100
          } else if (norm.lease_term_type.name === 'Fri') {
            this.FridayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.friId = norm.id
            this.friDe = +norm.deposit
            this.FriCostPrice = +norm.cost_price / 100
            this.FriOriginalPrice = +norm.original_price / 100
          } else if (norm.lease_term_type.name === 'Sat') {
            this.SaturdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.satId = norm.id
            this.satDe = +norm.deposit
            this.SaturCostPrice = +norm.cost_price / 100
            this.SaturOriginalPrice = +norm.original_price / 100
          } else if (norm.lease_term_type.name === 'Sun') {
            this.WeekdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.sunId = norm.id
            this.sunDe = +norm.deposit
            this.WeekCostPrice = +norm.cost_price / 100
            this.WeekOriginalPrice = +norm.original_price / 100
          } else {
            this.notdayPrice = (norm.price / 100) * (1 - this.rate / 100)
            this.notdayDeposit = +norm.deposit
            this.notdayCostPrice = +norm.cost_price / 100
            this.notdayOriginalPrice = +norm.original_price / 100
            this.notdayId = norm.id
          }
        }
      } else {
      // 规格 非天 的每天价格
        this.notdayPrice =
        (this.resourceNorms[0].price / 100) * (1 - this.rate / 100)
        // 规格 非天 的每天押金
        this.notdayDeposit = +this.resourceNorms[0].deposit

        this.notdayId = this.resourceNorms[0].id
        this.notdayCostPrice = +this.resourceNorms[0].cost_price / 100
        this.notdayOriginalPrice = +this.resourceNorms[0].original_price / 100
      }
    }

  },
  mounted() {
    // 当前限制时间为90天
    this.limitDay = new Date(new Date().setHours(0, 0, 0, 0)).getTime()
    const limit = 91 * 24 * 60 * 60 * 1000
    this.limitDay += limit
    this.year = new Date().getFullYear()
    this.miniYear = new Date().getFullYear()
    this.month = new Date().getMonth() + 1
    this.miniMonth = new Date().getMonth() + 1
    this.initDate()
  },
  methods: {
    initDate(miniBook) {
      if (miniBook !== 'miniBook') {
        this.bstop = true
        this.tempDateArr = [] // 每次清空数组
        this.miniDateArr = [] // 日历翻页，清空起订天数的日期数组
      }
      const today_d = new Date().getDate() // 当日
      const today_m = new Date().getMonth() + 1 // 当月
      const today_y = new Date().getFullYear() // 当年
      var todayTimestamp;
      var firstDay;
      if (miniBook === 'miniBook') {
        todayTimestamp = new Date(today_y, today_m - 1, today_d).getTime() // 当天时间戳
        firstDay = new Date(this.miniYear, this.miniMonth - 1, 1) // 本月第一天
      } else {
        todayTimestamp = new Date(today_y, today_m - 1, today_d).getTime() // 当天时间戳
        firstDay = new Date(this.year, this.month - 1, 1) // 本月第一天
      }
      const weekDay = firstDay.getDay() // 本月第一天星期几
      // 每月一号之前的日期都不显示
      for (let i = 0; i < weekDay; i++) {
        if (miniBook !== 'miniBook') {
          this.tempDateArr.push({
            day: '',
            price: '',
            cost_price: '',
            original_price: '',
            selling_resource_price_id: '',
            is_disabled: true
          })
        }
      }
      const y = firstDay.getFullYear()
      const m = firstDay.getMonth() + 1
      const lastDate = new Date(y, m, 0)
      const ld = lastDate.getDate() // 本月最后一天

      let candestineTimestamp = todayTimestamp
      // 处理提前预定天数 可能为0
      if (this.advanceDay || this.advanceDay === 0) {
        candestineTimestamp =
          todayTimestamp + (this.advanceDay - 1) * 24 * 3600 * 1000
      }
      // 循环拿到每月的日历数组( 包含不可预定的日期处理 ，今天的处理 ， 临时不可预定的处理 )
      for (let j = 1; j <= ld; j++) {
        var tempTimestamp;
        var week;
        if (miniBook === 'miniBook') {
          tempTimestamp = new Date(
            this.miniYear,
            this.miniMonth - 1,
            j
          ).getTime() // 每一天的时间戳
          week = new Date(this.miniYear, this.miniMonth - 1, j).getDay() // 每一天是星期几
        } else {
          tempTimestamp = new Date(this.year, this.month - 1, j).getTime() // 每一天的时间戳
          week = new Date(this.year, this.month - 1, j).getDay() // 每一天是星期几
        }

        if (
          tempTimestamp <= candestineTimestamp ||
          tempTimestamp >= this.limitDay
        ) {
          // 不可预定的日期  今天之前的资源都不可预定（包含有提前预定的资源）
          const temp = {
            day: j,
            price: '',
            cost_price: '',
            original_price: '',
            selling_resource_price_id: '',
            is_selected: false,
            is_disabled: true,
            timeStamp: tempTimestamp
          }
          if (tempTimestamp === todayTimestamp) {
            temp.today = true
            temp.day = '今天'
          }
          if (miniBook === 'miniBook') {
            this.miniBookDateArr.push(temp)
          } else {
            this.tempDateArr.push(temp)
          }
        } else {
          // 可以预定的日期
          // 不为活动的处理
          let temp = null
          if (this.calendars.length) {
            // 存在禁用日期
            for (const cancle of this.calendars) {
              if (cancle.exp_date === this.formatDate(tempTimestamp)) {
                // 禁用日期处理
                temp = {
                  day: j,
                  price: '不可预定',
                  cost_price: '',
                  original_price: '',
                  selling_resource_price_id: '',
                  is_selected: false,
                  is_disabled: true,
                  timeStamp: tempTimestamp
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (miniBook === 'miniBook') {
                  this.miniBookDateArr.push(temp)
                } else {
                  this.tempDateArr.push(temp)
                }
              }
            }
            if (!temp) {
              // 非禁用日期处理
              if (this.resourceNorms.length >= 1 && this.normsDays < 7) {
                // 为天的情况
                console.log('为天的情况')
                if (week === 1) {
                  // 周一
                  temp = {
                    day: j,
                    price: this.MondayPrice,
                    cost_price: this.MonCostPrice,
                    original_price: this.MonOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.monId,
                    deposit: this.monDe || 0
                  }
                  if (!this.MondayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else if (week === 2) {
                  // 周二
                  temp = {
                    day: j,
                    price: this.TuesdayPrice,
                    cost_price: this.TuesCostPrice,
                    original_price: this.TuesOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.tueId,
                    deposit: this.tueDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.TuesdayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else if (week === 3) {
                  // 周三
                  temp = {
                    day: j,
                    price: this.WednesdayPrice,
                    cost_price: this.WednesCostPrice,
                    original_price: this.WednesOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.wedId,
                    deposit: this.webDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.WednesdayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else if (week === 4) {
                  // 周四
                  temp = {
                    day: j,
                    price: this.ThursdayPrice,
                    cost_price: this.ThursCostPrice,
                    original_price: this.ThursOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.thuId,
                    deposit: this.thuDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.ThursdayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else if (week === 5) {
                  // 周五
                  temp = {
                    day: j,
                    price: this.FridayPrice,
                    cost_price: this.FriCostPrice,
                    original_price: this.FriOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.friId,
                    deposit: this.friDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.FridayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else if (week === 6) {
                  // 周六
                  temp = {
                    day: j,
                    price: this.SaturdayPrice,
                    cost_price: this.SaturCostPrice,
                    original_price: this.SaturOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.satId,
                    deposit: this.satDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.SaturdayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                } else {
                  // 周天
                  temp = {
                    day: j,
                    price: this.WeekdayPrice,
                    cost_price: this.WeekCostPrice,
                    original_price: this.WeekOriginalPrice,
                    is_selected: false,
                    timeStamp: tempTimestamp,
                    selling_resource_price_id: this.sunId,
                    deposit: this.sunDe || 0
                  }
                  if (tempTimestamp === todayTimestamp) {
                    temp.today = true
                    temp.day = '今天'
                  }
                  if (!this.WeekdayPrice) {
                    // 价格不存在  不可预定
                    temp.is_disabled = true
                  }
                }
                if (miniBook === 'miniBook') {
                  this.miniBookDateArr.push(temp)
                } else {
                  this.tempDateArr.push(temp)
                }
              } else {
                // 为其他周期的情况
                console.log('为其他周期的情况')
                temp = {
                  day: j,
                  price: this.notdayPrice,
                  cost_price: this.notdayCostPrice,
                  original_price: this.notdayOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  deposit: this.notdayDeposit || 0,
                  selling_resource_price_id: this.notdayId
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (miniBook === 'miniBook') {
                  this.miniBookDateArr.push(temp)
                } else {
                  this.tempDateArr.push(temp)
                }
              }
            }
          } else {
            // 不存在禁用日期
            if (this.resourceNorms.length >= 1 && this.normsDays < 7) {
              // 为天的情况
              if (week === 1) {
                // 周一
                temp = {
                  day: j,
                  price: this.MondayPrice,
                  cost_price: this.MonCostPrice,
                  original_price: this.MonOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.monId,
                  deposit: this.monDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.MondayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else if (week === 2) {
                // 周二
                temp = {
                  day: j,
                  price: this.TuesdayPrice,
                  cost_price: this.TuesCostPrice,
                  original_price: this.TuesOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.tueId,
                  deposit: this.tueDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.TuesdayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else if (week === 3) {
                // 周三
                temp = {
                  day: j,
                  price: this.WednesdayPrice,
                  cost_price: this.WednesCostPrice,
                  original_price: this.WednesOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.wedId,
                  deposit: this.wedDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.WednesdayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else if (week === 4) {
                // 周四
                temp = {
                  day: j,
                  price: this.ThursdayPrice,
                  cost_price: this.ThursCostPrice,
                  original_price: this.ThursOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.thuId,
                  deposit: this.thuDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.ThursdayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else if (week === 5) {
                // 周五
                temp = {
                  day: j,
                  price: this.FridayPrice,
                  cost_price: this.FriCostPrice,
                  original_price: this.FriOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.friId,
                  deposit: this.friDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.FridayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else if (week === 6) {
                // 周六
                temp = {
                  day: j,
                  price: this.SaturdayPrice,
                  cost_price: this.SaturCostPrice,
                  original_price: this.SaturOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.satId,
                  deposit: this.satDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.SaturdayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              } else {
                // 周天
                temp = {
                  day: j,
                  price: this.WeekdayPrice,
                  cost_price: this.WeekCostPrice,
                  original_price: this.WeekOriginalPrice,
                  is_selected: false,
                  timeStamp: tempTimestamp,
                  selling_resource_price_id: this.sunId,
                  deposit: this.sunDe || 0
                }
                if (tempTimestamp === todayTimestamp) {
                  temp.today = true
                  temp.day = '今天'
                }
                if (!this.WeekdayPrice) {
                  // 价格不存在  不可预定
                  temp.is_disabled = true
                }
              }
              if (miniBook === 'miniBook') {
                this.miniBookDateArr.push(temp)
              } else {
                this.tempDateArr.push(temp)
              }
            } else {
              // 为其他周期的情况
              temp = {
                day: j,
                price: this.notdayPrice,
                cost_price: this.notdayCostPrice,
                original_price: this.notdayOriginalPrice,
                is_selected: false,
                timeStamp: tempTimestamp,
                deposit: this.notdayDeposit || 0,
                selling_resource_price_id: this.notdayId
              }
              if (tempTimestamp === todayTimestamp) {
                temp.today = true
                temp.day = '今天'
              }
              if (miniBook === 'miniBook') {
                this.miniBookDateArr.push(temp)
              } else {
                this.tempDateArr.push(temp)
              }
            }
          }
        }
      }
      if (this.selectDate.length && this.bstop) { // this.bstop防止预加载后面日历数据重置selstart和selend
        this.selectedDate = this.selectDate;
        if (this.normsDays >= 7) {
          this.selstart = new Date(new Date(this.selectedDate[0].date).setHours(0, 0, 0, 0)).getTime();
          this.selend = new Date(new Date(this.selectedDate[this.selectedDate.length - 1].date).setHours(0, 0, 0, 0)).getTime();
        }
        // 已选 渲染
        if (this.selectedDate.length) {
          for (const t of this.selectedDate) {
            if (miniBook === 'miniBook') {
              for (const k of this.miniBookDateArr) {
                const date = this.formatDate(k.timeStamp)
                if (date === t.date) {
                  k.is_selected = true;
                  k.cost_price = t.cost_price;
                  k.deposit = t.deposit;
                  k.original_price = t.original_price;
                  k.price = +t.price;
                  k.selling_resource_price_id = t.selling_resource_price_id;
                }
              }
            } else {
              for (const k of this.tempDateArr) {
                const date = this.formatDate(k.timeStamp)
                if (date === t.date) {
                  k.is_selected = true;
                  k.cost_price = t.cost_price;
                  k.deposit = t.deposit;
                  k.original_price = t.original_price;
                  k.price = +t.price;
                  k.selling_resource_price_id = t.selling_resource_price_id;
                }
              }
            }
          }
        }
      }
    },
    nextMonth(miniBook) {
      if (miniBook === 'miniBook') {
        this.miniMonth++
        if (this.miniMonth > 12) {
          this.miniMonth = 1
          this.miniYear++
        }
        this.initDate(miniBook)
      } else {
        this.month++
        if (this.month > 12) {
          this.month = 1
          this.year++
        }
        this.miniMonth = this.month
        this.miniYear = this.year
        this.initDate()
      }
    },
    preMonth() {
      this.month--
      if (this.month < 1) {
        this.month = 12
        this.year--
      }
      this.miniMonth = this.month
      this.miniYear = this.year
      this.initDate()
    },
    formatDate(timestamp) {
      const y = new Date(timestamp).getFullYear()
      let m = new Date(timestamp).getMonth() + 1
      let d = new Date(timestamp).getDate()
      m = m < 10 ? '0' + m : m
      d = d < 10 ? '0' + d : d
      const date = y + '-' + m + '-' + d
      return date
    },
    dateClick(item) {
      // 如果禁止了不做任何处理
      if (item.is_disabled) {
        return
      }
      item.is_selected = !item.is_selected
      // 记录当前点击的日期的时间戳
      if (!item.is_selected) {
        const currentTime = this.formatDate(item.timeStamp)
        for (let i = this.selectedDate.length - 1; i >= 0; i--) {
          if (this.selectedDate[i].date === currentTime) {
            // 删除掉已保存数组中 的 取消日期
            this.selectedDate.splice(i, 1)
          }
        }
      }
      // 记录开始时间
      this.selstart = item.timeStamp
      // 为天的情况
      if (this.resourceNorms.length >= 1 && this.normsDays < 7) {
        if (
          item.is_selected &&
          !this.selectedDate.length &&
          this.miniBookingDays > 1
        ) {
          const startTime = this.selstart
          if (this.bstop) {
            this.bstop = false
            for (let i = 0; i < 6; i++) {
              // 解决起订天数超出当前月的问题,预先加载后面月份的日历数据 最高能选中起订天数为180天
              this.nextMonth('miniBook')
            }
            this.miniBookDateArr = this.tempDateArr.concat(this.miniBookDateArr)
          }
          //  console.log(this.miniBookDateArr)
          for (const k of this.miniBookDateArr) {
            if (k.day) {
              if (
                k.timeStamp >= startTime &&
                this.selectedDate.length < this.miniBookingDays
              ) {
                if (!k.is_disabled) {
                  k.is_selected = true
                  const date = this.formatDate(k.timeStamp)
                  this.selectedDate.push({
                    date: date,
                    price: k.price,
                    cost_price: k.cost_price,
                    original_price: k.original_price,
                    selling_resource_price_id: k.selling_resource_price_id,
                    deposit: k.deposit
                  })
                } else {
                  k.is_selected = false
                }
              } else {
                if (k.timeStamp >= startTime) {
                  break
                }
              }
            }
          }
        } else {
          if (item.is_selected) {
            const date = this.formatDate(item.timeStamp)
            this.selectedDate.push({
              date: date,
              price: item.price,
              cost_price: item.cost_price,
              original_price: item.original_price,
              selling_resource_price_id: item.selling_resource_price_id,
              deposit: item.deposit
            })
          }
        }

        // 去重
        const newArr = []
        if (this.selectedDate.length) {
          for (const item of this.selectedDate) {
            let flag = true
            for (const newitem of newArr) {
              if (item.date === newitem.date) {
                flag = false
              }
            }
            if (flag) {
              newArr.push(item)
            }
          }
        }
        // 重新赋值
        this.selectedDate = newArr
      } else {
        const arr = []
        const normsDaysArr = []
        const endTime = this.selstart + this.normsDays * 24 * 3600 * 1000
        this.selend = endTime - 24 * 3600 * 1000
        const startTime = this.selstart
        if (this.bstop) {
          this.bstop = false
          for (let i = 0; i < 12; i++) {
            // 预加载后面的日历数据
            this.nextMonth('miniBook')
          }
          this.miniBookDateArr = this.tempDateArr.concat(this.miniBookDateArr)
        }
        for (const k of this.miniBookDateArr) {
          if (k.day) {
            if (
              k.timeStamp >= startTime &&
              normsDaysArr.length < this.normsDays
            ) {
              if (!k.is_disabled) {
                k.is_selected = true
                normsDaysArr.push(k)
              } else {
                this.selend += 24 * 3600 * 1000
                k.is_selected = false
              }
            } else {
              if (k.timeStamp >= startTime) {
                break
              }
            }
          }
        }
        // 有开始时间才保存
        if (this.selstart) {
          for (let i = 0; i < this.normsDays; i++) {
            const time = this.selstart + i * 24 * 3600 * 1000
            const date = this.formatDate(time)
            arr.push({
              date: date,
              price: item.price,
              cost_price: item.cost_price,
              original_price: item.original_price,
              selling_resource_price_id: item.selling_resource_price_id,
              deposit: +item.deposit
            })
          }
        }
        this.selectedDate = arr
      }

      console.log(this.selectedDate, '选择的日期数组1')
      // 触发父级的监听事件
      this.$emit('pipeData', this.selectedDate)
    }
  }
}
</script>

<style lang="scss" scoped>
@import '../style/mixin';
.date-pick {
  padding: 0.2rem 0;
  background-color: #fff;
  .date-tit {
    padding: 0 0.25rem;
    height: 1.08rem;
    line-height: 1.08rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    .pre {
      display: inline-block;
      @include wh(0.48rem, 0.48rem);
      @include bg-image('../images/ic_back_black');
    }
    .next {
      display: inline-block;
      @include wh(0.48rem, 0.48rem);
      @include bg-image('../images/ic_next');
    }
    .month {
      font-size: 0.36rem;
      font-weight: bold;
    }
  }
  .date-week {
    height: 0.52rem;
    line-height: 0.52rem;
    display: flex;
    padding: 0 0.24rem;
    background-color: #f6f6fb;
    .week-item {
      width: 1rem;
      text-align: center;
      font-size: 0.24rem;
      color: #999;
    }
  }
  .date-num {
    display: flex;
    justify-content: flex-start;
    flex-wrap: wrap;
    padding: 0 0.24rem;
    max-height: 5rem;
    overflow-y: scroll;
    align-items:center;
    .item {
      @include wh(1rem, 1rem);
      &.is_today {
        .date-box {
          .date-day {
            font-size: 0.24rem;
            color:  #4585DD !important;
            font-weight: normal;
          }
        }
      }
      &.is_selected {
        .date-box {
          .date-day {
            display: inline-block;
            @include wh(0.44rem, 0.44rem);
            border-radius: 50%;
            color: #fff !important;
            background-color: #4585DD;
            overflow: hidden;
          }
        }
      }
      &.is_disabled {
        .date-box {
          .date-day {
            color: #d2d2d2;
          }
        }
      }
      &.is_start {
        .date-box {
          border-radius: 0.22rem 0 0 0.22rem;
          background-color: #4585DD;
          .date-day{
            color:#FFFFFF !important;
          }
        }
      }
      &.is_center {
        .date-box {
          background-color: #4585DD;
          .date-day{
            color:#FFFFFF;
          }
        }
      }
      &.is_end {
        .date-box {
          border-radius: 0 0.22rem 0.22rem 0;
          background-color: #4585DD;
          .date-day{
            color:#FFFFFF;
          }
        }
      }
      .date-box {
        position: relative;
        top: 0;
        left: 0;
        height: 0.44rem;
        line-height: 0.44rem;
        text-align: center;
        .date-day {
          font-size: 0.3rem;
          font-weight: bold;
          color: #666;
        }
      }
      .date-price {
        margin-top: 0.1rem;
        text-align: center;
        font-size: 0.18rem;
        color: #999;
        .not-order {
          color: $mainColor;
        }
      }
    }
  }
}
</style>
