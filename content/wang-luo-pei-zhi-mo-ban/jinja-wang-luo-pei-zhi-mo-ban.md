---
description: Jinja for Network Configuration Templates
---

# 网络配置模板Jinja

本章的剩余部分将特别关注一种模板语言——**Jinja**。我们将从一些基础知识开始，然后逐渐上升到更高级的主题，同时展示如何使用这些概念来生成一致\(consistent\)的网络设备配置。

## 为什么使用Jinja？

我们在本章的介绍中提到了Jinja，但也提到了其它七种模板语言。为什么我们只关注用于网络模板自动化的Jinja？为了避免本章的内容扯远了，我们选择Jinja是因为它与本书中提到的许多其它技术密切相关，包括Python\([第四章](https://app.gitbook.com/@uglysht/s/111/~/drafts/-MWcx1HR2sOhvEJgqhs4/content/python-zai-wang-luo-huan-jing-zhong-de-ying-yong)\)。事实上，Jinja是一种为Python而构建的模板语言。同时Jinja也被Ansible和Salt这两个自动化工具大量使用，并且这两个自动化工具也是用Python编写的，我们将在[第九章](https://app.gitbook.com/@uglysht/s/111/~/drafts/-MWcx1HR2sOhvEJgqhs4/content/zi-dong-hua-gong-ju)详述这两个工具。如果你已经很熟悉Python了，那么Jinja看起来和用起来也与Python非常相似。

能用其它模板语言构建网络配置模板吗？答案是肯定的。但是，如果你是使用模板语言的新手，特别是当你按照本书建议，学习一些Python技巧，你将会发现Jinja是你网络自动化历程中非常强大的工具。

## 将数据动态插入到基本的Jinja模板

我们从一个简单的例子开始，编写一个配置单个交换机接口的模板。这里有一个交换机端口\(使用行业标准CLI语法\)配置，我们想要将它转换为模板\(这样我们就可以在我们的环境中配置其它数百个接口\)：

```bash
interface GigabitEthernet0/1
 description Server Port
 switchport access vlan 10
 switchport mode access
```

这种代码段编写模板非常简单——我们只需要决定配置中的哪些部分保持不变，哪些部分需要变为动态插入。下一个例子中，我们移除具体的接口名\(“`GigabitEthernet0/1`”\)，然后将其转换为一个变量，当我们将模板渲染为实际配置时，我们就填入该变量：

```bash
interface {{ interface_name }}
 description Server Port
 switchport access vlan 10
 switchport mode access
```

这意味着我们可以在渲染这个模板时传入变量`interface_name`，并且这个位置会填入与`interface_name`相关联的值。

前面的例子假设每个网络接口都有一个相同的配置，如果我们想要配置不同VLAN或者某些接口配置不同的接口描述呢？在这种情况下，我们应该转换配置的一部分成为变量：

```bash
interface {{ interface_name }}
 description {{ interface_description }}
 switchport access vlan {{ interface_vlan }}
 switchport mode access
```

以上都是一些简单的例子，但它们**不是命名空间友好的\(namespace-friendely\)**。

当渲染一个模板时，通常会利用像Python语言中类和字典的概念。这可以是我们存储更多数据实例，可以在结果配置中循环并多次写入。我们将在后面的章节中探讨循环，但是现在，这里有一个重写的相同模板，被存储命名为template.j2，利用了类似Python类或字典的方式：

```bash
interface {{ interface.name }}
 description {{ interface.description }}
 switchport access vlan {{ interface.vlan }}
 switchport mode access
```

这是一个细微变动，但却是一个重要的变动。对象`interface`作为一个整体传入模板。如果`interface`是一个Python类，则`name`、`description`和`vlan`都是这个**类的属性**。同样的是，如果interface是一个字典——那么唯一的不同就是它们都是字典的**键**，而不是属性，所以渲染引擎会在渲染这个模板时自动为这些键放置放置对应的值。

## 在Python中渲染Jinja模板文件

在前面的例子中，我们看到了一个基本的交换机端口配置的Jinja模板，但我们并没有探索模板实际上是如何渲染的，是什么驱动数据嵌入到模板中，以生成最终配置文件的。现在，我们使用Python和Jinja2库来探索这个问题。

{% hint style="info" %}
虽然模板语言本身被称为Jinja，但**用于处理Jinja的Python库称为Jinja2**。
{% endhint %}

让我们使用上一个例子中的相同模板片段，然后使用Python将真实数据填充到那些动态字段中。我们将在这个例子中使用Python解释器，这样你就可以在自己的机器上操作。

{% hint style="info" %}
Python中的Jinja2渲染引擎并不是标准库的一部分，所以默认情况下没有安装。使用命令`pip install jinja2`用`pip`来安装，就像第四章中提到的，在**PyPI**中找到的其它Python包一样。
{% endhint %}

一旦安装完Jinja2库，我们应该首先导入我们需要的对象，以便渲染我们的模板。

```python
>>> from jinja2 import Environment, FileSystemLoader
```

接下来，我们需要设置环境，让渲染器知道去哪里找到模板。

```python
>>> ENV = Environment(loader=FileSystemLoader('.'))
>>> template = ENV.get_template("template.j2")
```









## 条件与循环

### 使用条件逻辑来创建交换机端口配置



### 使用循环创建多个交换机端口配置



### 使用循环和条件来创建交换机端口配置



### 在for循环中对变量进行循环以生成配置



### 在字典列表中生成接口配置



## Jinja Filter



## Jinja的模板继承





## Jinja的变量创建





