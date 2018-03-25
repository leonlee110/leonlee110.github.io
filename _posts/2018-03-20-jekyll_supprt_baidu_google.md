---
title: 网站增加百度&Google支持
layout: page
categories: Other
tags: blog
---

增加一些辅助功能，能让我们的博客更有价值。比如，
- 百度或者google的搜录能带来更多的流量
- 百度或者google的统计能告诉我们博客的压力

因此本文介绍怎样在博客中增加这些支持。

<!-- excerpt -->

## 1. Google搜录
### 1.1 添加Google收录
Google的搜录比较简单，在其console网站上提交相关信息，即可实现比较好的搜录。在这个网站中添加上博客地址：https://www.google.com/webmasters/tools/home?hl=zh-CN

然后在这个页面手动触发抓取操作：
![googlesoulu](/assets/jekyll/google_soulu.png){:class="img-responsive"}

过一会后，就可以在google中直接搜索"leonlee110.github.io"了，它会显示你的网站主页。

另外，为了让他搜录所有文章，一般会提交一个网站map文件，便于Google解析收录。

### 1.2 jekyll-sitemap
如果没有安装sitemap插件，可以通过如下命令安装：
```
gem install jekyll-sitemap
```

然后在_config.yaml中添加：
```
gems: ["jekyll-sitemap"]
```

如果想某些文章不被收录，只需要在相应md文件中，增加```gems: ["jekyll-sitemap"]```即可。

### 1.3 提交sitemap
在Google如下页面提交sitemap文件（比如https://leonlee110.github.io/sitemap.xml）即可。
![googlesitemap](/assets/jekyll/google_sitemap.png){:class="img-responsive"}

## 2. Google统计
直接使用jekyll-bootstrap模版的好处就是很多功能已经原生很好支持了，所以这也是前面文章推荐直接用jekyll-bootstrap的原因。

为了支持Google统计网站的一些访问信息，首先需要先注册账号获取相关的trace id。进行该页面操作即可：https://analytics.google.com/analytics。

注册成功后会在如下页面（管理->媒体资源设置）获取对应的跟踪id：
![googleanalytics](/assets/jekyll/google_analytics.png){:class="img-responsive"}

然后将_config.yml中如下代码修改成上述ID：
```
analytics :
  provider : google
  gauges :
      site_id : 'SITE ID'
  google :
      tracking_id : '这里这里填上trace id'
```
真实太容易了～

## 3. 百度收录
其实本来百度的收录和上面Google的收录方法是一样的，添加博客地址，上传sitemap，但是触发抓取的时候发现失败了，点击失败原因发现是拒绝的。
![baiduerror](/assets/jekyll/baiduerror.png){:class="img-responsive"}

网上搜索发现是百度不能抓取github page，但是可以通过一些geek的方式解决。有需要的小伙伴，参考这个知乎的内容吧：https://www.zhihu.com/question/30898326。

比较麻烦，以后有具体尝试了，再更新吧。
