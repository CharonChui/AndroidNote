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

title()方法是以首字母大写的方式显示每个单词，即将每个单词的首字母都改为大写。   
upper()方法将字符串改为全部大写。
lower()方法将字符串改为全部小写。   
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

rstrip(): 去除字符串末尾空白。注意去除只是暂时的，等你再次访问该变量是，你会发现这个字符串仍然包含末尾空白。      
要永久删除这个字符串中的空白，必须将删除的结果存回到变量中:   


first_name = "ada  "
print(first_name)
first_name = first_name.rstrip()
print(first_name)

lstrip(): 去除字符串开头空白       
strip(): 同时去除字符串两端的空白      


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


- 
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


## 字典(Map)


