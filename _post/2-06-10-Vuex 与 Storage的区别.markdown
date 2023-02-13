---
layout:  post
title:   Vuex 与 Storage的区别
date:   2-06-10 16:56:28 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-javascript
-前端
-缓存

---


Vuex 与 Storage的区别有几个：

- （1）从存储的位置来说，Vuex 用的是内存，而 Storage 用的是文件存储； 
- （2）从易失性来说，Vuex与页面的生存周期相同（如关闭页面、刷新等数据就会消失），而 localStorage是“永久”存储的，sessionStorage 生存于应用会话期间； 
- （3）从存储空间来看，Vuex取决于可用内存和浏览器的限制，Storage 都有个默认的大小（至少5MB，由浏览器决定），超出大小则需要用户同意增加空间； 
- （4）从共享来看，Vuex无法跨标签页、跨物理页面共享，则Storage则可以在同一域名底下共享； 
- （5）从用途来看，Vuex是用于管理页面内容及组件的状态，而Storage主要是用于存储数据； 
- （6）Storage是由浏览器提供的基础设施，而Vuex则是由JavaScript程序库提供的服务。 …

另外，缓存是应用级的概念，你可以根据需要，采用符合要求的存储来做缓存。因此，Vuex也可以做为缓存来使用。

同时，浏览器通常也会为应用提供缓存设施。现代浏览器都会提供几个类别的存储：

>（1）Storage: 包括 Local Storage、Session Storage、Web SQL、Index DB、Cookies等等；<br> （2）Cache: 包括 Cache Storage、Application Cache等；<br> （3）其它文件缓存，如图片、请求等；<br> 有些存府提供的服务和功能很相近，但都有一些区别和限制，可以查看相关文档来进一步了解，然后可以根据具体的需求，选择合适的存储来使用，或是性能，或是便利、或是生命周期、或是空间限制、或是隐私要求、或是兼容要求等竺，并非满足基本的数据存储就“完全够用”

