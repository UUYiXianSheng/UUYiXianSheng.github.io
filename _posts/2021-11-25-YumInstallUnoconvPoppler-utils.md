---
layout: post
title: linux实现xlsx文件转txt文件，实现在线浏览
tags: xlsx
---

### 介绍

背景：由于生产服务器服务器是命令行模式的，无法直接预览Windows上传的xlsx文件，影响排查系统问题。


1.安装unoconv

```     
$ yum install unoconv –y
```    

执行命令安装unoconv，unoconv是一个强大的办公文件，图片文件，数据表文件转换的命令。

![](/images/blog/yumUnoconv.jpg)


2.测试unoconv

```     
$ unoconv -f pdf 1.doc
```    

![](/images/blog/unoconvfpdf.jpg)

当能看见1.pdf文件的时候，说明linux预览功能环境已搭建好。

注意：在第一次运行转换命令的时候可能会报一些错误，比如：Error: Unable to connect or start own listener. Aborting.这些错误可以忽略，因为第一次转换识别不了，第二次运行就OK了。


3.安装poppler-utils

```     
$ yum install poppler-utils
```    

执行命令安装poppler-utils，他可以把pdf文件转为txt文件，这样就可以在线阅读了。


4.将pdf转换成txt

```     
$ pdftotext fail_wei-xin-invite-10.pdf fail.txt
```    

![](/images/blog/pdftotextPdfTxt.jpg)

