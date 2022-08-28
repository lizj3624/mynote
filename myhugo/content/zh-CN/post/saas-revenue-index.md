---
title: "美股SaaS公司财报中Bookings, Billings, Revenue指标详解"
date: 2022-08-28T15:21:51+08:00
tags:
   - SaaS估值
   - SaaS财务指标
   - 科技股
categories:
   - SaaS估值
   - SaaS财务指标
   - 科技股
toc: true
---

软件即服务(SaaS)行业充斥着大量的行业术语和定义。在SaaS的财务报告当中，我们经常会碰到bookings(签约合同金额)、
billings(订阅付费金额)以及收入(revenue)，这3个Top-Line财务指标密切相关却又各不相同。

为了帮助投资者了解这3个术语之间的差别和联系，我们假设一款名为"Help"的SaaS提供3款不同的套餐——Startup、Growth和Enterprise，
定价分别为每月$200、$500和$1000。以下是这些套餐年度客户订阅的一个样本数据集。
![01](./1-bookings-billings-table.png)

### SaaS的bookings（签约合同金额）
Booking是一个前瞻性指标，指的是SaaS公司与潜在客户签署的一定期限的合同金额，也就是客户承诺支付给服务提供商的资金额。
就拿上面的例子来说，客户A签署了1年期的Startup套餐合同，每月的订阅费是$200，1年下来就是$2400。
这$2400的合同金额就是SaaS提供商的"booking"。
![02](./2-bookings-billings-table.png)

上面的是1月份客户A的bookings，如果要算SaaS公司某一个月份的bookings，就要把当月签署合同的所有客户的合同金额加总。
比如说1月签约的有客户A和客户B，合同金额分别为$2400和$12000，1月合计bookings就是$14400。
![03](./3-bookings-billings-table.png)

### SaaS的billings（订阅付费金额）
相比bookings只是纸面上的收入，billings是SaaS公司实际向客户收取的资金。再拿上面的样本数据集举例。
你会注意到有的客户订阅的月套餐，有的则是年套餐。顾名思义，月套餐每月缴纳订阅费，年套餐一次性就预先就缴足了1年的订阅费。

按照这一定义，该SaaS提供商1月的bookings为$14400，其中客户A是年套餐，当月就缴足了1年$2400的订阅费。
不过，客户B订阅的是月套餐，虽然2年期合同金额高达$12000，但平摊到每个月就是$500，因此该公司1月的billings只有$2900。
![04](./4-bookings-billings-table.png)

### SaaS的revenue（收入）
虽然billing已经到手，但收入的确认要符合一套财务规则，即产品或服务已经交付给客户。就SaaS行业来说，每个月服务成功交付后，
公司才能确认当月的收入。按照美国通用会计准则（GAAP）的规定，收入一旦被"赚取"才能被确认。

回到例子中来。1月公司的bookings为$14400，其中billings为$2900，其中包括客户A缴纳$2400以及客户B 的$500。
按照GAAP的规定，客户A虽然已经支付了1年的订阅费，但实际已经提供的只是1月的服务，因此只能确认当月$200的收入。
相反客户B支付的$500是已经提供服务的费用。因此，该公司1月的收入为$700。

最后还有一个"递延收入"的概念，指的是实际已经收取但还无法确认为收入的资金额，计算公式也很简单，
就是billings减去已经确认的收入。
![05](./5-bookings-billings-table.png)
> 很多美股SaaS公司财务预测指标由"Deferred Revenue（递延收入，后称DR）"变成"Remaining Performance Obligation（剩余履约价值，后称RPO）"。
> 剩余履约价值（RPO）= 递延收入（DR）+ 未开票递延收入（UDR）

### 引用
1. [软件saas公司经营指标分析指南](https://finance.sina.com.cn/stock/stockzmt/2020-12-25/doc-iiznctke8379171.shtml)

2. [如何定义一家成熟的SaaS公司？](https://www.huxiu.com/article/357307.html)

3. [bookings，billings和收入：说说SaaS的这3个Top-Line财务指标](https://nai500.com/zh-hans/blog/2021/10/bookings-billings-%E6%94%B6%E5%85%A5-saas-top-line-%E8%B4%A2%E5%8A%A1%E6%8C%87%E6%A0%87/)
