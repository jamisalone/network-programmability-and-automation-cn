# JSON

* JSON是在网络开发者需要在web服务器和嵌入网页的小程序之间建立轻量通信机制的时候开发出来的。当然，XML在那个时候已经存在了，但它过于臃肿，无法满足互联网不断增长的需求。YAML和XML这两种数据格式在如何映射到大多数编程语言\(如Python\)的数据模型\(data model\)方面有很大的不同。有了PyYAML这样的库，将YAML文档导入源代码几乎是不费吹灰之力的。但是对于XML，基于你想做的，通常还需要一些其它步骤。由于这些原因和其他原因，JavaScript Object Notation\(JSON\)在2000年初崭露头角。它的目标是成为XML的轻量级版本，更适合流行的编程语言中的数据模型\(data model\)。同时，许多人还认为它更易于人类阅读，尽管这对数据格式来说是次要的考虑。
* 注意，JSON被广泛认为是YAML的子集。实际上，许多流行的YAML解析器能假设JSON数据好像是YAML，从而能够解析JSON数据。但是，YAML和JSON之间的关系细节是更加复杂的。
* 如果使用XML文档来展示一个表示三个authors的例子，将会是如下这样：

  ```text
  <authors>
  	<author>
  		<firstName>Jason</firstName>
  		<lastName>Edelman</lastName>
  	</author>
  	<author>
  		<firstName>Scott</firstName>
  		<lastName>Lowe</lastName>
  	</author>
  	<author>
  		<firstName>Matt</firstName>
  		<lastName>Oswalt</lastName>
  	</author>
  </authors>
  ```

  为了说明JSON和XML的不同，尤其是JSON更加轻量的特性，使用JSON来展示相同的数据：

  ```text
  {
  	"authors":[
  		{
  			"firstName": "Jason",
  			"lastName": "Edelman"
  		},
  		{
  			"firstName": "Scott",
  			"lastName": "Lowe"
  		},
  		{
  			"firstName": "Matt",
  			"lastName": "Oswalt"
  		}
  	]
  }
  ```

  * 整个结构都被封装在圆括号`{}`中。`{}`是常见的，这表示JSON对象\(JSON objects\)被包含在其中，这里认为里面的JSON对象就是若干键值对或者字典；JSON对象在描述这些结构中的键时，总是使用字符串作为值；
  * 键为`authors`，而这个键的值是一个JSON列表。这相当于在YAML中讨论的列表——一个空列表或包含多个值的有序列表。JSON列表被封装在方括号`[]`中。该列表中，包含了3个对象，对象之间用`,`和换行隔开，每个对象包含了两个键值对：第一个键值对描述了作者的名字，第二个键值对描述了作者的姓。

* 简单看下JSON支持的数据类型\(data types\)：Number\(有符号的十进制数\)，String，Boolean，Array\(值的有序列表，其中数组中的项不需要是相同类型，封装在`[]`中\)，Object\(键值对的一个无序集合，键必须为字符串，封装在`{}`中\)，Null\(空值，使用`null`表示\)。

## Working with JSON in Python

JSON享有广泛的编程语言支持。事实上，经常可以简单地将JSON数据结构导入\(import\)到特定语言的结构\(construct\)中，只需一条命令。下面看个例子：

* 我们的目标是下面这个文件中发现的数据导入到我们选择的编程语言所使用的结构\(construct\)中。下面是存储在简单文本文件中的JSON数据：

  ```text
  {
  	"hostname": "CORESW01",
  	"vendor": "Cisco",
  	"isAlive": true,
  	"uptime": 123456,
  	"users": {
  		"admin": 15,
  		"storage": 10,
  	},
  	"vlans": [
  		{
  			"vlan_name": "VLAN30",
  			"vlan_id": 30
  		},
  		{
  			"vlan_name": "VLAN20",
  			"vlan_id": 20
  		}
  	]
  }
  ```

* 当然，继续选择Python。Python在标准库中内置了处理JSON的工具，恰好叫json包。在这个例子中，我们在Python程序本身中定义了一个JSON数据结构\(data structure\)，但这也可以很容易地从一个文件或一个REST API中获取。正如所看到的，导入这个JSON也是相当直接的

  ```text
  # Python包含了处理JSON常用的工具，并且是作为标准库的一部分
  import json

  # We can load our JSON file into a variable called "data"
  # 我们可以加载JSON文件到一个名为data的变量中
  with open("json-example.json") as f:
      data = f.read()

  # json_dict是一个字典，json.loads负责将JSON数据赋值(place)到json_dict字典中
  json_dict = json.loads(data)

  # 打印结果-Python数据结构(data structure)的信息
  print("The JSON document is loaded as type {0}\\n".format(type(json_dict)))
  print("Now printing each item in this document and the type it contains")
  for k, v in json_dict.items():
      print(
          "-- The key {0} contains a {1} value.".format(str(k), str(type(v)))
      )
  ```

* 运行Python程序`$ python json-example.py`，得到结果：

  ```text
  The JSON document is loaded as type <type 'dict'>

  Now printing each item in this document and the type it contains
  -- The key uptime contains a <type 'int'> value.
  -- The key isAlive contains a <type 'bool'> value.
  -- The key users contains a <type 'dict'> value.
  -- The key hostname contains a <type 'unicode'> value.
  -- The key vendor contains a <type 'unicode'> value.
  -- The key vlans contains a <type 'list'> value.
  ```

  `unicode`数据类型可以大致等同于字符串类型。在 Python 中，`str`类型实际上只是一个字节序列\(a sequence of bytes\)，而`unicode`指定了一个实际的编码\(encoding\)。

## Using JSON Schema for Data Models

在YAML部分，解释了数据模型\(data model\)背后的思想【见Data Models in YAML】，并提到了 YAML没有内置任何数据模型的机制。在XML部分，谈到了XSD，它允许我们在XML中执行一个模式\(schema\)或数据模型。也就是说，我们可以对XML文档包含的数据类型进行详细说明。

JSON也有一个模式执行的机制，刚好就命名为JSON Schema。 这个规范被定义在[http://jsonschema.org/documentation.html](http://jsonschema.org/documentation.html)，但也还只是internet草案。

JSON Schema的[Python实现](https://pypi.org/project/jsonschema/)已经存在，其他语言的实现也可以找到。

