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

github链接
[链接名称](https://github.com/tanagang/mobile-calendar-simple)


### 使用方法
若在vue-cli项目中安装：npm install mobile-calendar-simple -S （若使用uniapp的工具HBuilderX导入，可以忽略此步骤）
```javascript
<template>
	<div>
		<!--用法一-->
		<calendar @callback="getDate" /> 
		<!--用法二（mode="2",酒店入住模式,mode="3",往返模式）start-date开始日期，end-date结束日期-->
		<calendar :start-date="startDate" :end-date="endDate" mode="2"  @callback="getDate" />
		<!--用法三,可以操作的日期范围-->
		<calendar  :between-start="startDate" :between-end="endDate"  @callback="getDate" />
		<!--设置主题色-->
		<calendar :themeColor="'#FF6600'"  @callback="getDate" />
		<!--如果需要solt-->
		<calendar @callback="getDate">
			<div>
				...此处也支持slot注入（不需要可以忽略此div）
			</div>
		</calendar>
	</div>
</template>
<script>
	import Calendar from '@/components/mobile-calendar-simple/Calendar.vue'
	export default {
		data(){
			return {//日期均为yyyy-mm-dd或者yyyy/mm/dd格式
				startDate:'',
				endDate:'',
				betweenStart:'',
				betweenEnd:'',
			}
		},
		methods:{
			//获取回调的日期数据
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
  *  start-date（默认当天）
  *  end-date 当mode=2、3的模式下为酒店\往返模式的离开日期
  *  between-start 和 between-end 可以操作的日期范围
  *  theme-color 日历的主题色，例 '#FF6600'或者简写'#f60'(默认#415FFB)
  *  mode 模式选择（默认1标准模式），2酒店模式，3往返模式
  *  title 日历顶部的tips提示
  *  init-month 初始月份数（默认6个月）最小1个月


### 回调函数
  *  @callback：日期选择后获取到的数据（所有你想要的都有）
***


