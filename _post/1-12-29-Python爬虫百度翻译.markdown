---
layout:  post
title:   Python爬虫百度翻译
date:   1-12-29 22:14:09 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-python
-爬虫
-百度

---


安装下列库

```java
import urllib.request      # 获取整个网页内容
from bs4 import BeautifulSoup	# 解析网页结构
import re						# 使用正则匹配数据
import xlwt						# 保存到Excel中
```


```java
import requests
import json

if __name__ == '__main__':
    post_url="https://fanyi.baidu.com/sug"
    # post请求
    data={
   
        'kw':'happy'
    }
    #UA 伪装
    head = {
     # 模拟成浏览器，伪装，反扒机制
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36'
    }
    response=requests.post(post_url,data=data,headers=head)
    # 获取响应数据
    dic_obj=response.json()
    # 存储到文件
    fp=open('./dog.json','w',encoding='utf-8')
    json.dump(dic_obj,fp=fp,ensure_ascii=False)
```

