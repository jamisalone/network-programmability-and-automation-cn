---
description: Dive into NETCONF
---

# 深入了解NETCONF

NETCONF是一个网络管理协议，在RFC 6241中定义，从根本上是专为配置管理和在网络设备上检索配置和运行状态数据而设计的。在这一点上，NETCONF在配置和运行状态之间有明确的划分；API请求用于执行不同的操作，如检索配置状态、检索运行状态和进行配置更改。

{% hint style="info" %}
我们在上一节中谈到，RESTful APIs并不是一项新技术，只是对于网络设备和SDN控制器来说是新的。当我们过渡到NETCONF API时，值得注意的是，NETCONF也不是新的技术。NETCONF实际上也已经存在十多年了。事实上，它是一个行业标准的协议，其最初的RFC已经在2005年被编写出来了。它甚至已经在各种网络设备上出现了很多年，尽管通常作为一个很少被使用的API。
{% endhint %}

NETCONF的**核心属性之一就是能够利用不同配置数据存储\(configuration datastores\)**。大多数网络工程师都熟悉运行配置\(running configuration\)和启动配置\(startup configuration\)。它们被看作两个配置文件，但是它们也是NETCONF背景下的两个配置数据存储。NETCONF世贤通常倾向于使用第三种称为候选配置\(candidate configuration\)的数据存储。候选配置存储保存未应用到设备上的配置对象\(如果你使用CLI来做配置，则配置对象就是CLI命令\)。举个例子，如果你在一台支持候选配置的设备上输入一条配置，配置不会立刻运行生效。相反，它们被保存在候选配置中，只有在执行\*`commit`_操作后才会被应用到设备上。当_`commit`\*执行时，候选配置才会被写到运行配置中。

候选配置数据存储已存在多年，从在NETCONF RFC中被定义到现在已经十多年了。行业中目前所面临的问题是，**NETCONF是否有可用的实现\(implementation\)来提供这个功能\(候选配置数据存储\)**。但是，不是所有实现都是未使用的——事实上，已经存在成功的实现。Juniper的Junos多年来一直拥有健壮的\(robust\)NETCONF实现，以及候选配置的能力；最近，CIsco IOS-XR增加了对NETCONF和候选配置的支持。HPE的Comware 7和Cisco IOS-XE等操作系统支持NETCONF，但是还不支持候选配置数据存储。

{% hint style="info" %}
经常检查你的硬件和软件平台，即使它们都是来自同一厂商。很可能，它们之间支持的功能是不同的。对候选配置的支持只是其中一个例子。
{% endhint %}

我们前面说到，对于候选配置，你输入各种配置，在执行提交操作\(commit operation\)之前，它们都不会在设备上应用。这就引出了启用NETCONF的设备的另一个核心属性——配置更改即事务\(transaction\)。在同样的例子中，这意味着所有的配置对象\(命令\)都是作为事务提交的。所有命令都要成功，否则它们就不会应用到设备上。这与更常见的场景相反，输入一系列命令，但中间某条命令配置失败，就会产生一个部分配置。

对候选配置的支持只是NETCONF的一个属性。现在让我们深入了解一下NETCONF的底层协议栈。

## 学习NETCONF协议栈

我们已经介绍了NETCONF的几个属性，但是是时候深入理解一下用于客户端和服务器通信的NETCONF协议栈了。在我们的例子中，客户端将是一个Python应用或SSH客户端，而服务器则是我们要进行自动化的目标网络设备。

NETCONF协议栈有四个核心层\(表7-3\)。我们将对每一层进行概述，并展示它们在客户端和服务器之间发送的XML对象的具体例子。

| 层 Layer | 示例 Example |
| :--- | :--- |
| 传输层 Transport | SSHv2, SOAP, TLS |
| 消息层 Messages | &lt;rpc&gt;, &lt;rpc-reply&gt; |
| 操作层 Operations | &lt;get-config&gt;, &lt;get&gt;, &lt;copy-config&gt;, &lt;lock&gt;, &lt;unlock&gt;, &lt;edit-config&gt;, &lt;delete-config&gt;, &lt;kill-session&gt;, &lt;close-session&gt; |
| 内容层 Content | Configuration/filers: XML representation of data models \(YANG, XSD\) |

{% hint style="info" %}
NETCONF只支持XML的数据编码。另一方面，要记住，RESTful API支持JSON和/或XML。
{% endhint %}

### 传输层

NETCONF通常使用SSH作为传输层来实现；它是自己的SSH子系统。虽然我们的所有例子都使用NETCONF over SSH，但从技术角度来说，通过NETCONF over SOAP、NETCONF over TLS或其它任意满足**NETCONF要求**的协议来实现NETCONF也是可能的。随着SOAP向RESTful API的迁移，NETCONF over SOAP的进一步发展是有限的，并且随着通过NETCONF over TLS成为可能，本书提到的平台目前都不支持它\(NETCONF over SOAP\)了。

下面是NETCONF的其中一些要求：

* 必须是一个面向连接的会话，因此客户端和服务器之间必须有一个一致的连接\(consistent connection\)；
* NETCONF会话必须提供认证\(authentication\)、数据完整性\(data integrity\)、保密性\(confidentiality\)和回复保护\(reply protection\)的方式；
* 虽然NETCONF可以用其它传输协议来实现，但每个实现都必须至少要支持SSH。

### 消息层

NETCONF消息是基于远程过程调用\(RPC-based, RPC: Remote Procedure Call\)的通信模型，每条消息都用XML编码。使用基于远程过程调用的模型，可以独立于传输层类型来使用XML消息。NETCONF支持两种消息类型，即`<rpc>`和`<rpc-reply>`。查看实际的XML编码对象有助于说明NETCONF，所以让我们看一看NETCONF的RPC请求。

最简单的例子，消息类型只有`<rpc>`和`<rpc-reply>`，并且它们将始终位于编码对象中最外层的XML标签：

```markup
<rpc message-id="101">
	<!-- rest of request as XML... -->
</rpc>
```

每个NETCONF`<rpc>`都包含一个`message-id`的**必需属性**。上面这个例子中的`<rpc>`就出现了`message-id="101"`。这是一段客户端发给服务器的任意字符串\(arbitrary string\)。服务器会在响应头部\(response header\)中重用这个ID，因此客户端就支持服务器响应的是哪一条消息。

另一个消息类型是`<rpc-reply>`。NETCONF服务器用`message-id`和从客户端接收的其它属性\(如XML命名空间\)来进行响应：

```markup
<rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
	<data>
		<!-- XML content/response... -->
	</data>
</rpc-reply>
```

上面这个&lt;rpc-reply&gt;的例子是假设了XML命名空间在客户端发送的&lt;rpc&gt;中。注意，来自NETCONF服务器的实际数据响应是嵌入在&lt;data&gt;标签中的。

接下来，我们将展示NETCONF请求是如何指定它向服务器请求哪种特定NETCONF操作\(RPC\)的。

### 操作层



### 内容层



