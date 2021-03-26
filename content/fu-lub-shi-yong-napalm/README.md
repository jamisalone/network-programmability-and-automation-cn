---
description: Using NAPALM
---

# 附录B 使用NAPALM

NAPALM，_Network Automation and Programmability Abstraction Layer with Multi-vendor support_，是一个Python库，它提供一套健壮的操作，使用一组通用的Python对象来管理网络设备，忽略\(regardless\)每个操作是如何在给定设备类型上进行的。

虽然NAPALM的特性越来越多，我们在本节中主要关注NAPALM的两个核心功能：

* 配置管理
* 从网络设备中检索信息

在这些操作之中，要注意，无论你使用那个厂商或操作系统，只要支持NAPALM驱动和给定操作的特性，执行任何给定操作都是一样的。

NAPALM支持大量的设备厂商，并使用不同的API来与每个厂商的设备进行通信。例如，Cisco Nexus现在使用NX-API，Arista EOS使用eAPI，Cisco IOS使用SSH，然后Juniper Junos驱动使用NETCONF。当评估NAPALM时，你应该清楚你使用的设备需要哪一种API。

有关支持的API和设备的更多细节，以及本附录未涉及的主题的更多细节，请查阅[NAPALM文档](https://napalm.readthedocs.io/en/latest/)。现在，我们先来看看用NAPALM管理配置。

