---
description: Jinja for Network Configuration Templates
---

# 网络配置模板Jinja

本章的剩余部分将特别关注一种模板语言——Jinja。我们将从一些基础知识开始，然后逐渐上升到更高级的主题，同时展示如何使用这些概念来生成一致\(consistent\)的网络设备配置。

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



## 在Python中渲染Jinja模板文件





## 条件与循环

### 使用条件逻辑来创建交换机端口配置



### 使用循环创建多个交换机端口配置



### 使用循环和条件来创建交换机端口配置



### 在for循环中对变量进行循环以生成配置



### 在字典列表中生成接口配置



## Jinja Filter



## Jinja的模板继承





## Jinja的变量创建





