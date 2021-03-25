---
description: XML
---

# XML

虽然YAML是人机交互的合适选择，但当软件元素需要相互通信时，其他格式如XML和JSON往往会受到青睐，成为数据表示\(data representation\)的选择。

XML在各种工具和语言中享有广泛的支持，例如Python中的LXML库。事实上，XML定义本身伴随着各种相关的定义，比如模式执行\(schema enforcement\)、转换\(transformation\)和高级查询\(advanced queries\)。

## Basics

* XML与YAML有一些相似之处，例如它们都是本质上都是有层次的，我们容易将数据嵌入到一个父结构\(parent construct\)中：

  ```text
  <device>
  	<vendor>Cisco</vendor>
  	<model>Nexus 7700</model>
  	<osver>NXOS 6.1</osver>
  </device>
  ```

  * `<device>`元素作为**根\(root\)**，它是XML文档中最外层的XML标签，它也是嵌套在它内部的元素的**父标签**\(parent of the elements nested within it\)：`<vnedor>`，`<model>`，`<osver>`；；它们反过来作为`<device>`元素的子元素；
  * 这有**利于存储关于网络设备的元数据**。在一个XML文档中，会存在多个`<device>`标签的实例，可能嵌套与一个更大的父元素`<devices>`中；
  * 各个子元素中也包含数据。根元素包含XML子元素，这些子元素标签中包含文本数据\(text data\)，在Python程序中，它们很可能用字符串类型表示值。

* **XML元素也可以有属性\(attributes\)**，例如`<device type = "datacenter-switch" />` ；当一条信息可能有一些相关的元数据时，可能不适合使用子元素，而应该使用属性。
* **XML规范还实现了一个命名空间系统\(namespace system\)，这有助于防止元素命名冲突**。开发人员在创建XML文档时可以使用任何他们想要的名称。当一个软件调用XML文档时，有可能会给该软件提供两个名称相同的XML元素，但这些元素的内容和目的是不同的。
  * 存在XML文档`<device>Palm Pilot</device>`，其使用`<device>`元素名称，但是显然它被用于某个目的而不是表示一个网络设备，因此这个定义是词不达意的；
  * 命名空间\(namespace\)可以通过定义和使用XML文档中的前缀\(prefixes\)来应对这种情况，使用**xmlns指定**\(xmlns designation\)：

    ```text
    <root>
    	<e:device **xmlns:c**="<http://example.org/enduserdevices>">Palm Pilot</e:device>
    	<n:device **xmlns:m**="<http://example.org/networkdevices>">
    		<n:vendor>Cisco</n:vendor>
    		<n:model>Nexus 7700</n:model>
    		<n:osver>NXOS 6.1</n:osver>
    	</n:device>
    </root>
    ```

## Using XML Schema Definition\(XSD\) for Data Models \| 使用XSD来建立数据模型

* YAML有一些内置的结构来帮助描述里面的数据类型\(连字符`-`和缩进的使用\)，但XML没有这些相同的机制。例如，许多XML解析器并没有像PyYAML和其他YAML解析器那样做出同样的假设。
* 像前面讨论的那样，数据格式\(data formats\)允许应用或设备\(网络设备\)使用标准化的方式进行信息交换。XML就是其中一种信息交换的标准化方式。然后，像XML这样的数据格式并没有强制要求各种字段和值中包含什么样的数据。为了帮助确保正确的数据类型在正确的XML元素中，我们使用XML Schema Definition\(XSD\)。
* XSD允许我们描述XML文档中的元素\(building blocks\)。使用XSD，我们可以**对数据应该\(或不应该\)在XML文档中的什么位置进行限制**。另外，XSD实际上是用XML编写的，这大大简化了事情。
* XSD——或者说任何一种模式\(schema\)或建模语言\(modeling language\)——的一个非常流行的用例是**生成与模式相匹配的源代码数据结构**。然后，我们可以**使用该源代码自动生成兼容该模式的XML**，而不是手工写出XML。
* 来看个例子
  * XML文档内容如下；

    ```text
    <device>
    	<vendor>Cisco</vendor>
    	<model>Nexus 7700</model>
    	<osver>NXOS 6.1</osver>
    </device>
    ```

  * 目标是打印该XML到Console界面。我们首先创建一个XSD文档，然后使用第三方工具从XSD文档生成Python代码，该代码被用于打印我们所需的XML。描述打算打印出来的数据的XSD模式文件\(schema file\)，`schema.xsd`如下：

    ```text
    <?xml version="1.0" encoding="utf-8"?>
    <xs:schema elementFormDefault="qualified" xmlns:xs="<http://www.w3.org/2001/>
    XMLSchema">
    	<xs:element name="device">
    	<xs:complexType>
    		<xs:sequence>
    			<xs:element name="vendor" type="xs:string"/>
    			<xs:element name="model" type="xs:string"/>
    			<xs:element name="osver" type="xs:string"/>
    		</xs:sequence>
    	</xs:complexType>
    </xs:element>
    </xs:schema>
    ```

    * 描述了每个`<device>`元素可以有三个子元素，并且在这些子元素中的数据必须是字符串；
    * 在 XSD 规范中支持指定必需的子元素，即你可以指定一个`<device>`元素必须有一个`<vendor>`子元素存在；

  * 使用Python工具**pyxb**来创建一个Python文件，该文件包含表示上述模式\(schema\)的类对象：`~$ pyxbgen -u schema.xsd -m schema`。这会在当前目录创建一个`schema.py`，所以，如果在此时打开一个Python提示，我们可以导入\(import\)这个schema文件。下例中，我们创建了一个生成对象的实例，为它设置属性`dev.x = "xxx"`，然后使用`toxml()`方法将设置的属性传递给XML：

    ```text
    **import schema**
    dev = schema.device()
    dev.vendor = "Cisco"
    dev.model = "Nexus"
    dev.osver = "6.1"
    dev.toxml("utf-8")
    ---------------------------------------------------------------------------------
    [output]
    '<?xml version="1.0" encoding="utf-8"?><device><vendor>Cisco</vendor><model>Nexus
    </model><osver>6.1</osver></device>'
    ```

    这仅仅是其中一个方法；还存在其它第三方库允许从XSD文件中生成代码。可以试一下generateDS，地址为[https://pypi.python.org/pypi/generateDS/](https://pypi.python.org/pypi/generateDS/) 。
* 一些RESTful API使用XML对软件端间的数据进行编码。使用XSD可以让开发人员更准确地生成兼容的XML，并且步骤更少。因此，如果你在网络设备上遇到RESTful API，让厂商提供模式文档\(schema documentation\)，这可以为你节省一些时间。

## Transforming XML with XSLT \| 用XSLT转换XML

## Searching XML Using XQuery

