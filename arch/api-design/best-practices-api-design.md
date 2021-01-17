

## API设计最佳实践

良好设计的API = 快乐的程序员 😃。



应用程序接口（API）是一种接口，它让应用程序可以轻松地使用另一个应用程序的数据和资源，API 对于一个产品或公司的成功至关重要。



如果没有 API，你大部分喜欢的软件今天就不会存在。例如，Google Maps API 可以让你在 app 或 Web 应用中使用 Google Maps。如果没有它，你将不得不设计和开发自己的地图数据库。这样的话，在地图上显示一个位置需要花费多少时间？



## **为什么要使用 API？**



1. API 可以让外部应用访问您的资源
2. API 扩展了应用程序的功能
3. API 允许开发者重用应用逻辑
4. API 是独立于平台的，它们传递数据不受请求平台的影响



![图片](https://mmbiz.qpic.cn/mmbiz_png/8XkvNnTiapOOEicFRTjZewWAJTeuAIUULUEyx2Nmursg95kIS2HLgHR5GeYaibuoNgIphfgRCfDt8XOrCnqDRt9aw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



在大多数实际场景中，数据模型 已经存在，但由于我们将讨论 API 设计最佳实践，我将从头开始说起。



## **数据建模与结构化**

以 API 为中心对您的数据进行建模，是设计易于创建、维护和更新 API 的第一步

在设计 API 时，尽量考虑使用通用的术语，而不是使用内部的复杂业务术语，因为这些术语在公司外可能不为人所知。你的 API 可能会对外开放，以允许外部开发人员使用你的 API 开发他们自己的应用。通过使用通用术语，你可以确保使用 API 的开发人员易于了解你的 API，并能快速上手。



假设到你正在建立一个门户网站，让用户点评不同作者的书籍。你的公司可能会使用特定的术语，如创作者、创作、系列等来指代图书作者、书籍和系列。但为了简单起见，并方便外部应用开发者使用你的 API，使用通用的概念而不是公司特定的术语来创建 API 路径。



```url
https://api.domain.com/authors 
https://api.domain.com/authors/{id}/books
```



这有助于新的开发人员快速了解你的 API 是什么，以及如何遍历你的数据模型。



## **编写面向资源的 API**

应用程序需要访问你的资源。维护一个资源层次结构可以帮助你更好地构建 API。资源层次结构是指路径中的每个节点，它由一个集合或一个资源组成。



资源可以是一个单一的数据，例如，上面例子中的作者简介。



集合是指一个资源的集合，在我们的例子中，它可以是一个作者所写的书的列表。



合适的资源层次结构可以是：

```
Base Path -> 作者 (集合) -> profile (资源)
Base Path -> 作者 (集合) -> 书 (集合) -> 书 (资源)
```



层次结构需要保持一致，以确保开发人员在将其应用程序接入 API 时遇到的问题最少。



为了保持简单性和一致性，这里有一些指导原则可以帮助你：

1. 命名集合和资源时使用美式英语（例如：color 而不是 colour）
2. 避免拼写错误
3. 使用更简单、更常用的词来保持清晰，例如 delete 而不是 remove
4. 如果你使用的资源与其他 API 使用的资源相同，请使用相同的术语以保持一致。
5. 对集合使用复数形式（例如：authors、books 等）。



## **RESTful 接口**

HTTP 形式的 API 最广泛接受的标准是 REST(Representational State Transfer)。它基本上意味着每个 URL 代表一个对象。



API 目的可以是以下之一：

1. 创建数据 Create
2. 读取数据 Read
3. 更新数据 Update
4. 删除数据 Delete



CRUD！猜对了!

API 通过使用一组 HTTP 命令来处理，这些命令定义了请求的性质和它应该做什么。

GET 从 API 中检索数据。它要求从 API 中获取数据的表示。GET请求可以包含查询参数，以过滤从API接收的结果。

POST 向 API 提交一条记录，该记录将在数据库中创建一个资源。

PUT 一般用于更新服务器上的现有资源。

DELETE 从服务器上删除一个资源。



## **API 版本控制**

应用程序和 API 的生命周期越长，应用和 API 对用户的承诺就越大。在某个时间点上，你的 API 将需要修改，因为你无法预见随着需求和业务政策而发生的变化。

因此需要对 API 进行更改。但是 API 可能已经有一个或多个开发者在使用了，所以，重要的是，你所做的更改不会破坏你的合作伙伴开发者的应用。



**了解主要和次要更新**



**小版本升级（Minor）**：当变更不会破坏客户端应用程序的运行时，可以使用小版本升级，例如添加可选字段或支持附加参数。这时候你可以为你的 API 增设小版本。



**大版本升级（Major）**：是那些肯定会破坏现有客户端应用的版本，比如在请求参数中添加一个新的必需参数，或改变返回结果中的字段。



可以通过多种方式来对 API 进行版本控制。



最常见的方法是将版本包含在 URI 中。

```
https://api.domain.com/v1.0/authors
```



另外一种方法是使用基于日期的版本控制。URI 中包括将版本发布日期。应用程序开发人员可以很方便了解 API 更改的频率。

```
https://api.domain.com/2020-06-15/authors
```



另一种方法是在请求标头中包含 API 版本。

```
https://api.domain.com/authors 
x-api-version：v1
```



最推荐和接受的版本控制方式是，在URI 中使用版本名称。



## **分页**

在数据量越来越大的世界里，不可能在一个屏幕上同时显示所有的数据。所以，让用户在再次请求数据之前，先取到一定数量的结果，这一点很重要。这就是所谓的分页，返回的数据集叫做页面。



建议你在请求和返回结果中使用特定的术语来启用 API 中的分页功能。这些术语有

1. STRING page_token（在请求中发送）
2. STRING next_page_token（由 API 返回）
3. INT page_size（在请求中发送）



page_token 请求 API 需要返回哪个页面。这通常是一个字符串。对于第一次API调用，page_token = "1"

page_size 定义了返回结果中应该返回多少条记录。例如page_size = 100，在API调用中最多返回100条记录。

next_page_token 定义了翻页的下一个 token。如果在page_token = "1" 之后有额外的数据，返回的值是应当是 next_page_token="2"



如果没有更多的数据可用，而且用户已经到达数据的终点，则返回一个空白值 next_page_token="" 。



这些就是设计 API 的最佳实践。它让你的 API 更健壮、简洁并易于与其他应用程序集成。



请记住。



**良好设计的API = 快乐的程序员 😃。**



英文原文：

https://codeburst.io/best-practices-api-design-61d4697d17ff



**参考阅读：**



- [为什么下一个十年的主战场在Serverless](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554526&idx=1&sn=a0f48a6e6e27604e8bd9e558c2e08689&chksm=813996c6b64e1fd0473816e95af0f3881193cae5565c22bba5af8a502e47f8f47c21dc6add8b&scene=21#wechat_redirect)
- [为什么我们不用数据库生成 ID？](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554513&idx=1&sn=3a9c720b5bc8b73174183420d9e21e0d&chksm=813996c9b64e1fdfdf26c48a14eb5fff97253a666966da3c05a33b2a25e1d98c7f31cd7117c4&scene=21#wechat_redirect)
- [C++服务编译耗时优化原理及实践](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554506&idx=1&sn=8a1a7f167e04eef7e06d740b0f7dceb9&chksm=813996d2b64e1fc406ad9a0b7eb852b01285dc5ab4a6382dc7b8b01aacadfeb6aa505a7f842e&scene=21#wechat_redirect)
- [基于Consul服务注册中心：一次故障分析及优化](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554462&idx=1&sn=5b63eeddcf3db77369f56c986179a1cc&chksm=81399686b64e1f904a5b54735fb9d3d01c79d647a4914560bc045bc76f139b44c78cb377cb56&scene=21#wechat_redirect)
- [API 分页设计与实现](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653554394&idx=1&sn=efb33ec244e404d87d995d4c1da525d1&chksm=81399742b64e1e544f69485bceb48d1c34ecf71f099297bbcd53e8c6a5e6245bf3ba062031e3&scene=21#wechat_redirect)
- [如何设计优雅的API](http://mp.weixin.qq.com/s?__biz=MzAwMDU1MTE1OQ==&mid=2653549140&idx=1&sn=04e5f06390d3f6c63e81d8cdc8ec655f&chksm=813a63ccb64deada854d98c37608001602f97274fd0cac8caa264d76268ca775d3c3e559d7e7&scene=21#wechat_redirect)