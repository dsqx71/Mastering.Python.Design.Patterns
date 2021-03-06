# Mastering.Python.Design.Patterns
  
《精通Python设计模式》
## 原版信息：
https://www.packtpub.com/application-development/mastering-python-design-patterns  

## 作者：
Sakis Kasampalis  

## 纸书出版日期：
January 2015  

## 特色：
Create various design patterns to master the art of solving problems using Python  

## 级别：
Progressing  


```开坑。
```
  

第一章预览 工厂模式
============
   创造模式处理一个对象的创建。创造模式旨在不按照约定而直接地创建。  
   
  在工厂模式中，用户查询对象但不知道这个对象来自于何处（即，它是由哪一个类生成的？）。在工厂背后的思想就是简化对象的创建。如果这个结果是通过一个中心函数来完成，相比之下要让用户直接地使用实例化的类来创建对象而言，跟踪哪一个对象被创建则会更容易些。通过分离将要使用的代码，工厂减少了应用维护的复杂性。
    
工厂通常以两中形式出现：**工厂模式**，是一个每次输入参数都会返回一个不同的方法（在Python术语中称为函数）；抽象工厂，它是一组工厂方法用于创建相关产品的系列。  

工厂方法
--------------
********
在工厂方法中，我们执行一个单独的函数，传递一个参数，它提供了我们想要的是*什么* 信息。我们不要求知道任何关于这个对象的*如何*实现的细节，以及它来自*哪里*。

### 真实的例子
一个工厂方法模式现实中的使用是用在塑料玩具厂。底模粉用于制造塑料玩具是相同的，但不同的外形可以使用不同的塑料模具生产它。这就像有一个工厂模式它有一个可以输入我们想要的外形名称的地方，而且输出是我们所要求的塑料外形。玩具制造的例子如下图表所示。   
###软件例子
Django框架使用工厂方法模式创建一个表单的字段。Django的forms模块支持创建不同的种类的字段（CharField, EmailField）和自定义的参数（max_length, required）。   

###使用案例
如果你意识到你不能够追踪应用的对象创建，那是因为这些代码被写在很多地方而不是一个单独的函数或者方法，你就应该考虑使用那个工厂方法模式。工厂方法使一个对象的创建集中化，追踪对象也变得非常简单。注意，它是完完全全可以创建不止一个工厂方法的。逻辑上，每一个工厂方法组的对象创建都是相似的。例如，一个工厂方法可以负责连接不同的数据库（MySQL， SQLite），另外一个工厂方法可以负责创建你所请求的几何对象（圆形，三角形），等等。   

工厂方法在你想要从对象使用中分离对象创建时也是大有裨益的。我们在创建一个对象时不合并或者绑定到一个指定的类，通过调用一个函数我们只提供我们所想要的部分信息。这意味着将改变引入到函数很简单而且不要求任何对所使用代码的改变。    

另外的用法值得一提是一个应用的提高性能和内存使用的关联。工厂方法可以由只创建一个绝对必要的新对象来提高性能和内存利用。当我们创建对象时使用一个直接的类实例，每次一个新对象（除非，类使用内部缓存，不过通常没有这种情况）都要额外的内存来分配。我们可以看到在下面代码（文件id.py）的的实践中，它创建同一个类A的两个实例，使用id()函数去对比这些实例的内存地址。   

这些地址也在输出中打印出来，所以我们可以检验它们。如下，内存地址的实际情况是两个有明显区别的对象的以不同方法来创建：   

```python
    Class A(object):
		pass
		
	if __name__ == '__main__':
		a = A()
		b = B()
		
		print(id(a) == id(b))
		print(a, b)
```
  
在电脑上执行id.py的给出如下输入内容：  
```python
    python3 id.py 
 	False
 	<__main__.A object at 0x7f5771de8f60> <__main__.A object at 0x7f5771df2208>”
``` 	
  
注意，如果你执行文件所见到的地址和我所见到不相同，那是因为它们依赖当前内存布局和分配。但是结果必须是一样：两个地址应该是不一样的。如果你在Python的读取-求职-打印 循环（REPL）（交互提示环境）中写和执行代码会有异常发生，但是针对REPL的优化是通常不可能有的。   

###实现
数据表现有很多形式。存储和重取数据有两种主要的文件类型：人类可读的文件和二进制文件。人类可读的文件例子有XML，Atom，YAML和JSON。二进制文件的例子有被SQLite所使用的 .sq3 文件格式，用于听音乐的 .mp3 文件格式。   

这个例子中，我们会关注两个流行的人类可读格式：XML和JSON。尽管，人类可读文件通常解析较慢于二进制文件，但是它们让数据交换，检查，和修改变得非常容易。因此，建议选择使用人类可读文件，除非有其他的限制让你不允许你这样做（主要地是难以接收的性能和专有的二进制格式）。   

这个问题中，我们在XML和JSON文件里已经有一些存储的数据，我们想要解析它们，重新取回一些信息。与此同时，我们想要集中化客户这些扩展服务的连接（以及未来所有的连接）。我们会使用工厂方法解决这个问题。这个例子只关注于XML和JSON，但是添加更多的服务支持也应该是简单明了的。   

首先，让我们看看数据文件。如下，XML文件person.xml，基于Wikipedia的例子，包含单独的信息（firstName, lastName, gender，等等）：

```python
    <persons>
    <person>
	<firstName>John</firstName>
    <lastName>Smith</lastName>
    <age>25</age>
    <address>
      <streetAddress>21 2nd Street</streetAddress>
      <city>New York</city>
      <state>NY</state>
      <postalCode>10021</postalCode>
    </address>
	<phoneNumbers>
	      <phoneNumber type="home">212 555-1234</phoneNumber>
	      <phoneNumber type="fax">646 555-4567</phoneNumber>
	    </phoneNumbers>
	    <gender>
	      <type>male</type>
	    </gender>
	  </person>
	  <person>
	    <firstName>Jimy</firstName>
	    <lastName>Liar</lastName>
	    <age>19</age>
	    <address>
	      <streetAddress>18 2nd Street</streetAddress>
	      <city>New York</city>
	      <state>NY</state>
	      <postalCode>10021</postalCode>
	    </address>
	    <phoneNumbers>
	      <phoneNumber type="home">212 555-1234</phoneNumber>
	  	</phoneNumbers>
		      <gender>
		        <type>male</type>
		      </gender>
		    </person>
		    <person>
		      <firstName>Patty</firstName>
		      <lastName>Liar</lastName>
		      <age>20</age>
		      <address>
		        <streetAddress>18 2nd Street</streetAddress>
		        <city>New York</city>
		        <state>NY</state>
		        <postalCode>10021</postalCode>
		      </address>
		      <phoneNumbers>
		        <phoneNumber type="home">212 555-1234</phoneNumber>
		        <phoneNumber type="mobile">001 452-8819</phoneNumber>
		      </phoneNumbers>
		      <gender>
		        <type>female</type>
				    </gender>
				  </person>
				</persons>
```
  
JSON文件donut.json，来自Github上面的Adobe账户，  包含甜甜圈信息（type,即价格和单元，ppu, topping，等等）如下：   

```python
	[
	  {
	    "id": "0001",
	    "type": "donut",
	    "name": "Cake",
	    "ppu": 0.55,
	    "batters": {
	      "batter": [
	        { "id": "1001", "type": "Regular" },
	        { "id": "1002", "type": "Chocolate" },
	        { "id": "1003", "type": "Blue”
		berry" },
		        { "id": "1004", "type": "Devil's Food" }
		      ]
		    },
		    "topping": [
		      { "id": "5001", "type": "None" },
		      { "id": "5002", "type": "Glazed" },
		      { "id": "5005", "type": "Sugar" },
		      { "id": "5007", "type": "Powdered Sugar" },
		      { "id": "5006", "type": "Chocolate with Sprinkles" },
		      { "id": "5003", "type": "Chocolate" },
		      { "id": "5004", "type": "Maple" }
		    ]
		  },
		  {
		    "id": "0002",
		"type": "donut",
		    "name": "Raised",
		    "ppu": 0.55,
		    "batters": {
		      "batter": [
		        { "id": "1001", "type": "Regular" }
		      ]
		    },
		    "topping": [
		      { "id": "5001", "type": "None" },
		      { "id": "5002", "type": "Glazed" },
		      { "id": "5005", "type": "Sugar" },
		      { "id": "5003", "type": "Chocolate" },
		      { "id": "5004", "type": "Maple" }
		    ]
		  },
		  {
		    "id": "0003",
		    "type": "donut",
		“type": "donut",
		    "name": "Old Fashioned",
		    "ppu": 0.55,
		    "batters": {
		      "batter": [
		        { "id": "1001", "type": "Regular" },
		        { "id": "1002", "type": "Chocolate" }
		      ]
		    },
		    "topping": [
		      { "id": "5001", "type": "None" },
		      { "id": "5002", "type": "Glazed" },
		      { "id": "5003", "type": "Chocolate" },
		      { "id": "5004", "type": "Maple" }
		    ]
		  }
		]
```
  
我们会对XML和JSON使用Python发行版中的两个库的一部分：xml.etree.ElementTree和json：

```python
	import xml.etree.ElementTree as etree
	import json
```
  
JSONConnector类解析JSON文件，含有一个将所有数据作为一个字典(dict)返回的parsed_data()方法。property装饰器用来使parsed_data()作为一个正常变量出现而不是作为一个方法出现：   

```python
    Class JSONConnector:
		def __init__(self, filepath):
			self.data = dict()
			with open(filepath, mode='r', encoding='utf-8') as f:
				self.data = json.load(f)
		
		@property
		def parsed_data(self):
			return self.data
```	
  
XMLConnector类解析XML文件并含有把所有数据作为一个xml.etree.Element的列表的一个parsed_data()方法：

```python
    Class XMLConnector:
		
		def __init__(self, filepath):
			self.tree = etree.parse(filepath)
			
		@property
		def parsed_data(self):
			return self.tree
```			
  
connection_factory()函数是一个工厂方法。它基于输入文件的扩展，返回一个JSONConnector或者XMLConnector的实例：

```python
    def connection_factory(filepath):
		if filepath.endwith('json'):
			connector = JSONConnector
		elif filepath.endwith('xml'):
			connector = XMLConnector
		else:
			raise ValueError('Cannot connect to {}'.format(filepaht))
		return connector(filepath)
```
  

抽象工厂
------------
******
抽象工厂设计模式是工厂模式的归纳。基本上，抽象工厂是一组(逻辑上的)工厂方法，这里每个工厂模式都负责生成一种不同的对象。

###一个真实的例子
抽象工厂用在车辆织造中。相同的机械装置用来冲压不同的车辆模型的部件（门，面板，车身罩体，挡泥板，以及各种镜子）。聚合了不同机械装置的模型是可配置的，而且在任何时候都可以轻松变更。在下图中我们可以看到一个车辆制造的抽象工厂的例子。   

###一个软件例子
**django_factory** 包是一个为了在测试中创建Django模型的抽象工厂实现。它用来创建支持指定测试属性的模型实例。这是很重要的因为测试变得可读，以及避免共享不必要的代码。
   
##使用案例
因为抽象工模式是工厂方法模式的归纳，它具有同样的优点：使得追踪一个对象创建更容易，它让对象的使用和创建分离开，并且给我们内存使用以及应用的性能提高的可能。   
  
但是问题来了：我们如何知道什么时候使用工厂模式还是抽象工厂？答案是我们通常以更简单的工厂模式开始。如果我们发现应用需要很多的工厂模式,要让它变得有意义就要合并同族对象，那么我们就使用它。抽象工厂的好处站在用户的角度来看不是很明显，但是在使用工厂模式时，通过改变活动的工厂方法它给了我们修改动态的应用（运行中的）的能力。经典的例子是给予改变一个应用的外观和感觉（例如，Apple以及Windows这样的系统，等等），对于用户来说就是在应用使用时，不需要终止它，启动它。   

##实现
  为了阐明抽象工厂模式，我将再次使用我最喜爱的例子之一。想象我们正在开发一款游戏，或者我们想把一款小游戏加入到自己的应用中来娱乐用户。我们想要至少包含两个游戏，一个给小孩子玩，一个给成人玩。基于用户输入，我们要决定在运用时创建哪一个游戏。一个抽象工厂便能做到游戏的创建部分。   
  
  让我们从儿童游戏开始吧。这个游戏叫做FrogWorld。主角是一个喜欢迟昆虫的青蛙。每个主角都需要的一个响亮的名号，在运行时，我们的这个例子中名字是由用户给出的。如下所示， interact_with()方法用于描述青蛙和障碍物（例如，虫子，迷宫以及别的青蛙）的互动：
  
```python
    Class Frog:
		def __init__(self, name):
			self.name = name
			
		def __str__(self):
			return self.name
			
		def interact_with(self, obstacle):
			print('{} the Frog encounters {} and {}!'.format(self, obstacle, obstacle.action()))
```
  
  可以有很多种类的障碍，不过对于我们的这个例子来说可以只有一个Bug。青蛙遇到一个虫子，只有一个行为被支持：吃掉它！  
   
```python
     Class Bug:
	 	def __str__(self):
			return 'a bug'
			   
		def action(self):
			return 'eats it'
```
  
  FrogWorld类是一个抽象工厂。它的主要责任是游戏的主角和障碍物。保持创建方法的独立和方法名称的普通（例如，make_character(), make _obstacle()）让我们可以动态的改变活动工厂（以及活动的游戏）而不设计任何的代码变更。如下所示，在一个静态类型语言中，抽象工厂可以是一个有着空方法的抽象类/接口，但是在Python中不要求这么做，因为类型在运行时才被检查。
  
```python
    Class FrogWorld:
		def __inti__(self, name):
			print(self)
			self.player_name = name
			
		def __str__(self):
			return '\n\n\t------ Frog World ------'
		
		def make_character(self):
			return Forg(self.player_name)
			
		def make_obstacle(self):
			return Bug()
```			
   
   游戏WizarWorld也类似。唯一的不同是巫师不吃虫子而是与半兽人这样的怪物战斗！
   
```python
     Class Wizard:
	 	def __init__(self, name):
			self.name = name
			
		def __str__(self):
			return self.name
			
		def interact_with(self, obstacle):
			print('{} the Wizard batteles against {} and {}!'.format(self, obstacle, obstacle.action()))
			
			
	Class Ork:
		def __str__(self):
			return 'an evial ork'
			
		def action(self):
			return 'kills it'
		
		
		
	Class WizardWorld:
		def __init__(self, name):
			print(self)
			self.player_name = name
			
		def __str__(self):
			return '\n\n\t------ Wizard World ------'
			
		def make_character(self):
			return Wizard(self.player_name)
			
		def make_obstacle(self):
			return Ork()
```			
   
   GameEnviroment是我们游戏的主要入口。它接受factory作为输入，然后用它创建游戏的世界。play()方法发起创造的hero和obstacle之间的交互如下：  
   
```python
    Class GameEnviroment:
		def __init__(self, factory):
			self.hero = factory.make_character()
			self.obstacle = factory.make_obstacle()
			
		def play(self):
			self.hero.interact_with(self.obstacle)
```
  
  validate_age()函数提示用户输入一个有效的年龄。若年龄无效，则返回一个第一个元素设置为False的元组。如下所示，若输入的年龄没问题，元组的第一个元素设置为True，以及我们所关心的该元组的第二个元素，即用户给定的年龄：
   
```python
    def validate_age(name):
		try:
			age = input('Welcome {}. How old are you？'.format(name))
			age = int(age)
		except ValueError as err:
			print("Age {} is valid, plz try again...".format(age))
			return (False, age)
		return (True, age)
		
```
  
   最后，尤其是main()函数的出现。它询问用户的名字和年龄，然后由用户的年龄来决定哪一个游戏应该被运行：
 
 ```python    
	 def main():
	 	name = input("Hello. What's your name?")
		valid_input = False
		while not valid_input:
			valid_input, age = validate_age(name)
			game = FrogWorld if age < 18 else WizardWorld
			enviroment = GameEnviroment(game(name))
			enviroment.play()
```			
  
  抽象工厂实现（abstrac_factory.py）的完整代码如下：
  
  
  该程序的简单输出如下：
  
  试着扩展游戏使它更加复杂。你的思想有多远就可以走多远：更多的障碍物，更多的敌人，以及任何你喜欢的东西。
  
#总结
  在这一章，我们已经见过如何使用工厂模式和抽象工厂设计模式。两种模式都可以在我们需要时使用（a）追踪对象创建，（b）分离对象的使用和创建，甚至（c）改善一个应用的性能和资源使用。情形（c）并不在章节中阐明。你可以把它当作一个不错的练习。
  
  工厂方法设计模式以一个不属于任何类的独立函数形式实现，它负责单一种类的对象（模型，接口，等等）的创建。我们看了工厂方法如何关联到玩具厂，注意到它如何被Django用来创建不同的表单字段，讨论了它的其他可能使用场合。像一个例子那样，我们实现了一个具有访问XML和JSON格式文件的工厂方法。
  
  抽象工厂设计模式通过一组属于一个单独类的工厂方法，它用于创建关联对象的家族（汽车的零部件，游戏的环境，等等）。我们注意到抽象工厂如何与汽车制造关联起来，Django包django_factory如何使用它去创建洁净测试，并覆盖了它的使用案例。抽象工厂的实现是一个向我们如何在一个单一类中使用多个相关工厂的小游戏。
  
  在下一章，我们会谈及生成器模式，它是另外一种可以用于复杂对象创建的精细控制的创建模式。
  
