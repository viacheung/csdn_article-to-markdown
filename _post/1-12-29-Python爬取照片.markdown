---
layout:  post
title:   Python爬取照片
date:   1-12-29 22:17:13 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-python
-开发语言
-后端

---


具体分析可以查看我的这篇 [文章](https://blog.csdn.net/qq_45774645/article/details/118071499)

```java
import urllib.request
from bs4 import BeautifulSoup
import re
import requests

# 影片图片的链接
findImgSrc=re.compile(r'<img alt=".*?".src="(.*?)"/>',re.S)    # re.S忽略换行符


def getData(baseurl):
    html=askURL(baseurl)    #一个页面的html
    # 解析每一个html
    data = []  # 保存每一个电影的信息
    soup=BeautifulSoup(html,'html.parser')
    for item in soup.findAll('img'):

        item=str(item)
        imgSrc=re.findall(findImgSrc,item)     #添加图片
        data.append(imgSrc)

    return data


# 得到一个指定的url网页内容
def askURL(url):
    head={
     # 模拟成浏览器，伪装，反扒机制
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36'
    }
    request=urllib.request.Request(url,headers=head)
    html=""
    try:
        response=urllib.request.urlopen(request)
        html=response.read().decode('utf-8')
    except urllib.error.URLError as e:
        if hasattr(e,'code'):
            print(e.code)
        if hasattr(e,'reason'):
            print(e.reason)
    return html



# 保存信息
def saveDate(datalist):

    for item in datalist:
        file_name=str(item[0]).split('/')[-1]
        response=requests.get(item[0])
        with open(file_name,'wb') as f:
            f.write(response.content)

def main():

    # 1.爬取网页
    datalist=getData("https://www.bizhizu.cn/nvsheng/qcka/")
    # 2.保存路径
    # 3.保存数据
    saveDate(datalist)

main()
```

