---
title: WEB小知识-HTTP请求对+和&的处理
author: 
  nick: 屈定
tags:
  - bug    
categories: web
date: 2017-05-22 22:18:00
updated: 2017-05-22 22:18:00
---

### 1.问题
在HTTP请求中如果传的参数有一些特殊字符则会被编码成空格,导致服务端获取不到响应的信息.
> 对于`+`号会被编码为空格
> 对于`&`也会被编码成空格

举个例子,需要向服务端提交如下代码:
```c++
#include <iostream>
using namespace std;
int main()
{
    int a,b;
    cin >> a >> b;
    cout << a+b << endl;
    return 0;
}
```
编码后的内容如下,可以发现`a+b`被转换成了`a b`导致服务端接收到后编译失败.
```java    
#include%20%3Ciostream%3E%0A%0Ausing%20namespace%20std;
%0A%0Aint%20main()%0A%7B%0A%20%20%20%20int%20a,b;
%0A%20%20%20%20cin%20%3E%3E%20a%20%3E%3E%20b;
%0A%20%20%20%20cout%20%3C%3C%20a b%20%3C%3C%20endl;
%0A%20%20%20%20return%200;%0A%7D
```
### 2.解决方案
使用函数`encodeURIComponent()`,该函数会把特殊字符都给转义,转义结果如下面所示,可见`a+b`转换成了`a%2Bb`
```java
%23include%20%3Ciostream%3E%0A%0Ausing%20namespace%20std%3B
%0A%0Aint%20main()%0A%7B%0A%20%20%20%20int%20a%2Cb%3B
%0A%20%20%20%20cin%20%3E%3E%20a%20%3E%3E%20b%3B
%0A%20%20%20%20cout%20%3C%3C%20a%2Bb%20%3C%3C%20endl%3B
%0A%20%20%20%20return%200%3B%0A%7D
```
服务端需要使用`URLDecoder`对其进行反转义,该问题到此解决.