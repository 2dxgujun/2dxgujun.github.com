---
layout: default
---
<!--博客首页-->
<div>
    <!--取最近一篇文章-->
    {% for post in site.posts limit:1 %}
        <article class="content">
            <!--带超链接的文章标题-->
            <section class="title">
                <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
            </section>
            <section class="meta">
                <!--写文章的时间-->
                <span class="time">
                  <time datetime="{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}">{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}</time>
                </span>
                <!--文章的标签-->
                {% if post.tags %}
                    <span class="tags">
                        {% for tag in post.tags %}
                            <a href="/tags/#{{ tag }}" title="{{ tag }}">#{{ tag }}</a>
                        {% endfor %}
                    </span>
                {% endif %}
            </section>

            <!--正文内容-->
            <section class="post">
                {{ post.content }}
            </section>
        </article>
    {% endfor %}

    <div class="divider"></div>
    
    <!--列举最近写的10篇文章列表-->
    <ul class="listing main-listing">
        <li class="listing-seperator">Earlier...</li>
            <!--获取当期年份-->
            <!--{% capture year %}{{ site.time | date:"%Y"}}{% endcapture %}-->
            
            <!--循环遍历最近10篇文章-->
            {% for post in site.posts offset:1 limit:10 %}
                <!--获取遍历到的文章年份-->
                <!--{% capture y %}{{ post.date | date:"%Y"}}{% endcapture %}-->
                <!--如果不是今年写的就跳出遍历循环-->
                <!--{% if year != y %}-->
                <!--    {% break %}-->
                <!--{% endif %}-->
                <li class="listing-item">
                    <time datetime="{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}">{{ post.date | date:"%Y-%m-%d %H:%M:%S" }}</time>
                    <a href="{{ post.url }}" title="{{ post.title }}">{{ post.title }}</a>
                </li>
            {% endfor %}
        <li class="listing-seperator"><a href="/archive">More earlier...</a></li>
    </ul>
</div>
