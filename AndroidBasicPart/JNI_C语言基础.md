JNI_C语言基础
===

1. JNI(java native interface)    
    `Java`本地开发接口，`JNI`是一个协议，这个协议用来沟通`Java`代码和外部的本地代码(`c/c++`).
    通过这个协议`Java`代码就可以调用外部的`c/c++`代码，外部的`c/c++`代码也可以调用java代码，
	使用JNI技术，其实就是在Java程序中，调用C语言的函数库中提供的函数，来完成一些Java语言无法完成的任务。由于Java语言和C语言结构完全不相同，因此若想让它们二者交互，则需要制定一系列的规范。
	JNI就是这组规范，此时 Java只和JNI交互，而由JNI去和C语言交互。
	
	JNI技术分为两部分：Java端和C语言端。且以Java端为主导。    
	- 首先，Java程序员在Java端定义一些native方法，并将这些方法以C语言头文件的方式提供给C程序员。
	- 然后，C程序员使用C语言，来实现Java程序员提供的头文件中定义的函数。
	- 接着，C程序员将函数打包成一个库文件，并将库文件交给Java程序员。
	- 最后，Java程序员在Java程序中导入库文件，然后调用native方法。
	在Java程序执行的时候，若在某个类中调用了native方法，则虚拟机会通过JNI来转调用库文件中的C语言代码。提示：C代码最终是在Linux进程中执行的，而不是在虚拟机中。
 
2. 为什么要用JNI
	- 首先，Java语言提供的类库无法满足要求(驱动开发 wifi等),且在数学运算,实时渲染的游戏上,音视频处理等方面上与C/C++相比效率稍低。 
	- 然后，Java语言无法直接操作硬件，C/C++代码不仅能操作硬件而且还能发挥硬件最佳性能。
	- 接着，使用Java调用本地的C/C++代码所写的库，省去了重复开发的麻烦，并且可以利用很多开源的库提高程序效率。Opencore
	- 接着，特殊的业务场景，java能反编译但是c不能，因此对于一些不想让别人知道的东西可以用c，加密等
 
3. 怎么用JNI
    1. C/C++语言 
	2. 掌握java jni流程 
	3. NDK (native develop kits ) 
  
4. 指针和指针变量的关系
	指针就是地址，地址就是指针          
	地址就是内存单元的编号          
	指针变量是存放地址(指针)的变量      
	指针和指针变量是两个不同的概念       
	但是要注意： 通常我们叙述时会把指针变量简称为指针，实际它们含义并不一样      

  - 未经过初始化的指针变量，不能够直接使用
  - 指针变量的类型 不能够相互转换 
  - 函数的变量(静态)不能够跨函数访问，失去了函数变量的访问范围(生命周期)，因为方法执行完之后会释放内存，所以方法中的变量就没有了，
	但是地址值还是能拿到的，因为地址值是内存中真实存在的地址位置 
  - 指针声明的三种方式
        int * p; //p 是变量的名字， int * 是一个类型，这个变量存放的是int类型变量的地址。
        int* p     int * p    int *p

5. *号的三种含义    
	1. 乘法 3*5
	2. 定义指针变量 int * p;
    3. 指针运算符，如果p是一个已经定义好的指针变量,则*p表示以p的内容为地址的变量
 
6. 为什么要使用指针(指针的重要性）
	直接访问硬件 (opengl 显卡绘图)       
	快速传递数据(指针表示地址)      
	返回一个以上的值(返回一个数组或者结构体的指针)     
	表示复杂的数据结构(结构体)    
	方便处理字符串    
	指针有助于理解面向对象

7. 指针和数组的关系
	一维数组的数组名是个指针常量，它存放的是一维数组第一个元素的地址    
	C中数组的定义比较死板，中括号必须放到名字的后面    
	int a[5];    
	printf("%#X\n",&a[0]);    
	printf("%#X\n",&a);    
     
	如果p是一维数组 则p[i] 等价于 *(p+i),都是得到一维数组中的第i个元素。    
	在c语言中不会检查角标越界如int a[5]写a[5]不会报错   

8. 动态分配内存Malloc
	```c
	采用malloc在椎内存中申请空间
	#include <malloc.h>  //不能省  malloc 是 memory(内存) allocate(分配)的缩写
	#include <stdio.h>

	main(){
		 //malloc() 在堆空间中动态的申请一块连续的内存空间(数组)
		 // 参数： 指定申请的内存空间的大小(字节)
		 // 返回值： 所申请空间的首地址(数组的第一个元素的地址),返回值是Void数据类型
	   
		 int* p = (int*) malloc( sizeof(int) );     //因为返回值是一个Void类型，所以要强转
		 *p = 99;
		
		 //free()释放已经分配的内存块
		 //参数： 指定释放哪块内存空间（地址）
	 
		 free(p);
		printf("内容是 %d\n", *p);     //上面的free只是释放内存块中的内容，但是打印这个内存块的地址还是能够打印出来的，因为这个内存块的地址是内存上的地址是真实存在的       
		 system("pause"); 
	}
	
	如果动态申请的内存不够用，那么可以继续申请内存
	用realloc
	/*
	  1\创建数组
	  2、赋值
	  3、打印 
	*/
	#include<stdio.h>
	#include<malloc.h>

	void printArr(int* arr, int len){
		 int i;
		 for( i = 0 ; i < len; i++){
				printf("arr[ %d ]= %d\n", i, *(arr+i));
		 }

	}

	main(){
		   printf("请您输入所要创建的数组大小： \n");
		   int len ;
		   scanf("%d", &len);  &是取地址符

		   //动态数组创建
		   int* arr = (int*) malloc( sizeof(int) *  len);

		   printf("请您为每个元素赋值： \n");

		   int i;
		   for(i = 0; i < len; i++){
				 int temp;
				 scanf("%d", &temp);

				 *(arr + i) = temp;
		   }

		   //打印
		   printf("数组元素的值为： \n");
		   printArr(arr, len);

		   //-----------------------------------------

		   printf("请输入增加的元素个数： \n");
		   int count;
		   scanf("%d", &count);

		   //更改数组大小
		   //realloc()
		   //参数1： 指定所需修改的数组
		   //参数2： 指定修改后的数组的大小
		   //返回值：修改后数组的首地址  （VOID）
		   arr = (int*) realloc(arr, len + count);

		   printf("请为新增加的元素赋值： \n");

		   int j;
		   for(j = len; j < len + count; j++){
				 int temp;
				 scanf("%d", &temp);

				 *(arr + j) = temp;
		   }

		   //打印 
		   printf("数组元素的值为： \n");
		   printArr(arr, len + count);

		   system("pause");       
	}
	或者有个简单的写法
	main()
	{       
		 int* arr =(int* ) malloc(sizeof(int)*len) ; //动态申请的内存 
		 int i=0;
		 for(;i<len;i++){
		   printf("请输入第%d个数据\n",i); 
		   scanf("%d",&arr[i]);           
		 } 
		  //打印显示这个数组的元素 
		  printArr(arr,len); 

			printf("请输入增加的数组的长度"); 
			int increase;
			scanf("%d", &increase); 

			arr = realloc(arr,(len+increase)*sizeof(int));
			i =len; 
		   for(;i<len+increase ;i++){
		   printf("请输入第%d个数据\n",i); 
		   scanf("%d",&arr[i]);           
		 } 

		  //打印显示这个数组的元素 
		   printf("新的数组长度为:%d\n",len+increase); 
			printArr(arr,len+increase); 

			system("pause"); 
	}
	``` 

9. 静态内存和动态内存                     
    静态内存是系统是程序编译执行后系统自动分配,由系统自动释放,静态内存是栈分配的.动态内存是堆分配的.       
    C中静态内存会自动释放，但是对于动态内存(堆内存)在c中是没有垃圾回收的，必须要靠程序员手动的去释放，不然就会一直存在        

10. C中的基本数据类型               
     char, int, float, double, signed, unsigned, long, short and void     
     c中char 占用1个字节       java char占用2个字节     
     c中long 占用4个字节       java long占用8个字节     

	int flag = 0,1  表示java中boolean类型

	c中用char数组  来表示java中String类型或者指针方式来表示java中String类型
		//内部转化为一个字符串的数组,并且在数组的最后一个元素拼装一个\0. 
		char* cc = "heima 15";//char* str ="hello" ;  //<--> char str[] ={'h','e','l','l','o','\0'};  
		char cc[20] = "heima 15";
		char cc[20] = {'h','e','i','m','a'};

11. c文件的后缀是.c          
	```c
	示例代码：
	c中的打印语句中要有类似占位符，在后面的参数对占位符的内容进行声明
	#include<stdio.h>
	main() {
		printf("%d\n",sizeof(int));       //sizeof() 得到制定数据类型的长度(占用字节数)参数  接受一个数据类型 
		printf("%d\n",sizeof(char));
		system("pause");//可以执行命令行中的命令如 system("shutdown -s -t 60");如果不加pause，运行窗口会一闪而过，因为会释放内存，把dos关闭了，所以要加上pause
	} 
	```

12. C中的输入输出            
	%d  -  int     
	%ld – long int     
	%c  - char    
	%f -  float     
	%lf – double     
	%x – 十六进制输出 int 或者long int 或者short int     
	%#x – 以0x开头 十六进制输出 int 或者long int 或者short int     
	%o -  八进制输出     
	%s – 字符串      
 
13. c语言从键盘输入一个字符串                   
	```c
    //scanf() 接收键盘输入的数据，参数1： 指定接收的数据的数据类型参数 2： 指定接收的数据存放的位置
	#include<stdio.h>
	main() {
		char c[20];
		scanf("%s", c);
		printf("%s", c);
		system("pause");
	}
	```

14. 取地址符  &(能得到一个对象的地址)    

15. C中两个数的交换      
	```c
	#include<stdio.h>
	void swap(int* i, int* j) {
		int temp = *i;
		*i = *j;
		*j = temp;
	}
	main() {
		int i = 3;
		int j = 5;
		swap(&i, &j);
		printf("%d",i);
		printf("%d",j);
		system("pause");      
	}
	```

16. C中的for循环             
	```c
	#include <stdio.h>

	/**
	打印输出数组的每一个元素
	*/
	void printArr(int* arr, int len){
		 int i;
		 for(i=0;i<len;i++){
			printf("arr[%d]=%d\n",i,*(arr+i));            
		  }
	}

	main()
	{    printf("请输入数组的长度");
		 int  len ;
		 scanf("%d", &len);//要用指针
		 int arr[len];
		 int i=0;
		 for(;i<len;i++){
		   printf("请输入第%d个数据\n",i);
		   scanf("%d",&arr[i]);          
		 }
		  //打印显示这个数组的元素
		  printArr(arr,len);
		  system("pause");
	}
	C中for循环不能像java那样for(int i=0, i<100; i++)C中不允许在for中进行变量的声明，必须要分开，像上面的这个代码这样
	```

17. 指针的运算               
    int i = 3;  //天津 解放路 33号      
    int j = 5;   // 北京 东北旺 9号      
    int* p = &i;      
    int* q = &j;       
    //p-q; 单纯的指针相加减 是没有任何的意义的.
    //指针的运算只有在连续的内存空间里面(数组)  才有意义. 
	因为指针的运算必须在数组中才有效，这就是为什么数组中能用     
	p[i] 等价于 *(p+i)，因为这个p是数组的名字也代表了数组中第一个元素的地址，p+i就是讲指针加几就是得到第几个元素的指针 

18. 函数的指针              
	```c
	/**
	1.定义int (*pf)(int x, int y);
	2.赋值 pf = add;
	3.引用 pf(3,5);
	*/
	#include <stdio.h>
	int add(int x,int y){
		return x+y;
	}
	main()
	{
		 int (*pf)(int x, int y); //定义一个函数的指针，就是讲函数的名字改了，其它的都和函数的定义一样    
		 pf = add; //将pf指向add
	   printf("result=%d\n",  pf(3,5)); //使用pf
		 
	   system("pause");
	}

	内存的四个部分 Stack  Heap CodeSegment DataSegment
	函数是存放到CodeSegment中的，这个函数的地址就是CodeSegment中的这个函数的地址，我们得到函数的地址如果去访问这个地址就相当于调用了这个函数
	```
    
19. 结构体(类似于java中的类)              
    ```c
    #include <stdio.h>
    struct Student
    {
    	 int age;  //4
    	 float score; //4
    	 long id; //4
    	 char sex; //1
    };
    int main(void)
    {
    	 struct Student st={80,55.6f,10001,'F'};
    	 printf("st.age=%d\n",st.age);
    	
    	 printf("结构体的长度为%d\n",sizeof(st));//打印出来的结果是16为什么呢？ 编译器为了方便起见 做了处理，它将所有的变量的长度都统一成最大的长度
    	
    	 struct Student* pst = &st;//结构体的指针
    	
    		  printf("st.age=%d\n",(*pst).age);//(*pst)就是得到结构体，由于*的优先级比较低，通常要用括号括起来。
    		 
    		 printf("age=%d\n",pst->age);//这一行是上一行的简单写法，pst->age 在计算机内部会被转换为 (*pst).age pst->age的含义: pst所指向的结构体变量中的age这个成员
    	system("pause");
    }
    ```
    结构体的三种写法          
    第一种  
    ```c
    struct Student
    {
    int age;
    float score;
    char sex;
    }
    ```
    第二种      
    ```c
    struct Student2
    {
    int age;
    float score;
    char sex;
    } st2;//相当于java中直接弄了一个对象
    ```
    第三种       
    ```c
    struct
    {
    int age;
    float score;
    char sex;
    } st3;
    ```

20. Union联合体      
	```c
	#include <stdio.h> 
	main() { 
		  struct date { int year, month, day; }today; 
		  union { long i; int k; char ii; double d; } mix; 

		  printf("date:%d\n",sizeof(struct date)); 
		  printf("mix:%d\n",sizeof(mix)); 
		  mix.i = 33;
		  mix.ii = 'a'; 
		  printf("i=%d\n",mix.i); //结果是96，因为联合体是一个公用的内存空间，在存ii的时候将i的值给覆盖了
		  //110100000101
	  
		  system("pause"); 
	} 
	联合体是一个公用的内存空间，联合体长度为： 占有字节数最大的元素的长度（字节数）
	```
    
21. 枚举         
	```c
	enum WeekDay {
	Monday=8,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday
	};//这里一定要加分号

	int main(void)
	{
	  enum WeekDay day = Sunday;
	  printf("%d\n",day);//打印出来是14，因为是从8开始往后逐个加1
	  system("pause");
	  return 0;
	}
	默认的情况是从0开始往后递加
	```		

22. typedef           
	定义别名          
	声明自定义数据类型，配合各种原有数据类型来达到简化编程的目的的类型定义关键字。 
    ```c
	typedef int haha;  //定义数据类型的别名. 
	int main(void)
	{
	  haha i = 3;这里haha就相当于int
	  printf("i=%d\n",i); 
	}
    ```
	每一个指针占四个字节
 
23. 多级指针
	```c
	#include <stdio.h>
	main() { 
		int i = 88;
		int* p = &i; 
		int** q = &p; //指针的指针前面要加两个*
		int*** r = &q; 
		printf("i=%d\n",***r);  
		system("pause"); 
	}
	```
   
C语言常见术语：      
库函数：     
- 为了代码重用，在C语言中提供了一些常用的、用于执行一些标准任务(如输入/出)的函数，这些函数事先被编译，并生成目标代码，然后将生成的目标代码打包成一个库文件，
以供再次使用。 库文件中的函数被称为库函数，库文件被称为函数库。        
通过头文件的方式 把函数库里面所有的函数暴露 .h         
- 在Windows中C语言库函数中的目标代码都是以.obj为后缀的，Linux中是以 .o为后缀。            
提示：单个目标代码是无法直接执行的，目标代码在运行之前需要使用连接程序将目标代码和其他库函数连接在一起后生成可执行的文件。Windows ->.exe .dll             
Linux -> .so 动态库         
       .a 静态库       
头文件：          
- 头文件中存放的是对某个库中所定义的函数、宏、类型、全局变量等进行声明，它类似于一份仓库清单。若用户程序中需要使用某个库中的函数，
	则只需要将该库所对应的头文件include到程序中即可。          
- 头文件中定义的是库中所有函数的函数原型。而函数的具体实现则是在库文件中。      
- 简单的说：头文件是给编译器用的，库文件是给连接器用的。       
- 在连接器连接程序时，会依据用户程序中导入的头文件，将对应的库函数导入到程序中。头文件以.h为后缀名。          
函数库：         
- 动态库：在编译用户程序时不会将用户程序内使用的库函数连接到用户程序的目标代码中，只有在运行时，且用户程序执行到相关函数时才会调用该函数库里的相应函数，因此动态函数库所产生的可执行文件比较小。 .so 动态库        
- 静态库：在编译用户程序时会将其内使用的库函数连接到目标代码中，程序运行时不再需要动态库。使用静态库生成可执行文件比较大。   
在Linux中：     
- 静态库命名一般为：lib+库名+.a 。  
- 如：libcxy.a 其中lib说明此文件是一个库文件，cxy是库的名称，.a说明是静态的。          
- 动态库命名一般为：lib+库名+.so 。.so说明是动态的。       
Windows 下的动态库  .dll    

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck!