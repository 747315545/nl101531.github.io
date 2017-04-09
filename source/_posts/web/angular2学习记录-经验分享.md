---
title: angular2学习记录-经验分享
tags:
  - angular
categories: web
date: 2017-04-08 23:00:00

---


# angular2学习记录-经验分享

标签（空格分隔）： web

---

### 1.前言

前几天刚下定决心把毕业设计改造下,因为毕业设计算是我学习的基石,学习到的东西都尽可能的在这个平台上施展,锻炼自己.改造为前后端分离,前端使用angular2,后端只提供接口.便于以后的维护.那么就要学习agular2了.

**这里就要说下个人观点了,安利一波**:我认为每个程序员都应该有自己的一个项目,一个可以让你学习的东西能施展到上面的项目,可能该项目一开始很简单,但是随着你不断的学习,不断的把新知识运用进去,这个项目就会伴随着你的成长而丰富起来,给你带来的则是更多的实战经验.

### 2.angular2简介
1. angular2是类似全家桶组合的框架,所需要的东西几乎都包办了,所以开发起来很迅速.
2. 使用TypeScript作为开发语言,对于Java和C#程序员可以快速上手,还有就是我比较喜欢强类型语言,每个变量各司其职,由其的类型来限定,开发人员也很明确知道变量的作用.
3. google和Microsoft支持
4. WebStorm对angular2的强大支持.
5. 一篇安利文章http://www.infoq.com/cn/articles/why-choose-angular2/

>一些学习资料
ECMAScript 6入门  http://es6.ruanyifeng.com/
TypeScript入门   http://www.imooc.com/learn/763
TypeScript中文网  https://www.tslang.cn/docs/tutorial.html
慕课网1小时快速上手视频  http://www.imooc.com/learn/789
官方文档  https://www.angular.cn/docs/ts/latest/cli-quickstart.html


### 3.遇到的问题

#### 3.1滚动监听
要实现页面滚动后导航栏会变色的效果,如下图(图来自我的csdn博客,没找到其他好图床)
![图来自我的csdn博客](http://img.blog.csdn.net/20170408234307620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMjcwNjgxMQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

之前使用Jq是
``` javaScript
$(window).scroll(function () {
    indexApp.scrollBar = parseInt(document.body.scrollTop||document.documentElement.scrollTop);
});
```
不打算依赖Jq,搜了点资料发现了下面两种写法.
```
//下面这种写法在TS下不会有效果.
  isAddBackColor(){
    if (this.getIsIndex()){
       var self = this;
       //该处使用匿名函数,而不是箭头函数.
      window.addEventListener('scroll',function () {
        let marginTop = document.body.scrollTop|| document.documentElement.scrollTop;
        self.isBackColor = marginTop > 20 && self.getIsIndex();
      });
    }
  }
```


```javaScript
/**
   * 判断是否需要加背景色(有效果的)
   * 使用isBackColor控制结果
   */
  isAddBackColor(){
    if (this.getIsIndex()){
      //监听事件使用箭头函数,这样ng2才会管理该变量
      window.addEventListener('scroll',() => {
        let marginTop = document.body.scrollTop|| document.documentElement.scrollTop;
        this.isBackColor = marginTop > 20 && this.getIsIndex();
      });
    }
  }
  
```
原因不明,猜想是`var self = this;`赋值操作后相当于一个全新的变量,self并不受angular管理,导致刷新的变量是self中的isBackColor.

#### 3.2http参数传递
按照下面代码传参数应该是没有问题的,但是我遇到了url被编码问题,例如输入`1111@qq.com`会被转换为`1111%40qq.com`,导致服务端解析失败,找了很多原因才发现是`URLSearchParams`这个对象用错了,angular2提供了这个对象,es6里面也有一个该对象,换成ng2中对象即可,`import {URLSearchParams} from "@angular/http";
`
```
    let urlParams = new URLSearchParams();
    urlParams.set('search',search);
    urlParams.set('order',order);
    urlParams.set('pageNum',pageNum.toString());
    urlParams.set('pageSize',pageSize.toString());
    return this.http.get(Config.url_problem_stage + stage,{params:urlParams}).toPromise()
              .then(response => response.json())
              .catch(LogService.handleError)
```


#### 3.3跨域问题
浏览器要求同源下才可请求,否则就产生跨域问题.

|URL|说明|是否允许通信|
|-----|-----|-----|
|http://www.a.com/a.js<br>http://www.a.com/b.js | 同一域名下 | 允许 |
|http://www.a.com/lab/a.js <br>http://www.a.com/script/b.js	|同一域名下不同文件夹	|允许|
|http://www.a.com:8000/a.js <br>http://www.a.com/b.js | 同一域名，不同端口 |不允许|
|http://www.a.com/a.js <br>https://www.a.com/b.js | 同一域名，不同协议 | 不允许|
|http://www.a.com/a.js <br>http://70.32.92.74/b.js |域名和域名对应ip |不允许|
|http://www.a.com/a.js <br>http://script.a.com/b.js |主域相同，子域不同|不允许|
|http://www.a.com/a.js <br>http://a.com/b.js |同一域名，不同二级域名（同上）| 不允许（cookie这种情况下也不允许访问）|
|http://www.cnblogs.com/a.js <br>http://www.a.com/b.js |不同域名 |不允许 |


解决方案是用nginx反向代理到不同端口,模拟同一域名下不同文件夹情况.nginx监听本地888端口,这个也是项目入口,对于带api标识的请求转到后端服务器,对于其他请求则到前端服务器.
```
    server {
        listen       8888;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location /api {
            proxy_pass   http://127.0.0.1:8080;
        }
        location / {
            proxy_pass   http://127.0.0.1:4200;
        }
    }
```

#### 3.4路由问题
angular2的路由匹配规则是从根路由也就是`forRoot()`的这个开始.在该处匹配寻找规则.

**根路由:**
```javaScript
export const appRoutes: Routes = [
  {
    path:'',
    component: IndexComponent,
    pathMatch:'full'
  },
  {
    path:'aust',
    loadChildren:'./content/content.module#ContentAndAsideModule'
  },
  {
    path:'index',
    component: IndexComponent,
  },
  {
    path:'**',
    loadChildren:'./content/content.module#ContentAndAsideModule'
  },
];

```
**子路由:**
```
export const childRouter : Routes = [
  {
    path: '',
    component:ContentAndAsideComponent,
    children:[
      {path:'',redirectTo:'/index',pathMatch:'full'},
      {path:'start',component:StartComponent},
    ]
  }
  ];
```
**举例:**
访问`/`,则先在根路由寻找,找到其跳转到IndexComponent,完成任务
访问`/aust`.则先在根路由找,发现需要到子路由里面寻找,到子路由后,在children中发现被重定向到`/index`,那么回到根路由,找到IndexComponent完成任务.
访问`/aust/start`,则先在根路由找,发现需要到子路由,到子路由匹配到StartComponent,完成任务.

#### 3.5组件通信
父->子:子组件使用input装饰器,接受父组件的属性,并且可使用ngOnChanges或则setter监听变化,做额外处理.
子->父:使用output装饰器加EventEmitter向上弹出事件到父组件,父组件监听后处理.
任意组件:使用service通讯,service提供Observable的next发布,其他组件引用service对象subscribe该发布,那么就实现了信息的流动,并且是在只要订阅了该发布的组件中都能获取.

#### 3.6单例?
agular2的service是providers提供的,该组件如果引用了这个service,那么会先在自己的providers中寻找service,找不到则再向上找父组件,直到module.那么意味着不同module中的service并不是共享的.