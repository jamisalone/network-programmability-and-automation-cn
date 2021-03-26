---
description: Understanding Configuration Management in NAPALM
---

# 理解NAPALM的配置管理

NAPALM提供了一种不同的方法来管理设备配置，同时仍然允许采用更传统的方法来配置设备。NAPALM 采用的独特方法被称为_**声明式配置管理**_。

声明式配置的唯一重点是你希望设备配置是什么。这与担心它是什么，以及如何从它是什么到你希望它是什么形成了鲜明的对比。虽然这是NAPALM的一个主要好处和特点，但它实际上是存在于实际网络工作设备上的特殊功能的副产品。其中一些以设备为中心的功能包括Juniper的候选配置、Arista的配置会话和Cisco IOS的配置替换功能。

在NAPALM术语中，以声明的方式管理一个完整的配置就是ac配置替换操作。

对于更传统的操作模式，NAPALM还提供了配置合并操作--就是将部分配置或仅有的几个设备命令，确保它们存在于目标网络设备上。

无论是哪种情况，主要是基于所支持的底层设备操作系统，只有在需要时才会进行更改。当我们通过几个例子时，你会看到这一点。

我们将从执行配置替换开始。

## 执行配置替换





```text
eos-spine1#show run
! Command: show running-config
! device: eos-spine1 (vEOS, EOS-4.15.2F)
!
! boot system flash:vEOS-lab.swi
!
transceiver qsfp default-mode 4x10G
!
hostname eos-spine1
ip domain-name ntc.com
!
snmp-server community networktocode ro
!
 
spanning-tree mode mstp
!
aaa authorization exec default local
!
no aaa root
!
username ntc privilege 15 secret 5 $1$KergS3bl$RFVho/GXf.3bQHhOCbeky1
!
vrf definition MANAGEMENT rd 100:100
!
interface Ethernet1 no switchport
!
interface Ethernet2 no switchport
!
interface Ethernet3 no switchport
!
interface Ethernet4 no switchport
!
...
!
interface Management1
vrf forwarding MANAGEMENT ip address 10.0.0.11/24
!
ip route vrf MANAGEMENT 0.0.0.0/0 10.0.0.2
!
ip routing
ip routing vrf MANAGEMENT
!
router ospf 100
router-id 100.100.100.100
network 10.0.0.10/32 area 0.0.0.0 network 10.0.1.10/32 area 0.0.0.0 network 10.0.2.10/32 area 0.0.0.0 network 10.0.3.10/32 area 0.0.0.0 network 10.0.4.10/32 area 0.0.0.0 max-lsa 12000
!
management api http-commands protocol http
no shutdown vrf MANAGEMENT
no shutdown
!
management ssh vrf MANAGEMENT
no shutdown
 
!
!
end
eos-spine1#

```





## 执行配置合并

