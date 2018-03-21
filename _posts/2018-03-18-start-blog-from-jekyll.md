---
title: 利用jekyll搭建简单的Blog
layout: page
categories: Other
tags: blog
---

爱分享的人总是喜欢记录下自己的一些想法，一是便于自己以后总结抽象，二也是为了给遇到相同问题或者考虑相同事情的人一些建议。

记录自己想法的方法很对，但对于程序员来说，不少都喜欢借助Github的Page来建立专属页面，既不会自己利用wordpress搭建博客那么复杂，又不会像一般都博客网站那样单一，可以自己定制。

本Blog也是利用Github的Page来搭建的。

## 1. 申请Github Page
直接新建一个repository，其中Repository name就是将来默认访问博客的网址，比如下图中的leonlee11.github.io，在创建博客成功后，可以直接通过该链接访问。
![applypage](/assets/jekyll/applypage.png){:class="img-responsive"}


## 2. 安装jekyll环境
jekyll是一个静态网站生成工具，它可以按照markdown、html等文件的内容生成对应的静态网站文件。同时它也是github默认的渲染引擎，因此使用它来搭建静态网站博客非常简单。

为了能使用jekyll，需要先安装相关环境。jekyll是ruby的一个第三方插件，通过gem能很好的进行管理。为了让安装更简单，最好将ruby、gem等都升级到最新版本。

### 2.1 安装或者升级最新ruby
```
1. 下载并安装rvm
curl -sSL https://get.rvm.io | bash -s stable
source ~/.bashrc
source ~/.bash_profile

2. 修改ruby安装源到ruby china服务器，加快安装速度
echo "ruby_url=https://cache.ruby-china.org/pub/ruby" > ~/.rvm/user/db

3. 查看ruby最新版本号
rvm list known

4. 安装并使用最新版ruby
rvm install ruby-2.4
rvm use ruby-2.4
```

### 2.2 升级gem
```
1. 修改gem源到ruby china服务器
gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.org/

2. 升级gem
sudo gem update --system
sudo gem update
```

### 2.3 安装jekyll
```
gem install jekyll
注：根据安装提示，安装一些相关的第三方库；在笔者环境下还需要安装bundler、jekyll-sitemap、pygments.rb
```
使用jekyll serve就会在本地启动一个小服务，如果能在127.0.0.1:4000下访问到正常内容，则证明环境安装成功。

## 3. 下载jekyll-bootstrap主题
jekyll-bootstrap提供了一个很好的quick start：http://jekyllbootstrap.com/usage/jekyll-quick-start.html

访问上面的链接，填入自己的github username，比如上面的leonlee11，就在该页面生成相应的命令。另外注意，这个start生成的命令中默认会认为你申请的reponsitory name是以.com结尾的，如果你如笔者一样是以.io结尾，注意修改下命令。

```
git clone https://github.com/plusjade/jekyll-bootstrap.git leonlee11.github.io
cd leonlee11.github.io
git remote set-url origin git@github.com:leonlee11/leonlee11.github.io.git
git push origin master
```

## 4. 修改必要信息
默认的jekyll-bootstrap可能有写问题或者不满足需求，需要进行定制化修改。

### 4.1 修改bootstrap3目录
在笔者环境和版本下，在leon11.github.io的目录下使用jekyll serve，生成的页面竟然没有bootstrap相关的样式，分析发现，是需要使用bootstrap3/bootstrap下的css等，但是这个目录不存在，需要将bootstrap3下的css、js等文件夹拷贝到一个新到bootstrap目录下。在bootstrap3目录下：
```
mkdir bootstrap
mv -f css fonts js bootstrap
```
在查看页面就正常了。

### 4.2 定制bootstrap样式
对于默认等字体，颜色什么等不满意的话，可以在该网站定制bootstrap样式：http://getbootstrap.com/docs/3.3/customize/
比如:
```
@font-size-base: 16px
@font-size-h1: floor((@font-size-base * 1.9))
@font-size-h2: floor((@font-size-base * 1.6))
@font-size-h3: ceil((@font-size-base * 1.25))

@container-large-desktop: (940px + @grid-gutter-width)
```
OK后，点击页面最下面的"Compile and Download"，然后将得到的文件替换掉上面的bootstrap目录就可以让定义的样式生效了。

不禁感慨，现在技术环境真是好啊！

## 5. 博客设置
### 5.1 添加about.md
演示一下怎样在jekyll-bootstrap中怎样添加导航项。在根目录下创建about.md，内容如下：
```
{%raw%}
---
title: 关于
layout: page
group: navigation
---
{% include JB/setup %}

## 自我介绍
LeonLee

半吊子架构师，全栈工程师，专业打杂，偶尔背锅。。。

对Python、GO、Kubernets、分布式系统。。。感兴趣，持续研究中

对硬件系统、计算机体系结构有些许研究，做软硬件结合相关对工作

有些懒，博客开了几次，没坚持；虽然目前来看，不能著书立传，但还是下定决心在世间留点什么
{%endraw%}
```
运行jekyll serve，就可以查看到最新更新。

是的，就是这么简单！

### 5.2 第一篇文章
在_post目录下，创建2018-03-18-the-first-page.md，然后编辑相关内容。注意jekyll下，文件名是year-month-day-title.md的格式。

### 5.3 修改首页
想在首页就按照时间现实写的文章，非常简单，编辑index.html，
```
{%raw%}
---
layout: page
title : 文章
header : Post Archive
group: navigation
---
{% include JB/setup %}

{% assign posts_collate = site.posts %}
{% include JB/posts_collate %}
{%endraw%}
```

还可以有很多定制或者其他功能，后面慢慢完善吧。

大家一起玩的开心～
