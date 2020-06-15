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
  
  
  
- 配置路由方式