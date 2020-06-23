---
layout: post
title: "[dudu] Product Requirement"
data: 2020-06-23 21:00:00 +0800
---

之前写了一个web服务器，名字还挺可爱的，叫`dudu`。烂尾到现在，想把这个坑填上，顺便体验node.js的全栈开发流程，熟悉一下常用的工具和插件。目标是让`dudu`成为一个原型项目框架，之后如果有web开发相关的需求，例如游戏后台，可以拿过来直接开始业务的开发。


需求是实现一个极简版的微博，业务和框架解耦。
- 用户
    - 注册
    - 登录
- 用户关系
    - 关注
    - 取消关注
    - 好友列表
- 核心业务
    - 发帖
    - 删帖
    - timeline

技术方案：
- 后端 node.js + koa2
- 数据库 mysql
- 测试 mocha
- 前端 react.js
- 首选语言 typescript