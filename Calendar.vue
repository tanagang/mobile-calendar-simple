<template>
    <transition :name="transition">
        <div class="calendar-tz" v-if="isShow" :class="isFixed&&'fixed'">
            <slot name="header"></slot>
            <div class="week-number">
                <span v-for="(item,index) in weekList" :style="{color:(index==0||index==weekList.length-1)&&themeColor}" :key="index">{{item}}</span>
            </div>
            <p class="tips" v-if="title">{{title}}</p>
            <div class="content" id="scrollWrap">
                <div class="con" v-for="(item,index) in calendar" :key="index"  :id="item.year+''+item.month">
                    <h3 v-text="item.year + '年' + item.month + '月'"></h3>
                    <span class="month-bg"  :style="{color:getBetweenColor}">{{item.month}}</span>
                    <ul class="each-month">
                        <li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" :class="[addClassBg(day, item.month, item.year)]" :style="{background:themeOpacityBg(day, item.month, item.year)}" @click="chooseDate(day, item.month, item.year)">
                            <div :class="[addClassName(day, item.month, item.year)]" :style="{background:themeBg(day, item.month, item.year)}">
                                <p class="day-tip" :style="{color:themeColor}"><i v-text="setTip(day, item.month, item.year,1)"></i></p>
                                <p class="day">{{day?day:''}}</p>
                                <p class="recent"><i v-text="setTip(day, item.month, item.year,2)"></i></p>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
            <slot name="footer"></slot>
        </div>
    </transition>
</template>

<script>
export default {
  props: {
    isShow: {//是否显示
        type: [Boolean],
        default() {
            return false;
        }
    },
    isFixed: {//是否定位全屏
        type: [Boolean],
        default() {
            return true;
        }
    },
    transition: {//动画类型slide
        type: [String],
        default() {
            return "";
        }
    },
    title: {//头部的一段文本
        type: [String, Object],
        default() {
            return "";
        }
    },
    mode: {//模式：1普通日历，2酒店，3飞机往返
        type: [String, Number],
        default() {
            return 1;
        }
    },
    startDate: {//开始日期
        type: [String, Object, Date]
    },
    endDate: {//结束日期
        type: [String, Object, Date]
    },
    betweenStart: {//日历可选范围开始
        type: [String, Object, Date],
        default() {
            return "";
        }
    },
    betweenEnd: { //日历可选结束日期
        type: [String, Object, Date],
        default() {
            return "";
        }
    },
    initMonth: {//初始化的月数
        type: [String, Number],
        default() {
            return 6;
        }
    },
    themeColor: {//主题色
        type: [String],
        default: "#415ffb"
    }
  },
  data() {
    return {
        startDates: "",
        endDates: "",
        betweenStarts: "",
        betweenEnds: "",
        calendar: [],
        weekList: ["日", "一", "二", "三", "四", "五", "六"]
    };
  },
  watch: {
    isShow() {
        this.init();
    },
    betweenStart() {
        this.init();
    },
    betweenEnd() {
        this.init();
    }
  },
  mounted() {
    this.init();
  },
  computed: {
    //设置主题色入住离开之间的背景色
    getBetweenColor() {
        if (!this.themeColor) return;
        var hex = this.themeColor;
        if (hex.length == 4) {
            hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`;
        }
        var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" + hex.slice(5, 7)) + ",0.1)";
        return str;
    }
  },
  methods: {
    init() {
        var date = new Date();
        this.year = date.getFullYear();
        this.month = date.getMonth() + 1;
        this.day = date.getDate();
        this.today = new Date(this.year + "/" + this.month + "/" + this.day) * 1;
        if (!this.startDate) {
            const year = date.getFullYear(),
                month = date.getMonth() + 1,
                day = date.getDate();
            this.startDates = this.resetTime(year + "/" + month + "/" + day);
            this.startYear = year;
            this.startMonth = month;
        } else {
            this.startDates = this.resetTime(this.startDate);
            var dd = this.startDate.replace(/-/g, "/").split("/");
            this.startYear = dd[0];
            this.startMonth = dd[1];
        }
        if (!this.endDate) {
            // var temp = this.startDates + 24 * 60 * 60 * 1000;
            // var dateTemp = new Date(temp);
            // const year = dateTemp.getFullYear(),
            //     month = dateTemp.getMonth() + 1,
            //     day = dateTemp.getDate();
            // this.endDates = this.resetTime(year + "/" + month + "/" + day);
            // this.endYear = year;
            // this.endMonth = month;
        } else {
            this.endDates = this.resetTime(this.endDate);
            var dd = this.endDate.replace(/-/g, "/").split("/");
            this.endYear = dd[0];
            this.endMonth = dd[1];
        }
        this.betweenStarts = this.resetTime(this.betweenStart);
        this.betweenEnds = this.resetTime(this.betweenEnd);
        this.createClendar(); //创建日历数据
    },
    //创建每个月日历数据，传入月份1号前面用null填充
    createDayList(month, year) {
        const count = this.getDayNum(month, year),
        _week = new Date(year + "/" + month + "/1").getDay();
        let list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28];
        for (let i = 29; i <= count; i++) {
            list.push(i);
        }
        for (let i = 0; i < _week; i++) {
            list.unshift(null);
        }
        return list;
    },
    //计算传入月份有多少天
    getDayNum(month, year) {
        let dayNum = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
        if ((year % 4 === 0 && year % 100 !== 0) || year % 400 === 0) {
            dayNum[1] = 29;
        }
        return dayNum[month - 1];
    },
    //根据当天和结束日期创建日历数据
    createClendar() {
        var yearTemp = this.year;
        var monthTemp = this.month;
        if (!!this.betweenStarts) {
            //如果有范围起始日期，可选范围从betweenStart开始
            yearTemp = new Date(this.betweenStarts).getFullYear();
            monthTemp = new Date(this.betweenStarts).getMonth() + 1;
        }
        this.calendar = [];
        for (let i = 0; i < this.initMonth; i++) {
            let year = yearTemp;
            let month = monthTemp + i;
            let _monthData = {
                dayList: [],
                month: "",
                year: ""
            };
            var m = Math.ceil(month / 12);
            if (m > 0) {
                year += m - 1;
            } else {
                year += m - 1;
            }
            if (month > 12) {
                month = month % 12 == 0 ? 12 : month % 12;
            }
            if (month <= 0) {
                month = 12 + month % 12;
            }
            _monthData.year = year;
            _monthData.month = month;
            _monthData.dayList = this.createDayList(month, year);
            this.calendar.push(_monthData);
        }
        //h5默认页面加载到当前日期start-date的位置
        if (document) {
            this.scrollTop(this.startYear, this.startMonth);
        }
    },
    scrollTop(year, month) {
        var id = year + "" + parseInt(month)
        setTimeout(() => {
            var obj = document.getElementById(id)
            if(!obj) return
            var wrap = document.getElementById("scrollWrap");
            wrap.scrollTop = obj.offsetTop - 40;
        }, 0);
    },
    //添加日历样式
    addClassName(day, month, year) {
        if (!day) return;
        const _date = new Date(year + "/" + month + "/" + day);
        let className = [];
        // if (_date.getDay() == 0 || _date.getDay() == 6) { //周末或周六样式
        //     className.push('weekend')
        // }
        if (_date * 1 == this.today) {
            className.push("today");
        }
        if (this.mode == 1) {
            if (_date * 1 == this.startDates) {
                className.push("trip-time");
            }
        } else {
            if (_date * 1 == this.startDates || _date * 1 == this.endDates) {
                className.push("trip-time");
            }
        }
        if (this.betweenStarts) {
            _date * 1 < this.betweenStarts && className.push("disabled");
        } else {
            _date * 1 < this.today && className.push("disabled"); //当天和结束日期之外不可选
        }
        _date * 1 > this.betweenEnds && className.push("disabled");
        return className.join(" ");
    },
    //入住离开的区间背景色
    addClassBg(day, month, year) {
        if (!day) return;
        const _date = this.resetTime(year + "/" + month + "/" + day);
        let className = [];
        if (_date >= this.startDates && _date <= this.endDates && this.mode > 1) {
            className.push("between");
        }
        return className.join(" ");
    },
    //theme入住离开的区间背景色
    themeOpacityBg(day, month, year) {
        if (!this.themeColor) return;
        if (!day) return;
        const _date = this.resetTime(year + "/" + month + "/" + day);
        if (_date >= this.startDates && _date <= this.endDates && this.mode > 1) {
            return this.getBetweenColor;
        }
    },
    //theme获取普通日期选中样式背景
    themeBg(day, month, year) {
        if (!this.themeColor) return;
        const _date = this.resetTime(year + "/" + month + "/" + day);
        //正常模式
        if (this.mode == 1) {
            if (_date == this.startDates) {
                return this.themeColor;
            }
        } else {
            //酒店和往返模式
            if (_date == this.startDates || _date == this.endDates) {
                return this.themeColor;
            }
        }
    },
    //清除时间 时 分 秒 毫秒
    resetTime(dateStr) {
        var date = new Date(dateStr.replace(/-/g, "/"));
        date.setHours(0);
        date.setMinutes(0);
        date.setSeconds(0);
        date.setMilliseconds(0);
        return date * 1;
    },
    //flag==1（返回今天，明天，后天)，flag==2（返回入住，离开，去返)
    setTip(day, month, year,flag) {
        if (!day) return
        var tip = ""
        var _date = this.resetTime(year + "/" + month + "/" + day);
        if(flag==1){
            if (_date == this.today) {
                tip = "今天";
            } else if (_date - this.today == 24 * 3600 * 1000) {
                tip = "明天";
            } else if (_date - this.today == 2 * 24 * 3600 * 1000) {
                tip = "后天";
            }
            return tip
        }else{
            if (this.mode == 2) {
                if (_date == this.endDates) {
                    tip = "离开";
                } else if (_date == this.startDates) {
                    tip = "入住";
                }
            } else if (this.mode == 3) {
                if (_date == this.startDates && !this.endDates) {
                    tip = "去/返";
                } else {
                    if (_date == this.endDates) {
                        tip = "返程";
                    } else if (_date == this.startDates) {
                        tip = "去程";
                    }
                }
            }
            return tip;
        }
    },
    //是否是选中当天，或者入住离开当天
    isCurrent(day, month, year) {
      if (!day) {
        return false;
      }
      const _date = this.resetTime(year + "/" + month + "/" + day);
      //正常模式
      if (this.mode == 1) {
        if (_date == this.startDates) {
          return true;
        }
      } else {
        //酒店和往返模式
        if (_date == this.startDates || _date == this.endDates) {
          return true;
        }
      }
    },
    dateFormat(times) {
        let date = new Date(times);
        let recent = "";
        if (times == this.today) {
            recent = "今天";
        } else if (times - this.today === 24 * 3600 * 1000) {
            recent = "明天";
        } else if (times - this.today === 2 * 24 * 3600 * 1000) {
            recent = "后天";
        }
        var year = date.getFullYear()
        var month = parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1)
        var day =  date.getDate() > 9 ? date.getDate() : '0' + date.getDate()
        return {
            dateStr: year + "-" + month + "-" + day,
            week: "周" + this.weekList[date.getDay()],
            recent
        };
    },
    chooseDate(day, month, year) {
        const _date = this.resetTime(year + "/" + month + "/" + day);
        const week = this.weekList[new Date(_date).getDay()];
        //判断日期区域是否可点击
        if (!day) return;
        if (this.betweenStarts) {
            if (_date * 1 < this.betweenStarts) return;
        } else {
            if (_date < this.today) return;
        }
        if (_date > this.betweenEnds) return;
        //判断酒店或者往返模式的选择逻辑
        if (this.startDates && this.endDates && _date > this.endDates) {
            this.startDates = _date;
            this.endDates = "";
        } else if (this.endDates && _date > this.endDates) {
            this.endDates = _date;
        } else if (_date >= this.startDates && _date <= this.endDates) {
            this.startDates = _date;
            this.endDates = "";
        } else if (_date < this.startDates) {
            this.startDates = _date;
            this.endDates = "";
        } else if (_date > this.startDates) {
            if (this.mode == 1) {
                this.startDates = _date;
            } else {
                this.endDates = _date;
            }
        }
        const choose = {
            startStr: this.dateFormat(this.startDates)
        };
        if (this.mode == 1) {
            this.$emit("callback", choose);
        } else if (this.mode == 2 && this.startDates && this.endDates) {
            choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000;
            choose.endStr = this.dateFormat(this.endDates);
            this.$emit("callback", choose);
        } else if (this.mode == 3) {
            if (this.startDates && this.endDates) {
                choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000;
                choose.endStr = this.dateFormat(this.endDates);
            } else if (this.startDates && !this.endDates) {
                choose.dayCount = 0;
                choose.endStr = this.dateFormat(this.startDates);
            }
            this.$emit("callback", choose);
        }
    }
  }
};
</script>

<style lang="less" scoped>
@color: #415ffb;
.calendar-tz {
    width: 100%;
    height: 100vh;
    background: #fff;
    display: -webkit-box;
    display: flex;
    -webkit-flex-direction: column;
    flex-direction: column;
    &.fixed{
        position: fixed;
        width:100%;
        height:100%;
        left:0;
        top:0;
        z-index: 900;
    }
    .week-number {
        background: #fff;
        padding: 0 1%;
        box-shadow: 0 2px 15px rgba(100, 100, 100, 0.1);
        span {
            display: inline-block;
            text-align: center;
            padding: 12px 0;
            font-size: 14px;
            width: 14.28%;
            &:first-child,
            &:last-child {
                color: @color;
            }
        }
    }
    .tips {
        padding: 6px 10px;
        background: #fff7dc;
        font-size: 12px;
        color: #9e8052;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
    .content{
        -webkit-box-flex: 1;
        flex:1;
        overflow-y: scroll;
        -webkit-overflow-scrolling: touch;
         .con {
            color: #333;
            padding-top: 10px;
            position: relative;
            h3 {
                width: 100%;
                font-weight: normal;
                text-align: center;
                font-size: 16px;
                padding: 10px 0;
            }
            .month-bg{
                position: absolute;
                text-align: center;
                opacity: 0.4;
                left:0;
                right:0;
                bottom:0;
                top:20%;
                font-size:220px;
                font-weight: bold;
                color:#f8f8f8;
            }
            .each-month {
                display: block;
                width: 98%;
                font-size: 0;
                margin: 0 auto;
                padding-left: 0;
                padding-bottom: 10px;
                border-bottom: 1px solid #eee;
                .each-day {
                    position: relative;
                    display: inline-block;
                    text-align: center;
                    vertical-align: middle;
                    width: 14.28%;
                    font-size: 16px;
                    height: 50px;
                    margin:2px auto;
                    div {
                        display: inline-block;
                        font-size: 14px;
                        width:98%;
                        height:100%;
                        justify-content: space-around;
                        display: -webkit-box;
                        display: flex;
                        -webkit-flex-direction: column;
                        flex-direction: column;
                        border-radius: 4px;
                    }
                    &.between {
                        background: rgba(75, 217, 173, 0.1);
                    }
                    .day{
                        font-size: 16px;
                    }
                    .day-tip,.recent{
                        font-size:10px;
                        height:14px;
                        i{
                            font-size:10px;
                        }
                    }
                    .recent {
                        color: #ccc;
                    }
                    .disabled {
                        color: #ccc !important;
                        .day-tip{
                            color: #ccc !important;
                        }
                    }
                    .today {
                        background: rgba(100,100,100,0.1);
                    }
                    .trip-time {
                        background: @color;
                        color: #fff !important;
                        .recent,.day-tip{
                            color: #fff!important;
                        }
                    }
                    .weekend {
                        color: @color;
                    }
                }
            }
        }
    }
}
/***右侧进入动画***/
.slide-enter-active,
.slide-leave-active {
  -webkit-transition: all 0.2s ease;
  transition: all 0.2s ease;
}
.slide-enter,
.slide-leave-to {
  -webkit-transform: translateX(100%);
  transform: translateX(100%);
}
</style>