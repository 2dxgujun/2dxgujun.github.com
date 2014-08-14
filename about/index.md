---
title: About me
layout: page
---
![dream](/media/files/2014/08/14/dream.jpg)
如果你想拥有你从未有过的东西

那么你必须去做你从未做过的事情

> 你可以在这些地方找到我：

{% for link in site.links %}
> {{link.title}}: [{{link.name}}]({{link.url}} "{{link.desc}}")
{% endfor %}