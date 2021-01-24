# uni-app  练手项目的一些笔记

## 1.Vue过滤器

### 1.1使用局部过滤器

**HTML代码**

```html
<view class="bottom-text">
    <!-- 在此处使用过滤器 默认把 | 左边的数据作为参数传给过滤器-->
	<view class="time">发表时间：{{item.add_time | timeFilter}}</view>
	<view class="num">浏览次数：{{item.click}}</view>
</view>
```

**js代码**

```js
	//自定义过滤器
		filters:{
            //过滤器名 传过来的参数
			timeFilter(myDate){
				const date=new Date(myDate)
				let year=date.getFullYear()
				let month=(date.getMonth()+1).toString().padStart(2,0)
				//padStart() ES2017新增方法 对字符串进行填充
				let Day=date.getDate().toString().padStart(2,0)
				let Hour=date.getHours().toString().padStart(2,0)
				let Min=date.getMinutes().toString().padStart(2,0)
				let Second=date.getSeconds().toString().padStart(2,0)
				return year+"-"+month+"-"+Day+" "+Hour+":"+Min+":"+Second
			}
		}
```

### 1.2使用全局过滤器

**在main.js文件中配置**

```js
Vue.filter("timeFilter",myDate=>{
	const date=new Date(myDate)
	let year=date.getFullYear()
	let month=(date.getMonth()+1).toString().padStart(2,0)
	//padStart() ES2017新增方法 对字符串进行填充
	let Day=date.getDate().toString().padStart(2,0)
	let Hour=date.getHours().toString().padStart(2,0)
	let Min=date.getMinutes().toString().padStart(2,0)
	let Second=date.getSeconds().toString().padStart(2,0)
	return year+"-"+month+"-"+Day+" "+Hour+":"+Min+":"+Second
})
```



## 2.字符串填充方法 padStart()

> ES2017 引入了字符串补全长度的功能。如果某个字符串不够指定长度，会在头部或尾部补全。`padStart()`用于头部补全，`padEnd()`用于尾部补全。

**此处对日期格式进行格式化，获取天数 只有一位数时前面补0**

**注意：这是字符串方法，不要漏掉toString()**

```js
let Day=date.getDate().toString().padStart(2,0)
```



## 附件：

### 1.[项目接口文档](https://www.yuque.com/suanmeitang-lsuno/interface/eagkm0)

### 2.[项目材料(含项目服务器)](https://github.com/ITaylorfan/uniapp-HeiMaShop-materials)

