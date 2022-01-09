---
title: 构建Web应用
date: 2021-12-19 18:51:09
tags: 
  - 网络编程
categories: Node
---
### 1. 基础功能
- 一个Web应用，主要包括请求方法判断、URL的路径解析，URL查询字符串解析，Cookie的解析，Basic认证，表单数据的解析和任意格式文件的上传处理
- http是一个无状态的协议，但是业务是需要一定的状态的，否则无法区分用户的身份
- Cookie的处理分为服务器向客户端发送Cookie，浏览器保存Cookie和每次请求浏览器都把Cookie发向服务器端三个步骤
- Cookie的格式是key=value;key=value这种字符串
- Cookie可能会影响性能，造成带宽的浪费，因此需要减少Cookie的大小。如果域名根节点设置Cookie，那么所有子路径都会发送。可以把静态资源和动态资源分开
- Session保存在服务器端，使用键值作为Session的口令
- 缓存的方式：添加Expires或者Cache-Control到报文中、配置ETag、让Ajax可缓存

### 2. 路由解析
- REST的全称是Representational State Transfer，表现层状态转化
