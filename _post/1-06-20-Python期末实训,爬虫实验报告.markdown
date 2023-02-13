---
layout:  post
title:   Python期末实训,爬虫实验报告
date:   1-06-20 16:37:19 发布
author:  'zhangtao'
header-img: 'img/post-bg-2015.jpg'
catalog:   false
tags:
-python
-python
-正则表达式
-爬虫

---



>版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接和本声明。<br> 本文链接：https://blog.csdn.net/qq_45774645/article/details/118071499<br> 简介：Python中有很多第三方库，可以快速处理网页，我们使用四个库来爬取网页，并且保存数据

```java
import urllib.request      # 获取整个网页内容
from bs4 import BeautifulSoup	# 解析网页结构
import re						# 使用正则匹配数据
import xlwt						# 保存到Excel中
```

## 第一步根据URL获取整个网页

​ 爬虫实际上就是程序模拟人类点击网页的过程。但是有些网站禁止爬取，他们是怎么做到禁止爬取的呢?首先就要知道，服务器是怎么区分是人或者是程序浏览网页的呢？

​ 实际上浏览器再像服务器发送请求时，我们的请求信息被封装在http对象当中，这个对象里面就有区分是人或者是程序的代码，在哪能找到这个信息呢？

以豆瓣top250为例：


![img](https://img-blog.csdnimg.cn/20210620163335949.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)


![img](https://img-blog.csdnimg.cn/20210620163415322.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)

这个信息就是人为访问的。


![img](https://img-blog.csdnimg.cn/20210620163522109.png)

当我们使用Python程序访问的时候，我们的身份直接就被识破了，好家伙，这也太实诚了。

那如何伪装成浏览器呢？

```java
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
```

## 第二步解析数据

```java
def getData(baseurl):
    datalist=[]
    for i in range(0,10):   #调用获取页面信息的函数
        url =baseurl+str(i*25)
        html=askURL(url)    #一个页面的html
        # 解析每一个html
        soup=BeautifulSoup(html,'html.parser')
        for item in soup.findAll('div',class_='item'):
            #print(item)
            data=[] # 保存每一个电影的信息
            item=str(item)
            # 影片详情的超链接
            link=re.findall(findLink,item)[0]  #re库用来通过正则表达式查找指定字符串
            data.append(link)                   #添加链接

            imgSrc=re.findall(findImgSrc,item)[0]      #添加图片
            data.append(imgSrc)

            titles=re.findall(findTitle,item) # 添加片名，片名可能有一个，也可能有两个
            if len(titles)==2:
                ctitle=titles[0]          #中文名
                data.append(ctitle)
                otitle=titles[1].replace("/","")   #英文名中有无关的符号，去除
                data.append(otitle)         #英文名
            else:
                data.append(titles[0])
                data.append(' ')    #留空 ，因为要放到数据库

            rating=re.findall(findRating,item)[0]
            data.append(rating)                         #评分

            judgeNum=re.findall(findJudge,item)[0]
            data.append(judgeNum)                   #添加评价人数

            inq=re.findall(findInq,item)
            if len(inq)!=0:
                inq=inq[0].replace("。","")
                data.append(inq)                        #添加概述
            else:
                data.append("无概述")

            bd=re.findall(findBd,item)[0]
            bd=re.sub('<br(\s+)?/>(\s+)?',' ',bd)  #去掉<br/>
            bd=re.sub('/',' ',bd)    #去掉/
            data.append(bd.strip())    #去掉前后空格

            datalist.append(data)    #将一个电影的信息放到datalist
    return datalist
```

## 保存数据

```java
def saveDate(datalist,savepath):
    book=xlwt.Workbook(encoding='utf-8',style_compression=0)        #创建Workbook对象
    sheet=book.add_sheet('豆瓣电影Top250',cell_overwrite_ok=True)   #创建工作表
    col=('电影详情链接','图片链接','影片中文片名','影片外国名','评分','评价数','概况','相关信息')
    for i in range(0,8):
        sheet.write(0,i,col[i])   # 列名
    for i in range(0,250):
        data=datalist[i]
        for j in range(0,8):
            sheet.write(i+1,j,data[j])    #写入数据

    book.save(savepath)       # 保存
```

## 完整代码

```java
import urllib.request
from bs4 import BeautifulSoup
import re
import xlwt



# 影片详情链接的规则
findLink=re.compile(r'<a href="(.*?)">') # 创建正则表达式对象，表示规则
# 影片图片的链接
findImgSrc=re.compile(r'<img.*src="(.*?)"',re.S)    # re.S忽略换行符
# 影片片名
findTitle=re.compile(r'<span class="title">(.*)</span>')
# 影片评分
findRating=re.compile(r'<span class="rating_num" property="v:average">(.*)</span>')
# 评价人数
findJudge=re.compile(r'<span>(\d*)人评价</span>')
# 概况
findInq=re.compile(r'<span class="inq">(.*)</span>')
# 找到影片相关内容
findBd=re.compile(r'<p class="">(.*?)</p>',re.S)



def getData(baseurl):
    datalist=[]
    for i in range(0,10):   #调用获取页面信息的函数
        url =baseurl+str(i*25)
        html=askURL(url)    #一个页面的html
        # 解析每一个html
        soup=BeautifulSoup(html,'html.parser')
        for item in soup.findAll('div',class_='item'):
            #print(item)
            data=[] # 保存每一个电影的信息
            item=str(item)
            # 影片详情的超链接
            link=re.findall(findLink,item)[0]  #re库用来通过正则表达式查找指定字符串
            data.append(link)                   #添加链接

            imgSrc=re.findall(findImgSrc,item)[0]      #添加图片
            data.append(imgSrc)

            titles=re.findall(findTitle,item) # 添加片名，片名可能有一个，也可能有两个
            if len(titles)==2:
                ctitle=titles[0]          #中文名
                data.append(ctitle)
                otitle=titles[1].replace("/","")   #英文名中有无关的符号，去除
                data.append(otitle)         #英文名
            else:
                data.append(titles[0])
                data.append(' ')    #留空 ，因为要放到数据库

            rating=re.findall(findRating,item)[0]
            data.append(rating)                         #评分

            judgeNum=re.findall(findJudge,item)[0]
            data.append(judgeNum)                   #添加评价人数

            inq=re.findall(findInq,item)
            if len(inq)!=0:
                inq=inq[0].replace("。","")
                data.append(inq)                        #添加概述
            else:
                data.append("无概述")

            bd=re.findall(findBd,item)[0]
            bd=re.sub('<br(\s+)?/>(\s+)?',' ',bd)  #去掉<br/>
            bd=re.sub('/',' ',bd)    #去掉/
            data.append(bd.strip())    #去掉前后空格

            datalist.append(data)    #将一个电影的信息放到datalist
    return datalist



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



# # askURL("https://www.bilibili.com/")

# 保存信息
def saveDate(datalist,savepath):
    book=xlwt.Workbook(encoding='utf-8',style_compression=0)        #创建Workbook对象
    sheet=book.add_sheet('豆瓣电影Top250',cell_overwrite_ok=True)   #创建工作表
    col=('电影详情链接','图片链接','影片中文片名','影片外国名','评分','评价数','概况','相关信息')
    for i in range(0,8):
        sheet.write(0,i,col[i])   # 列名
    for i in range(0,250):
        data=datalist[i]
        for j in range(0,8):
            sheet.write(i+1,j,data[j])    #写入数据

    book.save(savepath)       # 保存

def main():
    baseurl="https://movie.douban.com/top250?start="
    # 1.爬取网页
    datalist=getData("https://movie.douban.com/top250?start=")
    # 2.保存路径
    savepath="D://桌面//豆瓣电影Top250.xls"
    # 3.保存数据
    saveDate(datalist,savepath)
main()
```

```java
baseurl="https://movie.douban.com/top250?start="
# 1.爬取网页
datalist=getData("https://movie.douban.com/top250?start=")
# 2.保存路径
savepath="D://桌面//豆瓣电影Top250.xls"
# 3.保存数据
saveDate(datalist,savepath)
```


爬取结果：![img](https://img-blog.csdnimg.cn/20210625095309894.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1Nzc0NjQ1,size_16,color_FFFFFF,t_70)

