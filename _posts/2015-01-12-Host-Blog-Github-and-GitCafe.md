---
layout: post
title: 同时使用Github和GitCafe托管博客
category: Other
date: 2015-01-12
---

我之前一直使用[Github Pages](https://pages.github.com/)和[Jekyll](http://jekyllrb.com/)来搭建个人博客，但是不久前GFW好像加强了，Github虽然能访问但是会出现丢包现象，博客的访问速度明显不如从前，经常10多秒都打不开。之前没太在意，昨天同学说他把博客放在了[GitCafe](https://gitcafe.com/)上，访问速度要比Github快很多。

<!-- more -->

可能有些人因为Github用得久了不想迁移博客，其实你不必非此即彼，Github和GitCafe可以同时使用。具体做法就是国外访客解析到Github，国内访客解析到GitCafe。

我用的是万网的域名控制台做的解析，万网可以设置海外解析线路和默认解析线路：

![DNS](/media/2015/1/12/dns.png)

把海外IP解析到Github的A记录地址，默认线路解析到GitCafe的A记录地址。

解析设置完毕之后，在GitCafe上新建自己的博客项目，具体做法可以参考[GitCafe Pages的官方搭建教程](http://blog.gitcafe.com/?p=116)。新建完之后，进入项目管理页的域名管理，我们可以找到相应的设置项，如下所示：

![Custom Domain](/media/2015/1/12/domain.png)

项目创建完后，我们需要将原本提交到Github上的博客内容同步提交到GitCafe上。

### 设置 Remote 源

我原本只有一个`origin`指向Github的项目地址，现在我要同时提交到Github和GitCafe，我把原来的`origin`改名叫`github`，再新建一个`gitcafe`指向GitCafe的项目地址.

### 修改 Rakefile

原本我更新博客都是`git commit`之后再`git push`，现在由于要同时推送到两个代码仓库，执行的命令比较多，我把他们放在Rakefile里面，只要执行`rake deploy message="xxx"`命令就能轻松更新你的博客了。

{% highlight ruby %}
#Usage: rake deploy message="Message"
desc "Publishing the website via git"
task :deploy do
  message = ENV["message"] || "Empty Message"
  system "git commit -m \"#{message}\""
  puts "Publishing to Github"
  system "git push github master"
  puts "Publishing to GitCafe"
  system "git push gitcafe master:gitcafe-pages"
  puts "Your website is now published"
end
{% endhighlight %}

因为家里比较偏远有些延迟，但是GitCafe还是要比Github的速度快了近一倍。

![Ping Github](/media/2015/1/12/ping_github.png)
![Ping GitCafe](/media/2015/1/12/ping_gitcafe.png)

<br/>
本文出自[2dxgujun](/)，转载时请注明出处及相应链接。