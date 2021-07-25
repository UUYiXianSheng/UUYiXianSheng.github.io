---
layout: post
title: 配置Docker镜像加速
tags: Docerk
---

### 介绍
由于Docker默认的仓库站点是在国外，速度很慢，这就需要配置国内的镜像仓库，解决镜像下载过慢的问题，这里我选择的是阿里的镜像加速仓库。

1.登录阿里云平台。

PS:需要注册阿里云账号，阿里云的镜像加速服务是免费的。

2.获取链接

![](/images/blog/dockerExpediteone.png)

![](/images/blog/dockerExpediteTwo.png)

3.选择合适配置方式进行配置

配置方式上面写的很详细，我这里不再阐述，注意：需要重启docker服务才能使配置生效！

![](/images/blog/dockerExpediteThree.png)