Java基础面试题
===

本部分全部内容是根据张孝祥老师的Word文档整理而来。只不过是为了方便观看，把代码部分用`markdown`来展示。整理时脑海中不断回忆起张老师上课的情景，真是怀念。

1. 一个`.java`源文件中是否可以包括多个类（不是内部类）？有什么限制？            
    可以有多个类，但只能有一个public的类，并且public的类名必须与文件名相一致。
2. `Java`有没有`goto`?       
    `java`中的保留字，现在没有在`java`中使用。
3. 说说`&`和`&&`的区别。        
	`&`和`&&`都可以用作逻辑与的运算符，表示逻辑与`(and)`，当运算符两边的表达式的结果都为`true`时，整个运算结果才为`true`，
    否则，只要有一方为`false`，则结果为`false`。
	`&&`还具有短路的功能，即如果第一个表达式为`false`，则不再计算第二个表达式，例如，对于`if(str != null && !str.equals(“”))`表达式，当`str`为`null`时，
	后面的表达式不会执行，所以不会出现`NullPointerException`如果将`&&`改为`&`，则会抛出`NullPointerException`异常。

4. 在`JAVA`中如何跳出当前的多重嵌套循环？         
	在`Java`中，要想跳出多重循环，可以在外面的循环语句前定义一个标号，然后在里层循环体的代码中使用带有标号的`break`语句，即可跳出外层循环。例如，
	```java
	ok:
	for(int i=0;i<10;i++) {
		for(int j=0;j<10;j++) {
			System.out.println(“i=” + i + “,j=” + j);
			if(j == 5) break ok;
		}
	} 
	```
	另外，我个人通常并不使用标号这种方式，而是让外层的循环条件表达式的结果可以受到里层循环体代码的控制，例如，要在二维数组中查找到某个数字。
	```java
	int arr[][] = ...;
	boolean found = false;
	for(int i=0;i<arr.length && !found;i++) {
			for(int j=0;j<arr[i].length;j++) {
				System.out.println(“i=” + i + “,j=” + j);
				if(arr[i][j]  == 5) {
					found = true;
					break;
				}
			}
		} 
	}
	```
5. `switch`语句能否作用在`byte`上，能否作用在`long`上，能否作用在`String`上?                   
	在`switch（expr1）`中，`expr1`只能是一个整数表达式或者枚举常量（更大字体），整数表达式可以是`int`基本类型或`Integer`包装类型，由于，
	`byte`,`short`,`char`都可以隐含转换为`int`，所以，这些类型以及这些类型的包装类型也是可以的。显然，`long`和`String`类型都不符合`switch`的语法规定，
	并且不能被隐式转换成`int`类型，所以，它们不能作用于`swtich`语句中。 
6. `short s1 = 1; s1 = s1 + 1`;有什么错? `short s1 = 1; s1 += 1`;有什么错?               
	对于`short s1 = 1; s1 = s1 + 1`; 由于`s1+1`运算时会自动提升表达式的类型，所以结果是`int`型，再赋值给`short`类型`s1`时，编译器将报告需要强制转换类型的错误。
	对于`short s1 = 1; s1 += 1`;由于`+=`是`java`语言规定的运算符，`java`编译器会对它进行特殊处理，因此可以正确编译。 
7. `char`型变量中能不能存贮一个中文汉字?为什么?               
	`char`型变量是用来存储`Unicode`编码的字符的，`unicode`编码字符集中包含了汉字，所以`char`型变量中当然可以存储汉字啦。不过，
	如果某个特殊的汉字没有被包含在`unicode`编码字符集中，那么这个`char`型变量中就不能存储这个特殊汉字。补充说明：`unicode`编码占用两个字节，
	所以`char`类型的变量也是占用两个字节。
	备注：后面一部分回答虽然不是在正面回答题目，但是，为了展现自己的学识和表现自己对问题理解的透彻深入，可以回答一些相关的知识，做到知无不言，言无不尽。 
8. 用最有效率的方法算出2乘以8等於几?             
	`2 << 3`，            
	因为将一个数左移`n`位，就相当于乘以了`2`的`n`次方，那么，一个数乘以8只要将其左移3位即可，而位运算`cpu`直接支持的，效率最高，
	所以2乘以8等於几的最效率的方法是`2 << 3`。
9. 请设计一个一百亿的计算器                   
	首先要明白这道题目的考查点是什么，一是大家首先要对计算机原理的底层细节要清楚、要知道加减法的位运算原理和知道计算机中的算术运算会发生越界的情况，
	二是要具备一定的面向对象的设计思想。首先，计算机中用固定数量的几个字节来存储的数值，所以计算机中能够表示的数值是有一定的范围的，
	为了便于讲解和理解，我们先以`byte` 类型的整数为例，它用1个字节进行存储，表示的最大数值范围为-128到+127。-1在内存中对应的二进制数据为11111111，
	如果两个-1相加，不考虑Java运算时的类型提升，运算后会产生进位，二进制结果为1,11111110，由于进位后超过了byte类型的存储空间，所以进位部分被舍弃，
	即最终的结果为11111110，也就是-2，这正好利用溢位的方式实现了负数的运算。-128在内存中对应的二进制数据为10000000，如果两个-128相加，
	不考虑Java运算时的类型提升，运算后会产生进位，二进制结果为1,00000000，由于进位后超过了byte类型的存储空间，所以进位部分被舍弃，
	即最终的结果为00000000，也就是0，这样的结果显然不是我们期望的，这说明计算机中的算术运算是会发生越界情况的，
	两个数值的运算结果不能超过计算机中的该类型的数值范围。由于Java中涉及表达式运算时的类型自动提升，
	我们无法用byte类型来做演示这种问题和现象的实验，大家可以用下面一个使用整数做实验的例子程序体验一下：        
	```java
	int a = Integer.MAX_VALUE;          
	int b = Integer.MAX_VALUE;           
	int sum = a + b;            
	System.out.println(“a=”+a+”,b=”+b+”,sum=”+sum);             
	```
	先不考虑long类型，由于int的正数范围为2的31次方，表示的最大数值约等于2*1000*1000*1000，也就是20亿的大小，所以，要实现一个一百亿的计算器，
	我们得自己设计一个类可以用于表示很大的整数，并且提供了与另外一个整数进行加减乘除的功能，大概功能如下：            
	（）这个类内部有两个成员变量，一个表示符号，另一个用字节数组表示数值的二进制数       
	（）有一个构造方法，把一个包含有多位数值的字符串转换到内部的符号和字节数组中          
	（）提供加减乘除的功能    
	```java
	public class BigInteger {
		int sign;
		byte[] val;
		public Biginteger(String val) {
			sign = ;
			val = ;
		}
		public BigInteger add(BigInteger other) {
		}
		public BigInteger subtract(BigInteger other) {
		}
		public BigInteger multiply(BigInteger other) {
		}
		public BigInteger divide(BigInteger other) {
		}
	}
	```
	备注：要想写出这个类的完整代码，是非常复杂的，如果有兴趣的话，可以参看jdk中自带的java.math.BigInteger类的源码。
	面试的人也知道谁都不可能在短时间内写出这个类的完整代码的，他要的是你是否有这方面的概念和意识，他最重要的还是考查你的能力，
	所以，你不要因为自己无法写出完整的最终结果就放弃答这道题，你要做的就是你比别人写得多，证明你比别人强，你有这方面的思想意识就可以了，
	毕竟别人可能连题目的意思都看不懂，什么都没写，你要敢于答这道题，即使只答了一部分，那也与那些什么都不懂的人区别出来，拉开了距离，算是矮子中的高个，
	机会当然就属于你了。另外，答案中的框架代码也很重要，体现了一些面向对象设计的功底，特别是其中的方法命名很专业，用的英文单词很精准，
	这也是能力、经验、专业性、英语水平等多个方面的体现，会给人留下很好的印象，在编程能力和其他方面条件差不多的情况下，英语好除了可以使你获得更多机会外，
	薪水可以高出一千元。 
10. 使用`final`关键字修饰一个变量时，是引用不能变，还是引用的对象不能变？           
	使用`final`关键字修饰一个变量时，是指引用变量不能变，引用变量所指向的对象中的内容还是可以改变的。例如，对于如下语句：
	`final StringBuffer a=new StringBuffer("immutable");`         
	执行如下语句将报告编译期错误：         
	`a=new StringBuffer("");`         
	但是，执行如下语句则可以通过编译：        
	`a.append(" broken!");`              
	有人在定义方法的参数时，可能想采用如下形式来阻止方法内部修改传进来的参数对象：        
	```java
	public void method(final  StringBuffer  param) {       
	} 
	```
	实际上，这是办不到的，在该方法内部仍然可以增加如下代码来修改参数对象：
	`param.append("a");`
11. `==`和`equals`方法究竟有什么区别？    
	（单独把一个东西说清楚，然后再说清楚另一个，这样，它们的区别自然就出来了，混在一起说，则很难说清楚）
	`==`操作符专门用来比较两个变量的值是否相等，也就是用于比较变量所对应的内存中所存储的数值是否相同，要比较两个基本类型的数据或两个引用变量是否相等，
	只能用`==`操作符。如果一个变量指向的数据是对象类型的，那么，这时候涉及了两块内存，对象本身占用一块内存（堆内存），变量也占用一块内存，
	例如`Objet obj = new Object();`变量obj是一个内存，new Object()是另一个内存，此时，变量obj所对应的内存中存储的数值就是对象占用的那块内存的首地址。
	对于指向对象类型的变量，如果要比较两个变量是否指向同一个对象，即要看这两个变量所对应的内存中的数值是否相等，这时候就需要用==操作符进行比较。
	equals方法是用于比较两个独立对象的内容是否相同，就好比去比较两个人的长相是否相同，它比较的两个对象是独立的。例如，对于下面的代码：            
	`String a=new String("foo");`           
	`String b=new String("foo");`      
	两条new语句创建了两个对象，然后用a,b这两个变量分别指向了其中一个对象，这是两个不同的对象，它们的首地址是不同的，即a和b中存储的数值是不相同的，
	所以，表达式a==b将返回false，而这两个对象中的内容是相同的，所以，表达式a.equals(b)将返回true。
	在实际开发中，我们经常要比较传递进行来的字符串内容是否等，例如，String input = …;input.equals(“quit”)，许多人稍不注意就使用==进行比较了，
	这是错误的，随便从网上找几个项目实战的教学视频看看，里面就有大量这样的错误。记住，字符串的比较基本上都是使用equals方法。
	如果一个类没有自己定义equals方法，那么它将继承Object类的equals方法，Object类的equals方法的实现代码如下：       
	```java
	boolean equals(Object o){
		return this==o;
	}
	```
	这说明，如果一个类没有自己定义equals方法，它默认的equals方法（从Object 类继承的）就是使用==操作符，也是在比较两个变量指向的对象是否是同一对象，
	这时候使用equals和使用==会得到同样的结果，如果比较的是两个独立的对象则总返回false。如果你编写的类希望能够比较该类创建的两个实例对象的内容是否相同，
	那么你必须覆盖equals方法，由你自己写代码来决定在什么情况即可认为两个对象的内容是相同的。
12. 静态变量和实例变量的区别？ 
	在语法定义上的区别：静态变量前要加`static`关键字，而实例变量前则不加。     
	在程序运行时的区别：实例变量属于某个对象的属性，必须创建了实例对象，其中的实例变量才会被分配空间，才能使用这个实例变量。静态变量不属于某个实例对象，
	而是属于类，所以也称为类变量，只要程序加载了类的字节码，不用创建任何实例对象，静态变量就会被分配空间，静态变量就可以被使用了。
	总之实例变量必须创建对象后才可以通过这个对象来使用，静态变量则可以直接使用类名来引用。
	例如，对于下面的程序，无论创建多少个实例对象，永远都只分配了一个`static Var`变量，并且每创建一个实例对象，这个staticVar就会加1；但是，每创建一个实例对象，
	就会分配一个instanceVar，即可能分配多个instanceVar，并且每个instanceVar的值都只自加了1次。
	```java
	public class VariantTest {
		public static int staticVar = 0; 
		public int instanceVar = 0; 
		public VariantTest()
		{
			staticVar++;
			instanceVar++;
			System.out.println(“staticVar=” + staticVar + ”,instanceVar=” + instanceVar);
		}
	}
	```
	备注：这个解答除了说清楚两者的区别外，最后还用一个具体的应用例子来说明两者的差异，体现了自己有很好的解说问题和设计案例的能力，思维敏捷，
	超过一般程序员，有写作能力！
13. 是否可以从一个static方法内部发出对非static方法的调用？                
	不可以。因为非static方法是要与对象关联在一起的，必须创建一个对象后，才可以在该对象上进行方法调用，而static方法调用时不需要创建对象，可以直接调用。
	也就是说，当一个static方法被调用时，可能还没有创建任何实例对象，如果从一个static方法中发出对非static方法的调用，那个非static方法是关联到哪个对象上的呢？
	这个逻辑无法成立，所以，一个static方法内部发出对非static方法的调用。
14. Integer与int的区别                   
	int是java提供的8种原始数据类型之一。Java为每个原始类型提供了封装类，Integer是java为int提供的封装类。int的默认值为0，而Integer的默认值为null，
	即Integer可以区分出未赋值和值为0的区别，int则无法表达出未赋值的情况，例如，要想表达出没有参加考试和考试成绩为0的区别，则只能使用Integer。
	在JSP开发中，Integer的默认为null，所以用el表达式在文本框中显示时，值为空白字符串，而int默认的默认值为0，所以用el表达式在文本框中显示时，结果为0，
	所以，int不适合作为web层的表单数据的类型。在Hibernate中，如果将OID定义为Integer类型，那么Hibernate就可以根据其值是否为null而判断一个对象是否是临时的，
	如果将OID定义为了int类型，还需要在hbm映射文件中设置其unsaved-value属性为0。另外，Integer提供了多个与整数相关的操作方法，例如，将一个字符串转换成整数，
	Integer中还定义了表示整数的最大值和最小值的常量。
15. Math.round(11.5)等於多少? Math.round(-11.5)等於多少?                  
	Math类中提供了三个与取整有关的方法：ceil、floor、round，这些方法的作用与它们的英文名称的含义相对应，例如，ceil的英文意义是天花板，
	该方法就表示向上取整，所以，Math.ceil(11.3)的结果为12,Math.ceil(-11.3)的结果是-11；floor的英文意义是地板，该方法就表示向下取整，
	所以，Math.floor(11.6)的结果为11,Math.floor(-11.6)的结果是-12；最难掌握的是round方法，它表示“四舍五入”，算法为Math.floor(x+0.5)，
	即将原来的数字加上0.5后再向下取整，所以，Math.round(11.5)的结果为12，Math.round(-11.5)的结果为-11。
15. 下面的代码有什么不妥之处?    
    ```java
	if(username.equals(“zxx”){}
	int  x = 1;
	    return x==1?true:false;
    ```
	这个我就不说了吧，你能看出来的。
16. Overload和Override的区别。Overloaded的方法是否可以改变返回值的类型?          
	Overload是重载的意思，Override是覆盖的意思，也就是重写。     
	重载Overload表示同一个类中可以有多个名称相同的方法，但这些方法的参数列表各不相同（即参数个数或类型不同）。
	重写Override表示子类中的方法可以与父类中的某个方法的名称和参数完全相同，通过子类创建的实例对象调用这个方法时，将调用子类中的定义方法，
	这相当于把父类中定义的那个完全相同的方法给覆盖了，这也是面向对象编程的多态性的一种表现。子类覆盖父类的方法时，只能比父类抛出更少的异常，
	或者是抛出父类抛出的异常的子异常，因为子类可以解决父类的一些问题，不能比父类有更多的问题。子类方法的访问权限只能比父类的更大，不能更小。
	如果父类的方法是private类型，那么，子类则不存在覆盖的限制，相当于子类中增加了一个全新的方法。
	至于Overloaded的方法是否可以改变返回值的类型这个问题，要看你倒底想问什么呢？这个题目很模糊。如果几个Overloaded的方法的参数列表不一样，
	它们的返回者类型当然也可以不一样。但我估计你想问的问题是：如果两个方法的参数列表完全一样，是否可以让它们的返回值不同来实现重载Overload。
	这是不行的，我们可以用反证法来说明这个问题，因为我们有时候调用一个方法时也可以不定义返回结果变量，即不要关心其返回结果，
	例如我们调用map.remove(key)方法时，虽然remove方法有返回值，但是我们通常都不会定义接收返回结果的变量，这时候假设该类中有两个名称和参数列表完全相同的方法，
	仅仅是返回类型不同，java就无法确定编程者倒底是想调用哪个方法了，因为它无法通过返回结果类型来判断。 
17. 一个房子里有椅子，椅子有腿和背，房子与椅子是什么关系，椅子与腿和背是什么关系？                    
	如果房子有多个椅子，就是聚合关系，否则是一种关联关系，当然，聚合是一种特殊的关联。椅子与腿和背时组合关系。
	说说has a与is a的区别。        
	答：is-a表示的是属于得关系。比如兔子属于一种动物（继承关系）。has-a表示组合，包含关系。比如兔子包含有腿，头等组件；
18. 分层设计的好处；             
	把各个功能按调用流程进行了模块化，模块化带来的好处就是可以随意组合，举例说明：如果要注册一个用户，
	流程为显示界面并通过界面接收用户的输入，接着进行业务逻辑处理，在处理业务逻辑又访问数据库，如果我们将这些步骤全部按流水帐的方式放在一个方法中编写，
	这也是可以的，但这其中的坏处就是，当界面要修改时，由于代码全在一个方法内，可能会碰坏业务逻辑和数据库访问的码，同样，当修改业务逻辑或数据库访问的代码时，
	也会碰坏其他部分的代码。分层就是要把界面部分、业务逻辑部分、数据库访问部分的代码放在各自独立的方法或类中编写，这样就不会出现牵一发而动全身的问题了。
	这样分层后，还可以方便切换各层，譬如原来的界面是Swing，现在要改成BS界面，如果最初是按分层设计的，这时候不需要涉及业务和数据访问的代码，
	只需编写一条web界面就可以了。
   下面的仅供参考，不建议照搬照套，一定要改成自己的语言，发现内心的感受：             
	分层的好处：            
	- 实现了软件之间的解耦；
	- 便于进行分工
	- 便于维护
	- 提高软件组件的重用
	- 便于替换某种产品，比如持久层用的是hibernate,需要更换产品用toplink，就不用该其他业务代码，直接把配置一改。
	- 便于产品功能的扩展。
	- 便于适用用户需求的不断变化
19. 序列化接口的id有什么用？                 
   对象经常要通过IO进行传送，让你写程序传递对象，你会怎么做？把对象的状态数据用某种格式写入到硬盘，Person->“zxx,male,28,30000”?Person，
   既然大家都要这么干，并且没有个统一的干法，于是，sun公司就提出一种统一的解决方案，它会把对象变成某个格式进行输入和输出，
   这种格式对程序员来说是透明（transparent）的，但是，我们的某个类要想能被sun的这种方案处理，必须实现Serializable接口。      
   ObjectOutputStream.writeObject(obj);
   Object obj = ObjectInputStream.readObject();
   假设两年前我保存了某个类的一个对象，这两年来，我修改该类，删除了某个属性和增加了另外一个属性，两年后，我又去读取那个保存的对象，或有什么结果？
   未知！sun的jdk就会蒙了。为此，一个解决办法就是在类中增加版本后，每一次类的属性修改，都应该把版本号升级一下，这样，在读取时，
   比较存储对象时的版本号与当前类的版本号，如果不一致，则直接报版本号不同的错!
20. 面向对象的特征有哪些方面                 
	面向对象是相对于面向过程而言的，面向过程强调的是功能，面向对象强调的是将功能封装进对象强调具备功能的对象，
	- 思想特点好处：
		- 是符合人们思考习惯的一种思想；
		- 将复杂的事情简单化了；
		- 将程序员从执行者变成了指挥者；
	 注：比如我要达到某种结果我要是用某个功能，我就寻找能帮我达到该结果的功能的对象，如果该对象具备了该功能
	那么我拿到该对象，也就可以使用该对象的功能。如我要洗衣服我就买洗衣机，至于怎么洗我不管。		 

	面向对象的编程语言有封装、继承 、多态等3个主要的特征。          
	- 封装：隐藏对象的属性和实现细节，仅对外提供公共访问方式        
		封装是保证软件部件具有优良的模块性的基础，封装的目标就是要实现软件部件的“高内聚、低耦合”，防止程序相互依赖性而带来的变动影响。
		在面向对象的编程语言中，对象是封装的最基本单位，面向对象的封装比传统语言的封装更为清晰、更为有力。
		面向对象的封装就是把描述一个对象的属性和行为的代码封装在一个“模块”中，也就是一个类中，属性用变量定义，行为用方法进行定义，
		方法可以直接访问同一个对象中的属性。通常情况下，只要记住让变量和访问这个变量的方法放在一起，将一个类中的成员变量全部定义成私有的，
		只有这个类自己的方法才可以访问到这些成员变量，这就基本上实现对象的封装，就很容易找出要分配到这个类上的方法了，
		就基本上算是会面向对象的编程了。把握一个原则：把对同一事物进行操作的方法和相关的方法放在同一个类中，把方法和它操作的数据放在同一个类中。
		例如，人要在黑板上画圆，这一共涉及三个对象：人、黑板、圆，画圆的方法要分配给哪个对象呢？由于画圆需要使用到圆心和半径，圆心和半径显然是圆的属性，
		如果将它们在类中定义成了私有的成员变量，那么，画圆的方法必须分配给圆，它才能访问到圆心和半径这两个属性，人以后只是调用圆的画圆方法、
		表示给圆发给消息而已，画圆这个方法不应该分配在人这个对象上，这就是面向对象的封装性，即将对象封装成一个高度自治和相对封闭的个体，
		对象状态（属性）由这个对象自己的行为（方法）来读取和改变。一个更便于理解的例子就是，司机将火车刹住了，刹车的动作是分配给司机，
		还是分配给火车，显然，应该分配给火车，因为司机自身是不可能有那么大的力气将一个火车给停下来的，只有火车自己才能完成这一动作，
		火车需要调用内部的离合器和刹车片等多个器件协作才能完成刹车这个动作，司机刹车的过程只是给火车发了一个消息，通知火车要执行刹车动作而已。

	- 继承: 多个类中存在相同属性和行为时，将这些内容抽取到单独一个类中，那么多个类无需再定义这些属性和行为，只要继承那个类即可。   
		在定义和实现一个类的时候，可以在一个已经存在的类的基础之上来进行，把这个已经存在的类所定义的内容作为自己的内容，并可以加入若干新的内容，
		或修改原来的方法使之更适合特殊的需要，这就是继承。继承是子类自动共享父类数据和方法的机制，这是类之间的一种关系，提高了软件的可重用性和可扩展性。

	- 多态: 一个对象在程序不同运行时刻代表的多种状态，父类或者接口的引用指向子类对象
		多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，
		即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。
		因为在程序运行时才确定具体的类，这样，不用修改源程序代码，就可以让引用变量绑定到各种不同的类实现上，从而导致该引用调用的具体方法随之改变，
		即不修改程序代码就可以改变程序运行时所绑定的具体代码，让程序可以选择多个运行状态，这就是多态性。多态性增强了软件的灵活性和扩展性。
		例如，下面代码中的UserDao是一个接口，它定义引用变量userDao指向的实例对象由daofactory.getDao()在执行的时候返回，有时候指向的是UserJdbcDao这个实现，
		有时候指向的是UserHibernateDao这个实现，这样，不用修改源代码，就可以改变userDao指向的具体类实现，
		从而导致userDao.insertUser()方法调用的具体代码也随之改变，即有时候调用的是UserJdbcDao的insertUser方法，
		有时候调用的是UserHibernateDao的insertUser方法：
		```java
		UserDao userDao = daofactory.getDao();  
		userDao.insertUser(user);
		```
		比喻：人吃饭，你看到的是左手，还是右手？
		
	面向对象设计的重要经验：      
	**谁拥有数据，谁就对外提供操作这些数据的方法。**
	如：
	- 人在黑板上画圆：              
		画圆需要知道圆心的位置和圆的半径，而圆心的位置和圆的半径只有圆自己最清楚，所以画圆的方法是圆的方法。            
		
	- 列车司机紧急刹车：          
		刹车的方法是司机的还是车的呢？同样，具体刹车的动作、怎么刹车只有车是清楚的，人只是发个刹车的信号给车，所以刹车的方法是车的
		
	面向对象的面试题：           
	- 两块石头磨成一把石刀，石刀可以将树看成木材，木材可以做出椅子：              
		石头磨成石刀是哪个对象的方法呢？如果是石头内部的方法的话，那么石头磨成石刀，石头自己都没了，所以石头磨成石刀不是石头内部的方法，
		故石头变成石刀应该是一个石头加工厂的方法，将石头磨成石刀。石刀将树看成木材，所以石刀应该有一个将树看成木材的方法
		木材可以做成椅子，所以应该有一个木材加工厂，该加工厂有一个接收木材然后加工成椅子的方法。
	- 球从一根绳子的一端移动到另一端：
		球这个对象应该有移动到下一个点的方法
		绳子这个对象应该有提供下个点是哪个点的方法，以通知小球移动的方向		
			
21. java中实现多态的机制是什么？            
	靠的是父类或接口定义的引用变量可以指向子类或具体实现类的实例对象，而程序调用的方法在运行期才动态绑定，就是引用变量所指向的具体实例对象的方法，
	也就是内存里正在运行的那个对象的方法，而不是引用变量的类型中定义的方法。 
22. abstract class和interface有什么区别? 	          
	含有abstract修饰符的class即为抽象类，abstract类不能创建的实例对象。含有abstract方法的类必须定义为abstract class，abstract class类中的方法不必是抽象的。
	abstract class类中定义抽象方法必须在具体(Concrete)子类中实现，所以，不能有抽象构造方法或抽象静态方法。如果的子类没有实现抽象父类中的所有抽象方法，
	那么子类也必须定义为abstract类型。接口（interface）可以说成是抽象类的一种特例，接口中的所有方法都必须是抽象的。
	接口中的方法定义默认为public abstract类型，接口中的成员变量类型默认为public static final。
	下面比较一下两者的语法区别：        
	- 抽象类可以有构造方法，接口中不能有构造方法。
	- 抽象类中可以有普通成员变量，接口中没有普通成员变量
	- 抽象类中可以包含非抽象的普通方法，接口中的所有方法必须都是抽象的，不能有非抽象的普通方法。
	- 抽象类中的抽象方法的访问类型可以是public，protected和（默认类型,虽然eclipse下不报错，但应该也不行）,但接口中的抽象方法只能是public类型的，
		并且默认即为public abstract类型。
	- 抽象类中可以包含静态方法，接口中不能包含静态方法
	- 抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以任意，但接口中定义的变量只能是public static final类型，并且默认即为public static final类型。
	- 一个类可以实现多个接口，但只能继承一个抽象类。	
23. abstract的method是否可同时是static,是否可同时是native，是否可同时是synchronized?           
	abstract的method 不可以是static的，因为抽象的方法是要被子类实现的，而static与子类扯不上关系！
	native方法表示该方法要用另外一种依赖平台的编程语言实现的，不存在着被子类实现的问题，所以，它也不能是抽象的，不能与abstract混用。
	例如，FileOutputSteam类要硬件打交道，底层的实现用的是操作系统相关的api实现，例如，在windows用c语言实现的，所以，查看jdk 的源代码，
	可以发现FileOutputStream的open方法的定义如下：         
	`private native void open(String name) throws FileNotFoundException;`
	如果我们要用java调用别人写的c语言函数，我们是无法直接调用的，我们需要按照java的要求写一个c语言的函数，又我们的这个c语言函数去调用别人的c语言函数。
	由于我们的c语言函数是按java的要求来写的，我们这个c语言函数就可以与java对接上，java那边的对接方式就是定义出与我们这个c函数相对应的方法，
	java中对应的方法不需要写具体的代码，但需要在前面声明native。关于synchronized与abstract合用的问题，我觉得也不行，因为在我几年的学习和开发中，
	从来没见到过这种情况，并且我觉得synchronized应该是作用在一个具体的方法上才有意义。而且，方法上的synchronized同步所使用的同步锁对象是this，
	而抽象方法上无法确定this是什么。 
24. 什么是内部类？Static Nested Class 和 Inner Class的不同。      
	内部类就是在一个类的内部定义的类，内部类中不能定义静态成员（静态成员不是对象的特性，只是为了找一个容身之处，所以需要放到一个类中而已，这么一点小事，
	你还要把它放到类内部的一个类中，过分了啊！提供内部类，不是为让你干这种事情，无聊，不让你干。我想可能是既然静态成员类似c语言的全局变量，
	而内部类通常是用于创建内部对象用的，所以，把“全局变量”放在内部类中就是毫无意义的事情，既然是毫无意义的事情，就应该被禁止），
	部类可以直接访问外部类中的成员变量，内部类可以定义在外部类的方法外面，也可以定义在外部类的方法体中，如下所示：
	```java
	public class Outer {
		int out_x  = 0;
		public void method()
		{
			Inner1 inner1 = new Inner1();
			public class Inner2   //在方法体内部定义的内部类
			{
				public method()
				{
					out_x = 3;
				}
			}
			Inner2 inner2 = new Inner2();
		}
		public class Inner1   //在方法体外面定义的内部类
		{
		}			
	}
	```
	在方法体外面定义的内部类的访问类型可以是public,protecte,默认的，private等4种类型，这就好像类中定义的成员变量有4种访问类型一样，
	它们决定这个内部类的定义对其他类是否可见；对于这种情况，我们也可以在外面创建内部类的实例对象，创建内部类的实例对象时，
	一定要先创建外部类的实例对象，然后用这个外部类的实例对象去创建内部类的实例对象，代码如下：
	```java
	Outer outer = new Outer();
	Outer.Inner1 inner1 = outer.new Innner1();
	```
	在方法内部定义的内部类前面不能有访问类型修饰符，就好像方法中定义的局部变量一样，但这种内部类的前面可以使用final或abstract修饰符。
	这种内部类对其他类是不可见的其他类无法引用这种内部类，但是这种内部类创建的实例对象可以传递给其他类访问。这种内部类必须是先定义，
	后使用，即内部类的定义代码必须出现在使用该类之前，这与方法中的局部变量必须先定义后使用的道理也是一样的。这种内部类可以访问方法体中的局部变量，
	但是，该局部变量前必须加final修饰符。	
25. 内部类可以引用它的包含类的成员吗？有没有什么限制？             
	完全可以。如果不是静态内部类，那没有什么限制！ 
	如果你把静态嵌套类当作内部类的一种特例，那在这种情况下不可以访问外部类的普通成员变量，而只能访问外部类中的静态成员，例如，下面的代码：
    ```java
	class Outer {
		static int x;
		static class Inner
		{
			void test()
			{
				syso(x);
			}
		}
	}
    ```
	答题时，也要能察言观色，揣摩提问者的心思，显然人家希望你说的是静态内部类不能访问外部类的成员，但你一上来就顶牛，这不好，要先顺着人家，
	让人家满意，然后再说特殊情况，让人家吃惊。
26. Anonymous Inner Class (匿名内部类) 是否可以extends(继承)其它类，是否可以implements(实现)interface(接口)?                  
	可以继承其他类或实现其他接口。不仅是可以，而是必须!
27. super.getClass()方法调用                  
	下面程序的输出结果是多少？
	```java
	import java.util.Date;
	public  class Test extends Date{
		public static void main(String[] args) {
			new Test().test();
		}
		public void test(){
			System.out.println(super.getClass().getName());
		}
	}
	```
	很奇怪，结果是Test           
	这属于脑筋急转弯的题目，在一个qq群有个网友正好问过这个问题，我觉得挺有趣，就研究了一下，没想到今天还被你面到了，哈哈。         
	在test方法中，直接调用getClass().getName()方法，返回的是Test类名        
	由于getClass()在Object类中定义成了final，子类不能覆盖该方法，所以，在       
	test方法中调用getClass().getName()方法，其实就是在调用从父类继承的getClass()方法，等效于调用super.getClass().getName()方法，
	所以，super.getClass().getName()方法返回的也应该是Test。如果想得到父类的名称，应该用如下代码：       
	`getClass().getSuperClass().getName();`

28. jdk中哪些类是不能继承的？              
	不能继承的是类是那些用final关键字修饰的类。一般比较基本的类型或防止扩展类无意间破坏原来方法的实现的类型都应该是final的，
	在jdk中System,String,StringBuffer等都是基本类型。
29. String是最基本的数据类型吗?                
	基本数据类型包括byte、int、char、long、float、double、boolean和short。 
	java.lang.String类是final类型的，因此不可以继承这个类、不能修改这个类。为了提高效率节省空间，我们应该用StringBuffer类 

30. String s = "Hello";s = s + " world!";这两行代码执行后，原始的String对象中的内容到底变了没有？                    
	没有。因为String被设计成不可变(immutable)类，所以它的所有对象都是不可变对象。在这段代码中，s原先指向一个String对象，内容是 "Hello"，
	然后我们对s进行了+操作，那么s所指向的那个对象是否发生了改变呢？答案是没有。这时，s不指向原来那个对象了，而指向了另一个 String对象，
	内容为"Hello world!"，原来那个对象还存在于内存之中，只是s这个引用变量不再指向它了。
	通过上面的说明，我们很容易导出另一个结论，如果经常对字符串进行各种各样的修改，或者说，不可预见的修改，那么使用String来代表字符串的话会引起很大的内存开销。
	因为 String对象建立之后不能再改变，所以对于每一个不同的字符串，都需要一个String对象来表示。这时，应该考虑使用StringBuffer类，它允许修改，
	而不是每个不同的字符串都要生成一个新的对象。并且，这两种类的对象转换十分容易。
	同时，我们还可以知道，如果要使用内容相同的字符串，不必每次都new一个String。例如我们要在构造器中对一个名叫s的String引用变量进行初始化，
	把它设置为初始值，应当这样做：
	```java
	public class Demo {
		private String s;
		...
		public Demo {
		s = "Initial Value";
		}
		...
	}
	```
	而非
	`s = new String("Initial Value");`
	后者每次都会调用构造器，生成新对象，性能低下且内存开销大，并且没有意义，因为String对象不可改变，所以对于内容相同的字符串，只要一个String对象来表示就可以了。
	也就说，多次调用上面的构造器创建多个对象，他们的String类型属性s都指向同一个对象。
	上面的结论还基于这样一个事实：对于字符串常量，如果内容相同，Java认为它们代表同一个String对象。而用关键字new调用构造器，总是会创建一个新的对象，
	无论内容是否相同。至于为什么要把String类设计成不可变类，是它的用途决定的。其实不只String，很多Java标准类库中的类都是不可变的。
	在开发一个系统的时候，我们有时候也需要设计不可变类，来传递一组相关的值，这也是面向对象思想的体现。不可变类有一些优点，比如因为它的对象是只读的，
	所以多线程并发访问也不会有任何问题。当然也有一些缺点，比如每个不同的状态都要一个对象来代表，可能会造成性能上的问题。所以Java标准类库还提供了一个可变版本，
	即 StringBuffer。
 
31. String s = new String("xyz");创建了几个String Object? 二者之间有什么区别？         
	两个或一个，”xyz”对应一个对象，这个对象放在字符串常量缓冲区，常量”xyz”不管出现多少遍，都是缓冲区中的那一个。
	New String每写一遍，就创建一个新的对象，它一句那个常量”xyz”对象的内容来创建出一个新String对象。如果以前就用过’xyz’，
	这句代表就不会创建”xyz”自己了，直接从缓冲区拿。
32. String 和StringBuffer的区别               
	JAVA平台提供了两个类：String和StringBuffer，它们可以储存和操作字符串，即包含多个字符的字符数据。String类表示内容不可改变的字符串。
	而StringBuffer类表示内容可以被修改的字符串。当你知道字符数据要改变的时候你就可以使用StringBuffer。典型地，你可以使用StringBuffers来动态构造字符数据。
	另外，String实现了equals方法，new String(“abc”).equals(new String(“abc”)的结果为true,而StringBuffer没有实现equals方法，
	所以，new StringBuffer(“abc”).equals(new StringBuffer(“abc”)的结果为false。 
	接着要举一个具体的例子来说明，我们要把1到100的所有数字拼起来，组成一个串。
	```java
	StringBuffer sbf = new StringBuffer();  
	for(int i=0;i<100;i++) {
		sbf.append(i);
	}
	```
	上面的代码效率很高，因为只创建了一个StringBuffer对象，而下面的代码效率很低，因为创建了101个对象。
	```java
	String str = new String();  
	for(int i=0;i<100;i++) {
		str = str + i;
	}
	```
	在讲两者区别时，应把循环的次数搞成10000，然后用endTime-beginTime来比较两者执行的时间差异，最后还要讲讲StringBuilder与StringBuffer的区别。
	String覆盖了equals方法和hashCode方法，而StringBuffer没有覆盖equals方法和hashCode方法，所以，将StringBuffer对象存储进Java集合类中时会出现问题。

33. StringBuffer与StringBuilder的区别                 
	StringBuffer和StringBuilder类都表示内容可以被修改的字符串，StringBuilder是线程不安全的，运行效率高，如果一个字符串变量是在方法里面定义，
	这种情况只可能有一个线程访问它，不存在不安全的因素了，则用StringBuilder。如果要在类里面定义成员变量，并且这个类的实例对象会在多线程环境下使用，
	那么最好用StringBuffer。

34. 如何把一段逗号分割的字符串转换成一个数组?                   
	如果不查jdk api，我很难写出来！我可以说说我的思路：
	- 用正则表达式，代码大概为：String [] result = orgStr.split(“,”);
	- 用 StingTokenizer ,代码为：
		```java
		StringTokenizer  tokener = StringTokenizer(orgStr,”,”);
		String [] result = new String[tokener .countTokens()];
		Int i=0;
		while(tokener.hasNext(){
			result[i++]=toker.nextToken();
		}
		```
35. 数组有没有length()这个方法? String有没有length()这个方法？                
	数组没有length()这个方法，有length的属性。String有length()这个方法。
	
36. 下面这条语句一共创建了多少个对象：String s="a"+"b"+"c"+"d";                   
	答：对于如下代码：
	String s1 = "a";
	String s2 = s1 + "b";
	String s3 = "a" + "b";
	System.out.println(s2 == "ab");
	System.out.println(s3 == "ab");
	第一条语句打印的结果为false，第二条语句打印的结果为true，这说明javac编译可以对字符串常量直接相加的表达式进行优化，
	不必要等到运行期去进行加法运算处理，而是在编译时去掉其中的加号，直接将其编译成一个这些常量相连的结果。
	题目中的第一行代码被编译器在编译时优化后，相当于直接定义了一个”abcd”的字符串，所以，上面的代码应该只创建了一个String对象。写如下两行代码，
	```java
	String s = "a" + "b" + "c" + "d";
	System.out.println(s == "abcd");
	```
	最终打印的结果应该为true。 

37. try {}里有一个return语句，那么紧跟在这个try后的finally {}里的code会不会被执行，什么时候被执行，在return前还是后?                   
	也许你的答案是在return之前，但往更细地说，我的答案是在return中间执行，请看下面程序代码的运行结果： 
	```java
	public  class Test {
		/**
		 * @param args add by zxx ,Dec 9, 2008
		 */
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			System.out.println(new Test().test());;
		}
		static int test() {
			int x = 1;
			try
			{
				return x;
			}
			finally
			{
				++x;
			}
		}
	}
	```
	---------执行结果 ---------
	1

	运行结果是1，为什么呢？主函数调用子函数并得到结果的过程，好比主函数准备一个空罐子，当子函数要返回结果时，先把结果放在罐子里，
	然后再将程序逻辑返回到主函数。所谓返回，就是子函数说，我不运行了，你主函数继续运行吧，这没什么结果可言，结果是在说这话之前放进罐子里的。
38. 下面的程序代码输出的结果是多少？             
	```java
	public class  smallT {
		public static void  main(String args[]) {
			smallT t  = new  smallT();
			int  b  =  t.get();
			System.out.println(b);
		}
		public int  get() {
			try
			{
				return 1 ;
			}
			finally
			{
				return 2 ;
			}
		}
	}
	返回的结果是2。
	我可以通过下面一个例子程序来帮助我解释这个答案，从下面例子的运行结果中可以发现，try中的return语句调用的函数先于finally中调用的函数执行，
	也就是说return语句先执行，finally语句后执行，所以，返回的结果是2。Return并不是让函数马上返回，而是return语句执行后，将把返回结果放置进函数栈中，
	此时函数并不是马上返回，它要执行finally语句后才真正开始返回。
	在讲解答案时可以用下面的程序来帮助分析：   
	```java
	public  class Test {
		/**
		 * @param args add by zxx ,Dec 9, 2008
		 */
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			System.out.println(new Test().test());;
		}
		int test() {
			try
			{
				return func1();
			}
			finally
			{
				return func2();
			}
		}
		int func1() {
			System.out.println("func1");
			return 1;
		}
		int func2() {
			System.out.println("func2");
			return 2;
		}	
	}
	-----------执行结果-----------------
	func1
	func2
	2
	```
	结论：finally中的代码比return 和break语句后执行
39. final, finally, finalize的区别。          
	final 用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。 
	内部类要访问局部变量，局部变量必须定义成final类型.       
	finally是异常处理语句结构的一部分，表示总是执行。       
	finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源回收，例如关闭文件等。
	JVM不保证此方法总被调用

40. 运行时异常与一般异常有何异同？             
	异常表示程序运行过程中可能出现的非正常状态，运行时异常表示虚拟机的通常操作中可能遇到的异常，是一种常见运行错误。
	java编译器要求方法必须声明抛出可能发生的非运行时异常，但是并不要求必须声明抛出未被捕获的运行时异常。   
41. error和exception有什么区别?              
	error 表示恢复不是不可能但很困难的情况下的一种严重问题。比如说内存溢出。不可能指望程序能处理这样的情况。 
	exception 表示一种设计或实现问题。也就是说，它表示如果程序运行正常，从不会发生的情况。 

42. java中有几种方法可以实现一个线程？用什么关键字修饰同步方法? stop()和suspend()方法为何不推荐使用？           
	java5以前，有如下两种：     
	第一种：      
	```java
	new Thread(){}.start();这表示调用Thread子类对象的run方法，new Thread(){}表示一个Thread的匿名子类的实例对象，子类加上run方法后的代码如下：
	new Thread(){
		public void run(){
		}
	}.start();
	```
	第二种：         
	```java
	new Thread(new Runnable(){}).start();这表示调用Thread对象接受的Runnable对象的run方法，new Runnable(){}表示一个Runnable的匿名子类的实例对象,runnable的子类加上run方法后的代码如下：
	new Thread(new Runnable(){
			public void run(){
			}	
		}
	).start();
	```
	从java5开始，还有如下一些线程池创建多线程的方式：    
	```java
	ExecutorService pool = Executors.newFixedThreadPool(3)
	for(int i=0;i<10;i++) {
		pool.execute(new Runable(){public void run(){}});
	}
	Executors.newCachedThreadPool().execute(new Runable(){public void run(){}});
	Executors.newSingleThreadExecutor().execute(new Runable(){public void run(){}});
	```

	两种实现方法，分别是继承Thread类与实现Runnable接口 
	用synchronized关键字修饰同步方法 
	反对使用stop()，是因为它不安全。它会解除由线程获取的所有锁定，而且如果对象处于一种不连贯状态，那么其他线程能在那种状态下检查和修改它们。
	结果很难检查出真正的问题所在。suspend()方法容易发生死锁。调用suspend()的时候，目标线程会停下来，但却仍然持有在这之前获得的锁定。
	此时，其他任何线程都不能访问锁定的资源，除非被"挂起"的线程恢复运行。对任何线程来说，如果它们想恢复目标线程，同时又试图使用任何一个锁定的资源，
	就会造成死锁。所以不应该使用suspend()，而应在自己的Thread类中置入一个标志，指出线程应该活动还是挂起。若标志指出线程应该挂起，便用wait()命其进入等待状态。
	若标志指出线程应当恢复，则用一个notify()重新启动线程。 

43. 启动一个线程是用run()还是start()?             
	启动一个线程是调用start()方法，使线程就绪状态，以后可以被调度为运行状态，一个线程必须关联一些具体的执行代码，run()方法是该线程所关联的执行代码。 

44. 简述synchronized和java.util.concurrent.locks.Lock的异同 ？              
	主要相同点：Lock能完成synchronized所实现的所有功能                   
	主要不同点：Lock有比synchronized更精确的线程语义和更好的性能。synchronized会自动释放锁，而Lock一定要求程序员手工释放，
	并且必须在finally从句中释放。Lock还有更强大的功能，例如，它的tryLock方法可以非阻塞方式去拿锁。 
	
45. 设计4个线程，其中两个线程每次对j增加1，另外两个线程对j每次减少1。写出程序。             
	以下程序使用内部类实现线程，对j增减的时候没有考虑顺序问题。 
	```java
	public class ThreadTest1 
	{ 
	private int j; 
	public static void main(String args[]){ 
	   ThreadTest1 tt=new ThreadTest1(); 
	   Inc inc=tt.new Inc(); 
	   Dec dec=tt.new Dec(); 
	   for(int i=0;i<2;i++){ 
		   Thread t=new Thread(inc); 
		   t.start(); 
			   t=new Thread(dec); 
		   t.start(); 
		   } 
	   } 
	private synchronized void inc(){ 
	   j++; 
	   System.out.println(Thread.currentThread().getName()+"-inc:"+j); 
	   } 
	private synchronized void dec(){ 
	   j--; 
	   System.out.println(Thread.currentThread().getName()+"-dec:"+j); 
	   } 
	class Inc implements Runnable{ 
	   public void run(){ 
		   for(int i=0;i<100;i++){ 
		   inc(); 
		   } 
	   } 
	} 
	class Dec implements Runnable{ 
	   public void run(){ 
		   for(int i=0;i<100;i++){ 
		   dec(); 
		   } 
	   } 
	} 
	} 
	----------随手再写的一个-------------
	class A {
		JManger j =new JManager();
		main() {
			new A().call();
		}
		void call {
			for(int i=0;i<2;i++) {
				new Thread(
					new Runnable(){ public void run(){while(true){j.accumulate()}}}
				).start();
				new Thread(new Runnable(){ public void run(){while(true){j.sub()}}}).start();
			}
		}
	}
	class JManager {
		private j = 0;	
		public synchronized void subtract() {
			j--
		}
		public synchronized void accumulate() {
			j++;
		}
	}
	```
	
46. 子线程循环10次，接着主线程循环100，接着又回到子线程循环10次，接着再回到主线程又循环100，如此循环50次，请写出程序。         
	```java
	public class ThreadTest {
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			new ThreadTest().init();
		}
		public void init() {
			final Business business = new Business();
			new Thread(
					new Runnable() {
						public void run() {
							for(int i=0;i<50;i++) {
								business.SubThread(i);
							}						
						}
					}
			).start();
			for(int i=0;i<50;i++) {
				business.MainThread(i);
			}		
		}
		private class Business {
			boolean bShouldSub = true;//这里相当于定义了控制该谁执行的一个信号灯
			public synchronized void MainThread(int i) {
				if(bShouldSub)
					try {
						this.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}		
				for(int j=0;j<5;j++) {
					System.out.println(Thread.currentThread().getName() + ":i=" + i +",j=" + j);
				}
				bShouldSub = true;
				this.notify();
			}	
			public synchronized void SubThread(int i) {
				if(!bShouldSub)
					try {
						this.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}	 
				for(int j=0;j<10;j++) {
					System.out.println(Thread.currentThread().getName() + ":i=" + i +",j=" + j);
				}
				bShouldSub = false;				
				this.notify();			
			}
		}
	}
	备注：不可能一上来就写出上面的完整代码，最初写出来的代码如下，问题在于两个线程的代码要参照同一个变量，即这两个线程的代码要共享数据，
	所以，把这两个线程的执行代码搬到同一个类中去:   
	```java
	package com.huawei.interview.lym;
	public class ThreadTest {
		private static boolean bShouldMain = false;
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			/*new Thread(){
			public void run() {
				for(int i=0;i<50;i++) {
					for(int j=0;j<10;j++) {
						System.out.println("i=" + i + ",j=" + j);
					}
				}				
			}
		}.start();*/		
		//final String str = new String("");
		new Thread(
				new Runnable() {
					public void run() {
						for(int i=0;i<50;i++) {
							synchronized (ThreadTest.class) {
								if(bShouldMain) {
									try {
										ThreadTest.class.wait();} 
									catch (InterruptedException e) {
										e.printStackTrace();
									}
								}
								for(int j=0;j<10;j++) {
									System.out.println(
											Thread.currentThread().getName() + 
											"i=" + i + ",j=" + j);
								}
								bShouldMain = true;
								ThreadTest.class.notify();
							}							
						}						
					}
				}
		).start();
			
		for(int i=0;i<50;i++) {
			synchronized (ThreadTest.class) {
				if(!bShouldMain) {
					try {
						ThreadTest.class.wait();} 
					catch (InterruptedException e) {
						e.printStackTrace();
					}
				}				
				for(int j=0;j<5;j++) {
					System.out.println(
							Thread.currentThread().getName() + 						
							"i=" + i + ",j=" + j);
				}
				bShouldMain = false;
				ThreadTest.class.notify();				
			}			
		}
	}
	```
	下面使用jdk5中的并发库来实现的:   
	```java
	import java.util.concurrent.Executors;
	import java.util.concurrent.ExecutorService;
	import java.util.concurrent.locks.Lock;
	import java.util.concurrent.locks.ReentrantLock;
	import java.util.concurrent.locks.Condition;

	public class ThreadTest {
		private static Lock lock = new ReentrantLock();
		private static Condition subThreadCondition = lock.newCondition();
		private static boolean bBhouldSubThread = false;
		public static void main(String [] args) {
			ExecutorService threadPool = Executors.newFixedThreadPool(3);
			threadPool.execute(new Runnable(){
				public void run() {
					for(int i=0;i<50;i++) {
						lock.lock();					
						try {					
							if(!bBhouldSubThread)
								subThreadCondition.await();
							for(int j=0;j<10;j++) {
								System.out.println(Thread.currentThread().getName() + ",j=" + j);
							}
							bBhouldSubThread = false;
							subThreadCondition.signal();
						}catch(Exception e) {						
						}
						finally {
							lock.unlock();
						}
					}			
				}
				
			});
			threadPool.shutdown();
			for(int i=0;i<50;i++) {
					lock.lock();					
					try {	
						if(bBhouldSubThread)
								subThreadCondition.await();								
						for(int j=0;j<10;j++) {
							System.out.println(Thread.currentThread().getName() + ",j=" + j);
						}
						bBhouldSubThread = true;
						subThreadCondition.signal();					
					} catch(Exception e) {						
					}
					finally {
						lock.unlock();
					}					
			}
		}
	}
	```

47. ArrayList和Vector的区别             
	这两个类都实现了List接口（List接口继承了Collection接口），他们都是有序集合，即存储在这两个集合中的元素的位置都是有顺序的，相当于一种动态的数组，
	我们以后可以按位置索引号取出某个元素，，并且其中的数据是允许重复的，这是HashSet之类的集合的最大不同处，HashSet之类的集合不可以按索引号去检索其中的元素，
	也不允许有重复的元素（本来题目问的与hashset没有任何关系，但为了说清楚ArrayList与Vector的功能，我们使用对比方式，更有利于说明问题）。         
	接着才说ArrayList与Vector的区别，这主要包括两个方面：          
	- 同步性：
		Vector是线程安全的，也就是说是它的方法之间是线程同步的，而ArrayList是线程序不安全的，它的方法之间是线程不同步的。如果只有一个线程会访问到集合，
		那最好是使用ArrayList，因为它不考虑线程安全，效率会高些；如果有多个线程会访问到集合，那最好是使用Vector，因为不需要我们自己再去考虑和编写线程安全的代码。
		备注：对于Vector&ArrayList、Hashtable&HashMap，要记住线程安全的问题，记住Vector与Hashtable是旧的，是java一诞生就提供了的，它们是线程安全的，
		ArrayList与HashMap是java2时才提供的，它们是线程不安全的。所以，我们讲课时先讲老的。
	- 数据增长：
		ArrayList与Vector都有一个初始的容量大小，当存储进它们里面的元素的个数超过了容量时，就需要增加ArrayList与Vector的存储空间，每次要增加存储空间时，
		不是只增加一个存储单元，而是增加多个存储单元，每次增加的存储单元的个数在内存空间利用与程序效率之间要取得一定的平衡。Vector默认增长为原来两倍，
		而ArrayList的增长策略在文档中没有明确规定（从源代码看到的是增长为原来的1.5倍）。ArrayList与Vector都可以设置初始的空间大小，
		Vector还可以设置增长的空间大小，而ArrayList没有提供设置增长空间的方法。
		总结：即Vector增长原来的一倍，ArrayList增加原来的0.5倍。
		
48. HashMap和Hashtable的区别           
	HashMap是Hashtable的轻量级实现（非线程安全的实现），他们都完成了Map接口，主要区别在于HashMap允许空（null）键值（key）,由于非线程安全，
	在只有一个线程访问的情况下，效率要高于Hashtable。 
	HashMap允许将null作为一个entry的key或者value，而Hashtable不允许。 
	HashMap把Hashtable的contains方法去掉了，改成containsvalue和containsKey。因为contains方法容易让人引起误解。 
	Hashtable继承自Dictionary类，而HashMap是Java1.2引进的Map interface的一个实现。 
	最大的不同是，Hashtable的方法是Synchronize的，而HashMap不是，在多个线程访问Hashtable时，不需要自己为它的方法实现同步，而HashMap 就必须为之提供外同步。 
	Hashtable和HashMap采用的hash/rehash算法都大概一样，所以性能不会有很大的差异。
	就HashMap与HashTable主要从三方面来说。         
	- 历史原因:Hashtable是基于陈旧的Dictionary类的，HashMap是Java 1.2引进的Map接口的一个实现 
	- 同步性:Hashtable是线程安全的，也就是说是同步的，而HashMap是线程序不安全的，不是同步的 
	- 值：只有HashMap可以让你将空值作为一个表的条目的key或value 
	
49. List 和 Map 区别?             
	一个是存储单列数据的集合，另一个是存储键和值这样的双列数据的集合，List中存储的数据是有顺序，并且允许重复；Map中存储的数据是没有顺序的，
	其键是不能重复的，它的值是可以有重复的。
	
50. List, Set, Map是否继承自Collection接口?           
	List，Set是，Map不是 
	
51. List、Map、Set三个接口，存取元素时，各有什么特点？                
	这样的题属于随意发挥题：这样的题比较考水平，两个方面的水平：一是要真正明白这些内容，二是要有较强的总结和表述能力。如果你明白，
	但表述不清楚，在别人那里则等同于不明白。
	首先，List与Set具有相似性，它们都是单列元素的集合，所以，它们有一个功共同的父接口，叫Collection。Set里面不允许有重复的元素，所谓重复，
	即不能有两个相等（注意，不是仅仅是相同）的对象 ，即假设Set集合中有了一个A对象，现在我要向Set集合再存入一个B对象，但B对象与A对象equals相等，
	则B对象存储不进去，所以，Set集合的add方法有一个boolean的返回值，当集合中没有某个元素，此时add方法可成功加入该元素时，则返回true，
	当集合含有与某个元素equals相等的元素时，此时add方法无法加入该元素，返回结果为false。Set取元素时，没法说取第几个，只能以Iterator接口取得所有的元素，
	再逐一遍历各个元素。List表示有先后顺序的集合， 注意，不是那种按年龄、按大小、按价格之类的排序。当我们多次调用add(Obj e)方法时，
	每次加入的对象就像火车站买票有排队顺序一样，按先来后到的顺序排序。有时候，也可以插队，即调用add(int index,Obj e)方法，
	就可以指定当前对象在集合中的存放位置。一个对象可以被反复存储进List中，每调用一次add方法，这个对象就被插入进集合中一次，
	其实，并不是把这个对象本身存储进了集合中，而是在集合中用一个索引变量指向这个对象，当这个对象被add多次时，即相当于集合中有多个索引指向了这个对象，
	如图x所示。List除了可以以Iterator接口取得所有的元素，再逐一遍历各个元素之外，还可以调用get(index i)来明确说明取第几个。
	Map与List和Set不同，它是双列的集合，其中有put方法，定义如下：put(obj key,obj value)，每次存储时，要存储一对key/value，不能存储重复的key，
	这个重复的规则也是按equals比较相等。取则可以根据key获得相应的value，即get(Object key)返回值为key 所对应的value。另外，也可以获得所有的key的结合，
	还可以获得所有的value的结合，还可以获得key和value组合成的Map.Entry对象的集合。
	List 以特定次序来持有元素，可有重复元素。Set 无法拥有重复元素,内部排序。Map 保存key-value值，value可多值。
	HashSet按照hashcode值的某种运算方式进行存储，而不是直接按hashCode值的大小进行存储。
	例如，"abc" ---> 78，"def" ---> 62，"xyz" ---> 65在hashSet中的存储顺序不是62,65,78，LinkedHashSet按插入的顺序存储，
	那被存储对象的hashcode方法还有什么作用呢？学员想想!hashset集合比较两个对象是否相等，首先看hashcode方法是否相等，然后看equals方法是否相等。
	new 两个Student插入到HashSet中，看HashSet的size，实现hashcode和equals方法后再看size。
	同一个对象可以在Vector中加入多次。往集合里面加元素，相当于集合里用一根绳子连接到了目标对象。往HashSet中却加不了多次的。 

52. 说出ArrayList,Vector, LinkedList的存储性能和特性              
	ArrayList和Vector都是使用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，它们都允许直接按序号索引元素，
	但是插入元素要涉及数组元素移动等内存操作，所以索引数据快而插入数据慢，Vector由于使用了synchronized方法（线程安全），通常性能上较ArrayList差，
	而LinkedList使用双向链表实现存储，按序号索引数据需要进行前向或后向遍历，但是插入数据时只需要记录本项的前后项即可，所以插入速度较快。
	LinkedList也是线程不安全的，LinkedList提供了一些方法，使得LinkedList可以被当作堆栈和队列来使用。
	
53. 去掉一个Vector集合中重复的元素             
	```java
	Vector newVector = new Vector();
	For (int i=0;i<vector.size();i++) {
		Object obj = vector.get(i);
		if(!newVector.contains(obj)) {
			newVector.add(obj);
		}	
	}
	```
	还有一种简单的方式，`HashSet set = new HashSet(vector);`
	
54. Collection 和 Collections的区别。              
	Collection是集合类的上级接口，继承与他的接口主要有Set 和List.    
	Collections是针对集合类的一个帮助类，他提供一系列静态方法实现对各种集合的搜索、排序、线程安全化等操作。 
	
55. Set里的元素是不能重复的，那么用什么方法来区分重复与否呢? 是用==还是equals()? 它们有何区别?             
	Set里的元素是不能重复的，元素重复与否是使用equals()方法进行判断的。 
	equals()和==方法决定引用值是否指向同一对象equals()在类中被覆盖，为的是当两个分离的对象的内容和类型相配的话，返回真值。

56. 两个对象值相同(x.equals(y) == true)，但却可有不同的hash code，这句话对不对?            
	对。
	如果对象要保存在HashSet或HashMap中，它们的equals相等，那么，它们的hashcode值就必须相等。
	如果不是要保存在HashSet或HashMap，则与hashcode没有什么关系了，这时候hashcode不等是可以的，例如arrayList存储的对象就不用实现hashcode，当然，
	我们没有理由不实现，通常都会去实现的。
	
57. 字节流与字符流的区别                
	要把一片二进制数据数据逐一输出到某个设备中，或者从某个设备中逐一读取一片二进制数据，不管输入输出设备是什么，我们要用统一的方式来完成这些操作，
	用一种抽象的方式进行描述，这个抽象描述方式起名为IO流，对应的抽象类为OutputStream和InputStream ，不同的实现类就代表不同的输入和输出设备，
	它们都是针对字节进行操作的。在应用中，经常要完全是字符的一段文本输出去或读进来，用字节流可以吗？计算机中的一切最终都是二进制的字节形式存在。
	对于“中国”这些字符，首先要得到其对应的字节，然后将字节写入到输出流。读取时，首先读到的是字节，可是我们要把它显示为字符，我们需要将字节转换成字符。
	由于这样的需求很广泛，人家专门提供了字符流的包装类。底层设备永远只接受字节数据，有时候要写字符串到底层设备，需要将字符串转成字节再进行写入。
	字符流是字节流的包装，字符流则是直接接受字符串，它内部将串转成字节，再写入底层设备，这为我们向IO设别写入或读取字符串提供了一点点方便。
	字符向字节转换时，要注意编码的问题，因为字符串转成字节数组，
	其实是转成该字符的某种编码的字节形式，读取也是反之的道理。

58. 垃圾回收器的基本原理是什么？垃圾回收器可以马上回收内存吗？有什么办法主动通知虚拟机进行垃圾回收？              
	对于GC来说，当程序员创建对象时，GC就开始监控这个对象的地址、大小以及使用情况。通常，GC采用有向图的方式记录和管理堆(heap)中的所有对象。
	通过这种方式确定哪些对象是"可达的"，哪些对象是"不可达的"。当GC确定一些对象为"不可达"时，GC就有责任回收这些内存空间。可以。
	程序员可以手动执行System.gc()，通知GC运行，但是Java语言规范并不保证GC一定会执行。 
 
59. 什么时候用assert。              
	assertion(断言)在软件开发中是一种常用的调试方式，很多开发语言中都支持这种机制。在实现中，assertion就是在程序中的一条语句，
	它对一个boolean表达式进行检查，一个正确程序必须保证这个boolean表达式的值为true；如果该值为false，说明程序已经处于不正确的状态下，
	assert将给出警告或退出。一般来说，assertion用于保证程序最基本、关键的正确性。assertion检查通常在开发和测试时开启。为了提高性能，
	在软件发布后，assertion检查通常是关闭的。 
	```java
	package com.huawei.interview;
	public class AssertTest {
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			int i = 0;
			for(i=0;i<5;i++)
			{
				System.out.println(i);
			}
			//假设程序不小心多了一句--i;
			--i;
			assert i==5;		
		}
	}
	```

60. 能不能自己写个类，也叫java.lang.String？                  
	可以，但在应用的时候，需要用自己的类加载器去加载，否则，系统的类加载器永远只是去加载jre.jar包中的那个java.lang.String。
	由于在tomcat的web应用程序中，都是由webapp自己的类加载器先自己加载WEB-INF/classess目录中的类，然后才委托上级的类加载器加载，
	如果我们在tomcat的web应用程序中写一个java.lang.String，这时候Servlet程序加载的就是我们自己写的java.lang.String，但是这么干就会出很多潜在的问题，
	原来所有用了java.lang.String类的都将出现问题。
	虽然java提供了endorsed技术，可以覆盖jdk中的某些类，具体做法是….。但是，能够被覆盖的类是有限制范围，反正不包括java.lang这样的包中的类。
	（下面的例如主要是便于大家学习理解只用，不要作为答案的一部分，否则，人家怀疑是题目泄露了）例如，运行下面的程序：
    ```java
	package java.lang;
	public class String {
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			System.out.println("string");
		}
	}
    ```
	报告的错误如下：
	java.lang.NoSuchMethodError: main
	Exception in thread "main"
	这是因为加载了jre自带的java.lang.String，而该类中没有main方法。
	
算法与编程
---

1. 判断身份证：要么是15位，要么是18位，最后一位可以为字母，并写程序提出其中的年月日。                
	答：我们可以用正则表达式来定义复杂的字符串格式，(\d{17}[0-9a-zA-Z]|\d{14}[0-9a-zA-Z])可以用来判断是否为合法的15位或18位身份证号码。
	因为15位和18位的身份证号码都是从7位到第12位为身份证为日期类型。这样我们可以设计出更精确的正则模式，使身份证号的日期合法，
	这样我们的正则模式可以进一步将日期部分的正则修改为[12][0-9]{3}[01][0-9][123][0-9]，当然可以更精确的设置日期。
	在jdk的java.util.Regex包中有实现正则的类,Pattern和Matcher。以下是实现代码：
	```java
	import java.util.regex.Matcher;
	import java.util.regex.Pattern;
	public class RegexTest {
		/**
		 * @param args
		 */
		public static void main(String[] args) {
			// 测试是否为合法的身份证号码
			String[] strs = { "130681198712092019", "13068119871209201x",
					"13068119871209201", "123456789012345", "12345678901234x",
					"1234567890123" };
			Pattern p1 = Pattern.compile("(\\d{17}[0-9a-zA-Z]|\\d{14}[0-9a-zA-Z])");
			for (int i = 0; i < strs.length; i++) {
				Matcher matcher = p1.matcher(strs[i]);
				System.out.println(strs[i] + ":" + matcher.matches());
			}
			Pattern p2 = Pattern.compile("\\d{6}(\\d{8}).*"); // 用于提取出生日字符串
			Pattern p3 = Pattern.compile("(\\d{4})(\\d{2})(\\d{2})");// 用于将生日字符串进行分解为年月日
			for (int i = 0; i < strs.length; i++) {
				Matcher matcher = p2.matcher(strs[i]);
				boolean b = matcher.find();
				if (b) {
					String s = matcher.group(1);
					Matcher matcher2 = p3.matcher(s);
					if (matcher2.find()) {
						System.out
								.println("生日为" + matcher2.group(1) + "年"
										+ matcher2.group(2) + "月"
										+ matcher2.group(3) + "日");
					}
				}
			}
		}
	}
	```
	
2. 编写一个程序，将a.txt文件中的单词与b.txt文件中的单词交替合并到c.txt文件中，a.txt文件中的单词用回车符分隔，b.txt文件中用回车或空格进行分隔。             
	```java
	package cn.itcast;
	import java.io.File;
	import java.io.FileReader;
	import java.io.FileWriter;
	public class MainClass{
    	public static void main(String[] args) throws Exception{
    		FileManager a = new FileManager("a.txt",new char[]{'\n'});
    		FileManager b = new FileManager("b.txt",new char[]{'\n',' '});		
    		FileWriter c = new FileWriter("c.txt");
    		String aWord = null;
    		String bWord = null;
    		while((aWord = a.nextWord()) !=null ){
    			c.write(aWord + "\n");
    			bWord = b.nextWord();
    			if(bWord != null)
    				c.write(bWord + "\n");
    		}
    		while((bWord = b.nextWord()) != null){
    			c.write(bWord + "\n");
    		}	
    		c.close();
    	}
	}
	class FileManager{
    	String[] words = null;
    	int pos = 0;
    	public FileManager(String filename,char[] seperators) throws Exception{
    		File f = new File(filename);
    		FileReader reader = new FileReader(f);
    		char[] buf = new char[(int)f.length()];
    		int len = reader.read(buf);
    		String results = new String(buf,0,len);
    		String regex = null;
    		if(seperators.length >1 ){
    			regex = "" + seperators[0] + "|" + seperators[1];
    		}else{
    			regex = "" + seperators[0];
    		}
    		words = results.split(regex);
    	}
    	public String nextWord(){
    		if(pos == words.length)
    			return null;
    		return words[pos++];
    	}
	}
	```
	
3. 编写一个程序，将d:\java目录下的所有.java文件复制到d:\jad目录下，并将原来文件的扩展名从.java改为.jad。               
	listFiles方法接受一个FileFilter对象，这个FileFilter对象就是过虑的策略对象，不同的人提供不同的FileFilter实现，即提供了不同的过滤策略。
	```java
	import java.io.File;
	import java.io.FileInputStream;
	import java.io.FileOutputStream;
	import java.io.FilenameFilter;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	public class Jad2Java {
	public static void main(String[] args) throws Exception {
		File srcDir = new File("java");
		if(!(srcDir.exists() && srcDir.isDirectory()))
				throw new Exception("目录不存在");
		File[] files = srcDir.listFiles(
			new FilenameFilter(){
					public boolean accept(File dir, String name) {
						return name.endsWith(".java");
					}
				}
		);
		System.out.println(files.length);
		File destDir = new File("jad");
		if(!destDir.exists()) destDir.mkdir();
		for(File f :files){
			FileInputStream  fis = new FileInputStream(f);
			String destFileName = f.getName().replaceAll("\\.java$", ".jad");
			FileOutputStream fos = new FileOutputStream(new File(destDir,destFileName));
			copy(fis,fos);
			fis.close();
			fos.close();
		}
	}
	private static void copy(InputStream ips,OutputStream ops) throws Exception{
		int len = 0;
		byte[] buf = new byte[1024];
		while((len = ips.read(buf)) != -1){
			ops.write(buf,0,len);
		}
	}
	```
	由本题总结的思想及策略模式的解析：        
	1.    
	```java
	class jad2java{
		1. 得到某个目录下的所有的java文件集合
			1.1 得到目录 File srcDir = new File("d:\\java");
			1.2 得到目录下的所有java文件：File[] files = srcDir.listFiles(new MyFileFilter());
			1.3 只想得到.java的文件： class MyFileFilter implememyts FileFilter{
				public boolean accept(File pathname){
					return pathname.getName().endsWith(".java")
				}
			}
		2.将每个文件复制到另外一个目录，并改扩展名
			2.1 得到目标目录，如果目标目录不存在，则创建之
			2.2 根据源文件名得到目标文件名，注意要用正则表达式，注意.的转义。
			2.3 根据表示目录的File和目标文件名的字符串，得到表示目标文件的File。
				//要在硬盘中准确地创建出一个文件，需要知道文件名和文件的目录。 
			2.4 将源文件的流拷贝成目标文件流，拷贝方法独立成为一个方法，方法的参数采用抽象流的形式。
				//方法接受的参数类型尽量面向父类，越抽象越好，这样适应面更宽广。	
	}
	```
	分析listFiles方法内部的策略模式实现原理
	```java
	File[] listFiles(FileFilter filter){
		File[] files = listFiles();
		//Arraylist acceptedFilesList = new ArrayList();
		File[] acceptedFiles = new File[files.length];
		int pos = 0;
		for(File file: files){
			boolean accepted = filter.accept(file);
			if(accepted){
				//acceptedFilesList.add(file);
				acceptedFiles[pos++] = file;
			}		
		}
		Arrays.copyOf(acceptedFiles,pos);
		//return (File[])accpetedFilesList.toArray();
	}
	```
	
4. 编写一个截取字符串的函数，输入为一个字符串和字节数，输出为按字节截取的字符串，但要保证汉字不被截取半个，如“我ABC”，4，应该截取“我AB”，             
	输入“我ABC汉DEF”，6，应该输出“我ABC”，而不是“我ABC+汉的半个”。
	```java
	首先要了解中文字符有多种编码及各种编码的特征。
    假设n为要截取的字节数。
	public static void main(String[] args) throws Exception{
		String str = "我a爱中华abc我爱传智def';
		String str = "我ABC汉";
		int num = trimGBK(str.getBytes("GBK"),5);
		System.out.println(str.substring(0,num) );
	}
	public static int  trimGBK(byte[] buf,int n){
		int num = 0;
		boolean bChineseFirstHalf = false;
		for(int i=0;i<n;i++) {
			if(buf[i]<0 && !bChineseFirstHalf){
				bChineseFirstHalf = true;
			}else{
				num++;
				bChineseFirstHalf = false;				
			}
		}
		return num;
	}
	```
	
5. 有一个字符串，其中包含中文字符、英文字符和数字字符，请统计和打印出各个字符的个数。                    
	```java
	哈哈，其实包含中文字符、英文字符、数字字符原来是出题者放的烟雾弹。
	String content = “中国aadf的111萨bbb菲的zz萨菲”;
	HashMap map = new HashMap();
	for(int i=0;i<content.length;i++) {
		char c = content.charAt(i);
		Integer num = map.get(c);
		if(num == null)
			num = 1;
		else
			num = num + 1;
		map.put(c,num);
	} 
	for(Map.EntrySet entry : map) {
		system.out.println(entry.getkey() + “:” + entry.getValue());
	}
	估计是当初面试的那个学员表述不清楚，问题很可能是：
	如果一串字符如"aaaabbc中国1512"要分别统计英文字符的数量，中文字符的数量，和数字字符的数量，假设字符中没有中文字符、英文字符、数字字符之外的其他特殊字符。
	int engishCount;
	int chineseCount;
	int digitCount;
	for(int i=0;i<str.length;i++) {
		char ch = str.charAt(i);
		if(ch>=’0’ && ch<=’9’) {
			digitCount++
		} else if((ch>=’a’ && ch<=’z’) || (ch>=’A’ && ch<=’Z’)) {
			engishCount++;
		} else {
			chineseCount++;
		}
	}
	System.out.println(……………);
	```
	
6. 说明生活中遇到的二叉树，用java实现二叉树                 
	这是组合设计模式。
	我有很多个(假设10万个)数据要保存起来，以后还需要从保存的这些数据中检索是否存在某个数据，（我想说出二叉树的好处，该怎么说呢？那就是说别人的缺点），
	假如存在数组中，那么，碰巧要找的数字位于99999那个地方，那查找的速度将很慢，因为要从第1个依次往后取，取出来后进行比较。
	平衡二叉树（构建平衡二叉树需要先排序，我们这里就不作考虑了）可以很好地解决这个问题，但二叉树的遍历（前序，中序，后序）效率要比数组低很多，
	代码如下：
	```java
	package com.huawei.interview;
	public class Node {
		public int value;
		public Node left;
		public Node right;
		public void store(int value) {
			if(value<this.value) {
				if(left == null) {
					left = new Node();
					left.value=value;
				} else {
					left.store(value);
				}
			} else if(value>this.value) {
				if(right == null) {
					right = new Node();
					right.value=value;
				} else {
					right.store(value);
				}			
			}
		}
		public boolean find(int value) {	
			System.out.println("happen " + this.value);
			if (value == this.value) {
				return true;
			} else if(value>this.value) {
				if(right == null) return false;
				return right.find(value);
			} else {
				if(left == null) return false;
				return left.find(value);
			}
		}
		public  void preList() {
			System.out.print(this.value + ",");
			if(left!=null) left.preList();
			if(right!=null) right.preList();
		}
		public void middleList() {
			if(left!=null) left.preList();
			System.out.print(this.value + ",");
			if(right!=null) right.preList();		
		}
		public void afterList() {
			if(left!=null) left.preList();
			if(right!=null) right.preList();
			System.out.print(this.value + ",");		
		}	
		public static void main(String [] args) {
			int [] data = new int[20];
			for(int i=0;i<data.length;i++) {
				data[i] = (int)(Math.random()*100) + 1;
				System.out.print(data[i] + ",");
			}
			System.out.println();
			Node root = new Node();
			root.value = data[0];
			for(int i=1;i<data.length;i++) {
				root.store(data[i]);
			}
			root.find(data[19]);
			root.preList();
			System.out.println();
			root.middleList();
			System.out.println();		
			root.afterList();
		}
	}
	```
	-----------------又一次临场写的代码---------------------------
	```java
	import java.util.Arrays;
	import java.util.Iterator;
	public class Node {
		private Node left;
		private Node right;
		private int value;
		//private int num;
		public Node(int value){
			this.value = value;
		}
		public void add(int value){
			if(value > this.value) {
				if(right != null) {
					right.add(value);
				} else {
					Node node = new Node(value);				
					right = node;
				}
			} else {
				if(left != null) {
					left.add(value);
				} else {
					Node node = new Node(value);				
					left = node;
				}			
			}
		}
		public boolean find(int value){
			if(value == this.value) {
				return true;
			} else if(value > this.value){ 
				if(right == null) return false;
				else return right.find(value);
			}else{
				if(left == null) return false;
				else return left.find(value);			
			}
		}
		public void display(){
			System.out.println(value);
			if(left != null) left.display();
			if(right != null) right.display();
		}
		/*public Iterator iterator(){
		}*/
		public static void main(String[] args){
			int[] values = new int[8];
			for(int i=0;i<8;i++){
				int num = (int)(Math.random() * 15);
				//System.out.println(num);
				//if(Arrays.binarySearch(values, num)<0)
				if(!contains(values,num))
					values[i] = num;
				else
					i--;
			}
			System.out.println(Arrays.toString(values));
			Node root  = new Node(values[0]);
			for(int i=1;i<values.length;i++){
				root.add(values[i]);
			}
			System.out.println(root.find(13));
			root.display();
		}
		public static boolean contains(int [] arr, int value){
			int i = 0;
			for(;i<arr.length;i++){
				if(arr[i] == value) return true;
			}
			return false;
		}
	}
	```
	
7. 从类似如下的文本文件中读取出所有的姓名，并打印出重复的姓名和重复的次数，并按重复次数排序：               
	- 张三,28
	- 李四,35
	- 张三,28
	- 王五,35
	- 张三,28
	- 李四,35
	- 赵六,28
	- 田七,35
 
	程序代码如下（答题要博得用人单位的喜欢，包名用该公司，面试前就提前查好该公司的网址，如果查不到，现场问也是可以的。还要加上实现思路的注释）：

	```java
	package com.huawei.interview;

	import java.io.BufferedReader;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.InputStreamReader;
	import java.util.Comparator;
	import java.util.HashMap;
	import java.util.Iterator;
	import java.util.Map;
	import java.util.TreeSet;

	public class GetNameTest {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		//InputStream ips = GetNameTest.class.getResourceAsStream("/com/huawei/interview/info.txt");
		//用上一行注释的代码和下一行的代码都可以，因为info.txt与GetNameTest类在同一包下面，所以，可以用下面的相对路径形式
		Map results = new HashMap();
		InputStream ips = GetNameTest.class.getResourceAsStream("info.txt");
		BufferedReader in = new BufferedReader(new InputStreamReader(ips));
		String line = null;
		try {
			while((line=in.readLine())!=null)
			{
				dealLine(line,results);
			}
			sortResults(results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	static class User {
		public  String name;
		public Integer value;
		public User(String name,Integer value) {
			this.name = name;
			this.value = value;
		}
		@Override
		public boolean equals(Object obj) {
			// TODO Auto-generated method stub		
			//下面的代码没有执行，说明往treeset中增加数据时，不会使用到equals方法。
			boolean result = super.equals(obj);
			System.out.println(result);
			return result;
		}
	}
	private static void sortResults(Map results) {
		// TODO Auto-generated method stub
		TreeSet sortedResults = new TreeSet(
				new Comparator(){
					public int compare(Object o1, Object o2) {
						// TODO Auto-generated method stub
						User user1 = (User)o1;
						User user2 = (User)o2;
						/*如果compareTo返回结果0，则认为两个对象相等，新的对象不会增加到集合中去
						 * 所以，不能直接用下面的代码，否则，那些个数相同的其他姓名就打印不出来。
						 * */					
						//return user1.value-user2.value;
						//return user1.value<user2.value?-1:user1.value==user2.value?0:1;
						if(user1.value<user2.value) {
							return -1;
						}else if(user1.value>user2.value) {
							return 1;
						}else {
							return user1.name.compareTo(user2.name);
						}
					}
				}
		);
		Iterator iterator = results.keySet().iterator();
		while(iterator.hasNext()) {
			String name = (String)iterator.next();
			Integer value = (Integer)results.get(name);
			if(value > 1) {				
				sortedResults.add(new User(name,value));				
			}
		}
		printResults(sortedResults);
	}
	private static void printResults(TreeSet sortedResults) {
		Iterator iterator  = sortedResults.iterator();
		while(iterator.hasNext()) {
			User user = (User)iterator.next();
			System.out.println(user.name + ":" + user.value);
		}	
	}
	public static void dealLine(String line,Map map) {
		if(!"".equals(line.trim())) {
			String [] results = line.split(",");
			if(results.length == 3) {
				String name = results[1];
				Integer value = (Integer)map.get(name);
				if(value == null) value = 0;
				map.put(name,value + 1);
			}
		}
	}
	}
	``` 
9. 递归算法题                          
	一个整数，大于0，不用循环和本地变量，按照n，2n，4n，8n的顺序递增，当值大于5000时，把值按照指定顺序输出来。
	例：n=1237
	则输出为：      
	1237，    
	2474，      
	4948，       
	9896，      
	9896，      
	4948，       
	2474，      
	1237，            
	提示：写程序时，先只写按递增方式的代码，写好递增的以后，再增加考虑递减部分。       
	```java
	public static void doubleNum(int n) {
		System.out.println(n);
		if(n<=5000)
			doubleNum(n*2);
		System.out.println(n);		
	}
	```
 
10. 递归算法题                           
	第1个人10，第2个比第1个人大2岁，依次递推，请用递归方式计算出第8个人多大？
	```java
	package cn.itcast;
	import java.util.Date;
	public class A1 {
		public static void main(String [] args) {
			System.out.println(computeAge(8));
		}
		public static int computeAge(int n) {
			if(n==1) return 10;
			return computeAge(n-1) + 2;
		}
	}
	public static void toBinary(int n,StringBuffer result) {
		if(n/2 != 0)
			toBinary(n/2,result);
		result.append(n%2);		
	}
	```
	
11. 排序都有哪几种方法？请列举。用JAVA实现一个快速排序。            
	 本人只研究过冒泡排序、选择排序和快速排序，下面是快速排序的代码：
	```java
	public class QuickSort {
		/**
		* 快速排序
		* @param strDate
		* @param left
		* @param right
		*/
		public void quickSort(String[] strDate,int left,int right){
			String middle,tempDate;
			int i,j;
			i=left;
			j=right;
			middle=strDate[(i+j)/2];
			do{
				while(strDate[i].compareTo(middle)<0&& i<right)
				i++; //找出左边比中间值大的数
				while(strDate[j].compareTo(middle)>0&& j>left)
				j--; //找出右边比中间值小的数
				if(i<=j){ //将左边大的数和右边小的数进行替换 
				tempDate=strDate[i];
				strDate[i]=strDate[j];
				strDate[j]=tempDate;
				i++;
				j--;
				}
			}while(i<=j); //当两者交错6时停止
			if(i<right){
				quickSort(strDate,i,right);//从
			}
			if(j>left){
				quickSort(strDate,left,j);
			}
		}
		/**
		  * @param args
		  */
		public static void main(String[] args){
			String[] strVoid=new String[]{"11","66","22","0","55","22","0","32"};
			QuickSort sort=new QuickSort();
			sort.quickSort(strVoid,0,strVoid.length-1);
			for(int i=0;i<strVoid.length;i++){
				System.out.println(strVoid[i]+" ");
			}
		}
	}
	```
	
12. 有数组a[n]，用java代码将数组元素顺序颠倒             
	```java
	import java.util.Arrays;
	public class SwapDemo{
		public static void main(String[] args){
			int [] a = new int[]{
							(int)(Math.random() * 1000),
							(int)(Math.random() * 1000),
							(int)(Math.random() * 1000),
							(int)(Math.random() * 1000),						
							(int)(Math.random() * 1000)																		
			};	
			System.out.println(a);
			System.out.println(Arrays.toString(a));
			swap(a);
			System.out.println(Arrays.toString(a));		
		}
		public static void swap(int a[]){
			int len = a.length;
			for(int i=0;i<len/2;i++){
				int tmp = a[i];
				a[i] = a[len-1-i];
				a[len-1-i] = tmp;
			}
		}
	}
	```
	
13. 金额转换，阿拉伯数字的金额转换成中国传统的形式如：（￥1011）－>（一千零一拾一元整）输出。            
	```java
	// 去零的代码：
	return sb.reverse().toString().replaceAll("零[拾佰仟]","零").replaceAll("零+万","万").replaceAll("零+元","元").replaceAll("零+","零");
	public class RenMingBi {
		/**
		 * @param args add by zxx ,Nov 29, 2008
		 */
		private static final char[] data = new char[]{
				'零','壹','贰','叁','肆','伍','陆','柒','捌','玖'
			}; 
		private static final char[] units = new char[]{
			'元','拾','佰','仟','万','拾','佰','仟','亿'
		};
		public static void main(String[] args) {
			// TODO Auto-generated method stub
			System.out.println(
					convert(135689123));
		}
		public static String convert(int money) {
			StringBuffer sbf = new StringBuffer();
			int unit = 0;
			while(money!=0) {
				sbf.insert(0,units[unit++]);
				int number = money%10;
				sbf.insert(0, data[number]);
				money /= 10;
			}
			return sbf.toString();
		}
	}
	```

----
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 