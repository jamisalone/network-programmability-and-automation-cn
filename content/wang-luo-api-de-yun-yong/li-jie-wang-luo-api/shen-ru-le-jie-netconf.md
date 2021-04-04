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



| 层 Layer | 示例 Example |
| :--- | :--- |
| 传输层 Transport | SSHv2, SOAP, TLS |
| 消息层 Messages | &lt;rpc&gt;, &lt;rpc-reply&gt; |
| 操作层 Operations | &lt;get-config&gt;, &lt;get&gt;, &lt;copy-config&gt;, &lt;lock&gt;, &lt;unlock&gt;, &lt;edit-config&gt;, &lt;delete-config&gt;, &lt;kill-session&gt;, &lt;close-session&gt; |
| 内容层 Content | Configuration/filers: XML representation of data models \(YANG, XSD\) |



### 传输层



### 消息层



### 操作层



### 内容层



