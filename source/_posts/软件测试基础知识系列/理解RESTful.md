---
title: "理解RESTful "
date: 2017-04-02 21:56:11
tags: [Web-Test]
categories: [basic-knowledge]
---
在测试开发中，我们经常接触到API，在调用API时候特别是第三方API时候，我们常常陷入以下困惑：
以一个文件获取为例，我们要获取，上传，删除文件，需要以下接口：
>api/getfile.php - 获取文件信息，下载文件
api/uploadfile.php - 上传创建文件
api/deletefile.php - 删除文件

我们必须很熟悉每一个接口的名称才能使用
<!--more-->
如果我需要调用一个修改文件的接口该怎么办呢？这个接口叫modifyfile吗还是别的名字？如果接口维护人员不能及时提供帮助，我们是不是就无法正确引用了？ 既然都是对文件的操作，我们能不能用一个接口表述对这个文件的所有操作呢？
当然可以了，我们可以只用一个接口，但是用不同的http请求来表述不同的操作。：
api/file 只需要这一个接口
GET 方式请求 api/file - 获取文件信息，下载文件
POST 方式请求 api/file - 上传创建文件
DELETE 方式请求 api/file - 删除某个文件
是不是方便了很多？那么，这个是什么风格的API呢？是怎么实现的呢？我们就要从头说起了。
答案就是 RESTFUL。通常采用http+JSON实现。
（对API的调用通常包含两个部分，序列化和通信协议。常见的序列化协议包括json、xml、hession、protobuf、thrift、text、bytes等；通信比较流行的是http、soap、websockect，RPC。通常基于TCP实现，常用框架例如netty。web开发中常用的是json-rpc。 RESTful是另外一种方法)

要理解RESTful，首先要理解REST.
REST -- REpresentational State Transfer， 全称是 Resource Representational State Transfer
通俗来讲就是：资源在网络中以某种表现形式进行状态转移。REST是Web自身的架构风格。REST也是Web之所以取得成功的技术架构方面因素的总结。REST是世界上最成功的分布式应用架构风格（成功案例：Web，还不够吗？）。它是为 运行在互联网环境 的 分布式 超媒体系统量身定制的。互联网环境与企业内网环境有非常大的差别，最主要的差别是两个方面：
可伸缩性需求无法控制：并发访问量可能会暴涨，也可能会暴跌。
安全性需求无法控制：无法控制客户端发来的请求的格式，很可能会是恶意的请求。
而所谓的“超媒体系统”，即，使用了超文本的系统。可以把“超媒体”理解为超文本+媒体内容。

REST是HTTP/1.1协议等Web规范的设计指导原则，HTTP/1.1协议正是为实现REST风格的架构而设计的。新的Web规范，其设计必须符合REST的要求，否则整个Web的体系架构会因为引入严重矛盾而崩溃。这句话不是危言耸听，做个类比，假如苏州市政府同意在市区著名园林的附近大型土木，建造大量具有后现代风格的摩天大楼，那么不久之后世界闻名的苏州园林美景将不复存在。

上述这些关于“REST是什么”的描述，可以总结为一句话：REST是所有Web应用都应该遵守的架构设计指导原则。当然，REST并不是法律，违反了REST的指导原则，仍然能够实现应用的功能。但是违反了REST的指导原则，会付出很多代价，特别是对于大流量的网站而言。
要深入理解REST，需要理解REST的五个关键词：
资源（Resource）
资源的表述（Representation）
状态转移（State Transfer）
统一接口（Uniform Interface）
超文本驱动（Hypertext Driven）

什么是资源？
资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

什么是资源的表述？
资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

什么是状态转移？
状态转移（state transfer）与状态机中的状态迁移（state transition）的含义是不同的。状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

什么是统一接口？
REST要求，必须通过统一的接口来对资源执行各种操作。对于每个资源只能执行一组有限的操作。以HTTP/1.1协议为例，HTTP/1.1协议定义了一个操作资源的统一接口，主要包括以下内容：
7个HTTP方法：GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS
HTTP头信息（可自定义）
HTTP响应状态代码（可自定义）
一套标准的内容协商机制
一套标准的缓存机制
一套标准的客户端身份认证机制
REST还要求，对于资源执行的操作，其操作语义必须由HTTP消息体之前的部分完全表达，不能将操作语义封装在HTTP消息体内部。这样做是为了提高交互的可见性，以便于通信链的中间组件实现缓存、安全审计等等功能。

什么是超文本驱动？
“超文本驱动”又名“将超媒体作为应用状态的引擎”（Hypermedia As The Engine Of Application State，来自Fielding博士论文中的一句话，缩写为HATEOAS）。将Web应用看作是一个由很多状态（应用状态）组成的有限状态机。资源之间通过超链接相互关联，超链接既代表资源之间的关系，也代表可执行的状态迁移。在超媒体之中不仅仅包含数据，还包含了状态迁移的语义。以超媒体作为引擎，驱动Web应用的状态迁移。通过超媒体暴露出服务器所提供的资源，服务器提供了哪些资源是在运行时通过解析超媒体发现的，而不是事先定义的。从面向服务的角度看，超媒体定义了服务器所提供服务的协议。客户端应该依赖的是超媒体的状态迁移语义，而不应该对于是否存在某个URI或URI的某种特殊构造方式作出假设。一切都有可能变化，只有超媒体的状态迁移语义能够长期保持稳定。

一旦读者理解了上述REST的五个关键词，就很容易理解REST风格的架构所具有的6个的主要特征：
面向资源（Resource Oriented）
可寻址（Addressability）
连通性（Connectedness）
无状态（Statelessness）
统一接口（Uniform Interface）
超文本驱动（Hypertext Driven）

这6个特征是REST架构设计优秀程度的判断标准。其中，面向资源是REST最明显的特征，即，REST架构设计是以资源抽象为核心展开的。可寻址说的是：每一个资源在Web之上都有自己的地址。连通性说的是：应该尽量避免设计孤立的资源，除了设计资源本身，还需要设计资源之间的关联关系，并且通过超链接将资源关联起来。无状态、统一接口是REST的两种架构约束，超文本驱动是REST的一个关键词，在前面都已经解释过，就不再赘述了。

那么什么是RESTful呢？
如果一个架构符合REST原则，就称它为RESTful架构。

Server的API如何设计才满足RESTful要求?
REST 是面向资源的，这个概念非常重要，而资源是通过 URI 进行暴露。
一旦定义好了资源, 需要确定什么样的 actions 应用它们，这些 actions 怎么映射到你的 API 上。RESTful 原则提供了 HTTP methods 映射作为策略来处理 CRUD actions，如下：
GET /tickets - 获取 tickets 列表
GET /tickets/12 - 获取一个单独的 ticket
POST /tickets - 创建一个新的 ticket
PUT /tickets/12 - 更新 ticket #12
PATCH /tickets/12 - 部分更新 ticket #12
DELETE /tickets/12 - 删除 ticket #12

REST 非常棒的是，利用现有的 HTTP 方法在单个的 /tickets 接入点上实现了显著的功能。没有什么方法命名约定需要去遵循，URL 结构是整洁干净的。 REST 太棒了!

下面的是一个反例：
比如：左边是错误的设计，而右边是正确的
GET /rest/api/getDogs --> GET /rest/api/dogs 获取所有小狗狗 GET /rest/api/addDogs --> POST /rest/api/dogs 添加一个小狗狗 GET /rest/api/editDogs/:dog_id --> PUT /rest/api/dogs/:dog_id 修改一个小狗狗 GET /rest/api/deleteDogs/:dog_id --> DELETE /rest/api/dogs/:dog_id 删除一个小狗狗
左边的这种设计，很明显不符合REST风格，上面已经说了，URI 只负责准确无误的暴露资源，而 getDogs/addDogs...已经包含了对资源的操作，这是不对的。相反右边却满足了，它的操作是使用标准的HTTP动词来体现。总结起来就是：
URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作。

本文是我看了大量REST资料，根据自己理解总结而成，文中大量引用，糅合了不同资料，对这些乐于分享的人表示感谢。对REST的理解，可能有不当之处，欢迎交流指正。

参考资料如下：
https://www.zhihu.com/question/28557115
http://www.infoq.com/cn/articles/understanding-restful-style
https://www.zhihu.com/question/28570307 
http://blog.csdn.net/douliw/article/details/52592188