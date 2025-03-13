# python3入门

## 变量

```python
message = "Hello World"
print(message)
```

## 字符串

字符串就是一系列字符。在Python中，用引号括起来的都是字符串，其中的引号可以是单引号，也可以是双引号。例如:     
```python
message = 'Hello World'
print(message)
message = "Hello World"
print(message)
```

### 大小写

修改单词中的大小写:     

- title()方法是以首字母大写的方式显示每个单词，即将每个单词的首字母都改为大写。   
- upper()方法将字符串改为全部大写。
- lower()方法将字符串改为全部小写。   

```python
message = 'hello world'
print(message.title())
# 输出结果为Hello World
```

注意上面方法lower、upper、title等不会修改存储在变量message中的值。

### 拼接

合并: Python使用加号(+)来合并字符串。这种合并字符串的方法称为拼接。   
```python
first_name = "ada"
last_name = "lovelace"
full_name = first_name + " " + last_name
print(full_name)
# 输出： ada lovelace
```

### 空格

rstrip(): 去除字符串末尾空白。注意去除只是暂时的，等你再次访问该变量时，你会发现这个字符串仍然包含末尾空白。      
要永久删除这个字符串中的空白，必须将删除的结果存回到变量中:   


first_name = "ada  "
print(first_name)
first_name = first_name.rstrip()
print(first_name)

- lstrip(): 去除字符串开头空白       
- strip(): 同时去除字符串两端的空白      


## 整数


可对整数执行加(+)、减(-)、乘(`*`)、除(/)运算。     

Python使用两个乘号表示乘方运算。


## 浮点数

在Python中将带小数点的数字都称为浮点数。          
但是要注意的是，结果包含小数位数可能是不确定的，例如:     
```python
>>> 0.2 + 0.1
0.30000000000000004
```

命令行执行python后执行exit()或quit()可退出。     

## 类型转换

```python
age = 23
message = "Happy" + age + "Birthday"
print(message)
```

执行时会报错：  
```python
message = "Happy" + age + "Birthday"
              ~~~~~~~~^~~~~
TypeError: can only concatenate str (not "int") to str
```

类型错误，这个时候需要显式的指定希望Python将这个整数用作字符串。      
为此，可调用函数str()，它让Python将非字符串值表示为字符串： 

```python
age = 23
message = "Happy" + str(age) + "Birthday"
print(message)
```


## 注释

使用井号(#)标识注释


## 列表

在Python中，用方括号[]来表示列表，并用逗号来分割其中的元素。   
```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
print(bicycles)
print(bicycles[0])
print(bicycles[-1])
# 设置最后一个元素
bicycles[-1] = 'honda'
# 在列表最后添加元素
bicycles.append('ducati')
# 在某个位置添加元素
bicycles.insert(0, 'a')
print(bicycles)
# 删除某个位置的元素
del bicycles[0]
# 删除最后一个元素，并返回删除的值
popValue = bicycles.pop()
print(popValue)
# 通过值删除元素，如果列表中有多个重复的元素，那该方法只会删除第一个指定的值
bicycles.remove('trek')
# 排序
bicycles.sort()
# 反序排序
bicycles.sort(reverse=True)
```
Python为访问最后一个列表元素提供了一种特殊的语法，通过将索引指定为-1，可让Python返回最后一个列表元素。      
这是为了方便在不知道列表长度的情况下访问最后的元素。     
这种约定也适用于其他负数索引，例如，索引-2返回倒数第二个列表元素，索引-3返回倒数第三个列表元素，以此类推。   

### 临时排序

要保留列表元素原来的排列顺序，同时以特定的顺序呈现它们，可使用函数sorted()。       
函数sorted()让你能够按特定顺序显示列表元素，同时不影响它们在列表中的原始排列顺序。   

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
bicycles.sort(reverse=True)
# ['trek', 'specialized', 'redline', 'cannondale']
print(bicycles)
# ['cannondale', 'redline', 'specialized', 'trek']
print(sorted(bicycles))
# ['trek', 'specialized', 'redline', 'cannondale']
print(bicycles)
```


### 反转列表

```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
bicycles.reverse()
print(bicycles)
# 列表长度
size = len(bicycles)
print(size)
```

### 遍历

**Python根据缩进来判断代码行与前一个代码行的关系，在较长的Python程序中，你将看到缩进程度各不相同的代码块，这让你对程序的组织结构有大致的认识**


注意:    

- for in 后有个冒号`:`，for语句末尾的冒号告诉Python，下一行是循环的第一行。    
- 前面缩进的代码才是for循环中的部分
- 没有缩进的是for循环之外的


```python
bicycles = ['trek', 'cannondale', 'redline', 'specialized']
for bcy in bicycles:
    print(bcy)
    print(bcy.title())
print('end')
```


### 切片

你还可以处理列表的部分元素——Python称之为切片。
要创建切片，可指定要使用的第一个元素和最后一个元素的索引。
你可以生成列表的任何子集，例如，如果你要提取列表的第2～4个元素，可将起始索引指定为1，并将终止索引指定为4：

```python
players = ['charles','martina','michael','florence','eli']
print(players[1:4])
```

例如，如果要提取从第3个元素到列表末尾的所有元素，可将起始索引指定为2，并省略终止索引


### range函数

range()函数能够生成一系列的数字，例如range(1, 5)会生成1  2  3  4。     
要创建数字列表，可使用函数list()将range()的结果直接转换为列表:    
```ptyhon
numbers = list(range(1, 5))
print(numbers)

# 
print(min(numbers))
print(max(numbers))
print(sum(numbers))

```


## 元组(不可变的列表)

有时候你需要创建一系列不可修改的元素，元组可以满足这种需求。Python将不能修改的值称为不可变的，而不可变的列表被称为元组。
元组看起来犹如列表，但使用圆括号而不是方括号来标识。定义元组后，就可以使用索引来访问其元素，就像访问列表元素一样。
元组中的元素不能修改，访问和遍历都和列表一样。


相对于列表，元组是更简单的数据结构。       
如果需要存储的一组值在程序的整个生命周期内都不变，可使用元组。    



## if语句

```python
cars = ['audi', 'bmd', 'toyota']
for car in cars:
    if car == 'toyota':
        print(car.upper())
    else:
        print(car.title())
```
每条if语句的核心都是一个值为True或False的表达式，这种表达式被称为条件测试。 


```python
age = 12
if age < 4:
    print("cost $0")
elif age < 18:
    print("cost $5")
else:
    print("cost $10")
```

### 相等判断
```python
car = 'Audi'
# False
car == 'audi'
```

Python中使用两个等号(==)检测值是否相等，在Python中检测是否相等时区分大小写。   

判断两个值不相等，可结合使用惊叹号和等号(!=)来判断。   

条件语句中可包含各种数学比较，如小于<、小于等于<=、大于>、大于等于>=。   

### 多条件检测

有时候需要判断两个条件都为True或只要求一个条件为True时就执行相应的操作。     

在这些情况下，可以使用关键字and和or。    

### 值是否包含在列表中

要判断特定的值是否已包含在列表中，可使用关键字in。       
```python
cars = ['audi', 'bmd', 'toyota']
# True
print('audi' in cars)

# False
print('audi' not in cars)
```


## 字典(Key-Value)

在Python中，字典是一系列键-值对。字典用放在花括号{}中的一些列键-值对表示。           
每个键都与一个值相关联，你可以使用键来访问与之相关联的值。      
与键相关联的值可以是数字、字符串、列表乃至字典。事实上，可将任何Python对象用作字典中的值。     

```python
alien = {'color': 'blue', 'points': 5}
# 取值
print(alien['color'])
print(alien['points'])
# 添加值
alien['xPos'] = 10
alien['yPos'] = 20
print(alien['xPos'])
del alien['points']
```

### 遍历字典

- keys()方法返回所有的键列表。
- items()方法返回一个键-值对列表。    
- values()方法返回一个值列表。  

```ptyhon
alien = {'color': 'blue', 'points': 5}

for key,value in alien.items(): 
    print("key: " + key)
    print("value: " + str(value))


for key in alien.keys(): 
    print("key: " + key)    
```
字典总是明确地记录键和值之间的关联关系，但获取字典的元素时，获取顺序是不可预测的。      

这不是问题，因为通常你想要的只是获取与键相关联的正确的值。     

要以特定的顺序返回元素，一种方法是在for循环中对返回的键进行排序。。    

为此，可使用函数sorted()来获得特定顺序排列的键列表的副本:     
```python
favorite_languages = {
    'jen':'python',
    'sarah':'c',
    'edward':'ruby',
    'phil':'python',
    }
for name in sorted(favorite_languages.keys()):
    print(name.title()+",thank you for taking the poll.")
```

```python
favorite_languages = {
    'jen':'python',
    'sarah':'c',
    'edward':'ruby',
    'phil':'python',
    }
print("The following languages have been mentioned:")
for language in favorite_languages.values():
    print(language.title())
```

这种做法提取字典中所有的值，而没有考虑是否重复。涉及的值很少时，这也许不是问题，但如果被调查者很多，最终的列表可能包含大量的重复项。为剔除重复项，可使用集合(set)。集合类似于列表，但每个元素都必须是独一无二的：
```python
favorite_languages = {
    'jen':'python',
    'sarah':'c',
    'edward':'ruby',
    'phil':'python',
    }
print("The following languages have been mentioned:")
for language in set(favorite_languages.values()):❶
    print(language.title())
```
通过对包含重复元素的列表调用set()，可让Python找出列表中独一无二的元素，并使用这些元素来创建一个集合。

## 用户输入

函数input()让程序暂停运行，等待用户输入一些文本。     

获取用户输入后，Python将其存在一个变量中，以方便你使用。   
```python3
age = input("How old are you?")
print(age)
```
用户输入的是数字21，但我们请求Python提供age的值时，它返回的是`'21'`(用户输入的数值的字符串表示)。    

为了解决这个问题，可以使用函数int()：将数字的字符串表示转换为数值表示，如:  
```python
age = input("Please Input")
print(age)
# print(age >= 18)  报错
age = int(age)
print(age >= 18)
```

## while

```python
current_number = 1
while current_number <= 5:
    print(current_number)
    current_number+= 1
```

同样while中也可以结合使用break、continue等，和Java基本一样。  


## 函数

使用关键字def来定义一个函数，定义以冒号结尾。      
跟在def xxx:后面的所有缩进行构成了函数体。   

```python
def greet_user(username): 
    """函数功能的注释"""
    # 返回值
    return 'Hello ' + username 

# 调用函数    
print(greet_user('jack')) 
print(greet_user(username='lili'))
```
同样，在Python中函数也支持参数的默认值。   

上面三个引号的部分是文档字符串格式，用于简要的阐述其功能的注释。     

### 函数列表参数副本

将列表传递给函数后，函数就可对其进行修改。在函数中对这个列表所做的任何修改都是永久性的，这让你能够高效地处理大量的数据。

但是有些时候我们并不想让函数修改原始的列表。  

为解决这个问题，可向函数传递列表的副本而不是原件；这样函数所做的任何修改都只影响副本，而丝毫不影响原件。

要将列表的副本传递给函数，可以像下面这样做：
```
function_name(list_name[:])
```

切片表示法[:]创建列表的副本。

### 函数不定参数

有时候，你预先不知道函数需要接受多少个实参，Python允许函数从调用语句中收集任意数量的实参。  
```python
def make_pizza(*toppings):
    """打印顾客点的所有配料"""
    print(toppings)
make_pizza('pepperoni')
make_pizza('mushrooms','green peppers','extra cheese')
```
形参名`*toppings`中的星号让Python创建一个名为toppings的空元组，并将收到的所有值都封装到这个元组中。

现在，我们可以将这条print语句替换为一个循环，对配料列表进行遍历，并对顾客点的比萨进行描述：

```python
def make_pizza(*toppings):
    """概述要制作的比萨"""
    print("\nMaking a pizza with the following toppings:")
    for topping in toppings:
        print("- "+topping)
make_pizza('pepperoni')
make_pizza('mushrooms','green peppers','extra cheese')
```


简单来说，Python中所有的函数参数传递，统统都是基于传递对象的引用进行的。这是因为，在Python中，一切皆对象。而传对象，实质上传的是对象的内存地址，而地址即引用。


看起来，Python的参数传递方式是整齐划一的，但具体情况还得具体分析。在Python中，对象大致分为两类，即可变对象和不可变对象。可变对象包括字典、列表及集合等。不可变对象包括数值、字符串、不变集合等。如果参数传递的是可变对象，传递的就是地址，形参的地址就是实参的地址，如同“两套班子，一套人马”一样，修改了函数中的形参，就等同于修改了实参。如果参数传递的是不可变对象，为了维护它的“不可变”属性，函数内部不得不“重构”一个实参的副本。此时，实参的副本（即形参）和主调用函数提供的实参在内存中分处于不同的位置，因此对函数形参的修改，并不会对实参造成任何影响，在结果上看起来和传值一样。


看起来，Python的参数传递方式是整齐划一的，但具体情况还得具体分析。在Python中，对象大致分为两类，即可变对象和不可变对象。可变对象包括字典、列表及集合等。不可变对象包括数值、字符串、不变集合等。如果参数传递的是可变对象，传递的就是地址，形参的地址就是实参的地址，如同“两套班子，一套人马”一样，修改了函数中的形参，就等同于修改了实参。如果参数传递的是不可变对象，为了维护它的“不可变”属性，函数内部不得不“重构”一个实参的副本。此时，实参的副本（即形参）和主调用函数提供的实参在内存中分处于不同的位置，因此对函数形参的修改，并不会对实参造成任何影响，在结果上看起来和传值一样。


看起来，Python的参数传递方式是整齐划一的，但具体情况还得具体分析。在Python中，对象大致分为两类，即可变对象和不可变对象。可变对象包括字典、列表及集合等。不可变对象包括数值、字符串、不变集合等。如果参数传递的是可变对象，传递的就是地址，形参的地址就是实参的地址，如同“两套班子，一套人马”一样，修改了函数中的形参，就等同于修改了实参。如果参数传递的是不可变对象，为了维护它的“不可变”属性，函数内部不得不“重构”一个实参的副本。此时，实参的副本（即形参）和主调用函数提供的实参在内存中分处于不同的位置，因此对函数形参的修改，并不会对实参造成任何影响，在结果上看起来和传值一样。


看起来，Python的参数传递方式是整齐划一的，但具体情况还得具体分析。在Python中，对象大致分为两类，即可变对象和不可变对象。可变对象包括字典、列表及集合等。不可变对象包括数值、字符串、不变集合等。如果参数传递的是可变对象，传递的就是地址，形参的地址就是实参的地址，如同“两套班子，一套人马”一样，修改了函数中的形参，就等同于修改了实参。如果参数传递的是不可变对象，为了维护它的“不可变”属性，函数内部不得不“重构”一个实参的副本。此时，实参的副本（即形参）和主调用函数提供的实参在内存中分处于不同的位置，因此对函数形参的修改，并不会对实参造成任何影响，在结果上看起来和传值一样。


### 将函数存储在模块中

函数的优点之一是，使用它们可将代码块与主程序分离。通过给函数指定描述性名称，可让主程序容易理解得多。你还可以更进一步，将函数存储在被称为模块的独立文件中，再将模块导入到主程序中。import语句允许在当前运行的程序文件中使用模块中的代码。


要让函数是可导入的，得先创建模块。模块是扩展名为.py的文件，包含要导入到程序中的代码。下面来创建一个包含函数make_pizza()的模块。为此，我们将文件pizza.py中除函数make_pizza()之外的其他代码都删除：

```python
# pizza.py

def make_pizza(size,*toppings):
    """概述要制作的比萨"""
    print("\nMaking a "+str(size)+
          "-inch pizza with the following toppings:")
    for topping in toppings:
        print("- "+topping)
```

接下来，我们在pizza.py所在的目录中创建另一个名为making_pizzas.py的文件，这个文件导入刚创建的模块，再调用make_pizza()两次：

```python
# making_pizzas.py
import pizza
pizza.make_pizza(16,'pepperoni') ❶
pizza.make_pizza(12,'mushrooms','green peppers','extra cheese')

```
Python读取这个文件时，代码行import pizza让Python打开文件pizza.py，并将其中的所有函数都复制到这个程序中。你看不到复制的代码，因为这个程序运行时，Python在幕后复制这些代码。你只需知道，在making_pizzas.py中，可以使用pizza.py中定义的所有函数。

你还可以导入模块中的特定函数，这种导入方法的语法如下：
`from modulname import funcxx`

```python
from pizza import make_pizza
make_pizza(16,'pepperoni')
make_pizza(12,'mushrooms','green peppers','extra cheese')
```
若使用这种语法，调用函数时就无需使用句点。由于我们在import语句中显式地导入了函数make_pizza()，因此调用它时只需指定其名称。

还可以使用星号(`*`)导入模块中的所有函数:     

```python
from pizza import *
make_pizza(16,'pepperoni')
make_pizza(12,'mushrooms','green peppers','extra cheese')
```
`import *`会将模块pizza中的每个函数都复制到这个程序文件中。      
由于导入了每个函数，可通过名称来调用每个函数，而无需使用句点表示法。      
然而，使用并非自己编写的大型模块时，最好不要采用这种导入方法： 如果模块中有函数的名称与你的项目中使用的名称相同，可能导致意想不到的结果，因为Python可能遇到多个名称相同的函数或变量，进而覆盖函数，而不是分别导入所有的函数。   


### 别名

如果要导入的函数的名称与现有的名称冲突，或者函数的名称太长，可以通过别名的方式进行指定。

```python
from pizza import make_pizza as mp
mp(16,'pepperoni')
mp(12,'mushrooms','green peppers','extra cheese')
```

同样也可以通过as给模块指定别名:   

```python
import pizza as p

p.make_pizza(16,'pepperoni')
p.make_pizza(12,'mushrooms','green peppers','extra cheese')
```


## 类 

在Python中，首字母大写的名称指的是类。  
根据类来创建对象叫实例化。   

dog.py
```python
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def sit(self):
        print(self.name + " Sitting")

    def roll(self):
        print(self.name + " Rolling")
```

__init__()是一个特殊的方法，每当你根据Dog类创建新实例时，Python都会自动运行它。在这个方法中形参self必不可少，还必须位于其他形参的前面。        

为什么必须在方法定义中包含形参self呢？因为Python调用这个__init__()方法来创建Dog实例时，将自动传入实参self。    

每个与类相关联的方法调用都自动传入实参self，它是一个指向实例本身的引用，让实例能够访问类中的属性和方法。         

```python
import dog

dog = dog.Dog("xiaohei", 1)
# 属性
print(dog.name)
# 方法
dog.sit()
```


### 继承

一个类继承另一个类时，它将自动获得另一个类的所有属性和方法。       

创建子类实例时，Python首先需要完成的任务是给父类的所有属性赋值。因此，子类的__init__()方法需要调用父类的方法。   

```python
class Car:
    def __init__(self, make, model, year):
        self.make = make
        self.model = model
        self.year = year

    def get_des(self):
        name = str(self.year) + str(self.make) + str(self.model)
        return name.title()


class ElectricCar(Car):
    def __init__(self, make, model, year, battery):
        super().__init__(make, model, year)
        self.battery = battery

    # 重写父类方法
    def get_des(self):
        name = str(self.year) + str(self.make) + str(self.model) + str(self.battery)
        return name.title()
    def get_battery(self):
        print("battery : " + str(self.battery))


tesla = ElectricCar('Tesla', 'Model S', 2021, 80)
print(tesla.get_des())
tesla.get_battery()
```

- 创建子类时，父类必须包含在当前文件中，且位于子类前面。        
- 定义子类时，必须在括号内指定父类的名称
- 方法__init__()接受创建子类实例所需的信息

super()是一个特殊函数，帮助Python将父类和子类关联起来。这行代码让子类包含父类的所有属性。 


## 标准库

Python标准库是一组模块，安装的Python都包含它。
类名应采用驼峰命名法，即将类名中的每个单词的首字母都大写，而不使用下划线。实例名和模块名都采用小写格式，并在单词之间加上下划线。
每个模块也都应包含一个文档字符串，对其中的类可用于做什么进行描述。


### 读取文件

一次性读取整个文件:          
```python
with open('test.txt') as file_object:
    contents = file_object.read()
    print(contents)
```

要以每次一行的方式检查文件，可对文件对象使用for循环:     

```python
filename = 'test.txt'
with open(filename) as file_object:
    for line in file_object:
    print(line)
```    
这里使用了关键字with，让Python负责妥善地打开和关闭文件。       
使用关键字with时，open()返回的文件对象只在with代码块内可用。


要将文本写入文件，你在调用open()时需要提供另一个实参，告诉Python你要写入打开的文件。

```python
filename = 'test.txt'

with open(filename, 'w') as fo:
    fo.write("Hello World")
```    
调用open()时提供了两个实参:       

- 第一个实参是要打开的文件的名称
- 第二个实参('w')告诉Python，我们要以写入模式打开这个文件。

打开文件时，可指定读取模式('r')、写入模式('w')、附加模式('a')或让你能够读取和写入文件的模式('r+')。如果你省略了模式实参，Python将以默认的只读模式打开文件。      
如果你要写入的文件不存在，函数open()将自动创建它。然而，以写入('w')模式打开文件时千万要小心，因为如果指定的文件已经存在，Python将在返回文件对象前清空该文件。    
注意　Python只能将字符串写入文本文件。要将数值数据存储到文本文件中，必须先使用函数str()将其转换为字符串格式。

如果你要给文件添加内容，而不是覆盖原有的内容，可以附加模式打开文件。你以附加模式打开文件时，Python不会在返回文件对象前清空文件，而你写入到文件的行都将添加到文件末尾。如果指定的文件不存在，Python将为你创建一个空文件。   


### 异常

异常是使用try-except代码块处理的。try-except代码块让Python执行指定的操作，同时告诉Python发生异常时怎么办。使用了try-except代码块时，即便出现异常，程序也将继续运行：显示你编写的友好的错误消息，而不是令用户迷惑的traceback。


### 分割字符串

方法split()以空格为分隔符将字符串分拆成多个部分，并将这些部分都存储到一个列表中。

```python
title = "Alice in Wonderland"
title.split()
```

['Alice','in','Wonderland']


### pass

Python有一个pass语句，可在代码块中使用它来让Python什么都不要做。    

```python
def donothing():    
    pass    #空语句

```


pass语句如果啥事都不做，那要它有何用呢？实际上，pass可以作为一个占位符来用——视为未成熟代码的“预留地”。比如说，如果我们还没想好函数的内部实现，就可以先放一个pass，让代码先跑起来。

### json

函数json.dump()接受两个实参：要存储的数据以及可用于存储数据的文件对象。下面演示了如何使用json.dump()来存储数字列表：

```python
import json
numbers = [2,3,5,7,11,13]
filename = 'numbers.json'
with open(filename,'w') as f_obj:
    json.dump(numbers,f_obj) 
```

使用json.load()将这个列表读取到内存中：
```python
import json
filename = 'numbers.json'
with open(filename) as f_obj:
    numbers = json.load(f_obj) 
print(numbers)
```

### 网络请求

Web API是网站的一部分，用于与使用非常具体的URL请求特定信息的程序交互。这种请求称为API调用。请求的数据将以易于处理的格式（如JSON或CSV）返回。依赖于外部数据源的大多数应用程序都依赖于API调用，如集成社交媒体网站的应用程序。



requests包让Python程序能够轻松地向网站请求信息以及检查返回的响应。要安装requests，请执行类似于下面的命令：

$pip install --user requests
或者直接在ide中点击修复安装就可以:  

![Image](https://raw.githubusercontent.com/CharonChui/Pictures/master/python_install_requests.png)

```python
import requests
# 执行API调用并存储响应
url = 'https://api.github.com/search/repositories?q=language:python&sort=stars'
r = requests.get(url)
print("Status code:",r.status_code)
# 将API响应存储在一个变量中
response_dict = r.json()
# 处理结果
print(response_dict.keys())
```

执行结果为:    
```
Status code: 200
dict_keys(['total_count', 'incomplete_results', 'items'])
```

这个API返回JSON格式的信息，因此我们使用方法json()将这些信息转换为一个Python字典。我们将转换得到的字典存储在response_dict中。
最后，我们打印response_dict中的键。




在Python中，还可以定义可变参数。可变参数也称不定长参数，即传入函数中的实际参数可以是零个、一个、两个到任意个。

Python中的可变参数的表现形式为，在形参前添加一个星号（*），意为函数传递过来的实参个数不定，可能为0个、1个，也可能为n个（n≥2）。需要注意的是，不管可变参数有多少个，在函数内部，它们都被“收集”起来并统一存放在以形参名为某个特定标识符的元组之中。因此，可变参数也被称为“收集参数”。


除了用单个星号（*）表示可变参数，其实还有另一种标定可变参数的形式，即用两个星号（**）来标定。通过前文的介绍，我们知道，一个星号（*）将多个参数打包为一个元组，而两个星号（**）的作用是什么呢？它的作用就是把可变参数打包成字典模样。

这时调用函数则需要采用如“arg1=value1,arg2=value2”这样的形式。等号左边的参数好比字典中的键（key）[插图]，等号右边的数值好比字典中的值（value）​，

```python
def varFun(**x): 
    if len(x) == 0:
        print("None")
    else:
        print(x)


varFun(a = 1, b = 3)


```

定义可变参数时，主要有两种形式，一种是*parameter，另一种是**parameter。下面分别进行介绍。
1．*parameter
这种形式表示接收任意多个实际参数并将其放到一个元组中。
2．**parameter
这种形式表示接收任意多个类似关键字参数一样显式赋值的实际参数，并将其放到一个字典中。


除了用等号给可变关键字参数赋值，事实上，我们还可以直接用字典给可变关键字参数赋值:     

```python
def some_kwargs(name, age, sex):
    print(f"name: ${name}")

kdic = {'name' : 'Alice', 'age' : 11, 'sex' : 'nv'}
some_kwargs(**kdic)

```


在Python中，可以通过@property（装饰器）将一个方法转换为属性，从而实现用于计算的属性。将方法转换为属性后，可以直接通过方法名来访问方法，而不需要再添加一对小括号“()”，这样可以让代码更加简洁。
