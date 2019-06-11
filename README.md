vue开发，支持日历选择，支持（范围选择）酒店入住-离开，飞机往返。同时可以自定义主题色。（并兼容uniApp小程序）  
 * Calendar.vue 可以传参设置主题色（如:themeColor="'#f60'"），不依赖任何less，sass，stylus等预加载  
 * Calendar-simple.vue有固定主题色，不可以动态传参设置，但是可以通过源代码修改less主题色变量进行全局修改（依赖less）
  ![](https://file.40017.cn/tcyp/tz/calendar1.png)
  ![](https://file.40017.cn/tcyp/tz/calendar2.png)
  ![](https://file.40017.cn/tcyp/tz/calendar3.png)


### 使用方法
首先项目中安装：npm install mobile-calendar-simple -S （若使用HBuilderX导入的uniApp项目，可以忽略此步骤）
```javascript
<template>
	<div>
		<!--用法一-->
		<Calendar :date="'yyyy-mm-dd'" /> 如果默认今天可简写：<Calendar />
		<!--用法二（默认:mode="1",酒店入住模式）-->
		<Calendar :startDate="'yyyy-mm-dd'" :endDate="'yyyy-mm-dd'" />
		<!--用法三（:mode="2"）,往返模式-->
		<Calendar  :startDate="'yyyy-mm-dd'" :endDate="'yyyy-mm-dd'" :mode="2" />
		<!--设置主题色-->
		<Calendar :date="'yyyy-mm-dd'" :themeColor="#FF6600" />
		<!--如果需要solt-->
		<Calendar :date="'yyyy-mm-dd'">
			<div>
				...此处也支持slot注入（不需要可以忽略此div）
			</div>
		</Calendar>
	</div>
</template>
<script>
	import Calendar from 'mobile-calendar-simple'
	export default {
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
	 :mode：模式选择（默认1），1酒店模式，2往返模式
```diff

- 注意：date（日历模式）与startDate、endDate（酒店\往返日历模式）不能共存
- 要么 <Calendar :date="'2019-06-04'"  :themeColor="'#415FFB'" @callback="XXX" />
- 要么 <Calendar :startDate="'2019-06-06'" :endDate="'2019-06-08'" :themeColor="'#415FFB'"  @callback="XXX" />
```
### 回调函数
  *  @callback：日期选择后获取到的数据（所有你想要的都有）

***
github链接
[链接名称](https://github.com/tanagang/mobile-calendar-simple)
