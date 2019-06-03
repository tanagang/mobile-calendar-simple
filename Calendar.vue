<template>
	<div class="calendar-tz">
			<div class="calendar-header">
				<div class="week-number">
					<span v-for="(item,index) in weekList" :key="item"><p :style="{color:(index==0||index==weekList.length-1)?getThemeColor:''}">{{item}}</p></span>
				</div>
			</div>
			<div class="ti" :style='{paddingTop:"44px"}'>
				<div class="calendar-wrapper" v-for="(item,index) in calendar" :key="index">
					<view class="calendar-title">{{item.year}} 年 {{item.month}} 月</view>
					<!--如果普通日期选择-->
					<ul class="each-month" v-if="date||(!date&&!startDate&&!endDate)">
						<li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" @click="chooseDate($event,day, item.month, item.year)">
							<div :class="[addClassName(day, item.month, item.year)]"  :style="{background:getBackground(day, item.month, item.year),color:getWeekColor(day, item.month, item.year)}">
								{{ setFestival(day, item.month, item.year)!= 0 ? setFestival(day, item.month, item.year) : day}}
							</div>
							<span class="recent" v-text="setTip(day, item.month, item.year)" :style="{color:(index==0||index==weekList.length-1)?getThemeColor:''}"></span>
						</li>
					</ul>
					<!--如果酒店入住离开选择-->
					<ul class="each-month" v-else>
						<li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" :style="{background:addClassName2(day, item.month, item.year)}"
						 @click="chooseDate($event,day, item.month, item.year)">
							<div :class="[addClassName(day, item.month, item.year),{'trip-time': isCurrent(day, item.month, item.year)}]"  :style="{background:isCurrent(day, item.month, item.year)?getThemeColor:'',color:getWeekColor(day, item.month, item.year)}">
								{{ setFestival(day, item.month, item.year)!= 0 ? setFestival(day, item.month, item.year) : day}}
							</div>
							<span class="recent" v-text="setTip(day, item.month, item.year)" :style="{color:getThemeColor}"></span>
						</li>
					</ul>
				</div>
			</div>
		    <!--<div style="height:50px"></div>
				<div class="closeDialog">
	            <span class="icon-close" @click="closeDialog"></span>
	        </div> -->
	</div>
		
</template>

<script>
	export default {
		props: {
			date: { //选择的日期（此属性和startDate,endDate互斥）
				type: [String, Object],
				default () {
					return ''
				}
			},
			startDate: { //开始日期（入住酒店）
				type: [String, Object],
				default () {
					return ''
				}
			},
			endDate: { //结束日期（离开酒店）
				type: [String, Object, Date],
				default () {
					return ''
				}
			},
			themeColor:{
				type: [String, Object, Date],
				default () {
					return '#415FFB'
				}
			}
		},
		data() {
			return {
				endDates: '',
				startDates: '',
				dates: '',
				isDate:false,
				betweenDate: '', //显示日历的时间段
				weekList: ['日', '一', '二', '三', '四', '五', '六'],
				calendar: [],
				festival: {
					// 2018
					// "2018-1-1": "元旦",
					// "2018-2-15": "除夕",
					// "2018-2-16": "春节",
					// "2018-3-2": "元宵节",
					// "2018-4-5": "清明节",
					// "2018-5-31": "初四",
					// "2018-5-1": "五一",
					// "2018-5-13": "母亲节",
					// "2018-6-1": "六一",
					// "2018-6-18": "端午",
					// "2018-7-1": "建党节",
					// "2018-8-1": "建军节",
					// "2018-8-17": "七夕",
					// "2018-9-10": "教师节",
					// "2018-9-24": "中秋节",
					// "2018-10-1": "国庆",
					// "2018-10-17": "重阳节",
					// "2018-10-22": "感恩节",
					// "2018-12-24": "平安夜",
					// "2018-12-25": "圣诞节",
					// "2019-1-1": "元旦"
				}
			}
		},
		computed: {
			getThemeColor() {			
				var hex = this.themeColor
				if(hex.length==4){
					hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`
				}
				var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" + hex.slice(5, 7)) + ",1)";
				return str
			},
			getBetweenColor() {
				var hex = this.themeColor
				if(hex.length==4){
					hex = `#${hex[1]}${hex[1]}${hex[2]}${hex[2]}${hex[3]}${hex[3]}`
				}
				var str = "rgba(" + parseInt("0x" + hex.slice(1, 3)) + "," + parseInt("0x" + hex.slice(3, 5)) + "," + parseInt("0x" + hex.slice(5, 7)) + ",0.1)";
				return str
			},
		},
		mounted() {
			this.init()
		},
		methods: {
			init() {
				if (this.date) {
					this.dates = new Date(this.date.replace(/-/g, '/'))
					this.isDate = true
				}
				if (this.startDate) {
					this.startDates = new Date(this.startDate.replace(/-/g, '/'))
				}
				if (this.endDate) {
					this.endDates = new Date(this.endDate.replace(/-/g, '/'))
				}

				this.today = new Date(new Date().toLocaleDateString()).getTime()

				if (this.date && (this.startDate || this.endDate)) {
					console.warn(':date属性和 (:startDate,:endDate) 不能同时设置')
					this.isDate = true
				}
				if (!this.date && !this.startDate && this.endDate) {
					this.startDates = new Date(this.today * 1)
				}
				if (!this.date && !this.startDate && !this.endDate) {
					this.dates = new Date(this.today * 1)
					this.isDate = true
				}

				if (this.betweenDate === '') {
					//默认结束日期为180天后
					this.betweenDate = new Date(this.today * 1 + 180 * 24 * 3600 * 1000)
				}

				this.year = new Date().getFullYear();
				this.month = new Date().getMonth() + 1;
				this.createClendar(); //创建日历数据

			},
			closeDialog() {
				this.$emit("close");
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
					list.unshift("")
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
				const endY = this.betweenDate.getFullYear(),
					endM = this.betweenDate.getMonth() + 1,
					interval = (endY - this.year) * 12 + endM - this.month;
				for (let i = 0; i <= interval; i++) {
					let month = this.month + i,
						year = this.year,
						_monthData = {
							dayList: [],
							month: '',
							year: ''
						};

					if (month > 12) {
						month = month - 12;
						year += 1;
					}
					_monthData.year = year;
					_monthData.month = month;
					_monthData.dayList = this.createDayList(month, year);
					this.calendar.push(_monthData)
				}
			},
			//获取周末的样式
			getWeekColor(day, month, year){
				const _date = new Date(year + '/' + month + '/' + day)
				//设置周末的字体样式
				if (_date.getDay() === 0 || _date.getDay() === 6) { 
					return this.getThemeColor
				}
			},
			//获取普通日期选中样式背景
			getBackground(day, month, year){
				const _date = new Date(year + '/' + month + '/' + day)
				if (_date * 1 === this.dates * 1) {
					return this.getThemeColor
				}
			},
			//添加日历样式
			addClassName(day, month, year) {
				if (!day) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day)
				let className = [],
					festival = this.festival[year + "/" + month + "/" + day];
				// if (_date.getDay() === 0 || _date.getDay() === 6) { //周末或周六样式
				// 	className.push('weekend')
				// }
				if (_date * 1 === this.today) {
					className.push('today');
				}

				if (_date * 1 < this.today) { //当天之前不可选
					className.push('disabled')
				} else if (_date * 1 === this.dates * 1) {
					className.push(' trip-time');
				}

				className.push('festival')
				return className.join(' ');
			},
			addClassName2(day, month, year) {
				if (!day || this.date) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day) * 1
				//let className = []
				if (_date >= this.startDates * 1 && _date <= this.endDates * 1) {
					//className.push('between')
					return this.getBetweenColor
				}
				//return className.join(' ');
			},
			//清除时间 时 分 秒 毫秒
			resetTime(date) {
				date.setHours(0);
				date.setMinutes(0);
				date.setSeconds(0);
				date.setMilliseconds(0);
				return date;
			},
			//设置日期和假日
			setFestival(day, month, year) {
				const festivalStr = this.festival[year + "/" + month + "/" + day]
				if (festivalStr) {
					return festivalStr
				} else {
					return 0;
				}
			},
			//设置今天，明天，后天
			setTip(day, month, year) {
				if (!day) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day) * 1
				let tip;

				if (_date == this.today) {
					tip = '今天'
				} else if (_date - this.today === 24 * 3600 * 1000) {
					tip = '明天'
				} else if (_date - this.today === 2 * 24 * 3600 * 1000) {
					tip = '后天'
				}
				if (!this.date&&(this.startDate||this.endDate)) {
					if (_date === this.startDates * 1) {
						tip = '入住'
					} else if (_date === this.endDates * 1) {
						tip = '离开'
					}
				}

				return tip;
			},
			isCurrent(day, month, year) {
				if (!day) {
					return false;
				}

				const _date = new Date(year + '/' + month + '/' + day) * 1
				//设置入住和离开
				if (_date === this.startDates * 1 || (_date === this.endDates * 1)) {
					return true
				}
			},
			dateFormat(times) {
				let weekList = ['周日', '周一', '周二', '周三', '周四', '周五', '周六'];
				let date = new Date(times);
				return {
					y: date.getFullYear(),
					m: parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1),
					d: date.getDate() > 9 ? date.getDate() : '0' + date.getDate(),
					w: weekList[date.getDay()]
				}
			},
			chooseDate(e, day, month, year) {
				if (!day) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day) * 1

				if (_date < this.today) {
					return;
				}
				if (_date == this.today||this.dates*1) {
					this.dates = _date
				}
				
				if (this.startDates * 1 && this.endDates * 1 && _date > this.endDates * 1) {
					this.startDates = _date;
					this.endDates = "";
				} else if (this.endDates * 1 && _date > this.endDates) {
					this.endDates = _date;
				} else if (_date >= this.startDates * 1 && _date <= this.endDates * 1) {
					this.startDates = _date;
					this.endDates = '';
				} else if (_date < this.startDates * 1) {
					this.startDates = _date;
					this.endDates = '';
				} else if (_date > this.startDates * 1) {
					this.endDates = _date;
				}

				const choose = {
					dateStr: this.dates * 1,
					date: this.dateFormat(this.dates),
					recent: ''
				}
				const choose2 = {
					startDateStr: this.startDates,
					endDateStr: this.endDates,
					startDate: this.dateFormat(this.startDates),
					endDate: this.dateFormat(this.endDates),
					startRecent: '',
					endRecent: ''
				}
				
				if(this.isDate){//普通模式的recent
					if (_date == this.today) {
						choose.recent = '今天'
					} else if (_date - this.today == 24 * 3600 * 1000) {
						choose.recent = '明天'
					} else if (_date - this.today == 2 * 24 * 3600 * 1000) {
						choose.recent = '后天'
					}
				}else{//酒店模式的recent
					if (this.startDates == this.today) {
						choose2.startRecent = '今天'
					} else if (this.startDates - this.today == 24 * 3600 * 1000) {
						choose2.startRecent = '明天'
					} else if (this.startDates - this.today == 2 * 24 * 3600 * 1000) {
						choose2.startRecent = '后天'
					}
					
					if (this.endDates == this.today) {
						choose2.endRecent = '今天'
					} else if (this.endDates - this.today == 24 * 3600 * 1000) {
						choose2.endRecent = '明天'
					} else if (this.endDates - this.today == 2 * 24 * 3600 * 1000) {
						choose2.endRecent = '后天'
					}
				}

				if (this.isDate) { //普通日期选择模式
					this.$emit("callback", choose)
				} else { //酒店入住模式
					choose2.countDays = (this.endDates * 1 - this.startDates * 1) / 86400 / 1000;
					if (this.startDates && this.endDates) {
						this.$emit("callback", choose2)
					}
				}
			}
		}
	}
</script>

<style lang="less" scoped>
	@color: #415FFB;
	@background:rgba(80, 200, 180, 0.1);
	.calendar-tz {
		width: 100%;
		height: 100%;
		background: #fff;
		position: relative;
		overflow-y: scroll;
		z-index: 9;
	}
	.calendar-tz .closeDialog {
		position: fixed;
		bottom: 50px;
		z-index: 999;
		height: 50px;
		line-height: 90px;
		width: 100%;
		text-align: center;
		background: rgba(255, 255, 255, 0.9);
	}
	
	.calendar-tz .ti {
		color: #000;
		font-size: 16px;
	}
	
	.calendar-tz .calendar-header {
		position: fixed;
		width: 100%;
		left: 0;
		z-index: 9;
		box-shadow: 0 2px 15px rgba(100, 100, 100, 0.1);
	}
	.calendar-tz .week-number {
		background: #fff;
		width: 100%;
	}
	.calendar-tz .week-number span{
		display: inline-block;
		text-align: center;
		height: 40px;
		line-height: 40px;
		width: 14.28%;
		font-size: 16px;
	}
	
	.calendar-tz .calendar-wrapper {
		position: relative;
		color: #000;
		padding-top: 15px;
	}


	.calendar-tz .calendar-title {
		position: sticky;
		position: -webkit-sticky;
		z-index: 8;
		width: 100%;
		left: 0;
		color: #000;
		text-align: center;
		font-size: 16px;
		font-weight: 400;
		line-height: 40px;
		height: 40px;
	}

	.calendar-tz .each-month {
		display: inline-block;
		width: 98%;
		margin-left: 1%;
		padding-bottom: 10px;
		font-size: 0;
		border-bottom: 1px solid #F4F4F4;
	}
	
	.calendar-tz .each-day {
		position: relative;
		display: inline-block;
		text-align: center;
		vertical-align: middle;
		width: 14.2857143%;
		font-size: 16px;
		height: 50px;
		line-height: 50px;
	}
	

	.calendar-tz .each-day div {
		vertical-align: 10px;
		display: inline-block;
		height: 28px;
		width: 28px;
		line-height: 28px;
	}
	
	.calendar-tz .disabled {
		color: #ccc !important;
	}
	
	.calendar-tz .today {
		background: #E7E7E7;
		border-radius: 4px;
	}
	
	.calendar-tz .trip-time {
		background: @color;
		color: #fff !important;
		border-radius: 4px;
	}
	
	.calendar-tz .weekend {
		color: @color;
	}
	
	.calendar-tz .jia,.calendar-tz .recent {
		position: absolute;
		line-height: 12px;
		color: @color;
	}
	
	.calendar-tz .recent {
		font-size: 10px;
		width: 100%;
		text-align: center;
		bottom: 4px;
		left: 0;
	}
	
	.calendar-tz .festival {
		font-size: 14px;
	}
	
	
</style>
