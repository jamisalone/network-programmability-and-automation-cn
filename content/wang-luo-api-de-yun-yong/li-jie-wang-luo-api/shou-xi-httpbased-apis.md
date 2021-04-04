---
description: Getting Familiar with RESTful APIs
---

# 熟悉基于HTTP的APIs

在网络API的背景下，有两种类型的基于HTTP的API需要理解。它们是RESTful HTTP-based APIs和non-RESTFUL HTTP-based APIs。为了更好地理解它们以及RESTful一词的含义，我们将从研究RESTful API开始。一旦你理解了RESTful架构和原则，我们将继续前进，并将它们与non-RESTFUL HTTP-based APIs进行比较。

## 理解RESTful APIs

RESTful API在网络行业中越来越流行，使用也越来越普遍，虽然它们从2000年初就已经出现了。目前网络基础设施中存在的大多数API都是基本HTTP的RESTful API的。这意味着，当你听到网络设备或SDN控制器上的RESTful API时，它是一个将在客户端和服务器之间进行通信的API。

**客户端将是一个应用程序，如Python脚本或Web UI程序，而服务器讲师网络设备或控制器**。更多的，由于HTTP被用作传输\(transport\)，你会使用URLs来执行一些操作，就像你在浏览万维网时做的那样。因此，如果你明白当你浏览一个网站时，会执行HTTP GETs，当你填写一个web表单并点击提交时，HTTP POST将被执行，这时你已经了解了使用RESTful API的基本原理。

让我们看看从网站检索数据和通过RESTful API从网络设备检索数据的例子。在这两种情况下，都会像网络服务器发送HTTP GET请求\(如图7-1\)。

![&#x56FE;7-1 &#x89C2;&#x5BDF;HTTP GET&#x54CD;&#x5E94;&#x6765;&#x7406;&#x89E3;REST](../../../.gitbook/assets/image%20%281%29.png)

在图7-1中，其中一个主要区别是发送至和来自Web服务器的数据。当查看互联网内容\(browsing the Internet\)时，你会接收到HTML数据，是你的浏览器对这些数据进行解释\(interpret\)，以便能够正确地现实内容。另一方面，当向暴露RESTful API的Web服务器发出HTTP GET请求时\(记住，它是通过URLs暴露的\)，你会收到以JSON或XML编码的返回数据。谈到JSON或XML，就是我们将使用我们在第五章所涉及的内容。因为你收到以JSON或XML编码的返回数据，应用程序必须明白如何解释\(interpret\)JSON和/或XML。我们将继续进行概述，以便我们在开始探索RESTful HTTP API的使用之前有一个更完整的认识。

现在我们已经介绍了RESTful API的高层次概述，让我们再深入一步，看看RESTful API的起源。值得注意的是，现代基于web的RESTful APIs的诞生和结构来自于Roy Fielding在2000年发表的博士论文。在这篇名为_架构风格和基于网络的软件架构设计_ \(_Architectural Styles and the Design of Network-based Software Architectures_\)的论文中，他定义了在互联网\(用REST定义的架构\)上使用网络系统工作的复杂细节。

存在**六个架构约束**\(architectural constraints\)，一个接口必须符合这六个约束才能被认为是RESTful的。在本章旨在讨论其中三个约束。

* **客户端-服务器架构 \(Client-Server\)**

  当简化服务器的需求时，这要求提高系统的可用性。拥有**客户端-服务器架构**，能在不改变服务器组件的情况下，同时考虑到客户端应用的可移植性和可更改性。这意味着，你可以拥有不同API的客户端\(Web UI，CLI\)而这些客户端消耗的又是相同的服务器资源\(后端API，backend API\)。

* **无状态 \(Stateless\)**

  客户端和服务器之间的通信必须是无状态的。使用无状态通信形式的客户端必须在一次请求中发送服务器理解和执行请求的操作所需的所有数据。这与SSH等接口不同，SSH是客户端和服务器之间的持久连接。

* **统一接口\(Uniform interface\)**

  一个API调用范围内的单个**资源**在HTTP请求消息中可以被识别的。举个例子，在基于HTTP的RESTful系统中，所使用的**URL**引用了一个特定的**资源**。在网络的背景下，资源映射到一个网络设备构造，如主机名、接口、路由协议配置或设备上存在的任何其它资源。统一接口还规定，客户端应该有足够的资源信息来构建、修改或删除资源。

以上是REST架构六个核心约束中的其中三个，但我们已经可以看出RESTful系统之间和日常通过网页浏览来使用互联网的相似性。要记住，HTTP是应用RESTful API的主要方式，尽管传输类型在理论上可以是其它方式。为了真正地理解RESTful APIs，那么你必须也要理解HTTP基础。

## 理解HTTP请求类型

重要的是要明白，虽然每一个RESTful API都是基于HTTP的，但是最终我们会看到\*\*不遵守REST规则的基于HTTP的APIs，\*\*所以\(**不遵守REST规则的基于HTTP的APIs**\)不是RESTful的。但是，无论如何，这些API需要理解\(understanding\)HTTP。因为这些API使用HTTP作为传输方式，所以我们将使用的HTTP请求类型和响应码\(response codes\)与互联网上已经使用的HTTP请求类型和响应码\(response codes\)相同。

例如，常见的HTTP请求类型包括GET、POST、PATCH、PUT和DELETE。可想而知，GET求情用于向服务器请求数据，DELETE请求用于删除服务器上的资源，并且剩下的三个P\(POST、PATCH、PUT\)用于在服务器上进行更改\(make a change\)操作。

在网络背景下，我们可以把这些请求类型看作以下几种：

* GET：获得配置或操作数据
* PUT、POST、PATCH：进行配置更改
* DELETE：删除特定配置

下表描述了这些请求类型。我们将在本章后面的实际例子中使用这些类型。

| 请求类型Request Types | 说明Description |
| :--- | :--- |
| GET | 检索指定资源 |
| PUT | 创建或替换一个资源 |
| PATCH | 创建或更新一个资源对象 |
| POST | 创建一个资源对象 |
| DELETE | 删除一个指定资源 |



## 理解HTTP响应码

如果你使用浏览器访问互联网或使用RESTful API，就像上述的请求类型都是一样的，响应码也是如此。

当你尝试登陆一个网站而使用的是无效凭证时，是否见到过“401 Unauthorized”消息？那么，如果你试图登录一个使用RESTful API的系统，并且发送了错误凭证，你也会收到相同的响应码。对于成功连接的消息或如果服务器自身报错的情况，服务器也是会返回对应的响应码。表7-2描述了响应码常见类型的列表。注意，这个列表并没有包含所有类型\(只是列举了常见的\)——还有其它类型的响应码存在。

| 响应码Response Codes | 描述Description |
| :--- | :--- |
| 2XX | 成功\(successful\) |
| 4XX | 客户端错误\(Client error\) |
| 5XX | 服务器错误\(Server error\) |

记住，基于HTTP的API的响应码类型与标准HTTP响应码类型并没有差异。我们仅仅是提供了一个响应码系列的表格，并将其作为读者学习其中各个响应的联系。

既然我们已经对REST和HTTP原理有了一定的理解，那么学习一下基于HTTP的非RESTful API也是重要的。

## 理解基于HTTP的非RESTful APIs

虽然RESTful API是首选，但你也可能会遇到基于HTTP的非RESTful API。在网络行业中，这种情况常见于**位于CLI之上的API**，这就意味着API调用实际上是向设备发送一条命令，而不是发送原生结构化数据\(native structured data\)。首选的方法是让所有现代网络平台的CLI或Web UI使用底层API \(underlying API\)，但是对于使用命令构建的“传统”或现有系统来说，事实上非RESTful API的使用是常见的，因为这种**添加API的方式** \(\[译者注\]指的是在CLI或者Web UI之上构建API\) 比重构底层系统更容易。

基于HTTP的RESTful API和基于HTTP的非RESTful API有两个主要差异。我们之前介绍了HTTP请求类型的概念，这些请求类型映射到一个特定的动词\(verb\)，如GET、POST、PATCH、PUT和DELETE。RESTful API使用特定动词来指定所请求的目标服务器配置更改的类型 \(type of change\)。举个例子，在网络的背景下，如果你发送一个HTTP GET，那么配置更改的动作则不会发生，因为你仅仅是执行数据检索。但是，**基于HTTP但不遵守RESTful规则的系统会对每一个API调用使用相同的HTTP动词 \(verb\)**。这意味着，**如果你检索数据或进行配置更改，所有的API调用都可以使用一个POST请求**。如果你在一个给定的API类型中看到这种情况，它仍然是一个基于HTTP的API，但不是一个基于HTTP的RESTful API。

另一个需要知道的常见差异主要在每次API调用中被使用的URL。如果你使用的是基于HTTP的API，则API总是使用相同的URL并且不允许你通过URL更改来访问某个特定资源，那么这个API就是基于HTTP的RESTful API。

在网络基础设施上使用基于HTTP的API，对行业来说是向着正确方向迈进了一大步，但是理想的是，所有HTTP API都将遵循REST的规则。

尽管你用相同工具来使用基于HTTP的RESTful API和基于HTTP的非RESTful API，但重要的是要意识到这些网络API的差异，因为基于HTTP的非RESTful API通常不像RESTful API那样灵活。

现在我们已经介绍了基于HTTP的API，让我们转移注意力，接着介绍一下NETCONF API。

