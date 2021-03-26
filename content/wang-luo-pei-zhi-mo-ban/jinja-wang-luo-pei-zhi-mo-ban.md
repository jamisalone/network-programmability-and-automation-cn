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

当渲染一个模板时，通常会利用像Python语言中类和字典的概念。这可以是我们存储更多数据实例，可以在结果配置中循环并多次写入。我们将在后面的章节中探讨循环，但是现在，这里有一个重写的相同模板，被存储命名为_`template.j2`_，利用了类似Python类或字典的方式：

{% code title="template.j2" %}
```bash
interface {{ interface.name }}
 description {{ interface.description }}
 switchport access vlan {{ interface.vlan }}
 switchport mode access
```
{% endcode %}

这是一个细微变动，但却是一个重要的变动。对象`interface`作为一个整体传入模板。如果`interface`是一个Python类，则`name`、`description`和`vlan`都是这个**类的属性**。同样的是，如果interface是一个字典——那么唯一的不同就是它们都是字典的**键**，而不是属性，所以渲染引擎会在渲染这个模板时自动为这些键放置放置对应的值。

## 在Python中渲染Jinja模板文件

在前面的例子中，我们看到了一个基本的交换机端口配置的Jinja模板，但我们并没有探索模板实际上是**如何渲染的，是什么驱动数据嵌入到模板中，以生成最终配置文件的**。现在，我们使用Python和Jinja2库来探索这个问题。

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

第一行设置`Environment`对象，指定一个点\(`.`\)来表示模板存在于启动 Python解释器的同一目录下。第二行通过静态指定模板名称`template.j2`，从环境中产生一个模板对象。同样，此模板文件的内容与我们以前保存的`template.j2`相同。

现在，驱动配置和模板准备好了，我们需要**数据**。对于该例，我们将使用**Python字典**。注意该字典的**键**对应于我们**模板中引用的字段名称**。

```python
>>> interface_dict = {
...     "name": "GigabitEthernet0/1",
...     "description": "Server Port",
...     "vlan": 10,
...     "uplink": False
... }
```

{% hint style="info" %}
重要的是要记住，你很少需要在Python 中手动创建数据结构来填充数据模板。在本书中，只是为了举例说明的目的，但你应该总是编写你的软件来从其他来源获取数据，而不是将数据嵌入到你的软件中。
{% endhint %}

现在我们拥有了渲染模板所需的一切。我们将调用模板对象的`render()`函数将数据传递到模板引擎中，然后使用`print()`函数将渲染后的结果输出到屏幕。

```python
>>> print(template.render(interface=interface_dict))
interface GigabitEthernet0/1
 description Server Port
 switchport access vlan 10
 switchport mode access
```

注意这里我们给模板对象的`render()`函数传递了一个参数`interface=interface_dict`。注意名称——参数`interface`这个关键词对应的是我们Jinja模板中对`interface`的引用。这就是如何将接口字典\(`interface_dict`\)传递给模板引擎的——当模板引擎看到**接口**或**接口字典的键的引用**时，它将使用这里传递的字典来满足该引用。

正如所看到的，呈现的输出如我们所料。然而，我们不一定要用一个Python字典。将其它Python库中的数据驱动到Jinja模板中是常见的，则我们还可以采用Python类的形式。

下一个例子展示了一个Python程序，类似于我们刚刚举的例子，但没有使用字典，而是使用Python类。

```python
from jinja2 import Environment, FileSystemLoader

ENV = Environment(loader=FileSystemLoader('.'))

template = ENV.get_template("template.j2")

class NetworkInterface(object):
    def __init__(self, name, description, vlan, uplink=False):
        self.name = name
        self.description = description
        self.vlan = vlan
        self.uplink = uplink
    
interface_obj = NetworkInterface("GigabitEthernet0/1", "Server Port", 10)

print(template.render(interface=interface_obj))
```

这个程序的输出与之前的程序输出是一样的。因此，没有一种“正确”的方式来用数据填充Jinja模板——这取决于数据的来源。幸运的是，Python Jinja2库在数据填充这方面允许一些**灵活性**。

{% hint style="info" %}
在这本书中，我们并没有过多地涉及Python类。这是因为还有许多其他资源可以学习如何在 你的Python代码中实现类，并且通常网络工程师会使用API来很好地映射到像列表或字典的简单结构。这本书旨在弥合软件基础知识和存在于下一代行业的工具之间的差距。

然而，面向对象编程和类的使用有很多好处。使用得当的话，面向对象编程可以更易读、更易维护，甚至更易测试。记住一点，当你编写代码时，要决定是采取一个成熟的对象定义，还是满足于采用简单的字典。
{% endhint %}

## 条件与循环

是时候让我们的模板真正地为我们工作了。前面的例子对于理解如何将动态数据插入到文本文件中是很有用的，但这只是扩大网络模板到适当自动化网络配置的过程的一部分。

{% hint style="info" %}
Jinja允许我们将Python式的逻辑嵌入到我们的模板文件中，以便执行决策或将重复数据压缩成一块，该块将在渲染时通过for循环解压缩。虽然这些工具是非常强大的，但他们也可能是一个带来事故。 

重要的是不要太沉迷于把各种高级逻辑放入你的模板中——Jinja有一些非常有用的功能，但它从来没有打算成为一个成熟的\(full-blown\)编程语言，所以最好保持良好的平衡。 

阅读[Jinja FAQ](http://jinja.pocoo.org/docs/dev/faq/)-特别是标题为“将逻辑放入模板不是一个可怕的想法吗？”的部分，以获得一些提示。
{% endhint %}

### 使用条件逻辑来创建交换机端口配置

让我们继续配置单个交换机端口\(switchport\)的例子——但在这种情况下，我们希望通过使用模板文件本身的条件来决定要渲染什么。

通常情况下，一些交换机端口将配置为VLAN trunk接口，另一些将配置为“mode access”。一个好的例子是接入层交换机，其中两个或多个接口时“uplink”接口且需要被配置为允许所有VLAN通过。我们前面的例子展示了一个“uplink”布尔属性，如果接口是上行端口则设置值为True，如果只是接入接口，则设置为False。我们可以在模板中使用一个条件语句来检查这个值：

```text
interface {{ interface.name }}
 description {{ interface.description }}
{% if interface.uplink %}
 switchport mode trunk
{% else %}
 switchport access vlan {{ interface.vlan }}
 switchport mode access
{% endif %}
```

简而言之，如果接口的`uplink`属性是`True`，则我们想要配置该接口为VLAN Trunk。否则，我们要确保它被设置为正确的access mode。

在前面的例子中，我们看到了一种新的语言—— `{% ... %}`括号是一个特殊的Jinja标签，它表示某种类型的逻辑。此模板的建立旨在配置GigabitEthernet0/1为VLAN Trunk，然后其它接口被置于vlan 10中，配置为access mode。

### 使用循环创建多个交换机端口配置

当目前为止，我们仅配置了一个接口，所以让我们看看是否可以使用Jinja循环来创建多个交换端口的配置。为此，我们使用for循环，这与我们通常使用的Python中的for循环语法极为相似。

```text
{% for n in range(10) %}
interface GigabitEthernet0/{{ n+1 }}
 description {{ interface.description }}
 switchport access vlan {{ interface.vlan }}
 switchport mode access
{% endfor %}
```

需要注意的是，这里再次使用到`{% ... %}`语法来包含每个逻辑表达\(logic statements\)。在这个模板中，我们调用`range()`函数在一个整数列表中进行迭代，并且对于每轮迭代，我们都打印出“**n+1**”轮的结果，因为`range()`从0开始迭代而通常交换端口编号从1开始。

### 使用循环和条件逻辑来创建交换机端口配置

这使得我们得到10个相同配置的交换端口——但是如果我们想要对其中的一些端口进行不同的配置呢？以我们探讨Jinja条件语句时的例子为例——也许第一个端口是一个VLAN trunk。我们结合上面学到的条件语句和for循环来实现不同配置：

```text
{% for n in range(10) %}
interface GigabitEthernet0/{{ n+1 }}
 description {{ interface.description }}
{% if n+1 == 1 %}
 switchport mode trunk
{% else %}
 switchport access vlan {{ interface.vlan }}
 switchport mode access
{% endif %}

{% endfor %}
```

结果是GigabitEthernet0/1被配置为VLAN Trunk，但是GigabitEthernet0/2–10依然为access mode。下面是使用模拟数据生成的接口描述：

```text
interface GigabitEthernet0/1
 description TRUNK INTERFACE
 switchport mode trunk
interface GigabitEthernet0/2
 description ACCESS INTERFACE
 switchport mode access
interface GigabitEthernet0/3
 description ACCESS INTERFACE
 switchport mode access
...
```

### 在for循环中对变量进行循环以生成配置

在前一个例子中我们能够在Jinja模板中访问字典中的键\(`interface.description`或`interface.vlan`\)，但如果我们通过使用for循环对字典和列表进行实际的迭代呢？

让我们想象一下，将下面的列表作为\*`interface_list`\*传入模板。相关Python代码为：

```text
intlist = [
	"GigabitEthernet0/1",
	"GigabitEthernet0/2",
	"GigabitEthernet0/3"
]
print(template.render(interface_list=intlist))
```

然后我们在循环中引用interface\_list，这样我们可以获取接口列表中的各个成员接口并对它们分别生成一个交换端口配置。注意，嵌套条件已经被修改了，因为我们不会使用计数变量n了：

```text
{% for iface in interface_list %}
 interface {{ iface }}
{% if iface == "GigabitEthernet0/1" %}
 switchport mode trunk
{% else %}
 switchport access vlan 10
 switchport mode access
{% endif %}

{% endfor %}
```

现在我们可以在每个循环的迭代中简单引用`iface`\(line 2\)来检索到该列表中的当前项。

我们也可以用字典来做同样的事情。下面是一段相关的Python代码，用于构造和传递用于Jinja模板的字典。这次我们从简，仅仅传递一组接口名\(interface names\)作为键和对应的端口描述作为值:

```text
intdict = {
	"GigabitEthernet0/1": "Server port number one",
	"GigabitEthernet0/2": "Server port number two",
	"GigabitEthernet0/3": "Server port number three"
}
print(template.render(interface_dict=intdict))
```

我们可以用与Python相同的方式，对用于迭代该字典的循环进行调整：

```text
{% for name, desc in interface_dict.items() %}
interface {{ name }}
 description {{ desc }}
{% endfor %}
```

`for name`、`desc`意味着在循环的每一次迭代中，`name`将作为字典的键，然后`desc`作为对应键的值。不要忘记添加例子中使用的`.item()`符号，以便正确将\*`interface_dict`\*拆开为`name`和`desc`。

这样我们就可以在模板中轻松地引用`name`和`desc`，结果如下所示：

```text
interface GigabitEthernet0/3
 description Server port number three

interface GigabitEthernet0/2
 description Server port number two

interface GigabitEthernet0/1
 description Server port number one
```

{% hint style="info" %}
你可能已经注意到，示例中的输出是无需的。这是由于字典是无序的，而我们使用for循环来迭代字典中的各项。你可能还记得我们在第四章中第一次讨论字典的情况。
{% endhint %}

你可能已经注意到前几个例子中的一些限制。在这几个例子中，我们使用range\(\)函数进行迭代，意味着我们没有像使用类或字典时那样拥有关于接口的所有有价值的元数据。即使我们在后续的例子中使用了字典，它的结构也不过是存储一个接口名与该接口的描述。

### 在字典列表中生成接口配置



## Jinja Filter



## Jinja的模板继承





## Jinja的变量创建





