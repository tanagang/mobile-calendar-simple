<template>
	<div class="calendar">
		<div class="calendar-header">
			<div class="week-number">
				<span v-for="(item,index) in weekList" :style="{color:(index==0||index==weekList.length-1)&&themeColor}" :key="index">{{item}}</span>
			</div>
            <p class="calendar-title" v-if="title">{{title}}</p>
		</div>
        <p :style="{height:title?'68px':'40px'}"></p>
        <div class="calendar-wrapper" v-for="(item,index) in calendar" :key="index">
            <h3 v-text="item.year + '年' + item.month + '月'"></h3>
            <ul class="each-month">
                <li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" :class="[addClassBg(day, item.month, item.year)]" :style="{background:themeOpacityBg(day, item.month, item.year)}" @click="chooseDate(day, item.month, item.year)">
                    <div :class="[addClassName(day, item.month, item.year),{'trip-time': isCurrent(day, item.month, item.year)}]" :style="{background:themeBg(day, item.month, item.year)}">{{day?day:''}}</div>
                    <span class="recent" :style="{color:themeColor}" v-text="setTip(day, item.month, item.year)"></span>
                </li>
            </ul>
        </div>
        <slot></slot>
	</div>
</template>

<script>
    export default {
        props: {
            title: {
                type: [String, Object],
                default () {
                    return ''
                }
            }, 
            mode: {
                type: [String, Number],
                default () {
                    return 1
                }
            },
            startDate: { //开始日期
                type: [String, Object,Date],
                default () {
                    const year = new Date().getFullYear(),
                        month = new Date().getMonth()+1,
                        day = new Date().getDate()
                    return year+'/'+month+'/'+day
                }
            },
            endDate: { //结束日期
                type: [String, Object, Date],
                default () {
                    var temp = new Date().getTime() + 24 * 60 * 60 * 1000
                    var date = new Date(temp)
                    const year = date.getFullYear(),
                        month = date.getMonth()+1,
                        day = date.getDate()
                    return year+'/'+month+'/'+day
                }
            },
            betweenStart: { //日历可选范围开始
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            betweenEnd: { //日历可选结束日期
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            initMonth:{//初始化的月数
                type:[String,Number],
                default(){
                    return 6
                }
            },
            themeColor: {//主题色
				type: [String],
				default: ''
			}
        },
        data() {
            return {
                startDates:'',
                endDates:'',
                betweenStarts:'',
                betweenEnds:'',
                calendar: [],
                weekList: ['日', '一', '二', '三', '四', '五', '六']
            }
        },
        mounted() {
            this.init()
        },
        computed: {
            //theme
			getBetweenColor() {
                if(!this.themeColor) return
                var hex = this.themeColor
				if (hex.length == 4) {
					hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`
				}
				var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" +
					hex.slice(5, 7)) + ",0.1)";
				return str
			},
		},
        methods: {
            init() {
                this.year = new Date().getFullYear()
                this.month = new Date().getMonth()+1
                this.day = new Date().getDate()
                this.today = new Date(this.year+'/'+this.month+'/'+this.day) * 1

                if(this.mode==1){//普通模式不提供默认值，第一次让用户选择日期
                    this.startDates = ''
                    this.endDates = ''
                }else{//酒店和往返模式提供默认值
                    this.startDates = this.resetTime(this.startDate)
                    this.endDates = this.resetTime(this.endDate)
                }
                
                this.betweenStarts = this.resetTime(this.betweenStart)
                this.betweenEnds = this.resetTime(this.betweenEnd)

                this.createClendar(); //创建日历数据
            },
            //创建每个月日历数据，传入月份1号前面用null填充
            createDayList(month, year) {
                const count = this.getDayNum(month, year),
                    _week = new Date(year + '/' + month + '/1').getDay();
                let list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28]

                for (let i = 29; i <= count; i++) {
                    list.push(i)
                }
                for (let i = 0; i < _week; i++) {
                    list.unshift(null)
                }
                return list;
            },
            //计算传入月份有多少天
            getDayNum(month, year) {
                let dayNum = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

                if ((year % 4 === 0) && (year % 100 !== 0) || (year % 400 === 0)) {
                    dayNum[1] = 29;
                }
                return dayNum[month - 1]
            },
            //根据当天和结束日期创建日历数据
            createClendar() {
                var yearTemp = this.year
                var monthTemp = this.month
                if(!!this.betweenStarts){//如果有范围起始日期，可选范围从betweenStart开始
                    yearTemp = new Date(this.betweenStarts).getFullYear()
                    monthTemp = new Date(this.betweenStarts).getMonth()+1
                }
                for (let i = 0; i < this.initMonth; i++) {
                    let year = yearTemp
                    let month = monthTemp + i
                    let  _monthData = {
                            dayList: [],
                            month: '',
                            year: ''
                        };

                    var m = Math.ceil(month / 12)
					if (m > 0) {
						year += m - 1
					} else {
						year += m - 1
					}
					if (month > 12) {
						month = month % 12 == 0 ? 12 : month % 12;
					}
					if (month <= 0) {
						month = 12 + month % 12
                    }
                    
                    _monthData.year = year;
                    _monthData.month = month;
                    _monthData.dayList = this.createDayList(month, year);
                    this.calendar.push(_monthData)
                }
            },
            //添加日历样式
            addClassName(day, month, year) {
                if (!day) return
                const _date = new Date(year + '/' + month + '/' + day)
                let className = []
                // if (_date.getDay() == 0 || _date.getDay() == 6) { //周末或周六样式
                //     className.push('weekend')
                // }
                if (_date * 1 == this.today) {
                    className.push('today');
                }
                if(this.betweenStarts){
                   (_date * 1 < this.betweenStarts)&&className.push('disabled')
                }else{
                    (_date * 1 < this.today)&&className.push('disabled')  //当天和结束日期之外不可选
                }
                (_date * 1 > this.betweenEnds)&&className.push('disabled')
                return className.join(' ');
            },
            //入住离开的区间背景色
            addClassBg(day, month, year) {
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                let className = [];
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    className.push("between");
                }
                return className.join(" ");
            },
            //theme入住离开的区间背景色
            themeOpacityBg(day, month, year) {
				if(!this.themeColor) return
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    return this.getBetweenColor
                }
            },
			//theme获取普通日期选中样式背景
			themeBg(day, month, year) {
                if(!this.themeColor) return
				const _date = this.resetTime(year + '/' + month + '/' + day)
                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return this.themeColor
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return this.themeColor
                    }
                }
			},
            //清除时间 时 分 秒 毫秒
            resetTime(dateStr) {
                var date = new Date(dateStr.replace(/-/g, '/'))
                date.setHours(0);
                date.setMinutes(0);
                date.setSeconds(0);
                date.setMilliseconds(0);
                return date * 1;
            },
            //设置今天，明天，后天
            setTip(day, month, year) {
                if (!day) {
                    return;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)
                let tip = "";

                if (_date == this.today) {
                    tip = '今天'
                } else if (_date - this.today == 24 * 3600 * 1000) {
                    tip = '明天'
                } else if (_date  - this.today == 2 * 24 * 3600 * 1000) {
                    tip = '后天'
                }
                if(this.mode == 2){
                    if (_date == this.endDates) {
                        tip = "离开";
                    } else if (_date == this.startDates) {
                        tip = "入住";
                    }
                }else if (this.mode == 3) {
                    if (_date == this.startDates && !this.endDates) {
                        tip = '去/返'
                    } else {
                        if (_date == this.endDates) {
                            tip = '返程'
                        } else if (_date == this.startDates) {
                            tip = '去程'
                        }
                    }
                }
                return tip;
            },
            isCurrent(day, month, year) {
                if (!day) {
                    return false;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)

                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return true
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return true
                    }
                }
                
            },
            dateFormat(times) {
                let date = new Date(times);
                let recent = ''  
                if (times == this.today) {
                    recent = '今天'
                } else if (times - this.today === 24 * 3600 * 1000) {
                    recent = '明天'
                } else if (times - this.today === 2 * 24 * 3600 * 1000) {
                    recent = '后天'
                }
               
                var year = date.getFullYear()
				var month = parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1)
				var day =  date.getDate() > 9 ? date.getDate() : '0' + date.getDate()
				return {
					dateStr:year+'-'+month+'-'+day,
                    week: '周'+this.weekList[date.getDay()],
                    recent
				}
			},
            chooseDate(day, month, year) {
                const _date = this.resetTime(year + '/' + month + '/' + day)
                const  week = this.weekList[new Date(_date).getDay()]

                //判断日期区域是否可点击
                if (!day) return
                if(this.betweenStarts){
                    if(_date * 1 < this.betweenStarts) return
                }else{
                    if (_date < this.today) return
                }
                if(_date > this.betweenEnds) return
                
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
                    if(this.mode==1){
                        this.startDates = _date
                    }else{
                        this.endDates = _date;
                    }
                }

                const choose = {
                    startStr:this.dateFormat(this.startDates)
                }

                if(this.mode==1){
                    this.$emit('callback',choose)
                }else if(this.mode==2&&this.startDates && this.endDates){
                    choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                    choose.endStr = this.dateFormat(this.endDates)
                    this.$emit('callback',choose)
                }else if(this.mode==3){
                    if (this.startDates && this.endDates) {
                        choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                        choose.endStr = this.dateFormat(this.endDates)
                    }else if(this.startDates&&!this.endDates){
                        choose.dayCount = 0
                        choose.endStr = this.dateFormat(this.startDates)
                    }
                    this.$emit('callback',choose)
                }
               
            }
        }
    }
</script>

<style lang="less" scoped>
    @color: #415FFB;
    .calendar {
        width: 100%;
        height: 100%;
        background: #fff;
        position:fixed;
        left:0;
        top:0;
        overflow-y: scroll;
        z-index:999;
        .calendar-header {
            position: fixed;
            width: 100%;
            top: 0;
            left: 0;
            z-index: 9991;
            box-shadow: 0 2px 15px rgba(100,100,100,0.1);
            .calendar-title {
                padding:6px 10px;
                background: #fff7dc;
                font-size: 12px;
                color: #9e8052;
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
            .week-number {
                background: #fff;
                padding: 0 1%;
                span {
                    display: inline-block;
                    text-align: center;
                    padding:12px 0;
                    font-size:14px;
                    width: 14.28%;
                    &:first-child,
                    &:last-child {
                        color: @color;
                    }
                }
            }
        }
        .calendar-wrapper {
            position: relative;
            color: #000;
            padding-top:10px;
            h3 {
                position: sticky;
                position: -webkit-sticky;
                z-index: 999;
                width: 100%;
                left: 0;
                color:#000;
                text-align: center;
                font-size: 16px;
                font-weight: 400;
                padding:10px 0;
            }
            .each-month {
                display: block;
                width: 98%;
                font-size: 0;
                margin:0 auto;
                padding-left:0;
                padding-bottom: 10px;
                border-bottom:1px solid #F4F4F4;
                .each-day {
                    position: relative;
                    display: inline-block;
                    text-align: center;
                    vertical-align: middle;
                    width: 14.28%;
                    font-size: 16px;
                    height: 52px;
                    margin-top:4px;
                    padding-top:4px;
                    div {
                        display: inline-block;
                        font-size:14px;
                        padding:8px 0;
                        width: 32px;
                    }
                    &.between {
                        background: rgba(75, 217, 173, 0.1);
                    }
                    .disabled {
                        color: #ccc !important;
                        &.trip-time{
                           background: #e7e7e7; 
                        }
                        &.today{
                            color:#ccc!important;
                            background: none;
                        }
                    }
                    .disabled+.recent{
                        color:#ccc;
                    }
                    .today {
                        background: #e7e7e7;
                        border-radius: 4px;
                    }
                    .trip-time {
                        background: @color;
                        color: #fff !important;
                        border-radius: 4px;
                    }
                    .weekend {
                        color: @color;
                    }
                    .recent {
                        position: absolute;
                        font-size: 10px;
                        width: 100%;
                        text-align: center;
                        color: @color;
                        bottom: 0;
                        left: 0;
                    }
                }
            }
        }
    }
    
</style><template>
	<div class="calendar">
		<div class="calendar-header">
			<div class="week-number">
				<span v-for="(item,index) in weekList" :style="{color:(index==0||index==weekList.length-1)&&themeColor}" :key="index">{{item}}</span>
			</div>
            <p class="calendar-title" v-if="title">{{title}}</p>
		</div>
        <p :style="{height:title?'68px':'40px'}"></p>
        <div class="calendar-wrapper" v-for="(item,index) in calendar" :key="index">
            <h3 v-text="item.year + '年' + item.month + '月'"></h3>
            <ul class="each-month">
                <li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" :class="[addClassBg(day, item.month, item.year)]" :style="{background:themeOpacityBg(day, item.month, item.year)}" @click="chooseDate(day, item.month, item.year)">
                    <div :class="[addClassName(day, item.month, item.year),{'trip-time': isCurrent(day, item.month, item.year)}]" :style="{background:themeBg(day, item.month, item.year)}">{{day?day:''}}</div>
                    <span class="recent" :style="{color:themeColor}" v-text="setTip(day, item.month, item.year)"></span>
                </li>
            </ul>
        </div>
        <slot></slot>
	</div>
</template>

<script>
    export default {
        props: {
            title: {
                type: [String, Object],
                default () {
                    return ''
                }
            }, 
            mode: {
                type: [String, Number],
                default () {
                    return 1
                }
            },
            startDate: { //开始日期
                type: [String, Object,Date],
                default () {
                    const year = new Date().getFullYear(),
                        month = new Date().getMonth()+1,
                        day = new Date().getDate()
                    return year+'/'+month+'/'+day
                }
            },
            endDate: { //结束日期
                type: [String, Object, Date],
                default () {
                    var temp = new Date().getTime() + 24 * 60 * 60 * 1000
                    var date = new Date(temp)
                    const year = date.getFullYear(),
                        month = date.getMonth()+1,
                        day = date.getDate()
                    return year+'/'+month+'/'+day
                }
            },
            betweenStart: { //日历可选范围开始
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            betweenEnd: { //日历可选结束日期
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            initMonth:{//初始化的月数
                type:[String,Number],
                default(){
                    return 6
                }
            },
            themeColor: {//主题色
				type: [String],
				default: ''
			}
        },
        data() {
            return {
                startDates:'',
                endDates:'',
                betweenStarts:'',
                betweenEnds:'',
                calendar: [],
                weekList: ['日', '一', '二', '三', '四', '五', '六']
            }
        },
        mounted() {
            this.init()
        },
        computed: {
            //theme
			getBetweenColor() {
                if(!this.themeColor) return
                var hex = this.themeColor
				if (hex.length == 4) {
					hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`
				}
				var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" +
					hex.slice(5, 7)) + ",0.1)";
				return str
			},
		},
        methods: {
            init() {
                this.year = new Date().getFullYear()
                this.month = new Date().getMonth()+1
                this.day = new Date().getDate()
                this.today = new Date(this.year+'/'+this.month+'/'+this.day) * 1

                if(this.mode==1){//普通模式不提供默认值，第一次让用户选择日期
                    this.startDates = ''
                    this.endDates = ''
                }else{//酒店和往返模式提供默认值
                    this.startDates = this.resetTime(this.startDate)
                    this.endDates = this.resetTime(this.endDate)
                }
                
                this.betweenStarts = this.resetTime(this.betweenStart)
                this.betweenEnds = this.resetTime(this.betweenEnd)

                this.createClendar(); //创建日历数据
            },
            //创建每个月日历数据，传入月份1号前面用null填充
            createDayList(month, year) {
                const count = this.getDayNum(month, year),
                    _week = new Date(year + '/' + month + '/1').getDay();
                let list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28]

                for (let i = 29; i <= count; i++) {
                    list.push(i)
                }
                for (let i = 0; i < _week; i++) {
                    list.unshift(null)
                }
                return list;
            },
            //计算传入月份有多少天
            getDayNum(month, year) {
                let dayNum = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

                if ((year % 4 === 0) && (year % 100 !== 0) || (year % 400 === 0)) {
                    dayNum[1] = 29;
                }
                return dayNum[month - 1]
            },
            //根据当天和结束日期创建日历数据
            createClendar() {
                var yearTemp = this.year
                var monthTemp = this.month
                if(!!this.betweenStarts){//如果有范围起始日期，可选范围从betweenStart开始
                    yearTemp = new Date(this.betweenStarts).getFullYear()
                    monthTemp = new Date(this.betweenStarts).getMonth()+1
                }
                for (let i = 0; i < this.initMonth; i++) {
                    let year = yearTemp
                    let month = monthTemp + i
                    let  _monthData = {
                            dayList: [],
                            month: '',
                            year: ''
                        };

                    var m = Math.ceil(month / 12)
					if (m > 0) {
						year += m - 1
					} else {
						year += m - 1
					}
					if (month > 12) {
						month = month % 12 == 0 ? 12 : month % 12;
					}
					if (month <= 0) {
						month = 12 + month % 12
                    }
                    
                    _monthData.year = year;
                    _monthData.month = month;
                    _monthData.dayList = this.createDayList(month, year);
                    this.calendar.push(_monthData)
                }
            },
            //添加日历样式
            addClassName(day, month, year) {
                if (!day) return
                const _date = new Date(year + '/' + month + '/' + day)
                let className = []
                // if (_date.getDay() == 0 || _date.getDay() == 6) { //周末或周六样式
                //     className.push('weekend')
                // }
                if (_date * 1 == this.today) {
                    className.push('today');
                }
                if(this.betweenStarts){
                   (_date * 1 < this.betweenStarts)&&className.push('disabled')
                }else{
                    (_date * 1 < this.today)&&className.push('disabled')  //当天和结束日期之外不可选
                }
                (_date * 1 > this.betweenEnds)&&className.push('disabled')
                return className.join(' ');
            },
            //入住离开的区间背景色
            addClassBg(day, month, year) {
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                let className = [];
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    className.push("between");
                }
                return className.join(" ");
            },
            //theme入住离开的区间背景色
            themeOpacityBg(day, month, year) {
				if(!this.themeColor) return
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    return this.getBetweenColor
                }
            },
			//theme获取普通日期选中样式背景
			themeBg(day, month, year) {
                if(!this.themeColor) return
				const _date = this.resetTime(year + '/' + month + '/' + day)
                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return this.themeColor
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return this.themeColor
                    }
                }
			},
            //清除时间 时 分 秒 毫秒
            resetTime(dateStr) {
                var date = new Date(dateStr.replace(/-/g, '/'))
                date.setHours(0);
                date.setMinutes(0);
                date.setSeconds(0);
                date.setMilliseconds(0);
                return date * 1;
            },
            //设置今天，明天，后天
            setTip(day, month, year) {
                if (!day) {
                    return;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)
                let tip = "";

                if (_date == this.today) {
                    tip = '今天'
                } else if (_date - this.today == 24 * 3600 * 1000) {
                    tip = '明天'
                } else if (_date  - this.today == 2 * 24 * 3600 * 1000) {
                    tip = '后天'
                }
                if(this.mode == 2){
                    if (_date == this.endDates) {
                        tip = "离开";
                    } else if (_date == this.startDates) {
                        tip = "入住";
                    }
                }else if (this.mode == 3) {
                    if (_date == this.startDates && !this.endDates) {
                        tip = '去/返'
                    } else {
                        if (_date == this.endDates) {
                            tip = '返程'
                        } else if (_date == this.startDates) {
                            tip = '去程'
                        }
                    }
                }
                return tip;
            },
            isCurrent(day, month, year) {
                if (!day) {
                    return false;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)

                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return true
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return true
                    }
                }
                
            },
            dateFormat(times) {
                let date = new Date(times);
                let recent = ''  
                if (times == this.today) {
                    recent = '今天'
                } else if (times - this.today === 24 * 3600 * 1000) {
                    recent = '明天'
                } else if (times - this.today === 2 * 24 * 3600 * 1000) {
                    recent = '后天'
                }
               
                var year = date.getFullYear()
				var month = parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1)
				var day =  date.getDate() > 9 ? date.getDate() : '0' + date.getDate()
				return {
					dateStr:year+'-'+month+'-'+day,
                    week: '周'+this.weekList[date.getDay()],
                    recent
				}
			},
            chooseDate(day, month, year) {
                const _date = this.resetTime(year + '/' + month + '/' + day)
                const  week = this.weekList[new Date(_date).getDay()]

                //判断日期区域是否可点击
                if (!day) return
                if(this.betweenStarts){
                    if(_date * 1 < this.betweenStarts) return
                }else{
                    if (_date < this.today) return
                }
                if(_date > this.betweenEnds) return
                
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
                    if(this.mode==1){
                        this.startDates = _date
                    }else{
                        this.endDates = _date;
                    }
                }

                const choose = {
                    startStr:this.dateFormat(this.startDates)
                }

                if(this.mode==1){
                    this.$emit('callback',choose)
                }else if(this.mode==2&&this.startDates && this.endDates){
                    choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                    choose.endStr = this.dateFormat(this.endDates)
                    this.$emit('callback',choose)
                }else if(this.mode==3){
                    if (this.startDates && this.endDates) {
                        choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                        choose.endStr = this.dateFormat(this.endDates)
                    }else if(this.startDates&&!this.endDates){
                        choose.dayCount = 0
                        choose.endStr = this.dateFormat(this.startDates)
                    }
                    this.$emit('callback',choose)
                }
               
            }
        }
    }
</script>

<style lang="less" scoped>
    @color: #415FFB;
    .calendar {
        width: 100%;
        height: 100%;
        background: #fff;
        position:fixed;
        left:0;
        top:0;
        overflow-y: scroll;
        z-index:999;
        .calendar-header {
            position: fixed;
            width: 100%;
            top: 0;
            left: 0;
            z-index: 9991;
            box-shadow: 0 2px 15px rgba(100,100,100,0.1);
            .calendar-title {
                padding:6px 10px;
                background: #fff7dc;
                font-size: 12px;
                color: #9e8052;
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
            .week-number {
                background: #fff;
                padding: 0 1%;
                span {
                    display: inline-block;
                    text-align: center;
                    padding:12px 0;
                    font-size:14px;
                    width: 14.28%;
                    &:first-child,
                    &:last-child {
                        color: @color;
                    }
                }
            }
        }
        .calendar-wrapper {
            position: relative;
            color: #000;
            padding-top:10px;
            h3 {
                position: sticky;
                position: -webkit-sticky;
                z-index: 999;
                width: 100%;
                left: 0;
                color:#000;
                text-align: center;
                font-size: 16px;
                font-weight: 400;
                padding:10px 0;
            }
            .each-month {
                display: block;
                width: 98%;
                font-size: 0;
                margin:0 auto;
                padding-left:0;
                padding-bottom: 10px;
                border-bottom:1px solid #F4F4F4;
                .each-day {
                    position: relative;
                    display: inline-block;
                    text-align: center;
                    vertical-align: middle;
                    width: 14.28%;
                    font-size: 16px;
                    height: 52px;
                    margin-top:4px;
                    padding-top:4px;
                    div {
                        display: inline-block;
                        font-size:14px;
                        padding:8px 0;
                        width: 32px;
                    }
                    &.between {
                        background: rgba(75, 217, 173, 0.1);
                    }
                    .disabled {
                        color: #ccc !important;
                        &.trip-time{
                           background: #e7e7e7; 
                        }
                        &.today{
                            color:#ccc!important;
                            background: none;
                        }
                    }
                    .disabled+.recent{
                        color:#ccc;
                    }
                    .today {
                        background: #e7e7e7;
                        border-radius: 4px;
                    }
                    .trip-time {
                        background: @color;
                        color: #fff !important;
                        border-radius: 4px;
                    }
                    .weekend {
                        color: @color;
                    }
                    .recent {
                        position: absolute;
                        font-size: 10px;
                        width: 100%;
                        text-align: center;
                        color: @color;
                        bottom: 0;
                        left: 0;
                    }
                }
            }
        }
    }
    
</style><template>
	<div class="calendar">
		<div class="calendar-header">
			<div class="week-number">
				<span v-for="(item,index) in weekList" :style="{color:(index==0||index==weekList.length-1)&&themeColor}" :key="index">{{item}}</span>
			</div>
            <p class="calendar-title" v-if="title">{{title}}</p>
		</div>
        <p :style="{height:title?'68px':'40px'}"></p>
        <div class="calendar-wrapper" v-for="(item,index) in calendar" :key="index">
            <h3 v-text="item.year + '年' + item.month + '月'"></h3>
            <ul class="each-month">
                <li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" :class="[addClassBg(day, item.month, item.year)]" :style="{background:themeOpacityBg(day, item.month, item.year)}" @click="chooseDate(day, item.month, item.year)">
                    <div :class="[addClassName(day, item.month, item.year),{'trip-time': isCurrent(day, item.month, item.year)}]" :style="{background:themeBg(day, item.month, item.year)}">{{day?day:''}}</div>
                    <span class="recent" :style="{color:themeColor}" v-text="setTip(day, item.month, item.year)"></span>
                </li>
            </ul>
        </div>
        <slot></slot>
	</div>
</template>

<script>
    export default {
        props: {
            title: {
                type: [String, Object],
                default () {
                    return ''
                }
            }, 
            mode: {
                type: [String, Number],
                default () {
                    return 1
                }
            },
            startDate: { //开始日期
                type: [String, Object,Date],
                default () {
                    const year = new Date().getFullYear(),
                        month = new Date().getMonth()+1,
                        day = new Date().getDate()
                    return year+'/'+month+'/'+day
                }
            },
            endDate: { //结束日期
                type: [String, Object, Date],
                default () {
                    var temp = new Date().getTime() + 24 * 60 * 60 * 1000
                    var date = new Date(temp)
                    const year = date.getFullYear(),
                        month = date.getMonth()+1,
                        day = date.getDate()
                    return year+'/'+month+'/'+day
                }
            },
            betweenStart: { //日历可选范围开始
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            betweenEnd: { //日历可选结束日期
                type: [String, Object, Date],
                default () {
                    return ''
                }
            },
            initMonth:{//初始化的月数
                type:[String,Number],
                default(){
                    return 6
                }
            },
            themeColor: {//主题色
				type: [String],
				default: ''
			}
        },
        data() {
            return {
                startDates:'',
                endDates:'',
                betweenStarts:'',
                betweenEnds:'',
                calendar: [],
                weekList: ['日', '一', '二', '三', '四', '五', '六']
            }
        },
        mounted() {
            this.init()
        },
        computed: {
            //theme
			getBetweenColor() {
                if(!this.themeColor) return
                var hex = this.themeColor
				if (hex.length == 4) {
					hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`
				}
				var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" +
					hex.slice(5, 7)) + ",0.1)";
				return str
			},
		},
        methods: {
            init() {
                this.year = new Date().getFullYear()
                this.month = new Date().getMonth()+1
                this.day = new Date().getDate()
                this.today = new Date(this.year+'/'+this.month+'/'+this.day) * 1

                if(this.mode==1){//普通模式不提供默认值，第一次让用户选择日期
                    this.startDates = ''
                    this.endDates = ''
                }else{//酒店和往返模式提供默认值
                    this.startDates = this.resetTime(this.startDate)
                    this.endDates = this.resetTime(this.endDate)
                }
                
                this.betweenStarts = this.resetTime(this.betweenStart)
                this.betweenEnds = this.resetTime(this.betweenEnd)

                this.createClendar(); //创建日历数据
            },
            //创建每个月日历数据，传入月份1号前面用null填充
            createDayList(month, year) {
                const count = this.getDayNum(month, year),
                    _week = new Date(year + '/' + month + '/1').getDay();
                let list = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28]

                for (let i = 29; i <= count; i++) {
                    list.push(i)
                }
                for (let i = 0; i < _week; i++) {
                    list.unshift(null)
                }
                return list;
            },
            //计算传入月份有多少天
            getDayNum(month, year) {
                let dayNum = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

                if ((year % 4 === 0) && (year % 100 !== 0) || (year % 400 === 0)) {
                    dayNum[1] = 29;
                }
                return dayNum[month - 1]
            },
            //根据当天和结束日期创建日历数据
            createClendar() {
                var yearTemp = this.year
                var monthTemp = this.month
                if(!!this.betweenStarts){//如果有范围起始日期，可选范围从betweenStart开始
                    yearTemp = new Date(this.betweenStarts).getFullYear()
                    monthTemp = new Date(this.betweenStarts).getMonth()+1
                }
                for (let i = 0; i < this.initMonth; i++) {
                    let year = yearTemp
                    let month = monthTemp + i
                    let  _monthData = {
                            dayList: [],
                            month: '',
                            year: ''
                        };

                    var m = Math.ceil(month / 12)
					if (m > 0) {
						year += m - 1
					} else {
						year += m - 1
					}
					if (month > 12) {
						month = month % 12 == 0 ? 12 : month % 12;
					}
					if (month <= 0) {
						month = 12 + month % 12
                    }
                    
                    _monthData.year = year;
                    _monthData.month = month;
                    _monthData.dayList = this.createDayList(month, year);
                    this.calendar.push(_monthData)
                }
            },
            //添加日历样式
            addClassName(day, month, year) {
                if (!day) return
                const _date = new Date(year + '/' + month + '/' + day)
                let className = []
                // if (_date.getDay() == 0 || _date.getDay() == 6) { //周末或周六样式
                //     className.push('weekend')
                // }
                if (_date * 1 == this.today) {
                    className.push('today');
                }
                if(this.betweenStarts){
                   (_date * 1 < this.betweenStarts)&&className.push('disabled')
                }else{
                    (_date * 1 < this.today)&&className.push('disabled')  //当天和结束日期之外不可选
                }
                (_date * 1 > this.betweenEnds)&&className.push('disabled')
                return className.join(' ');
            },
            //入住离开的区间背景色
            addClassBg(day, month, year) {
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                let className = [];
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    className.push("between");
                }
                return className.join(" ");
            },
            //theme入住离开的区间背景色
            themeOpacityBg(day, month, year) {
				if(!this.themeColor) return
                if (!day) return
                const _date = this.resetTime(year + "/" + month + "/" + day);
                if (_date >= this.startDates && _date <= this.endDates && this.mode>1) {
                    return this.getBetweenColor
                }
            },
			//theme获取普通日期选中样式背景
			themeBg(day, month, year) {
                if(!this.themeColor) return
				const _date = this.resetTime(year + '/' + month + '/' + day)
                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return this.themeColor
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return this.themeColor
                    }
                }
			},
            //清除时间 时 分 秒 毫秒
            resetTime(dateStr) {
                var date = new Date(dateStr.replace(/-/g, '/'))
                date.setHours(0);
                date.setMinutes(0);
                date.setSeconds(0);
                date.setMilliseconds(0);
                return date * 1;
            },
            //设置今天，明天，后天
            setTip(day, month, year) {
                if (!day) {
                    return;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)
                let tip = "";

                if (_date == this.today) {
                    tip = '今天'
                } else if (_date - this.today == 24 * 3600 * 1000) {
                    tip = '明天'
                } else if (_date  - this.today == 2 * 24 * 3600 * 1000) {
                    tip = '后天'
                }
                if(this.mode == 2){
                    if (_date == this.endDates) {
                        tip = "离开";
                    } else if (_date == this.startDates) {
                        tip = "入住";
                    }
                }else if (this.mode == 3) {
                    if (_date == this.startDates && !this.endDates) {
                        tip = '去/返'
                    } else {
                        if (_date == this.endDates) {
                            tip = '返程'
                        } else if (_date == this.startDates) {
                            tip = '去程'
                        }
                    }
                }
                return tip;
            },
            isCurrent(day, month, year) {
                if (!day) {
                    return false;
                }
                const _date = this.resetTime(year + '/' + month + '/' + day)

                //正常模式
                if(this.mode == 1){
                    if (_date == this.startDates) {
                        return true
                    }
                }else{//酒店和往返模式
                    if (_date == this.startDates||_date == this.endDates) {
                        return true
                    }
                }
                
            },
            dateFormat(times) {
                let date = new Date(times);
                let recent = ''  
                if (times == this.today) {
                    recent = '今天'
                } else if (times - this.today === 24 * 3600 * 1000) {
                    recent = '明天'
                } else if (times - this.today === 2 * 24 * 3600 * 1000) {
                    recent = '后天'
                }
               
                var year = date.getFullYear()
				var month = parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1)
				var day =  date.getDate() > 9 ? date.getDate() : '0' + date.getDate()
				return {
					dateStr:year+'-'+month+'-'+day,
                    week: '周'+this.weekList[date.getDay()],
                    recent
				}
			},
            chooseDate(day, month, year) {
                const _date = this.resetTime(year + '/' + month + '/' + day)
                const  week = this.weekList[new Date(_date).getDay()]

                //判断日期区域是否可点击
                if (!day) return
                if(this.betweenStarts){
                    if(_date * 1 < this.betweenStarts) return
                }else{
                    if (_date < this.today) return
                }
                if(_date > this.betweenEnds) return
                
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
                    if(this.mode==1){
                        this.startDates = _date
                    }else{
                        this.endDates = _date;
                    }
                }

                const choose = {
                    startStr:this.dateFormat(this.startDates)
                }

                if(this.mode==1){
                    this.$emit('callback',choose)
                }else if(this.mode==2&&this.startDates && this.endDates){
                    choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                    choose.endStr = this.dateFormat(this.endDates)
                    this.$emit('callback',choose)
                }else if(this.mode==3){
                    if (this.startDates && this.endDates) {
                        choose.dayCount = (this.endDates - this.startDates) / 24 / 3600 / 1000
                        choose.endStr = this.dateFormat(this.endDates)
                    }else if(this.startDates&&!this.endDates){
                        choose.dayCount = 0
                        choose.endStr = this.dateFormat(this.startDates)
                    }
                    this.$emit('callback',choose)
                }
               
            }
        }
    }
</script>

<style lang="less" scoped>
    @color: #415FFB;
    .calendar {
        width: 100%;
        height: 100%;
        background: #fff;
        position:fixed;
        left:0;
        top:0;
        overflow-y: scroll;
        z-index:999;
        .calendar-header {
            position: fixed;
            width: 100%;
            top: 0;
            left: 0;
            z-index: 9991;
            box-shadow: 0 2px 15px rgba(100,100,100,0.1);
            .calendar-title {
                padding:6px 10px;
                background: #fff7dc;
                font-size: 12px;
                color: #9e8052;
                overflow: hidden;
                text-overflow: ellipsis;
                white-space: nowrap;
            }
            .week-number {
                background: #fff;
                padding: 0 1%;
                span {
                    display: inline-block;
                    text-align: center;
                    padding:12px 0;
                    font-size:14px;
                    width: 14.28%;
                    &:first-child,
                    &:last-child {
                        color: @color;
                    }
                }
            }
        }
        .calendar-wrapper {
            position: relative;
            color: #000;
            padding-top:10px;
            h3 {
                position: sticky;
                position: -webkit-sticky;
                z-index: 999;
                width: 100%;
                left: 0;
                color:#000;
                text-align: center;
                font-size: 16px;
                font-weight: 400;
                padding:10px 0;
            }
            .each-month {
                display: block;
                width: 98%;
                font-size: 0;
                margin:0 auto;
                padding-left:0;
                padding-bottom: 10px;
                border-bottom:1px solid #F4F4F4;
                .each-day {
                    position: relative;
                    display: inline-block;
                    text-align: center;
                    vertical-align: middle;
                    width: 14.28%;
                    font-size: 16px;
                    height: 52px;
                    margin-top:4px;
                    padding-top:4px;
                    div {
                        display: inline-block;
                        font-size:14px;
                        padding:8px 0;
                        width: 32px;
                    }
                    &.between {
                        background: rgba(75, 217, 173, 0.1);
                    }
                    .disabled {
                        color: #ccc !important;
                        &.trip-time{
                           background: #e7e7e7; 
                        }
                        &.today{
                            color:#ccc!important;
                            background: none;
                        }
                    }
                    .disabled+.recent{
                        color:#ccc;
                    }
                    .today {
                        background: #e7e7e7;
                        border-radius: 4px;
                    }
                    .trip-time {
                        background: @color;
                        color: #fff !important;
                        border-radius: 4px;
                    }
                    .weekend {
                        color: @color;
                    }
                    .recent {
                        position: absolute;
                        font-size: 10px;
                        width: 100%;
                        text-align: center;
                        color: @color;
                        bottom: 0;
                        left: 0;
                    }
                }
            }
        }
    }
    
</style>