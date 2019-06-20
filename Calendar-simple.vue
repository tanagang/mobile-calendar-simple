<template>
	<div class="calendar">
		<slot name="top"></slot>
		<div class="calendar-header">
			<div class="week-number">
				<span v-for="item in weekList" v-text="item" :key="item"></span>
			</div>
		</div>
		<div class="ti">
			<div class="calendar-wrapper" v-for="(item,index) in calendar" :key="index">
				<div class="calendar-title" v-if="lang=='cn'">{{item.year}} 年 {{item.month}} 月</div>
				<div class="calendar-title" v-else>{{monthEn[item.month-1]}} {{item.year}}</div>
				<!--如果普通日期选择-->
				<ul class="each-month" v-if="date||(!date&&!startDate&&!endDate)">
					<li class="each-day" v-for="(day,idx) in item.dayList" :key="idx" @click="chooseDate(day, item.month, item.year)">
						<div :class="[addClassName(day, item.month, item.year)]">
							{{!!day?day:''}}
						</div>
						<span class="recent">{{setTip(day, item.month, item.year)}}</span>
					</li>
				</ul>
				<!--如果酒店入住离开选择-->
				<ul class="each-month" v-else>
					<li class="each-day" v-for="(day,idx) in item.dayList" :key="idx"  :class="[addClassName2(day, item.month, item.year)]"
					 @click="chooseDate(day, item.month, item.year)">
						<div :class="[addClassName(day, item.month, item.year),{'clicktime': isCurrent(day, item.month, item.year)}]">
							{{!!day?day:''}}
						</div>
						<span class="recent" v-text="setTip(day, item.month, item.year)"></span>
					</li>
				</ul>
			</div>
		</div>
		<slot></slot>
	</div>
</template>

<script>
	let {keys,values,entries} = Object
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
			mode: { //模式（默认1），1酒店，2飞机往返 
				type: [String, Number],
				default () {
					return '1'
				}
			},
			preDisabled: { //小于初始的日期的全部disabled置灰
				type: [String, Boolean],
				default () {
					return false
				}
			},
			lang: { 
				type: [String],
				default () {
					return "cn"
				}
			}
		},
		data() {
			return {
				endDates: '',
				startDates: '',
				dates: '',
				isDate: false, //是否是普通日历模式
				betweenDate: '', //显示日历的时间段
				weekList: ['日', '一', '二', '三', '四', '五', '六'],
				monthEn:['January','February','March','April','May','June','July','August','September','October','November'],
				calendar: [],
				festival: {
					'2019':{
						"2019/6/1": "儿童节",
						"2019/6/7": "端午",
						"2019/7/1": "建党节",
						"2019/8/1": "建军节",
						"2019/8/7": "七夕",
						"2019/9/10": "教师节",
						"2019/9/13": "中秋",
						"2019/10/1": "国庆",
						"2019/10/28": "重阳",
						"2019/10/22": "感恩节",
						"2019/12/24": "平安夜",
						"2019/12/25": "圣诞",
						"2020/1/1": "元旦"
					},
					'2020':{
						"2020/1/1": "元旦",
						"2020/1/17": "小年",
						"2020/1/24": "除夕",
						"2020/1/25": "春节",
						"2020/2/8": "元宵",
						"2020/2/14": "情人节",
						"2020/3/8": "妇女节",
						"2020/3/12": "植树节",
						"2020/4/1": "愚人节",
						"2020/4/4": "清明节",
						"2020/5/1": "劳动节",
						"2020/5/10": "母亲节",
						"2020/6/1": "儿童节",
						"2020/6/21": "父亲节",
						"2020/6/25": "端午节",
						"2020/7/1": "建党节",
						"2020/8/1": "建军节",
						"2020/8/25": "七夕",
						"2020/9/10": "教师节",
						"2020/10/1": "国庆中秋",
						"2020/10/25": "重阳节",
						"2020/11/26": "感恩节",
						"2020/12/24": "平安夜",
						"2020/12/25": "圣诞节",
						"2021/1/1": "元旦"
					}
				}
			}
		},
		mounted() {
			this.init()
			//查看今年是否设置节假日
			this.festivalNew= entries(this.festival).find((item,index)=>{
				return item[index]==this.year
			})
		},
		methods: {
			init() {
				if (this.date) {
					//disableDate用于addClassName方法preDisabled==true的时候使用
					this.dates = this.disableDate = new Date(this.date.replace(/-/g, '/'))
					this.isDate = true
				}
				if (this.startDate) {
					this.startDates = this.disableStartDate = new Date(this.startDate.replace(/-/g, '/'))
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
					this.startDates =  this.disableStartDate = new Date(this.today * 1)
				}
				if (!this.date && !this.startDate && !this.endDate) {
					this.dates = new Date(this.today * 1)
					this.isDate = true
				}
				
				//最后可以选择的日期范围
				this.lastDate = this.today +  180 * 24 * 3600 * 1000
				if (this.betweenDate === '') {
					if(this.isDate){
						this.betweenDate = new Date(this.dates * 1 + 180 * 24 * 3600 * 1000)
					}else if(!this.isDate){
						this.betweenDate = new Date(this.startDates * 1 + 180 * 24 * 3600 * 1000)
					}else{
						//默认结束日期为180天
						this.betweenDate = new Date(this.today * 1 + 180 * 24 * 3600 * 1000)
					}
				}
				
				this.year = new Date().getFullYear();
				this.month = new Date().getMonth() + 1;
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
				const endY = this.betweenDate.getFullYear(),
					endM = this.betweenDate.getMonth() + 1,
					interval = (endY - this.year) * 12 + endM - this.month;
				for (let i = 0; i < interval; i++) {
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
			//添加日历样式
			addClassName(day, month, year) {
				if (!day) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day)
				let className = []
				if (_date.getDay() === 0 || _date.getDay() === 6) { //周末或周六样式
					className.push('weekend')
				}
				if (_date * 1 === this.today) {
					className.push('today');
				}

				if (_date * 1 < this.today|| _date*1 > this.lastDate) { //当天之前和180天之后不可选) { //当天之前不可选
					className.push('disabled')
				} else if (_date * 1 === this.dates * 1) {
					className.push(' clicktime');
				}
				//preDisabled==true时设置小于disableDate的都disable
				if(this.preDisabled&&this.isDate&&_date*1 < this.disableDate*1){
					className.push('disabled')
				}
				if(this.preDisabled&&!this.isDate&&_date*1 < this.disableStartDate*1){
					className.push('disabled')
				}
				return className.join(' ');
			},
			//日期范围选择背景色
			addClassName2(day, month, year) {
				if (!day) {
					return;
				}
				const _date = new Date(year + '/' + month + '/' + day) * 1
				let className = []
				if (_date >= this.startDates * 1 && _date <= this.endDates * 1) {
					return 'between'
				}
			},
			//清除时间 时 分 秒 毫秒
			resetTime(date) {
				date.setHours(0);
				date.setMinutes(0);
				date.setSeconds(0);
				date.setMilliseconds(0);
				return date;
			},
			//设置今天，明天，后天
			setTip(day, month, year) {
				if (!day) {
					return;
				}
				const td = year + '/' + month + '/' + day
				const _date = new Date(td) * 1
				const lang = this.lang
				
				let tip;
				
				//设置节假日
				if(!!this.festivalNew&&lang=="cn"){// && (_date >= this.today && _date <= this.lastDate) 180范围外是否显示节假日
					tip = this.festivalNew[1][year + "/" + month + "/" + day]
				}
				
				if (_date == this.today) {
					tip = lang =='cn' ? '今天' : 'Today'
				} else if (_date - this.today === 24 * 3600 * 1000) {
					tip = lang =='cn' ? '明天' : 'TMR'
				} else if (_date - this.today === 2 * 24 * 3600 * 1000) {
					tip = lang =='cn' ? '后天' :''
				}
				
				if (!this.date && (this.startDate || this.endDate)) {
					if (_date === this.startDates * 1) {
						if(this.mode==2){
							if(this.endDates*1==0){
								tip = lang =='cn' ? '去/返' : 'Go/Back'
							}else{
								tip = lang =='cn' ? '去程' : 'Go'
							}	
						}else{
							tip = lang =='cn' ? '入住' : 'Into'
						}	
						
					} else if (_date === this.endDates * 1) {
						if(this.mode==2){
							tip = lang =='cn' ? '返程' :  'Back'
						}else{
							tip = lang =='cn' ? '离开' : 'Leave'
						}
					}
				}
				
				return tip;
			},
			isCurrent(day, month, year) {
				if (!day) {
					return false;
				}

				const _date = new Date(year + '/' + month + '/' + day) * 1

				if (_date === this.startDates * 1 || (_date === this.endDates * 1)) {
					return true
				}
			},
			dateFormat(times) {
				let weekList = ['日', '一', '二', '三', '四', '五', '六'];
				let date = new Date(times);
				return {
					year: date.getFullYear(),
					month: parseInt(date.getMonth() + 1) > 9 ? parseInt(date.getMonth() + 1) : '0' + parseInt(date.getMonth() + 1),
					day: date.getDate() > 9 ? date.getDate() : '0' + date.getDate(),
					week: weekList[date.getDay()]
				}
			},
			chooseDate(day, month, year) {
				if (!day) {
					return;
				}
				
				const _date = new Date(year + '/' + month + '/' + day) * 1
				
				//超出180天范围之前和之后disable灰色的区域不可点击
				if (_date < this.today|| _date > this.lastDate) {
					return;
				}
				//如果设置preDisabled==true，小于disableDate的日期都不能点击
				if(this.preDisabled&&this.isDate&&_date*1 < this.disableDate*1){
					return;
				}
				if(this.preDisabled&&!this.isDate&&_date*1 < this.disableStartDate*1){
					return;
				}
				
				if (_date == this.today || this.dates * 1) {
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
				
				const dateChoose = this.dateFormat(this.dates)
				const choose = {
					dateTime: this.dates * 1,
					date: dateChoose,
					dateStr: dateChoose.year + "-" + dateChoose.month + "-" + dateChoose.day,
					recent: ''
				}
				
				const startDateChoose = this.dateFormat(this.startDates)
				const endDateChoose = this.dateFormat(this.endDates)
				const startDateStr = startDateChoose.year + "-" + startDateChoose.month + "-" + startDateChoose.day
				const endDateStr = endDateChoose.year + "-" + endDateChoose.month + "-" + endDateChoose.day
				const choose2 = {
					startDateTime: this.startDates,
					endDateTime: this.endDates,
					startDate: startDateChoose,
					endDate: endDateChoose,
					startDateStr: startDateStr,
					endDateStr: endDateStr,
					startRecent: '',
					endRecent: ''
				}
				
				if (this.isDate) { //普通模式的recent
					if (_date == this.today) {
						choose.recent = '今天'
					} else if (_date - this.today == 24 * 3600 * 1000) {
						choose.recent = '明天'
					} else if (_date - this.today == 2 * 24 * 3600 * 1000) {
						choose.recent = '后天'
					}
				} else { //酒店和往返模式的recent
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
				} else { 
					choose2.countDays = (this.endDates * 1 - this.startDates * 1) / 86400 / 1000;
					if(this.mode==2){//往返模式
						if (this.startDates&&!this.endDates) {//单日往返
							choose2.endDate = choose2.startDate
							choose2.endDateStr = choose2.startDateStr
							choose2.endDateTime = choose2.startDateTime
							choose2.endRecent = choose2.startRecent
							this.$emit("callback", choose2)
						}else if(this.startDates){//去程-返程
							this.$emit("callback", choose2)
						}
					}else{//酒店模式
						if (this.startDates && this.endDates) {
							this.$emit("callback", choose2)
						}
					}	
				}
			}
		}
	}
</script>

<style lang="less" scoped>
	@color: #415FFB;
	@background: rgba(80, 200, 180, 0.1);

	div,ul,li,p,span,i,b,a {
		margin: 0;
		padding: 0;
		font-size:14px;
	}

	.calendar {
		width: 100%;
		height: 100%;
		background: #fff;
		position: relative;
		z-index: 9;

		&:-webkit-scrollbar {
			display: none
		}
		.ti {
			font-size: 16px;
			padding-top:44px;
		}
		.calendar-header {
			position: fixed;
			width: 100%;
			left: 0;
			z-index: 9;
			box-shadow: 0 2px 15px rgba(100, 100, 100, 0.1);
			.week-number {
				background: #fff;
				width: 100%;
				span {
					display: inline-block;
					text-align: center;
					height: 40px;
					line-height: 40px;
					width: 14.28%;
					font-size: 16px;

					&:first-child,
					&:last-child {
						color: @color;
					}
				}
			}
		}

		.calendar-wrapper {
			color: #333;
			padding-top: 10px;
			.calendar-title {
				width: 100%;
				color: #333;
				text-align: center;
				font-size: 16px;
				font-weight: 400;
				line-height: 40px;
				height: 40px;
			}

			.each-month {
				display: inline-block;
				width: 98%;
				margin-left: 1%;
				padding-bottom: 10px;
				font-size: 0;
				border-bottom: 1px solid #F4F4F4;

				.each-day {
					position: relative;
					display: inline-block;
					text-align: center;
					vertical-align: middle;
					width: 14.28%;
					font-size: 16px;
					height: 50px;
					line-height: 50px;

					&.between {
						background: @background;
					}

					div {
						vertical-align: 8px;
						display: inline-block;
						height: 28px;
						width: 28px;
						line-height: 28px;
						&.weekend {
							color: @color;
						}
						&.today {
							background: #E7E7E7;
							border-radius: 4px;
						}
						&.clicktime {
							background: @color;
							color: #fff;
							border-radius: 4px;
						}
						&.disabled {
							color: #ccc;
							~.recent{
								color:#ccc;
							}
						}
					}
					.recent {
						position: absolute;
						line-height: 12px;
						color: @color;
						font-size: 10px;
						width: 100%;
						text-align: center;
						bottom: 4px;
						left: 0;
					}
				}
			}
		}
	}
</style>
