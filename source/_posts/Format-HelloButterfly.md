---
title: Hello World
tags:
 - Butterfly
description: Front Matter范式及一些备注
date: 2023-05-30
top_img: https://qiuhanblog-imgsubmit.oss-cn-beijing.aliyuncs.com/img/background3.jpg
swiper_index: 1
---

## Hello Butterfly

```yaml
title: #标题
date: #创建日期
tags: #标签
 - tag1
 - tag2
categories: #种类
keywords: '#关键词'
description: #封面处的描述
top_img: #文章顶部图片
cover: #封面图片
swiper_index: 1 #置顶轮播图顺序，非负整数，数字越大越靠前 （目前暂时还没有开）
katex: true #此页是否使用katex渲染数学公式
```



## 主页双行显示

博客的config文件（不是主题的）中的buterfly_article_double_row



## 代码自动换行

主题配置文件中的code_word_wrap



## 副标题打字效果

主题配置文件中的subtitle



# 折叠框使用

{% tabs test1, 2%}
<!-- tab -->
**This is Tab 1.**
<!-- endtab -->

<!-- tab -->
**This is Tab 2.**
<!-- endtab -->

<!-- tab -->
**This is Tab 3.**
<!-- endtab -->
{% endtabs %}
