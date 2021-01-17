对于设计和实现 API 来说，当结果集包含成千上万条记录时，返回一个查询的所有结果可能是一个挑战，它给服务器、客户端和网络带来了不必要的压力，于是就有了分页的功能。



通常我们通过一个 offset 偏移量或者页码来进行分页，然后通过 API 实现类似请求：

```url
GET /api/products?page=10
{"items": [...100 products]}
```



如果要继续访问后续数据，则修改分页参数即可。

```url
GET /api/products?page=11
{"items": [...another 100 products]}
```

在使用 offset 的情况下，通常使用 ?offset=1000 和 ?offset=1100 这种大家都熟悉的方法。它要么直接调用 OFFSET 1000 LIMIT 100 的 SQL 查询数据库，要么使用 LIMIT 乘以 page 作为查询参数。无论如何，**这是一个次优的解决方案**，因为无论哪种数据库都要跳过前面 offset 指定的 1000 行。而跳过额外的offset，不管是 PostgreSQL，ElasticSearch还是 MongoDB 都存在额外开销，数据库需要对它们进行排序，计数，然后将前面不用的数据扔掉。



这是一种低效的方法，但由于它使用简单，所以大家重复地用这个方法，也就是直接把 API 参数映射到数据库查询上。



那合适的方法是什么？介绍之前我们可以先看看数据库的实现。在数据库中有一个游标（cursor）的概念，它是一个指向行的指针，然后可以告诉数据库："在这个游标之后返回 100 行"。这个指令对数据库来说很容易，因为你很有可能通过一个索引字段来识别这一行。然后就不需要去取和跳过前面那些没用到的记录了。



举个例子。

```
GET /api/products
{"items": [...100 products], "cursor": "qWe"}
```



API 返回一个无业务意义的字符串（游标），你可以用它来检索下一个页面。

```url
GET /api/products?cursor=qWe
{"items": [...100 products], "cursor": "qWr"}
```



实现游标有很多方法。一般来说，可以通过一些排序字段比如产品 id 来实现。在这种情况下，你可以用一些可逆算法对产品 id 进行编码。而在接收到一个带有游标的请求时，你会对它进行解码，并生成一个类似 WHERE id > :cursor LIMIT 100 的查询。



下面是一个小小的性能对比，先看看 offset 是如何工作：

```uri
=# explain analyze select id from product offset 10000 limit 100;                                                           QUERY PLAN                                                            --------------------------------------------------------------------------------------------------------------------------------- 
Limit  (cost=1114.26..1125.40 rows=100 width=4) (actual time=39.431..39.561 rows=100 loops=1)   ->  Seq Scan on product  (cost=0.00..1274406.22 rows=11437243 width=4) (actual time=0.015..39.123 rows=10100 loops=1) Planning Time: 0.117 ms Execution Time: 39.589 ms
```



再看看 where (cursor) 语句如何工作：

```
=# explain analyze select id from product where id > 10000 limit 100;                                                          QUERY PLAN                                                          ------------------------------------------------------------------------------------------------------------------------------ Limit  (cost=0.00..11.40 rows=100 width=4) (actual time=0.016..0.067 rows=100 loops=1)   ->  Seq Scan on product  (cost=0.00..1302999.32 rows=11429082 width=4) (actual time=0.015..0.052 rows=100 loops=1)         Filter: (id > 10000) Planning Time: 0.164 ms Execution Time: 0.094 ms
```



这是几个数量级的差异! 当然，实际的差异取决于表的大小以及过滤器和存储的实现。有一篇不错的文章 (1) 提供了更多的技术信息，里面有 ppt，性能比较见第 42 张幻灯片。



(1) https://use-the-index-luke.com/no-offset



当然，用户不会按 id 来检索商品，而是会按一些相关性来查询（然后按 id 作为关联字段）。在现实世界中，需要根据你的业务来决定该怎么做。订单可以按 id 排序（因为它是单调增加的）。购买清单可以按 wishlist 时间排序。在我们的案例中，产品来自 ElasticSearch，自然支持游标的特性。



我们可以看到的一个不足是，使用无状态的 API， 无法支持翻到“上一页”这样的功能。所以在面向用户界面中，如果有 prev/next 或者 “直接进入第10页” 这样的按钮，就没有办法绕过前面提到的 offset/limit 这种实现。但是在其他情况下，使用基于游标的分页可以极大地提高性能，特别是在真正的大表和真正的深度分页上。



英文原文：

https://solovyov.net/blog/2020/api-pagination-design/



HackerNews 评论：

https://news.ycombinator.com/item?id=25547716



HN网友 et1337：



使用游标的另一个原因是避免由于并发编辑而导致元素重复或跳过的问题，比如你使用 offset 正在第 10 页上，而有人在第 1 页上删除了一个项目，则整个列表会移动，你可能会意外跳过第 11 页上的一行数据。同样，如果有人在第 1 页上添加了一条记录而你正在第 10 页上，第 10 页中的一项也会重复显示在第 11 页上。



游标优雅地回避了这些问题。



HN 网友 chrismorgan：



有时候，你需要一个游标，这样你就可以从你刚才的地方继续前进，而不用担心新的记录进来扰乱你的分页。



有时你想要基于位置的查询，因为你明确地希望所有的东西都是位置的。



有时你想把这两种技术结合起来，例如，如果你跳到一个大的、不断变化的列表中间，然后想在刚才的位置之后检索下一批结果。



我喜欢 JMAP 最后的设计(https://tools.ietf.org/html/rfc8620#page-45)：你可以指定一个位置整数，或者一个锚 ID 和可选的 anchorOffset 整数。锚是游标的一种实现，它使用结果集中一个实体 ID，而不是一个可以嵌入其他信息（比如 coroutine 地址）的不透明类型，，它有一个明显的优点，就是可以由客户端控制。

HN 网友 vincnetas



我认为作者在使用 OFFSET 时忽略了一些关键点。至少 postgres 文档对此有明确的的说法（https://www.postgresql.org/docs/13/queries-limit.html）



> When using LIMIT, it is important to use an ORDER BY clause that constrains the result rows into a unique order. Otherwise you will get an unpredictable subset of the query's rows. You might be asking for the tenth through twentieth rows, but tenth through twentieth in what ordering?



看起来作者提供的分页查询没有考虑到排序，这意味着第 100 页上的项目的 ID 大于 10000，但顺序未定义。

> explain analyze select id from product where id > 10000 limit 100



HN 网友 boulos



鉴于对“游标”一词的重用感到困惑，我更喜欢 Google 为分页所使用的术语：页面令牌和页面大小，详细可以参阅：



https://google.aip.dev/158



![图片](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOPxePRn526libhCD2hQcWdCLbex0gOtnEY1BhHESertJVMDxOdP9AdqamIr4TbBNHtq1ThbwvcvmPA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)





**参考阅读：**



- [Rust 和 C 性能对比：排序](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554384&idx=1&sn=892e4c588949c5c4f27e546cb5ead428&chksm=81399748b64e1e5e5c429d8464c521c2d9883020538a7fdcc9daf91031c83cd406057cfeaee0&scene=21#wechat_redirect)
- [白话高并发中的协程](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554383&idx=1&sn=0379e7a7c4c0fb467a652168b7fcc46f&chksm=81399757b64e1e415aae9800868fe9348d607c3fb4d3a9ae5ae523b8c246795ad144603f1dd1&scene=21#wechat_redirect)
- [聊聊 Service 命名与设计](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554354&idx=1&sn=11d89714ebf643aab4bbe04cfa03907c&chksm=8139972ab64e1e3cae8cffee0dc88207aff5a8124f1e091f3992500edce22820018754caa411&scene=21#wechat_redirect)
- [一个有关tcp的非常有意思的问题](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554348&idx=1&sn=e7849a4362b859bc4f928d2a8726bcd0&chksm=81399734b64e1e227ad94ecc89e838c093374fe5435a87f624d305991efbffb7750ed1c5a2b0&scene=21#wechat_redirect)
- [Dubbo 3.0 前瞻系列：服务发现支持百万集群，带来可伸缩微服务架构](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554342&idx=1&sn=ab7184beca87daf68b75e1573023a234&chksm=8139973eb64e1e282f3d8b751b3130fc124720a24876f0ae61b2270b4cd1ce0e60701cbc0d37&scene=21#wechat_redirect)
- [如何设计优雅的API](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653549140&idx=1&sn=04e5f06390d3f6c63e81d8cdc8ec655f&chksm=813a63ccb64deada854d98c37608001602f97274fd0cac8caa264d76268ca775d3c3e559d7e7&scene=21#wechat_redirect)