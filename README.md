一款结合携程、同程艺龙开发的日历(支持日期选择模式，酒店模式，往返模式,价格日历模式)，可自定义主题色，支持英文版  
 * Calendar.vue 可以传参设置主题色（如:themeColor="'#f60'"），不依赖任何less，sass，stylus等预加载  
 * Calendar-simple.vue（有默认主题色）不可以动态传参设置主题色，但是可以通过源代码less变量@color进行全局修改（依赖less）
  ![](https://file.40017.cn/tcyp/tz/calendar1.png)
  ![](https://file.40017.cn/tcyp/tz/calendar2.png)
  ![](https://file.40017.cn/tcyp/tz/calendar3.png)
  ![](https://file.40017.cn/tcyp/tz/calendar4.png)

  
  
```diff
+ 为了及时响应大家的各种功能需求，可谓不遗余力
+ 所以希望能在上面github链接里点个star，也算是鼓励一下了！
+ 同时有什么新的需求和建议可以继续联系我，我及时更新...
```

### 使用方法
首先项目中安装：npm install mobile-calendar-simple -S （若使用HBuilderX导入的uniApp项目，可以忽略此步骤）
```javascript
<template>
	<div>
		<!--用法一-->
		<Calendar :date="'yyyy-mm-dd'" @callback="getDate"/> 如果默认今天可简写：<Calendar  @callback="getDate" />
		<!--用法二（默认:mode="1",酒店入住模式）-->
		<Calendar :startDate="'yyyy-mm-dd'" :endDate="'yyyy-mm-dd'"  @callback="getDate" />
		<!--用法三（:mode="2"）,往返模式-->
		<Calendar  :startDate="'yyyy-mm-dd'" :endDate="'yyyy-mm-dd'" :mode="2"  @callback="getDate" />
		<!--用法四,价格日历模式-->
		<Calendar  :priceList="priceList"  @callback="getDate" />
		<!--用法五,不可操作的日期-->
		<Calendar  :notDateList="notDateList"  @callback="getDate" />
		<!--设置主题色-->
		<Calendar :date="'yyyy-mm-dd'" :themeColor="#FF6600"  @callback="getDate" />
		<!--如果需要solt-->
		<Calendar :date="'yyyy-mm-dd'"  @callback="getDate">
			<div>
				...此处也支持slot注入（不需要可以忽略此div）
			</div>
		</Calendar>
	</div>
</template>
<script>
	import Calendar from 'mobile-calendar-simple'
	export default {
		data(){
			return {
				notDateList:['2019-6-25','2019-6-26'],//不可操作的日期
				priceList:[//价格日历列表
					{date:'2019-6-24',price:'¥199'},
					{date:'2019-6-25',price:'¥500'},
					{date:'2019-6-26',price:'¥199'},
					{date:'2019-6-27',price:'¥500'},
					{date:'2019-6-28',price:'¥500'}
				]
			}
		},
		methods:{
			//获取选择的日期回调数据
			getDate(date){
				console.log(date)
			}
		},
		components:{
			Calendar
		}
	}
</script>
```
### 参数如下
  *  :date：传入初始日期（默认当天）
  *  :startDate：酒店\往返模式的入住日期
  *  :endDate：酒店\往返模式的离开日期
  *  :themeColor：日历的主题色，例:themeColor="#FF6600"(默认#415FFB)  
  *  :mode：模式选择（默认1），1酒店模式，2往返模式
  *  :preDisabled:默认（false），当设置为true时，所有小于初始日期（date和startDate）都disabled置灰
  *  :lang(默认cn)，值包含中文版cn和英文版en
  *  :initMonthCount 要初始多少个月份（默认6个月）最小1个月
  *	 :allAbled="true"  解除日期范围外的disabled限制（比如今天之前的日期为disabled），即所有日期均都可操作
  *	 :priceList='priceList' 添加价格日历列表
  *	 :notDateList="notDateList" 设置不可点击的日期
  *  :switchMonth="true" 当设置此值时，可以手动切换月份（上一月，下一月）  
  

```diff

- 注意：date（日历模式）与startDate、endDate（酒店\往返日历模式）尽量不要共存
- 要么 <Calendar :date="'2019-06-04'"  :themeColor="'#415FFB'" @callback="XXX" />
- 要么 <Calendar :startDate="'2019-06-06'" :endDate="'2019-06-08'" :themeColor="'#415FFB'"  @callback="XXX" />
```
### 回调函数
  *  @callback：日期选择后获取到的数据（所有你想要的都有）

***
github链接
[链接名称](https://github.com/tanagang/mobile-calendar-simple)
