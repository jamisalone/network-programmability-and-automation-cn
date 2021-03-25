---
description: Data Models Using YANG
---

# YANG数据模型

在本文中，我们一直在讨论特定数据格式背景下的数据模型—例如，如何在XML或JSON中执行数据模型。在这一节中，我们想退一步，更笼统地看待数据模型，然后在最后讨论如何使用YANG来描述特定于网络的数据模型。

为了帮助巩固数据模型的概念，快速回顾一下关于数据模型的一些关键特征。

* 数据模型以**模式语言**（schema language，例如XSD）的形式描述一组受约束的数据。
* 数据模型使用定义明确的类型\(well-defined types\)和参数来对数据进行结构化\(structured\)和标准的\(standard\)表示。
* 数据模型**不传输**数据，数据模型不关心使用的底层传输协议\(例如，你可以通过HTTP使用JSON，也可以通过HTTP使用XML\)。

## YANG Overview

YANG是RFC 6020中定义的一种数据建模语言。

* YANG类似于我们提到的针对通用XML数据的XSD，但YANG特别关注网络构造。
* YANG用于模拟配置和操作状态数据，也用于模拟一般的RPC数据。一般RPC数据和任务允许我们对通用任务进行建模，例如升级设备。
* YANG提供了定义语法和语义\(syntax and smantics\)的能力，以便使用内置和可定制的类型更容易地定义数据。您可以强制执行语义，例如VLAN ID必须在1和4094之间。您可以强制执行接口的操作状态，即它必须是 "Up "或 "Down"。模型\(model\)定义了这些类型的构造，然后这些结构最终成为网络设备上允许配置的来源。

YANG模型有多种类型。其中一些YANG模型是由终端用户创建的；另一些则是由厂商或开放组织创建的。

* 有来自IETF和Open-Config工作组等团体的行业标准模型。这些模型是厂商和平台中立的。由开放标准组制作的每个模型都是为了给特定功能提供一套基本的选择。
* 当然，也有针对厂商的模型。如你所知，几乎每个厂商都有自己的多机箱链路聚合解决方案\(solution of multichassis link aggregation, 例如VSS、VPC、MCLAG、虚拟机箱\)。这意味着，如果每个厂商采用**模型驱动的架构\(model-driven architecutre\)**，就需要有自定义的YANG模型。
* 在每家厂商的设备上，你甚至可能看到某个功能的差异。因此，也就有针对平台的\(platform-specific\)模型。也许OSPF在X平台与同一厂商的Y平台上的操作方式不同，这也需要不同的模型。

## Taking a Deeper Dive into YANG

我们继续研究YANG statements，来说明YANG如何转化为两个系统用来通信的数据。要注意的是我们并没有展示如何创建一个自定义的YANG模型，因为这不属于本书的范围。但接下来介绍一些YANG statements，以帮助你更好地理解YANG。

* **leaf** — 允许定义一个单一实例的对象，只有一个值且没有子示例\(children\)

  ```text
  leaf hostname {
  	type string;
  	mandatory true;
  	config true;
  	description "Hostname for the network device";
  }
  ```

  我们可以从这个leaf statement中推测出代码在做什么。它定义了一个结构，这个结构包含了网络设备`hostname`的值。这个leaf被命名为`hostname`，类型为`string`且是设备上必需的值\(required，`mandatory`置为`true`\)，同时他也是可以配置的\(configurable，`config`置为`true`\)。

  也可以通过设置`config`为`false`在YANG的leaf statements中定义操作数据\(operational data\)。【笔者没有明白这个operational data是什么意思，希望有大佬能够帮忙指出。】

  YANG leaf在XML或JSON中被表示为单一元素或单一键值对，如下：【第一个为XML文件，第二个为JSON文件】

  ```text
  <hostname>NYC-R1</hostname>
  ```

  ```text
  {
  	"hostname": "NYC-R1"
  }
  ```

* **leaf-list** — 与leaf相似，但是leaf-list支持多个实例；因为leaf-list是一个list对象，存在一个叫做`ordered-by`的参数，基于顺序是否重要，这个参数可以用来设置_user_列表或_system_列表 — 即ACLs的顺序、SNMP community列表和域名服务器

  ```text
  leaf-list name-server {
  	type string;
  	ordered-by user;
  	description “List of DNS servers to query";
  }
  ```

  YANG leaf-list在XML或JSON中被表示为单一元素，如下：【第一个为XML文件，第二个为JSON文件】

  ```text
  <name-server>8.8.8.8</name-server>
  <name-server>4.4.4.4</name-server>
  ```

  ```text
  {
  	"name-server": [
  		"8.8.8.8",
  		"4.4.4.4"
  	]
  }
  ```

* **list**— 允许我们创建一个leaf或leaf-list的列表

  ```text
  list vlan {
  	key "id";
  	leaf id {
  		type int;
  		range 1..4094;
  	}
  	leaf name {
  		type string;
  	}
  }
  ```

  对环境来说更重要的是理解这个list在XML和JSON中如何被构建为模型。下面是分别使用XML和JSON来表示这个YANG list模型的例子：

  ```text
  <vlan>
  	<id>100</id>
  	<name>web_vlan></name>
  </vlan>
  <vlan>
  	<id>200</id>
  	<name>app_vlan></name>
  </vlan>
  ```

  ```text
  {
  	"vlan": [
  		{
  			"id": "100",
  			"name": "web_vlan"
  		},
  		{
  			"id": "200",
  			"name": "app_vlan"
  		}
  	]
  }
  ```

* **container** — 容器直接映射到XML和JSON中的层次结构。在上面list的例子中，我们有一个VLAN的列表，但在XML中没有包含所有VLAN的外部结构或标签元素。我们可以添加一个名为`vlans`的容器来描述包含所有VLAN的结构

  ```text
  container vlans {
  	list vlan {
  		key "id";
  		leaf id {
  			type int;
  			range 1..4094;
  		}
  		leaf name {
  			type string;
  		}
  	}
  }
  ```

  相比于上个list的例子，这个例子仅仅是在外部增加了一层容器：

  ```text
  <vlans>
  	<vlan>
  		<id>100</id>
  		<name>web_vlan></name>
  	</vlan>
  	<vlan>
  		<id>200</id>
  		<name>app_vlan></name>
  	</vlan>
  </vlans>
  ```

  ```text
  {
  	"vlans": {
  		"vlan": [
  			{
  				"id": "100",
  				"name": "web_vlan"
  			},
  			{
  				"id": "200",
  				"name": "app_vlan"
  			}
  		]
  	}
  }
  ```

上面对于YANG的介绍和相关statements的例子都在于说明YANG是一个简单的建模语言\(modeling language\)。这是一种对数据输入实施约束的方式，并且这些输入在API中使用时以XML和JSON表示。它也可以被任何其他数据格式表示。然后，设备本身会执行检查，查看数据是否符合正在使用的底层模型\(undelying model\)。

像YANG这样的建模语言可以让你为一个给定的数据集定义语义和变量约束。网络设备如何确保VLAN在1到4094之间？网络设备如何确保管理状态是打开或关闭的？可以通过对**数据的正确定义**来回答这些类型的问题。

这些定义是在模式文档\(schema document\)或特定的建模语言中定义的。其中一个选择是使用XML模式定义\(XSD\)，在前面介绍过。然而，XSD是通用的。虽然它们是定义XML文档的模式的好方法，但XSD并不是_network smart_的，因此业界看到模式和模型的编写方式正在发生转变。

* YANG是一种专门为网络创造的建模语言，它了解网络构造\(networking constructs\)。举个例子，它有内置的类型来验证一个输入是否是有效的IPv4地址、BGP AS号或MAC地址。
* YANG对编码类型也是中立的。一个模型可以用YANG编写，然后用JSON或XML的数据格式来表示。**这是RESTCONF提供的功能**。RESTCONF是一个REST API，它使用XML或JSON编码的数据，而这些数据恰好代表由YANG模型定义的数据。

