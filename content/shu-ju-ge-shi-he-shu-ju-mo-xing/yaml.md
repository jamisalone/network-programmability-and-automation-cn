# YAML

YAML是 "YAML Ain't Markup Language "的缩写，这似乎告诉我们，YAML的创造者并不希望它只是成为某种新的标记标准，而是以一种人类可读的方式来表示数据的独特尝试。YAML是一种对人类极友好的数据格式。

## Basics

* 越来越多工具如Ansible在使用YAML作为定义自动化工作流\(automation workflow\)的一种方式或提供一种可使用的数据集\(如提供VLAN列表\)
* Example 1 — 使用string来表示各厂商名称

  ```text
  ---
  - Cisco
  - Juniper
  - Brocade
  - VMware
  ```

  * `---` 表示YAML文本的开始
  * 每个`- x` 都是一个字符串\(string\)，并且字符串不需要单引号或双引号来表示，Python会通过YAML解析器\(PyYAML\)来自动发现字符串
  * 4个厂商名称都属于同一级，没有缩进出现，并且可以说4个项组成了长度为4的列表

* Example 2 — YAML趋近于Python数据结构的灵活性：可以使用混合数据类型\(Python\)

  ```text
  ---
  - Core Switch
  - 7700
  - False
  - ['switchport', 'mode', 'access']
  ```

  * 同样，这是长度为4的列表；但每个项都是完全不同的数据类型，第一项Core Switch是字符串类型，第二项7700是整数类型，第三项是布尔型，第四项`['switchport', 'mode', 'access']` 是列表\(list\)，这个列表包含了三个字符串项。
  * YAML的布尔类型也是非常灵活的，可以接受各种各样的值，当YAML解析器解释这些值时，这些各种各样的值都是一样的。在该例中使用`False`，或者你也可以写`no`，`off`或者甚至是简单的`n`，它们都代表同样的事情：一个False的布尔值。这种灵活性成为YAML常常用于软件项目的人机界面的原因；
  * 通过该例可以知道，外部的4个项是一个列表，第四项又含有一个列表，这是YAML表示列表的两种方式；
  * 有时我们需要帮助解析器搞清楚待通信的数据类型。当我们想要`- 7700` 被解析器解释为字符串，我们使用双引号对整数进行封装`"7700"`；另一个使用引号的原因是，如果一个字符属于YAML语法，比如冒号`:`;

* Example 3 — YAML支持键值对\(字典\)

  ```text
  ---
  Juniper: Also a plant
  Cisco: 6500
  Brocade: True
  VMware:
  - esxi
  - vcenter
  - nsx
  ```

  * 键为冒号左边的字符串，与键相关的值在冒号右边；当需要在Python中查找某个值时，就引用要查找值得对应键
  * 与列表类似，YAML键值对\(YAML字典\)的灵活性体现在值可以支持多种数据类型
  * YAML字典可以用多种方式表示，如下表示方式与上例等同：

    ```text
    ---
    {Juniper: Also a plant, Cisco: 6500, Brocade: True,
    VMware: ['esxi', 'vcenter', 'nsx']}
    ```

* Example 4 — 使用`#`进行注释

  ```text
  ---
  - Cisco # ocsiC
  - Juniper # repinuJ
  - Brocade # edacorB
  - VMware # erawMV
  ```

  * 在`#`后面的内容会被YAML解析器忽略掉

* 特点
  * YAML提供了一种人类和软件系统的友好交互方式
  * 就数据格式而言，YAML是相当新的
  * 关于软件元素之间的直接通信（即不需要人的交互），其他格式如XML和JSON要流行得多，而且有更成熟的工具，有利于实现软件间的直接交互。

## Working with YAML in Python

使用前面使用过的例子来说明YAML表示某个数据类型的不同方式：

* example.yml

  ```text
  ---
  Juniper: Also a plant
  Cisco: 6500
  Brocade: True
  VMware:
  - esxi
  - vcenter
  - nsx
  ```

* 使用Python读取YAML文件，并进行解析\(parse\)，然后将解析数据表示为某种变量

  ```text
  import yaml
  with open("example.yml") as f:
  result = yaml.load(f)
  print(result)
  type(result)
  ```

  ```text
  {'Brocade': True, 'Cisco': 6500, 'Juniper': 'Also a plant',
  'VMware': ['esxi', 'vcenter', 'nsx']}
  <type 'dict'>
  ```

  * 使用内容管理器`open(filename)`来打开文件以读取文件；
  * 使用内置在YAML模块中的`load()`方法来将YAML文件中的内容直接加载到字典中，并将字典命名为`result`；

## Data Models in YAML

* 有一组YAML数据

  ```text
  ---
  Juniper: vSRX
  Cisco: Nexus
  Brocade: VDX
  VMware: NSX
  ```

  明显地，我们可以将这组YAML数据认作一组厂商列表和对应厂商的网络设备。这些字符串组成了一个键值对组成的字典，`:` 左边的字符串是厂商，右边的字符串是设备。

* 当我们更改上例YAML数据

  ```text
  ---
  Juniper: vSRX
  Cisco: 6500
  Brocade: True
  VMware:
  - esxi
  - vcenter
  - nsx
  ```

  这是可用的YAML文档，但其中的数据是不可用的。即使Brocade有产品值为`True`，但是大多数YAML解析器会将这个数据读取为布尔类型而不是字符串类型；当我们的软件使用这组数据运行时，它会**期待该值为字符串类型**而不是布尔类型，所以很大可能这会造成我们的软件产生错误结果\(incorrect result\)或崩溃\(crash\)。

* 数据模型是定义存储在YAML等数据格式中的数据结构\(structure\)和内容的一种方法。使用数据模型，我们可以明确\(显式，explicitly\)规定YAML文档中的数据必须是一个键值列表，而且每个值必须是一个字符串。\*\*但不幸的是，YAML并没有提供任何内置的机制来描述或执行数据模型。\*\*有可用的第三方工具提供描述或执行数据模型的机制\(如Kwalify\)。这也是为什么YAML非常适合人机交互，但不一定那么适合机器与机器交互的原因之一。
* YAML被认为是JSON的超集；理论上，这意味着用于验证JSON schema\(JSON文档的数据模型\)的工具也可以验证YAML文档。【关于这点，后面JSON还会提到】

