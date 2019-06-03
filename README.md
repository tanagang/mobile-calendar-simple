vue开发，支持日历选择，支持酒店入住-离开范围选择，同时可以自定义主题色（兼容uniApp）
下图gif可能有卡顿，[图片原地址](https://file.40017.cn/tcyp/tz/refresh2.gif)  

  ![](https://file.40017.cn/tcyp/tz/refresh2.gif)

### 使用方法
首先项目中安装：npm install mobile-calendar-simple -S
<template>
	<div>
		<Calendar></Calendar>
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

```diff
- 注意：date（日历模式）与startDate、endDate（酒店模式）不能共存
- 要么 <Calendar :date="'2019-06-04'"  :themeColor="'#415FFB'" @callback="XXX" ></Calendar>
- 要么 <Calendar :startDate="'2019-06-06'" :endDate="'2019-06-08'" :themeColor="'#415FFB'"  @callback="XXX"></Calendar>
```
### 参数如下
  *  :date：传入初始日期（默认当天）
  *  :startDate：酒店模式的入住日期
  *  :endDate：酒店模式的离开日期
  *  :themeColor：日历的主题色，例:themeColor="#FF6600"(默认#415FFB)
 
### 回调函数
  *  @callback：日期选择后获取到的数据（所有你想要的都有）

***
github链接
[链接名称](https://github.com/tanagang/mobile-calendar-simple)
