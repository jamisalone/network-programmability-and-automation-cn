# 数据格式简介

* 计算机程序通常使用各种不同的工具来存储和使用程序产生的数据。它们可能会使用简单变量（单值），数列（多值），哈希值（键值对）或甚至是根据所使用语言的语法创建的自定义对象。有时需要一种**更抽象的、可移植**的格式。例如，一个非程序员可能需要将数据移入和移出这些程序，另一个程序可能要以类似的方式与这个程序进行通信，而且这些程序甚至可能不是用同一种语言编写的，就像传统的客户机-服务器通信之类的东西一样。我们需要一个标准的格式来让不同的软件之间进行通信，并让人类与它们进行交互。
* 对于数据格式，我们所说的是**基于文本表示的数据**\(text-based representation of data\)，要不然这些数据会在内存中以内部软件构造的形式表示。**本篇讨论的所有数据格式都在多种语言和操作系统上得到了广泛的支持**。
* 首先，从原始网络协议的角度来说，这种程度的标准化已经到位。像BGP、OSPF和TCP/IP这样的协议，是根据网络设备在互联网\(全球分布式系统\)中拥有单一语言的需要而构想出来的。本文中的数据格式也是出于非常类似的原因而设想的——使计算机系统能够公开地相互理解和通信。
* 所安装、配置或升级的每一台设备都是由考虑到该主题\(使计算机系统能够公开地相互理解和通信\)的软件开发人员赋予生命的。一些网络供应商认为有必要\(see fit\)提供机制来允许运维人员使用这些广泛支持的数据格式与网络设备进行交互；而另一些厂商则认为这是不必要的；
* 举个例子，一些配置模型对于自动化方法是很友好的，通过如XML或JSON等数据格式来表示配置模型。从下例中可以看出，在Junos中很容易看到一个特定数据集的XML表示：

  ```text
  root@vsrx01> show interfaces | display xml
  <rpc-reply xmlns:junos="<http://xml.juniper.net/junos/12.1X47/junos>">
  	<interface-information xmlns="<http://xml.juniper.net/junos/12.1X47/>
  	junos-interface" junos:style="normal">
  		<physical-interface>
  			<name>ge-0/0/0</name>
  			<admin-status junos:format="Enabled">up</admin-status>
  			<oper-status>up</oper-status>
  			<local-index>134</local-index>
  			<snmp-index>507</snmp-index>
  			<link-level-type>Ethernet</link-level-type>
  			<mtu>1514</mtu>
  			<source-filtering>disabled</source-filtering>
  			<link-mode>Full-duplex</link-mode>
  			<speed>1000mbps</speed>
  ... output truncated ...
  ```

  从编程角度来看，这种数据集的XML表示是理想的，因为每条数据都有自己的易解析字段\(easily parseable field\)。软件需要找到接口名称时，就可以直接找到位于有“name”标记的地方。当与基础设置组件交互时，这是理解软件系统和操作CLI的网工的不同需求的关键。

* 在高层次上思考数据格式时，首先了解打算用我们所掌握的各种数据格式做什么是重要的。每种格式都是为不同的用例\(use cases\)而创建的，了解这些用例将帮助你决定哪种格式是适合你的。

## 数据类型

由于这些格式的意义是在软件实例之间交换文字\(words\)、数字甚至是复杂的类，所以简单谈一下哪种类型的数据被这些格式表示。

如下：

* **String**：可以说，最基本的数据类型是字符串。这是一种非常常见的表示数字、字母或符号序列的方式。在Python中，String用`str`类型或`unicode`类型表示。
* **Integer**：实际上有很多数据类型与数值\(numerical values\)有关，但对于大多数人来说，在讨论数值数据类型时，首先想到的是整数。还有其他数据类型，如 float 或 decimal，用它们来描述非整数值。Python 使用`int`类型来表示整数。
* **Boolean**：最简单的数据类型之一是boolean，一个简单的值，要么是 True，要么是False。这是一个非常流行的类型，当希望知道一个操作的结果或者两个值是否相互相等时，就使用该类型。这在 Python 中使用`bool` 类型，值为True或False。
* **Advanced data structures**：数据类型也可以组织成复杂的结构。
  * 我们讨论的所有格式都支持一个称为数组的基本概念，或者有时候是一个列表。这是一个可以用某种索引来表示和引用的值或对象的列表。
  * 还有键值对，还可以称为字典、哈希、哈希图、哈希表或映射图。这与数组类似，但值是根据键值对组织的，其中键和值都可以是几种数据类型中的一种，如字符串、整数等。
  * 在Python中，数组可以有多种形式：集合、元组和列表都是用来表示一个很多项的序列，但它们提供的灵活性是不同的。键值对由`dict`类型表示。

