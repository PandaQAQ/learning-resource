---
title: Vue 路由跳转的传参方式
---

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