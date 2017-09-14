Android编码规范
===

介绍
---

1. 为什么需要编码规范
	编码规范对于程序员而言尤为重要，有以下几个原因： 
	- 一个软件的生命周期中，80%的花费在于维护 
	- 几乎没有任何一个软件，在其整个生命周期中，均由最初的开发人员来维护
	- 编码规范可以改善软件的可读性，可以让程序员尽快而彻底地理解新的代码
	
源文件规范
---

1. 文件名
	源文件名必须和它包含的顶层类名保持一致，包括大小写，并以.java作为后缀名。

2. 文件编码
	所有源文件编码必须是`UTF-8`
	
	
	
命名
---

1. 包名
	命名规则：一个唯一包名的前缀总是全部小写的`ASCII`字母并且是一个**顶级域名**,通常是`com,edu,gov,mil,net,org`。      
	包名的后续部分根据不同机构各自内部的命名规范而不尽相同。这类命名规范可能以特定目录名的组成来区分部门 `(department)`,      
	项目`(project)`,机器`(machine)`,或注册名`(login names)`.         
	例如： `com.domain.xx`           
	
    包命名必须以`com.domain`开始,后面跟有项目名称（或者缩写）,再后面为模块名或层级名称。           
		如：`com.domain.项目缩写.模块名` `com.domain.xx.bookmark`        
		如：`com.domain.项目缩写.层级名` `com.domain.xx.activity`          

2. 类和接口命名
	命名规则：类名是个一名词，采用大小写混合的方式，每个单词的首字母大写。尽量使你的类名简洁而富于描述。使用完整单词，
	避免缩写词(除非该缩写词被更广泛使用，像 URL，HTML)        
	
	接口一般要使用`able`,`ible`,`er`等后缀              

	类名必须使用**驼峰规则**，即首字母必须大写，如果为词组，则每个单词的首字母也必须要大写，类名必须使用名词，或名词词组。
	要求类名简单，不允许出现无意义的单词。

3. 方法的命名
	命名规则：方法名是一个动词，采用大小写混合的方式，第一个单词的首字母小写，其后单词的首字母大写。
	例如： `public void run(); public String getBookName();`     
	
	类中常用方法的命名：              
		- 类的获取方法（一般具有返回值）一般要求在被访问的字段名前加上`get`.     
		    如`getFirstName()`,`getLastName()`.
			一般来说,`get`前缀方法返回的是单个值,`find`前缀的方法返回的是列表值.
			
		- 类的设置方法(一般返回类型为`void`)：被访问字段名的前面加上前缀 `set`.    
			如`setFirstName()`,`setLastName()`.
			
		- 类的布尔型的判断方法一般要求方法名使用单词`is`或`has`做前缀.      
			如`isPersistent()`,`isString()`.或者使用具有逻辑意义的单词,例如`equal`或`equals`.
			
		- 类的普通方法一般采用完整的英文描述说明成员方法功能,第一个单词尽可能采用动词,首字母小写.     
			如`openFile()`, `addCount()`.
		- 构造方法应该用递增的方式写,(参数多的写在后面).
		- `toString()`方法：一般情况下，每个类都应该定义`toString()`.

4. 变量命名
	命名规则：第一个单词的首字母小写，其后单词的首字母大写。变量名不应以下划线或美元符号开头，尽管这在语法上是允许的。变量名应简短且富于描述。
	变量名的选用应该易于记忆，即，能够指出其用途。尽量避免单个字符的变量名，除非是一次性的临时变量。临时变量通常被取名为 `i,j,k,m,n`它们一般用于整型；
	`c,d,e` 它们一般用于字符型。         
	在`Android`中成员变量
	非`public`非`static`的变量可以使用`m`开头
	非常量的`static`变量可以使用`s`开头
	

	变量命名也必须使用**驼峰规则**，但是首字母必须小写，变量名尽可能的使用名词或名词词组。同样要求简单易懂，不允许出现无意义的单词。
	例如：`private String mBookName; ` 
	
5. 常量命名
	命名规则：类常量的声明，应该全部大写，单词间用下划线隔开。
	例如：`private static final int MIN_WIDTH = 4;`
	
6. 异常命名
	自定义异常的命名必须以`Exception`为结尾。已明确标示为一个异常。

7. layout命名
	`layout.xml`的命名必须以全部单词小写，单词间以下划线分割，并且使用名词或名词词组，即使用 模块名_功能名称_所属页面类型 来命名。
	如：`video_controller_player_activity`     视频模块下的-控制栏-属于播放器的-Activity页
	
8. id命名
	·layout`中所使用的`id`必须以全部单词小写，单词间以下划线分割，并且使用名词或名词词组，并且要求能够通过`id`直接理解当前组件要实现的功能。       
	如：某`TextView @+id/tv_book_name_show`         
	如：某`EditText @+id/`et_book_name_edit`          

9. 资源命名
	`layout`中所使用的所有资源(如`drawable`,`style`等),命名必须以全部单词小写，单词间以下划线分割，并且尽可能的使用名词或名词组，        
	即使用 模块名_用途 来命名。如果为公共资源，如分割线等，则直接用用途来命名                
	如：`menu_icon_navigate.png`      
	如：某分割线：`line.png` 或 `separator.png`        

注释
---

`Java`程序有两类注释：实现注释`(implementation comments)`和文档注释`(document comments)`。
实现注释是使用/*...*/和//界定的注释。文档注释(被称为"doc comments")由/**...*/界定。文档注释可以通过javadoc 工具转换成HTML 文件。

1. 类注释
	每一个类都要包含如下格式的注释，以说明当前类的功能等。
	/**
	 * 类名
	 * @author 作者 <br/>
	 *	实现的主要功能。
	 *	创建日期
	 *	修改者，修改日期，修改内容。
	 */

2. 方法注释
	每一个方法都要包含 如下格式的注释 包括当前方法的用途，当前方法参数的含义，当前方法返回值的内容和抛出异常的列表。
	/**
	 * 
	 * 方法的一句话概述
	 * <p>方法详述（简单方法可不必详述）</p>
	 * @param s 说明参数含义
	 * @return 说明返回值含义
	 * @throws IOException 说明发生此异常的条件
	 * @throws NullPointerException 说明发生此异常的条件
	 */

3. 类成员变量和常量注释
	成员变量和常量需要使用`java doc`形式的注释，以说明当前变量或常量的含义
	/**
	 * XXXX含义
	 */

4. 其他注释
	方法内部的注释 如果需要多行 使用/*…… */形式，如果为单行是用//……形式的注释。
	不要再方法内部使用 java doc 形式的注释“/**……*/”

代码风格
---

1. 缩进
    除了换行符之外，ASCII空格（0x20）是唯一合法的空格字符。这意味着
    不允许使用`Tab`进行缩进，应该使用空格进行缩进，推荐缩进为4个空格		
	`Eclipse`中将`Tab`替换为4个空格的设置方法(很多人都习惯直接按4次空格，感觉不设置习惯了也挺好)
		- 代码设置
			`Window->Preferences->General->Editors->Text Editors->`勾选`Insert spaces for tabs``
		- XML文件的Tab配置
			`Window->Preferences->XML->XML Files->Editor>`选择右侧区域的`Indent using spaces`


2. 空行
	空行将逻辑相关的代码段分隔开，以提高可读性。       
	
 	下列情况应该总是使用空行： 
	- 一个源文件的两个片段之间
 	- 类声明和接口声明之间
	- 两个方法之间
	- 方法内的局部变量和方法的第一条语句之间
	- 一个方法内的两个逻辑段之间，用以提高可读性  

    通常在 变量声明区域之后要用空行分隔，常量声明区域之后要有空行分隔，方法声明之前要有空行分隔。

3. 方法
 	- 一个方法尽量不要超过15行(可能会有难度，但是尽量不要太多，弄个方法几千行这是绝对不允许的)，如果方法太长，说明当前方法业务逻辑已经非常复杂，
		那么就需要进行方法拆分，保证每个方法只作一件事。
	- 不要使用`try catch`处理业务逻辑！！！！

4. 参数和返回值
    - 一个方法的参数尽可能的不要超过4个(根据情况可能也会有些难度)
    - 如果一个方法返回的是一个错误码，请使用异常！！
    - 尽可能不要使用`null`替代为异常

5. 神秘数字
	代码中不允许出现单独的数字，字符！如果需要使用数字或字符，则将它们按照含义封装为静态常量!(`for`语句中除外)
	
6. 控制语句
	判断中如有常量，则应将常量置于判断式的右侧。如： 
	`if (true == isAdmin())...`
	尽量不要使用三目条件的嵌套。

	在`if`、`else`、`for`、`do`和`while`语句中，即使没有语句或者只有一行，也不得省略花括号：
	```java
	if (true){
		//do something......
	}
	```
	不要使用下面的方式:
	```java
	if (true)
		i = 0; //不要使用这种
	```
	
	对于循环：
	//将操作结构保存在临时变量里，减少方法调用次数
	```java
	final int count = products.getCount();
	while(index < count){
		// ...
	}
	```
	而不是
	```java
	while(index < products.getCount()){
		//每此都会执行一次getCount()方法，
		//若此方法耗时则会影响执行效率
		//而且可能带来同步问题，若有同步需求，请使用同步块或同步方法
	}
	```


7. 访问控制
	若没有足够理由，不要把实例或类变量声明为公有。
	
8. 变量赋值
	不要使用内嵌(embedded)赋值运算符试图提高运行时的效率，这是编译器的工作。例如： 
	 `d = (a = b + c) + r;`
	应该写成 
	```java
	a = b + c; 
	d = a + r; 
	```
	
9. 圆括号的试用
	一般而言，在含有多种运算符的表达式中使用圆括号来避免运算符优先级问题，是个好方法。
	即使运算符的优先级对你而言可能很清楚，但对其他人未必如此。你不能假设别的程序员和你一样清楚运算符的优先级。 
	不要这样写：
	`if (a == b && c == d)`
	正确的方式为： 
	`if ((a == b) && (c == d))`

10. 返回值
	设法让你的程序结构符合目的。例如： 
	```java
	if (booleanExpression) { 
		return true; 
	} else { 
		return false; 
	} 
	```
	应该代之以如下方法： 
	`return booleanExpression`

	类似地： 
	```java
	if (condition) { 
		return x; 
	} 
	return y; 
	```
	应该写做： 
    `return (condition ? x : y);`

11. 条件运算符`?`前的表达式    
	如果一个包含二元运算符的表达式出现在三元运算符" ? : "的"?"之前，那么应该给表达式添上一对圆括号。例如：  
	`(x >= 0) ? x : -x`

12.  所有未使用的import语句应该被删除。	

13. 重载（Overload）方法必须放在一起

14. 非空块中花括号的使用
	在非空代码块中使用花括号时要遵循`K&R`风格`(Kernighan and Ritchie Style)：
	左花括号（{）前不能换行，在其后换行。
	在右花括号（}）前要有换行。
	
15. 空代码块中花括号的使用	
	如果一个代码块是空的，可以直接使用`{}`。除了`if/else-if/else`和`try/catch/finally`这样的多块语句.
	
16. 列宽
	列宽必须为120字符，以下情况可以不遵守列宽限制：
	无法限制宽度的内容，比如注释里的长`URL`
	`package`和`import`语句
	注释中需要被粘贴到`Shell`里去执行的命令

17. 枚举
	用逗号分割每个枚举变量，并且变量要单独在一行
	```java
	enum Color {
		RED,
		GREEN,
		YELLOW
	}
	```
	
18. 长整形数字
	长整型数字必须使用大写字母`L`结尾，不能使用小写字母`l`，以便和数字1进行区分。例如使用3000000000L而不是3000000000l
  
  
开发格式统一
---

1. Eclipse
    `Windows -> Preferences -> Java -> Code Style`      
	然后选择`Import`导入相应的`Clean Up`、`Code Templates`、`Formatter`等XML文件。     
	如果不需要Copyright信息，想要自定义的，可以不导入Code Templates。
	
2. IDEA 
	`File -> Import Settings`选择下载链接中的`IDEA_Style.jar`文件，
	可以看到两个选项，只需代码风格的，可以仅选择`Code style schemes`,
	如果需要默认的`Copyright`信息，选择`Default Project settings`。


代码严谨性要求
---

1. `ArrayList`通过`get`方法使用下标获取元素，如果使用的下标不在`ArrayList`大小范围内，将产生`java.lang.IndexOutOfBoundsException`的异常，导致`app`出现`Crash`。

2. 方法中存在`return null`返回对象直接进行方法调用隐患, 在使用时需要先判断是否为`null`，一般尽量不要在方法中直接`return null`，最好用异常代替。

3. 销毁`Dialog`前是否`isShowing`未判断隐患
	调用`Android.app.Dialog.cancel()`方法前，如果这个`dialog`不处于`showing`状态时，会抛出`java.lang.IllegalArgumentException`的异常，导致`app`出现`Crash`。

4. 使用`String.split`结果未判断长度隐患
	在使用`String.split`得到的结果数组前，未对数组进行长度检查，取字段的时候可能发生越界而导致`Crash`。
	```java
	String source = "<br/>";
	String[] infos = source.split("<br/>");
	if(0 < infos.length){
	   String poiName = infos[0];
	}
	```




---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 