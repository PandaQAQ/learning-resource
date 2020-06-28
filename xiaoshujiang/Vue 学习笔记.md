---
title: Vue 学习笔记
---

# 路由跳转传递参数
- query 方式传递参数
```javascript
	//跳转
    goDetail (article) {
      this.$router.push({
        path: '/index/article',
        query: {
          link: article.link,
          title: article.title
        }
      })
    },
```

```javascript
	//接收
	mounted () {
    	this.params = this.$route.query
  	}
```
这种方式传递参数有个问题，参数将会被拼接到 url 中，如果不想传递的参数直接显示在 url 中，可以使用后面的两种方式

- params 方式传递参数
  
```javascript
	//发送
    goDetail (article) {
      this.$router.push({
        path: '/index/article',
        name:'ArticleDetail',
        params: {
          link: article.link,
          title: article.title
        }
      })
    },
```
```javascript
	//接收
	mounted () {
    	this.params = this.$route.params
  	}
```
 使用上述方式路由跳转时传递参数，不会将参数拼接到 url 中，外部不可见参数内容。需要注意 `name:'ArticleDetail',`一定要加上目标路由页面的 name ，否则在目标页面将无法接收到参数
 
 # 页面数据缓存
 H5 页面每次跳转都会重走创建页面的生命周期，如果在构造页面时去获取数据则会每次返回页面都重新拉取数据。为了避免这个问题，可以临时保存数据也可以将整个页面缓存下来，浏览器回退时不去重新渲染加载。
 
 ## localStorage 保存页面临时数据
 类似于 Android 中的 SharePreference 存值
```javascript
	
	export default {
		data(){
			return {
				announce:Object
			}
		},
		created () { //获取数据
	 		 getData() 
		},
		methods{
			getData(){
				let condition = localStorage.getItem('announce')
				if (condition == null) { 
					let ann = asyncData //服务器获取数据
        			localStorage.setItem('announce', JSON.stringify(ann))
        			this.announce = ann
      			}else{ // 使用缓存数据
					this.announce = JSON.parse(condition)
				}
			}
		},
		destroyed () { // 页面从路由栈中清除销毁时移除临时缓存数据
    		localStorage.removeItem('announce')
  		}
	｝
```

 ## keepAlive 缓存页面
 `localStorage`方式能保留数据不去重新加载，但路由返回当前页面时仍然会重新走生命周期渲染页面。如果想保存数据的同时返回界面不刷新页面组件可使用 keepAlive 包裹页面，达到整个页面状态保留
 
 ```html
     <div v-bind:class="showTab?'app-content':'app-content-full'">
      <transition>
        <keep-alive include="Index,Explore,Announce"> // 使用 keep-alive 包裹显示的内容
          <router-view class="router-view"/>
        </keep-alive>
      </transition>
    </div>
 ```
详情查看： [官方文档](https://cn.vuejs.org/v2/api/#keep-alive)
- 使用`keep-alive`后触发的组件将增加两个生命周期函数 `activated ` 和 `deactivated `分部在离开和进入组件时触发
- `include` 和 `exclude` 其中 `include` 通过组件名指定缓存的组件，`exclude` 通过组件名排除不缓存的组件 
- `max` 指定缓存组件最大的个数，超过缓存数后将不再缓存新的组件
 
 
 ## 保留页面滚动位置
 使用 `keepAlive` 发现从详情页返回列表也虽然未重新请求数据，created 方法也没有触发，即页面没有完全重新创建渲染。但是列表的滚动位置没有得到包留，滚到中间部位的列表重新滚到了顶部。为了解决这个问题，我们可以在 `keepAlive` 的基础上去做滚动状态的恢复：
 - 1、路由离开时保存滚动位置到 `data`，`keepAlive` 会保留 `data` 中的数据
 - 2、路由返回时触发 `actived` 方法，在此方法中从 `data` 恢复组件的滚动位置

```javascript
data () {
    return {
      scrollTop: 0, // 顶部滚动的位置
      mescroll: null, // 滚动组件
      dataList: []
    }
  },
  .
  .
  .
methods:{
	goDetail (item) {
      this.scrollTop = this.mescroll.getScrollTop()  // 将滚动的值赋值给 data 中的 scrollTop
      this.$router.push({ // 路由跳转到其他页面
        path: '/index/announcedetail',
        name: 'AnnounceDetail',
        params: {
          announce: item
        }
      })
    }
}
	.
	.
	.
activated () {
	// 返回缓存页触发 activated 钩子函数将列表滚动到之前的位置
    this.mescroll.scrollTo(this.scrollTop,0)
}
```