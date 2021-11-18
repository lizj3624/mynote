- [Comparing Ingress controllers for Kubernetes](#comparing-ingress-controllers-for-kubernetes)
  - [**Criteria**](#criteria)
    - [**Supported protocols**](#supported-protocols)
    - [Based on (underlying software)](#based-on-underlying-software)
    - [**Traffic routing**](#traffic-routing)
    - [**Namespace limitations**](#namespace-limitations)
    - [**Upstream probes**](#upstream-probes)
    - [**Load balancing algorithms**](#load-balancing-algorithms)
    - [**Authentication**](#authentication)
    - [**Traffic distribution**](#traffic-distribution)
    - [**Paid subscription**](#paid-subscription)
    - [**Graphical user interface (Web UI)**](#graphical-user-interface-web-ui)
    - [**JWT validation**](#jwt-validation)
    - [**Customization of configuration**](#customization-of-configuration)
    - [**Basic DDOS protection mechanisms**](#basic-ddos-protection-mechanisms)
    - [**Requests tracing**](#requests-tracing)
    - [WAF](#waf)
  - [**Ingress Controllers**](#ingress-controllers)
    - [**Kubernetes Ingress Controller**](#kubernetes-ingress-controller)
    - [**NGINX Ingress Controller**](#nginx-ingress-controller)
    - [**Kong Ingress**](#kong-ingress)
    - [Traefik](#traefik)
    - [HAProxy Ingress](#haproxy-ingress)
    - [**Voyager**](#voyager)
    - [**Contour**](#contour)
    - [**Istio Ingress**](#istio-ingress)
    - [**Ambassador**](#ambassador)
    - [Gloo](#gloo)
    - [Skipper](#skipper)
    - [Other notes](#other-notes)
  - [**Comparison table**](#comparison-table)
  - [**Summary**](#summary)
  - [P.S. Other articles/reviews to consider](#ps-other-articlesreviews-to-consider)
  - [*Afterword*](#afterword)
  - [Related posts:](#related-posts)
# Comparing Ingress controllers for Kubernetes

- Kubernetes Ingress：即 Kubernetes 推荐默认使用的 Nginx Ingress。它的主要优点为简单、易接入。缺点是Nginx reload耗时长的问题根本无法解决。另外，虽然可用插件很多，但插件扩展能力非常弱。
- Istio Ingress 和 Ambassador Ingress 都是基于非常流行的 Envoy。说实话，我认为这两个 Ingress 没有什么缺点，可能唯一的缺点是他们基于 Envoy 平台，如果大家对这个平台都不是很熟悉，上手门槛会比较高。
- Kong：其本身就是一个 API 网关，它也算是开创了先河，将 API 网关引入到 Kubernetes 中当Ingress。另外相对边缘网关，Kong 在鉴权、限流、灰度部署等方面做得非常好。Kong Ingress 还有一个很大的优点：提供了一些 API、服务的定义，可以抽象成 Kubernetes 的 CRD，通过K8S Ingress 配置便可完成同步状态至 Kong 集群。缺点就是部署特别困难，另外在高可用方面，与 APISIX 相比也是相形见绌。
- APISIX Ingress：它具有非常强大的路由能力、灵活的插件拓展能力，在性能上表现也非常优秀。同时，它的缺点也非常明显，尽管APISIX开源后有非常多的功能，但是缺少落地案例，没有相关的文档指引大家如何使用这些功能。
- Traefik ：基于 Golang 的 Ingress，它本身是一个微服务网关，在 Ingress 的场景应用比较多。他的主要平台基于 Golang，自身支持的协议也非常多，总体来说是没有什么缺点。如果大家熟悉 Golang 的话，也推荐一用。
- Nginx Ingress：主要优点是在于它完全支持 TCP 和 UDP 协议，但是缺失了鉴权方式、流量调度等其他功能。
- HAproxy：是一个久负盛名的负载均衡器。它主要优点是具有非常强大的负载均衡能力，其他方面并不占优势。

Deploying a Kubernetes cluster for a specific application, you need to realize the requirements from the application itself, business and developers. Having this understanding, you can make an architectural choice and select an appropriate Ingress controller for K8s. We have prepared this review to help you to get a rough idea of what is available on the market. Hopefully, all notable, production-ready Ingress controllers are taken into consideration, so you can save your research time — at least, it might be a good starting point followed by your own practical experimentations.

Since comparing a lot of criteria for many solutions is a long journey, **any feedback correcting our information is very welcome** — especially from those who have a real experience with the products listed.

***NB\****: This comparison has been inspired by a* [*kubedex.com article*](https://kubedex.com/ingress/) *which has been lacking some actual data we need. Please check it and other useful links listed in the end of this text.*

## **Criteria**

In order to make a high-grade comparison and to get some useful results, you have to have a good understanding of the subject, but that’s not all. You also need a specific suite of criteria that would set a research vector. We do not pretend to analyze *all* possible Kubernetes Ingress/API gateways/Service Mesh use cases, but try to highlight the *most common* requirements for controllers. Studying all the specifics and particularities of each case still has to be done to succeed in your own case.

Let’s start with features that have become so common that they are **already implemented in all solutions** and therefore not considered:

- Open Source *(we just don’t want to have other options at this level of our K8s stack)*;
- dynamic service discovery;
- SSL termination;
- support for WebSockets.

Now, we’re coming to the points of comparison:

### **Supported protocols**

This is the fundamental parameter for making your choice. Will a regular HTTP(S) proxy be enough for your software? Or does it work through gRPC, HTTP/2.0?.. Maybe it also needs TCP (with SNI) or even UDP? If your case is non-standard, make sure you consider it thoroughly so that you won’t have to reconfigure the cluster later. Each controller has its own set of supported protocols.

### Based on (underlying software)

There are several types of applications that can be at the core of the controller. Popular ones are NGINX, Traefik, HAProxy, Envoy. In general case, this choice may not heavily affect how your traffic is processed, however it is always useful to know the potential characteristics and habits of what is “under the hood”.

### **Traffic routing**

What is the basis of the decision on routing traffic to a particular service? Usually, you can employ host and path, but there are additional possibilities as well. Does matching these values support RegEx (regular expressions) as well?

### **Namespace limitations**

Namespaces provide a way to logically separate resources in Kubernetes (for example, between your production, staging, etc). There are Ingress controllers that must be installed in each namespace separately (they allow traffic to the pods belonging to that namespace only). And there are those (actually, a clear majority) that operate globally for the entire cluster. In this case, traffic can go to any pod regardless of its namespace.

### **Upstream probes**

How do you direct traffic to healthy instances of an application, its services? There are solutions with active and passive checks, retries, circuit breakers, custom health checks, etc. It is a very important parameter if you have strict requirements for the availability and prompt removing of failed services from load balancing.

### **Load balancing algorithms**

Here we have a lot of choices, from the traditional [round-robin](https://en.wikipedia.org/wiki/Round-robin_DNS) to much more unconventional like the [rdp-cookie](http://www.loadbalancer.org/blog/load-balancing-windows-terminal-server-haproxy-and-rdp-cookies/). [Sticky sessions](https://access.redhat.com/solutions/900933) are also quite common in this category.

### **Authentication**

What authorization schemes does the controller support? Basic, digest, OAuth, external auth… I think, you know them already. This is an important parameter if we use many environments (tiers) for developers and/or simply private tiers that are accessed via Ingress.

### **Traffic distribution**

Does the controller support often used mechanisms of traffic distribution like canary deployments, A/B testing, mirroring/shadowing? This is a really sensitive subject for applications that require accurate and careful traffic management for fruitful testing, debugging of production errors with minimal impact, traffic analysis, etc.

### **Paid subscription**

Is there a paid version of the controller with extended functionality and/or technical support?

### **Graphical user interface (Web UI)**

Is there any graphical interface for controller configuration? This is important for those who prefer the convenience and/or who need to make some changes to Ingress’ configuration preferring to avoid working with “raw” templates. It can be also useful if developers want to experiment with the traffic “on the fly.”

### **JWT validation**

Is there a built-in validation of JSON Web Tokens for authentication and validation of the user to the end application?

### **Customization of configuration**

Expandability of templates in the sense of having the mechanisms that allow you to add your own directives, flags, etc to standard configuration templates.

### **Basic DDOS protection mechanisms**

Basic rate limit algorithms or more sophisticated variants of traffic filtering based on addresses, whitelists, countries, and so on.

### **Requests tracing**

Ability to monitor, trace and debug requests from Ingresses to specific services/pods (and ideally between services/pods) via OpenTracing or other options.

### WAF

Support for a [web application firewall](https://en.wikipedia.org/wiki/Web_application_firewall).

## **Ingress Controllers**

The list of controllers has been started from the [official Kubernetes documentation](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#additional-controllers) and extended with other well-known options. However some of them were excluded due to specific nature or low popularity / early stages of development. The remaining are examined below. We would start with a general description and provide the summary table at the end.

### **Kubernetes Ingress Controller**

- [*github.com/kubernetes/ingress-nginx*](https://github.com/kubernetes/ingress-nginx)
- *Implemented in: Go/Lua (while nginx is written in C)*
- *License: Apache 2.0*

The “official” Kubernetes controller. *(We will call it this way not to be confused with an offer from NGINX company which will follow shortly.)* It is being developed by the community. As you might guess from the name, it is based on nginx web server and is supplemented with a set of Lua plugins used to implement extra features.

Thanks to the popularity of NGINX and minimal modifications over it when using as a controller, it can be the simplest and most straightforward option for an average engineer dealing with K8s.

### **NGINX Ingress Controller**

- [*github.com/nginxinc/kubernetes-ingress*](https://github.com/nginxinc/kubernetes-ingress)
- *Implemented in: Go/Python*
- *License: Apache 2.0*

This is the official product from NGINX developers which also has a commercial version based on [NGINX Plus](https://www.nginx.com/products/nginx/). NGINX controller features high stability level, constant backward compatibility, absence of any third-party modules, promised high speed (in comparison with the official controller) achieved thanks to the elimination of Lua code.

The freeware version is significantly limited, even when compared with the official controller (due to the absence of mentioned above Lua modules). At the same time, the paid version has a fairly wide additional functionality: real-time metrics, JWT validation, active health checks, and so on. The important advantage over NGINX Ingress is the full support for TCP/UDP traffic (and in the community version too!).

Some prominent features (including requests tracing with OpenTracing and improved traffic splitting) have been added to this controller with [its v1.6.0 release](https://www.nginx.com/blog/announcing-nginx-ingress-controller-for-kubernetes-release-1-6-0/) in Dec’19.

### **Kong Ingress**

- [*github.com/Kong/kubernetes-ingress-controller*](https://github.com/Kong/kubernetes-ingress-controller)
- *Implemented in: Go*
- *License: Apache 2.0*

Kong Ingress is being developed by Kong Inc and also has two versions: commercial and free. Kong Ingress is built on NGINX with the addition of Lua modules that extend its capabilities.

Initially, it was focused on processing and routing of API requests, acting as an *API Gateway*. However, as of now, it has become a full-fledged Ingress controller. Its main advantage is a large number of additional modules/plugins (including those from third-party developers) that are easy to install and configure. It opens the way to a wide range of additional features. Incidentally, the built-in functions already offer many possibilities. The configuration is performed using CRD.

An important feature of Kong is its ability to run within *one environment* only (instead of supporting *cross-namespace*). It is quite a controversial topic: some consider it a disadvantage (you have to produce instances for each environment), while others believe it is a special feature (a higher level of isolation so the impact of malfunction of one controller is limited to its environment).

### Traefik

- [*github.com/containous/traefik*](https://github.com/containous/traefik)
- *Implemented in: Go*
- *License: MIT*

Originally, this proxy was created for the routing of requests for microservices and their dynamic environment, hence many of its useful features: continuous update of configuration (no restarts), support for multiple load balancing algorithms, web UI, metrics export, support for various protocols, REST API, canary releases and so on. The support for Let’s Encrypt certificates right out of the box is another nice feature. The main disadvantage is that in order to organize the high availability of the controller you have to install and connect its own KV-storage.

While a lot of nice new features (including TCP/SSL with SNI, canary deployments and traffic mirroring/shadowing, revamped Web UI) have arrived in a fresh (September’19) [Traefik v2.0 release](https://blog.containo.us/traefik-2-0-6531ec5196c2), some (like [WAF support](https://github.com/containous/traefik/issues/1336)) are still discussed.

*In September’19, another service mesh solution from the same developers* [*has been introduced*](https://blog.containo.us/announcing-maesh-a-lightweight-and-simpler-service-mesh-made-by-the-traefik-team-cb866edc6f29)*. It is called* [*Maesh*](https://github.com/containous/maesh) *and built on top of Traefik.*

### HAProxy Ingress

- [*github.com/jcmoraisjr/haproxy-ingress*](https://github.com/jcmoraisjr/haproxy-ingress)
- *Implemented in: Go (while HAProxy is written in C)*
- *License: Apache 2.0*

HAProxy is well known as a proxy server and load balancer. As part of the Kubernetes cluster, it offers a “soft” configuration update (without traffic loss), DNS-based service discovery, dynamic configuration through API. HAProxy also supports the full customization of a config-files template (via replacing a ConfigMap) as well as using Spring Boot functions there. In general, developers put emphasis on high speed, optimization, and efficiency in consumed resources. One of HAProxy advantages is a great number of supported balancing algorithms.

It’s worth mentioning a lot of new features have appeared in a recent (June’19) [v2.0 release](https://www.haproxy.com/blog/haproxy-2-0-and-beyond/), and even more (including OpenTracing support) is expected with upcoming v2.1.

### **Voyager**

- [*github.com/appscode/voyager*](https://github.com/appscode/voyager)
- *Implemented in: Go*
- *License: Apache 2.0*

Voyager is based on HAProxy and is offered as a universal solution that is well placed for a large number of providers. Its notable features include traffic balancing on L7 and L4. By all means, Voyager’ TCP L4-traffic balancing might be called one of the key features of this solution.

While full HTTP/2 and gRPC protocols support has appeared earlier this year (with [v9.0.0 release](https://github.com/appscode/voyager/releases/tag/9.0.0)), the support for cert-manage (for Let’s Encrypt certificates) might be the latest prominent feature integrated in Voyager since then.

### **Contour**

- [*github.com/heptio/contour*](https://github.com/heptio/contour)
- *Implemented in: Go*
- *License: Apache 2.0*

Contour is not only based on Envoy, but it was also developed jointly with authors of this popular proxy. The ability to manage Ingress resources via special CRD (new API called IngressRoute) is its special feature. For organizations with multiple development teams using one cluster concurrently, this helps to secure traffic in neighboring environments and protect them from errors arising when there are changes in Ingress resources.

It also offers an expanded set of balancing algorithms (mirroring, auto-repetition, limiting the rate of requests, and much more), detailed monitoring of traffic flow and failures. Perhaps, for some, the lack of support for sticky sessions would be a serious flaw ([ongoing efforts](https://github.com/projectcontour/contour/issues/361) in this direction have made a long journey already).

### **Istio Ingress**

- [*istio.io/docs/tasks/traffic-management/ingress*](https://istio.io/docs/tasks/traffic-management/ingress/)
- *Implemented in: Go*
- *License: Apache 2.0*

Istio — created as a joint project of IBM, Google, and Lyft (original authors of Envoy)— is a comprehensive service mesh solution. It can manage not just all incoming outside traffic (as an Ingress controller) but control all traffic inside the cluster as well. Under the hood, Istio uses Envoy as a sidecar-proxy for each service. In essence, it is a large processor that can do almost anything. Its central idea is maximum control, extensibility, security, and transparency.

With Istio Ingress, you can fine tune traffic routing, access authorization between services, balancing, monitoring, canary releases and much more.

*Here is a great intro to learn about Istio: “*[*Back to microservices with Istio*](https://blog.flant.com/google-cloud/back-to-microservices-with-istio-p1-827c872daa53)*”.*

### **Ambassador**

- [*github.com/datawire/ambassador*](https://github.com/datawire/ambassador)
- *Implemented in: Python*
- *License: Apache 2.0*

Ambassador is another Envoy-based solution. It has free and commercial versions. Ambassador is described as a «Kubernetes-native API gateway for microservices» and it brings corresponding benefits — such as the tight integration with the primitives of K8s. Having a pack of features you’d expect from an Ingress controller, it can also be used with a variety of service mesh solutions (Consul, Linkerd, Istio).

By the way, Ambassador engineers have recently published their [benchmarking results](https://www.getambassador.io/resources/envoyproxy-performance-on-k8s/) comparing performance of Envoy (as in Ambassador Pro), HAProxy and NGINX (as in official Kubernetes Ingress from the community).

### Gloo

- [*github.com/solo-io/gloo*](https://github.com/solo-io/gloo)
- *Implemented in: Go*
- *License: Apache 2.0*

Gloo is a new software ([announced](https://blog.flant.com/solo-io/announcing-gloo-the-function-gateway-3f0860ef6600) in Mar’18) built on top of Envoy and described as a “Function Gateway” since its authors insist “gateways should build APIs out of *functions* rather than *services*.” Its “function-level routing” means it can be used to route traffic for hybrid apps with backends implemented as microservices, serverless functions, and/or legacy apps.

Having a pluggable architecture, Gloo offers most of features you might expect, however some of them are available in its commercial version (Gloo Enterprise) only. Its documentation explains how easily it can be integrated with service mesh solutions like Linkerd & Istio.

### Skipper

- [*github.com/zalando/skipper*](https://github.com/zalando/skipper)
- *Implemented in: Go*
- *License: Apache 2.0*

Skipper is a *HTTP* router and reverse proxy, so it doesn’t support a big variety of protocols (for example, [no enthusiasm](https://github.com/zalando/skipper/issues/906) is seen in adding gRPC). Technically, it uses Endpoints API (instead of Kubernetes Services) to route traffic to the pods. Its huge advantage are advanced HTTP routing capabilities through its rich set of filters that can create, update, and delete all kind of HTTP data. Skipper’s routing rules can be updated without downtime. As Skipper’s authors state, it can be nicely used with other solutions like AWS ELB providing other features it misses (compared to other Ingress solutions).

[*Here is a comparison*](https://opensource.zalando.com/skipper/kubernetes/ingress-controller/#comparison-with-other-ingress-controllers) *of Skipper with other Kubernetes Ingress controllers made by its authors (Zalando).*

### Other notes

Having Traefik and Istio here, you might notice missing another popular service mesh solution — Linkerd. [Here’s](https://linkerd.io/2/features/ingress/) why it’s not listed:

> For reasons of simplicity, Linkerd does not provide its own ingress controller. Instead, Linkerd is designed to work alongside your ingress controller of choice.

## **Comparison table**

The culmination of the article is this huge summary matrix:

[![Kubernetes Ingress comparison](https://blog.flant.com/wp-content/uploads/sites/2/2019/10/kubernetes-ingress-comparison-1024x665.png)](https://blog.flant.com/wp-content/uploads/sites/2/2019/10/kubernetes-ingress-comparison.png)Kubernetes Ingress controllers comparison by Flant

You may click on it for more detailed viewing. The table is also available in [Google Sheets](https://docs.google.com/spreadsheets/d/1DnsHtdHbxjvHmxvlu7VhzWcWgLAn_Mc5L1WlhLDA__k/edit) format.

## **Summary**

The purpose of this article is to provide a more complete understanding (though not at all exhaustive!) of what can be done in each particular case. As usual, each controller has its advantages and disadvantages.

Official Kubernetes’ Ingress is easy to use, mature and offers nice features which are enough for the most cases. You should pay attention to a commercial version of Ingress based on NGINX Plus if there are high requirements to reliability and the quality of features implementation. Kong has the richest set of plug-ins (and opportunities they provide) and offers even more in its commercial version, it also boasts dynamic configuration based on custom resources.

Take a look at Traefik and HAProxy if there are increased demands for balancing and authorization methods. They are Open Source projects that proved their capabilities over the years, very stable and actively evolving. Contour has been around for a couple of years, but it still looks young and has mostly basic features built on top of Envoy. If you need a WAF in front of the application, pay attention to the same old Ingress by Kubernetes or HAProxy.

Envoy-based products have the richest set of features (especially Istio). It is a complex solution that “can do almost anything.” Incidentally, this means a broader relevant experience is needed for configuring/running/operating them. In some other cases (Gloo), many of its features may be offered in paid version only. Skipper can be a good solution if your application requires advanced or frequently changing HTTP routing tables.

If we speak about the trends in what a worldwide K8s community is choosing, the dominance of Istio and Traefik is evident — even the “official” Kubernetes Ingress is noticeably behind when we compare GitHub stars (25000+ and almost 20000 against less than 6000 correspondingly). Kong Ingress and HAProxy Ingress are on the opposite side having the smallest number of stars (less than 1000).

In [Flant](https://flant.com/), we have chosen the “official” Kubernetes’ Ingress as the standard controller and still use it. It covers 80–90% of our needs. Kubernetes’ Ingress is quite reliable, easily configurable and expandable. In the absence of specific requirements, it should fit well for the majority of clusters/applications. Traefik, HAProxy are other universal and relatively simple products that we can recommend.

**Our kind reminder: please feel free to express your corrections to keep this comparison more precise and up-to-date. Thanks in advance!**

## P.S. Other articles/reviews to consider

Here are some other articles which you may find useful if you’re choosing an Ingress solution:

- [Ingress](https://kubedex.com/ingress/) by kubedex — a nice table (with a brief text) comparing NGINX Ingress, Kong, Traefik, HAProxy, Voyager, Contour, Ambassador, Istio Ingress, Gloo Solo *(we have used this table to select options for our comparison)*;
- [Service Mesh Landscape](https://layer5.io/landscape/) by Layer5 — a huge selection (presented as a table) of service meshes (about 30 of them!) compared by many parameters, including supported protocols, multi-cluster and multi-tenant features, Prometheus and tracing integrations;
- [What’s the right ingress controller for my Kubernetes environment?](https://www.tfir.io/2019/05/17/whats-the-right-ingress-controller-for-my-kubernetes-environment/) *(May’19)* by TFiR — a text comparison of NGINX Ingress (both from community and NGINX Inc), Envoy, Kong, Google Cloud and AWS solutions, Citrix Ingress;
- [Comparison of Kubernetes Top Ingress Controllers](https://caylent.com/kubernetes-top-ingress-controllers) *(September’19)* by Cayent — a brief text comparison of Kong, Traefik, HAProxy, Istio Ingress, Nginx, and Ambassador;
- [Kubernetes Ingress Controllers: How to choose the right one: Part 1](https://itnext.io/kubernetes-ingress-controllers-how-to-choose-the-right-one-part-1-41d3554978d2) by ITNEXT — quite in-depth text comparison of Nginx Ingress controllers vs AWS ALB Ingress Controller.

## *Afterword*

*This article has been originally posted on [Medium](https://medium.com/flant-com/comparing-ingress-controllers-for-kubernetes-9b397483b46b). New texts from our engineers are placed here, on [blog.flant.com](https://blog.flant.com/). Please follow our [Twitter](https://twitter.com/flant_com) or subscribe below to get last updates!*

## Related posts:

- [Running Cassandra in Kubernetes: challenges and solutions](https://blog.flant.com/running-cassandra-in-kubernetes-challenges-and-solutions/)
- [Go? Bash! Meet the shell-operator](https://blog.flant.com/go-bash-meet-the-shell-operator/)
- [shell-operator & addon-operator news: hooks as admission webhooks, Helm 3, OpenAPI, Go hooks, and more!](https://blog.flant.com/shell-operator-addon-operator-v1-rc1-changes/)