---
description: Working with Network APIs
---

# 第七章 网络API的运用

从Python和数据格式到使用Jinja的配置模板，我们已经探讨过了使你成为一个更好的网络工程师的关键基础技术和技能。在本章中，我们将把这些技能付诸实践，并开始使用\(consume\)和交流不同类型的网络设备API。 



为了最好地帮助你了解如何开始自动化网络，本章将组织成三个部分：

_**了解网络API**_

我们研究不同API的架构和基础，包括基于RESTful HTTP的API、基于非RESTful HTTP的API和NETCONF。

_**探索网络API**_

我们介绍常用的工具，用于测试和学习如何使用每种API类型。

_**使用网络API实现自动化**_

最后，我们看一下允许你开始自动化网络的Python库。我们将看看用于消费基于 HTTP 的 API 的 Python requests 库，用于与 NETCONF 设备交互的 ncclient，以及用于使用 SSH 自动化设备的 netmiko。



当你阅读本章时，请记住一点 — 本章不是任何特定API的综合指南，也不应该被视为API文档。因为在多供应商环境中工作是很常见的，我们将提供示例，这些示例使用不同产商实现\(different vendor implementation\)的给定API。同样重要的是，要看到同一类API不同实现之间的共同模式\(common pattern\)和独特对比\(unique contrasts\)。

