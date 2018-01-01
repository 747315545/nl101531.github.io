---
title: Angular中引入第三方JS库
subtitle: Angular中引入第三方JS库
cover: http://oobu4m7ko.bkt.clouddn.com/1514805636.png
author: 
  nick: 屈定
tags:
  - angular
categories: web
date: 2017-10-09 23:34:43
updated: 2017-10-09 23:34:43
---
最近写[http://www.itoolshub.com/](http://www.itoolshub.com/)的时候用到了日期时间选择器,Angular本身material2只有日期选择器,也不知道为什么官方不提供日期时间选择器,也可能是Angular2以及如今的4有些年轻,很多库都不是很成熟,于是乎搜索到的解决方案就是借助第三方的库来使用一些优秀的组件.本文以[https://github.com/sentsin/laydate](https://github.com/sentsin/laydate)组件为例.

### 引入js与css
[https://github.com/sentsin/laydate](https://github.com/sentsin/laydate)是采用原生js实现的组件,因此不需要考虑相关依赖,直接入手.
1.使用npm下载该组件`npm install layui-laydate -save`
2.在`.angular-cli.json`文件中配置
```json
   "styles": [
        "styles.scss",
        "../node_modules/layui-laydate/dist/theme/default/laydate.css"
      ],
      "scripts": [
        "../node_modules/layui-laydate/dist/laydate.js"
      ],
```
Angular在编译的时候会把上述的js引用都打包到`scripts.bundle.js`文件中

### ts编译识别laydate
第一步完成后如果在TS中使用laydate变量,编译器是会直接报错的,因为其找不到这个变量,因此这一步要做的就是让ts识别该变量.做法很简单,在`typings.d.ts`中加入声明
```js
/* SystemJS module definition */
declare var module: NodeModule;
interface NodeModule {
  id: string;
}
// laydate声明
declare var laydate: any;
```

### 使用laydate功能
`laydate`是需要更改Dom节点的,因此该步骤必须放到Angular对视图渲染之后,也就是生命周期中的`AfterViewInit`函数中执行.另外该渲染会使得双向绑定失效,需要处理结果则可以在`laydate`的回调函数中处理.
另外使用的时候就可以按照ts的语法来使用了,最终都会解析成原生js.比如下方的箭头函数.
```js
  ngAfterViewInit(): void {
    let done = (value, date, endDate) =>{
      let selectTime = new Date(value);
      this.timeStampOut = selectTime.getTime() / 1000;
      this.timeStampWeek = TimestampComponent.WEEKS[selectTime.getDay()] == null ? "Invalid Week": TimestampComponent.WEEKS[selectTime.getDay()]
    };
    laydate.render({
      elem: '#layerdate',
      type: 'datetime',
      change: done,
      done: done
    });
  }
```
### 备注
很多库都是直接对DOM进行操作,这对于Angular这种虚拟Dom操作会导致绑定失效等各种异常问题,一般情况下不建议混编,尤其是大项目,到后期会出现各种折磨人的小问题.

更多Angular实战代码可以参考我的开源项目:
> github: [https://github.com/nl101531/IToolsHub](https://github.com/nl101531/IToolsHub)