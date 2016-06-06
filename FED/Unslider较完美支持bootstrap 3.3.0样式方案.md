---
title: Unslider较完美支持bootstrap 3.3.0样式方案
tags: [css]
date: 2015/05/21
---

效果图：

![效果图](http://limeng.u.qiniudn.com/wp_220_01.png)

css样式：

```
    .banner {
      position: relative;
      overflow: auto;
      padding: 0;
    }
    .banner li {
      list-style: none;
    }
    .banner * {
      -webkit-margin-before: 0;
      -webkit-margin-after: 0;
      -webkit-margin-start: 0;
      -webkit-margin-end: 0;
      -webkit-padding-start: 0;
    }
    .banner ul li {
      float: left;
      height: 248px;
    }
    .banner .dots{
      position: absolute;
      bottom: 0;
      width: 100%;
      text-align: center;
    }
    .banner .dot{
      display: inline-block;
      width: 24px;
      height: 24px;
      border-radius: 12px;
      background-color: #ccc;
      line-height: 24px;
      text-align: center;
      margin: 10px;
    }
    .banner .dot.active{
      background-color: #333;
      color: #fff;
    }
```
