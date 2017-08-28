技术博客及内容站点的推荐系统设计
===

过去的两周里，我一直忙于为 『[玩点什么](https://www.wandianshenme.com/)』 设计一个推荐系统（即，recommend system）。在这个过程中，参考了之前的 几本书籍，查找了一系列的资料。想着这些资料上，大部分都是大同小民的，实现了几个简单的推荐功能，改进了标签推荐算法，便想着写篇文章记录一下。

『[玩点什么](https://www.wandianshenme.com/)』，是一个基于 Django、Python 的 CMS 系统（Mezzanine）。是的，和我的博客使用的是同一个 CMS 系统。由于使用的是 Python 语言，因此对于机器学习具有天生的优势。

**推荐系统**

> 推荐系统是一种信息过滤系统，用于预测用户对物品的“评分”或“偏好”。

对于推荐系统系统来说，目前采用的主要方式是：

 - 基于内容推荐：内容之间的相似度，如文章的标签、电影的属性、书籍的分类。
 - 协同过滤（待实现）：用户之间的相似度，如喜欢看科幻片的 A、B 用户，A 喜欢看的 c 电影，B 也可能喜欢。

要实现这两种方式有一个前提是，拥有的数据。特别是协同过滤，需要有大量的用户行为数据。

而对于一个 CMS 系统来说，还有两种简单的方式：

 - 基于统计学推荐，诸如文章的阅读量、分享量，又或者文章的评分数。
 - 基于标签推荐，对于专业领域的文章来说，作者提交的标签往往比机器生成更加可靠。

因此，让我们先从这两种简单的方式讲起。

基于访问量推荐
---

在很多内容网站上，一般都会把流量比较大的

 - 博客的访问量
 - 用户的点评数据


基于 评分及 IMDB 加权算法推荐
---


https://www.biaodianfu.com/imdb-rank.html

贝叶斯统计的算法得出的加权分(Weighted Rank-WR)，公式如下：

 WR， 加权得分（weighted rating）。
 R，该电影的用户投票的平均得分（Rating）。
 v，该电影的投票人数（votes）。
 m，排名前 250 名的电影的最低投票数（现在为 3000）。
 C， 所有电影的平均得分（现在为6.9）。
 
```
from math import sqrt

def confidence(ups, downs):
    n = ups + downs

    if n == 0:
        return 0

    z = 1.0 #1.44 = 85%, 1.96 = 95%
    phat = float(ups) / n
    return ((phat + z*z/(2*n) - z * sqrt((phat*(1-phat)+z*z/(4*n))/n))/(1+z*z/n))
```


基于文章标签过滤
---

id  | xx | keywords | votes | rating_sum | rating_average | published | title
----|----|----------|-------|------------|----------------|-----------|-------
"323 " | "0 " | "tech tools " | "45 " | "165 " | "3.66666666666667 " | "1 " | "每个程序员必知之:程序员差别的本质"
"63 " | "0 " | "programmer resume latex " | "24 " | "66 " | "2.75 " | "1 " | "程序员该如何去写自己的简历(草稿）"
"207 " | "0 " | "beageek geek anywherehtml " | "20 " | "79 " | "3.95 " | "1 " | "be a geek 1:无处不在的html"
"38 " | "0 " | "iot osiot laravel ajax RESTful " | "19 " | "84 " | "4.42105263157895 " | "1 " | "一个最小的物联网系统设计方案及源码"
"361 " | "0 " | "write type writer programmer " | "17 " | "65 " | "3.82352941176471 " | "1 " | "编程同写作，写代码只是在码字"
"259 " | "0 " | "gitbook " | "12 " | "51 " | "4.25 " | "1 " | "gitbook 制作书籍"
"37 " | "0 " | "iot ajax laravel RESTful serial " | "11 " | "40 " | "3.63636363636364 " | "1 " | "最小物联网系统（一）——系统组成"
"391 " | "0 " | "javascript anonymous encapsulation " | "11 " | "50 " | "4.54545454545455 " | "1 " | "Javascript 匿名函数与封装"
"512 " | "0 " | " " | "9 " | "36 " | "4.0 " | "1 " | "如何通过github提升自己"
"508 " | "0 " | "emacs vim github " | "8 " | "40 " | "5.0 " | "1 " | "努力只是因为想去做想做的事"
"548 " | "0 " | "full stack mustache django rework " | "8 " | "35 " | "4.375 " | "1 " | "全栈工程师的思考"

1. 获取文章的所有标签
2. 对所有文章的标签进行统计，计数
3. 获取文章标签中计数最多的 tag，查找相同标签的博客
4. 在剩余的博客中，选择第二多 tag，再过滤剩余的博客

```
keywords_name = model.get_keywordsfield_name()
assigned = getattr(model, keywords_name).all()
all_keywords = Keyword.objects.filter(assignments__in=assigned)
keywords = all_keywords.annotate(item_count=Count("assignments")).order_by('-item_count')

# TODO: filter most popular tag
first_keyword = keywords.first()
if first_keyword:
    first_filtered_blogposts = BlogPost.objects.published().filter(keywords__keyword__title__contains=first_keyword.title)
    first_filtered_blogposts = first_filtered_blogposts.filter(~Q(id=post_id))
    second_keyword = keywords[1]

    if second_keyword:
        blog_posts = first_filtered_blogposts.filter(keywords__keyword__title__contains=second_keyword.title)
        return blog_posts[:3]
    else:
        return []
```                

https://stackoverflow.com/questions/10029588/python-implementation-of-the-wilson-score-interval

添加基于 Google 搜索的标签权重
---

单一的关键词，只对于网站本身是有价值的，对于用户来说，则不是如此。

基于 Google Search Console 的关键词权重，比如

Queries | Clicks | Impressions | CTR | Position
--------|--------|------------|------|------
homebridge-miio | 7 | 28 | 25% | 8.2
home assistant broadlink | 4 | 10 | 40% | 15
amazon echo raspberry pi | 3 | 10 | 30% | 5.0
raspberry pi homebridge | 2 | 6 | 33.33% | 7.7
raspberry pi alexa gpio | 2 | 4 | 50% | 10
nodemcu homekit | 2 | 3 | 66.67% | 13
arduino homekit | 1 | 3 | 33.33% | 9.7


### 相关性搜索

用户搜索 Raspberry Pi，那么它可能还会结合 Arduino ??

### 标签权重排

这也就是上面算法中的问题，假如我们的文章中，出现一系列的 home assistant、raspberry pi 相关的文章，那么它对于网站来说，表明它是没有价值的。

并且如果同一系列的文章太多，如网上各类的 Vue 高仿站点，那么用户可能已经掌握了，或者没有价值。因此，它并不能在搜索结果上体现，

在站点内，它有重要的意义，标签数量多。但是它并不能真正地解决用户的问题？也不能真正地体现出网站的价值。

假如用户搜索了一篇 raspberry pi + homebridge 的文章，那么它确实可以阅读一些相关的文章，而诸如 raspberry pi alexa gpio 从上图来看似乎是一个用户更加喜欢的选择。

示例：http://localhost:8000/play/howto-install-home-assistant-in-raspberry-pi/

这个时候由编辑推荐出来的，反而比较准确。

 - 权重
 - 加权计算法


基于内容的推荐（Content-based Recommendations）
--

1. Item Representation：为每个item抽取出一些特征（也就是item的content了）来表示此item；
2. Profile Learning：利用一个用户过去喜欢（及不喜欢）的item的特征数据，来学习出此用户的喜好特征（profile）；
3. Recommendation Generation：通过比较上一步得到的用户profile与候选item的特征，为此用户推荐一组相关性最大的item。

步骤一：文章的标签

步骤二：用户行为 

步骤三：分析用户可能性

### 特征提取

分词库：[jieba](https://github.com/fxsjy/jieba) 功能：

 - 基于前缀词典实现高效的词图扫描，生成句子中汉字所有可能成词情况所构成的有向无环图 (DAG)
 - 采用了动态规划查找最大概率路径, 找出基于词频的最大切分组合
 - 对于未登录词，采用了基于汉字成词能力的 HMM 模型，使用了 Viterbi 算法

分词功能

 - 基于 TF-IDF 算法的关键词抽取
 - 基于 TextRank 算法的关键词抽取

**TextRank**

> TextRank算法可以用来从文本中提取关键词和摘要（重要的句子）

 - 关键词提取
 - 关键短语提取
 - 摘要生成

中文库：[https://github.com/letiantian/TextRank4ZH](https://github.com/letiantian/TextRank4ZH)

Content Engine: https://github.com/groveco/content-engine

User Filtering: https://github.com/fcurella/django-recommends

**LDA**

> 隐含狄利克雷分布简称LDA(Latent Dirichlet allocation)，是一种主题模型，它可以将文档集中每篇文档的主题按照概率分布的形式给出。

### 用户喜好

**最近邻方法（k-Nearest Neighbor，简称kNN）**

**Rocchio算法**

**决策树算法（Decision Tree，简称DT）**

**线性分类算法（Linear Classifer，简称LC）**

**朴素贝叶斯算法（Naive Bayes，简称NB）**

### 生成推荐


http://blog.untrod.com/2016/06/simple-similar-products-recommendation-engine-in-python.html


协同过滤
---


### 技术实现原型

 - hitcounts
 - rating
 
 
django + celery + redis 调度/定期/afterpost ?

ajax 生成


小结
---

> 大多数情况下，对于我而言，又或者大多数程序员而言，我们只需要了解相关领域的算法（无论是机器学习、深度学习），有一个总体的认识：了解有哪些相似的算法、以及每个算法的用途、**能读懂原理**。除非自己去造轮子，要不仍然只是使用 API 而已。即便你熟悉某一个算法的原理，只要是不能深入理解，在未来也中会是使用 API。

参考内容：

 - 《驾驭文本》
 - 《推荐系统》
 - 《推荐系统：技术、评估及高效算法》
 - 《机器学习实战》

文章：[http://www.cnblogs.com/breezedeus/archive/2012/04/10/2440488.html](http://www.cnblogs.com/breezedeus/archive/2012/04/10/2440488.html)

