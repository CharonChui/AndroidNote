剑指Offer(上)
===

最近面试，遇到一些笔试题，写不上来，内心是崩溃的，该好好复习下了，所以决定仔细做一遍，随便也整理下，方便大家学习。

1. 我没找到第一题是什么- -!,谁知道的给补充下吧

2. 实现单例模式                 
    单例的实现分为好几种:     
    - 饿汉式
	- 懒汉式
	- 枚举              
	
	具体实现:      
	- 饿汉式              

		```java
		public class Singleton {
			private Singleton() {
			}

			private static final Singleton SINGLETON = new Singleton();

			public static Singleton getInstance() {
				return SINGLETON;
			}
		}
		```
	- 懒汉式

		```java
		public class Singleton {
			private Singleton() {
			}

			private static Singleton singleton = null;

			public static Singleton getInstance() {
				// 同步会导致效率低，这里采用双重判断的方式来提高效率
				if (singleton == null) {
					synchronized (Singleton.class) {
						if (singleton == null) {
							singleton = new Singleton();
						}
					}
				}
				return singleton;
			}
		}
		```
	- 枚举	

	    ```java
		public enum Singleton {
			INSTANCE;
			private Singleton() {

			}
		}		
		```
	
	- 我这里写一种自我感觉是单例最完美的实现方式
	　　
		```java
		public class Singleton {
			// Private constructor prevents instantiation from other classes
			private Singleton() { }

			/**
			* SingletonHolder is loaded on the first execution of Singleton.getInstance() 
			* or the first access to SingletonHolder.INSTANCE, not before.
			*/
			private static class SingletonHolder { 
					public static final Singleton INSTANCE = new Singleton();
			}
			public static Singleton getInstance() {
					return SingletonHolder.INSTANCE;
			}
		}
		```
	
3. 二维数组中的查找            
    题目描述：一个二维数组，每一行从左到右递增，每一列从上到下递增．输入一个二维数组和一个整数，判断数组中是否含有整数。
	             
	分析:     
	```
	1   6   11
	5   9   15
	7   13   20
	```
	
	假设我们要找7，那怎么找呢？                
	我们先从第一行找，从后往前找，因为他是递增的，先是11，这里11>7所以肯定不是第三列的。这时候我们就找第二列，       
	这个值是6,6 < 7,所以我们可以从第二列往下找，这个数可能会再第二列或者第一列。把行数加1，来到第二行第二列的9            
	这时候一判断9 > 7，所以不可能是第二列了，这时候把列数再前移，来到第一列，刚才是第二行，所以我们取第一列第二行        
	的数，也就是5,5 < 7，所以还要继续往后找，就是把行数加1，就来到了第三行第一列，也就是7，一判断就是他了。         
	整体思路就是从右上角开始，逐渐前移列数或者增加行数。          
	```java
	public static boolean find(int[][] array, int number) {
		if (array == null) {
			return false;
		}
		
		// 从第一行最后一列的数开始
		int column = array[0].length - 1;
		int row = 0;
		while (row < array.length && column >= 0) {
			if (array[row][column] == number) {
				return true;
			}
			
			if (array[row][column] > number) {
				// 如果这个数比要找的数大，那肯定不是这一列了，只能是前一列
				column--;
			} else {
				// 小的话，那肯定在该数的下面，就要增大行数
				row++;
			}
		}
		return false;
	}
	```
	
4. 替换空格            
    请实现一个函数，把字符串中的每个空格替换成`%20`。              
	思路: 很简单，就是判断每个字符是否为空着，但也要注意使用`StringBuilder`会比`StringBuffer`效率稍高。                    
	```java
	public String replaceBlank(String input) {
		if (input == null) {
			return null;
		}	
		StringBuilder  sb = new StringBuilder ();
		for (int i = 0; i < input.length(); i++) {
			if (input.charAt(i) == ' ') {
				sb.append("%");
				sb.append("2");
				sb.append("0");
			} else {
				sb.append(String.valueOf(input.charAt(i)));
			}
		}
		return new String(sb);
	}
	```
	
5. 从尾到头打印链表            
    输入一个链表的头结点，从尾到头反过来打印出每个节点的值。            	
	思路: 我们可以从头开始遍历，但是要让先遍历的最后打印，这就是一个吃进去、吐出来的方式，最适合的就是栈.            
	- 遍历的方式
	```java
	public class ListNodeTest {
		public static void main(String args[]) {
			ListNode node1 = new ListNode();
			ListNode node2 = new ListNode();
			ListNode node3 = new ListNode();
			node1.data = 1;
			node2.data = 2;
			node3.data = 3;
			node1.next = node2;
			node2.next = node3;

			ListNodeTest.reversePrint(node1);
		}

		public static void reversePrint(ListNode headNode) {
			Stack<ListNode> stack = new Stack<ListNode>();
			while (headNode != null) {
				// 遍历，然后用栈来保存
				stack.push(headNode);
				headNode = headNode.next;
			}
			while (!stack.isEmpty()) {
				System.out.println(stack.pop().data);
			}
		}
	}

	class ListNode {
		public ListNode() {

		}

		ListNode next;
		int data;
	}
	```
	- 递归
	```java
	public static void printListReverse(ListNode headNode) {
		if (headNode != null) {
			if (headNode.next != null) {
				printListReverse(headNode.next);
			}
			System.out.println(headNode.data);
		}
	}
	```
	
6. 重建二叉树	    
    输入二叉树的前序遍历和中序遍历的结果，重建出该二叉树。假设前        
    序遍历和中序遍历结果中都不包含重复的数字，例如输入的前序遍历序列        
    {1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}重建出如图所示的二叉树。          
		  
	![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/binary_offer.png)	  
	思路: 前序遍历序列中，第一个数字总是树的根节点的值。          
	中序遍历根节点在序列的中间，左边是左子树，右边是右子树。          
	所以我们可以根据前序遍历的第一个值去中旬遍历数组中查找，就能找出来中序的分割点是谁。           
	这样在他左边的就都是左子树，右边是右子树。这样也就能知道左子树的个数，再用这个数量去前子树中去前几个数。          
	就这样再递归下去排列二级左子树的根节点。          
	```java
	import java.util.Arrays;

	public class BinaryTree {
		public static void main(String[] args) throws Exception {
			int[] preorder = { 1, 2, 4, 7, 3, 5, 6, 8 };
			int[] inorder = { 4, 7, 2, 1, 5, 3, 8, 6 };
			BinaryTreeNode root = constructCore(preorder, inorder);
		}

		public static BinaryTreeNode constructCore(int[] preorder, int[] inorder)
				throws Exception {
			if (preorder == null || inorder == null) {
				return null;
			}
			if (preorder.length != inorder.length) {
				throw new IllegalArgumentException("error params");
			}
			BinaryTreeNode root = new BinaryTreeNode();
			for (int i = 0; i < inorder.length; i++) {
				if (inorder[i] == preorder[0]) {
					root.value = inorder[i];
					// 递归让再去构建左子树的下一个根节点
					root.leftNode = constructCore(
							// 前序遍历从第二个开始后的i个都是左子树的
							Arrays.copyOfRange(preorder, 1, i + 1),
							// 中序遍历最边边这i个也是左子树
							Arrays.copyOfRange(inorder, 0, i));
					root.rightNode = constructCore(
							Arrays.copyOfRange(preorder, i + 1, preorder.length),
							Arrays.copyOfRange(inorder, i + 1, inorder.length));
				}
				// 就这样循环递归下去，就OK了。
			}
			return root;
		}
	}

	class BinaryTreeNode {
		public static int value;
		public BinaryTreeNode leftNode;
		public BinaryTreeNode rightNode;
	}
	```

7. 用两个栈实现队列                      
    用两个栈实现一个队列，实现队列的两个函数`appendTail`和     
    `deleteHead`，分别完成在队列尾插入结点和在队列头部删除结点的功能。        
	思路： 栈是啥？栈是先进后出，因为栈是一个出口啊，先进入的被压在最下面了，出要从上面开始出，也就是吃了吐出来。         
	队列是啥？两头的，就想管道一样，先进先出。不雅的说，就是吃了拉出来。            
	队列尾插入节点好说啊，就是在栈中往里放。         
	那队列头部删除怎么弄？因为他在栈的最底部啊，你没法直接删他啊，不要忘了，我们是用两个栈来实现。所以自然想到
	就是把这个栈中的数据都取出放入到第二个栈中，然后删除第二个栈的最上面的元素就可以了。         
	```java
	public class StackListTest<T> {
		private Stack<T> stack1 = new Stack<T>();
		private Stack<T> stack2 = new Stack<T>();

		public void appendTail(T t) {
			// 往栈1中存
			stack1.push(t);
		}

		public T deleteHead() throws Exception {
			if (stack2.isEmpty()) {
				// 把栈1的数都放到栈2中
				while (!stack1.isEmpty()) {
					stack2.push(stack1.pop());
				}
			}
			// 转移到栈2后，就相当于把栈1倒序了。
			if (stack2.isEmpty()) {
				throw new Exception("队列为空，不能删除");
			}
			// 直接取栈2中最上层的就可以了。
			return stack2.pop();
		}

		public static void main(String args[]) throws Exception {
			StackListTest<String> p7 = new StackListTest<String>();
			p7.appendTail("1");
			p7.appendTail("2");
			p7.appendTail("3");
			p7.deleteHead();
		}
	}
	```
	
8. 	旋转数组的最小数字               
    把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的        
    旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如数      
    组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为 1.        
	
	思路: 开始看到这道题，我感觉很简单，就是循环比较下找出最下的就完了，我感觉什么旋转数组都是面试官
	放的烟雾弹。后来我发现我错了。旋转数组是有用的。
	```java
	public class FindTest {
		public static void main(String[] args) {
			// int[] array={1,1,1,2,0};
			// int[] array={3,4,5,1,2};
			int[] array = { 1, 0, 1, 1, 1 }; // 这是0，1，1，1，1的旋转
			System.out.println(findMinNum(array));
		}
	
		public static Integer findMinNum(int[] array) {
			if (array == null) {
				return null;
			}
			int leftIndex = 0;
			int rightIndex = array.length - 1;
			int mid = 0;
			// 最小的数就在这中间
			while (array[leftIndex] >= array[rightIndex]) {
				if (rightIndex - leftIndex <= 1) {
					// 这就是最小的了
					mid = rightIndex;
					break;
				}
				// 去中间的值，类似二分法查找
				mid = (leftIndex + rightIndex) / 2;
				// 前、中、后三个值都相等的情况，主要就是为了区分0，1，1，1，1这种数值相同的情况
				if (array[leftIndex] == array[rightIndex] && array[leftIndex] == array[mid]) {
					// 把指针在移动一下，不相等就继续变mid的值
					if (array[leftIndex + 1] != array[rightIndex - 1]) {
						mid = array[leftIndex + 1] < array[rightIndex - 1] ? (leftIndex + 1) : (rightIndex - 1);
						break;
					} else {
						leftIndex++;
						rightIndex--;
					}
				} else {
					if (array[mid] >= array[leftIndex])
						leftIndex = mid;
					else {
						if (array[mid] <= array[rightIndex])
							rightIndex = mid;
					}
				}
				return array[mid];
			}
			return null;
		}
	}
    ```
9. 斐波那契数列	     
    什么是斐波那契数列呢？ 就是f(0)=0;f(1)=1;f(n)=f(n-1)+f(n-2);           
    写一个函数，输入n，求斐波那契数列的第n项。           
    思路:标准的一个递归。
    ```java
	public long fibonacci1(int n) {
		if (n == 0) {
			return 0;
		}
		
		if (n == 1) {
			return 1;
		}
		
		return fibonacci1(n-1) + fibonacci1(n-2);
	}
    ```
    貌似很合理，但其实也是有问题的，这样会导致重复计算，例如我们在算f(10)，需要先求出f(9)和f(8)，而算f(9)又要求出f(8)和f(7)，很显然重复了。显然面试官不会满意的。那该怎么做呢？ 那就是累加。 
    ```java
	public class Fibonacci {
		public static long fibonacci(int n) {
			long result = 0;
			long preOne = 0;
			long preTwo = 1;
			if (n == 0) {
				return preOne;
			}
			if (n == 1) {
				return preTwo;
			}
			for (int i = 2; i <= n; i++) {
				result = preOne + preTwo;
				preOne = preTwo;
				preTwo = result;
			}
			return result;
		}
	}
	```
10. 2进制中1的个数               
    请实现一个函数,输入一个整数,输出该数二进制表示中 1 的个数。例如 把 9 表示成二进制是 1001;有 2 位是 1,因此如果输入 9,函数输出 2.      
    思路: 把一个整数减去1，再和原整数做与运算，会把该整数最右边的一个1变成0.那么一个整数的二进制表示中有多少个1，就可以进行多少次运算。       
    ```java
	public int numberOf1(int n) {
		int count = 0;
		while (n != 0) {
			count++;
			n = (n - 1) & n;
		}
		return count;
	}
    ```
    
11. 数值的整数次方           
    实现函数double Power(double base,int exponent),求base的exponent次方。不得使用库函数，同时不需要考虑大数问题。        
    思路:就是不断的累计去乘.            
    ```java
    public double powerWithExponent(double base, int exponent) {
		double result = 1.0;
		for (int i = 1; i <= exponent; i++) {
			result = result * base;
		}
		return result;
	}
    ```
    本来想着挺简单，其实已经写错了。因为exponent如果是0或者负数呢？         
    思路：当指数为负数的时候，可以先对指数求绝对值，然后算出次方的结果之后再取倒数。既然有求倒数，我们很自然的就要想到有没有可能对0求倒数，如果对0求倒数怎么办？当底数base是零且指数是负数的时候，我们不做特殊的处理，就会发现对0求倒数从而导致程序运行出错。怎么告诉函数的调用者出现了这种错误？在Java中可以抛出异常来解决。
    ```java
    public static double power(double base, int exponent) throws Exception {
		double result = 0.0;
		// 如果是求0的负数次幂
		if (equal(base, 0.0) && exponent < 0) {
			throw new Exception("0的负数次幂没有意义");
		}
		if (exponent < 0) {
			// 负数次幂，先取绝对值算出次方后再求倒数
			result = 1.0 / powerWithExpoment(base, -exponent);
		} else {
			result = powerWithExpoment(base, exponent);
		}
		return result;
	}

	private static double powerWithExpoment(double base, int exponent) {
		if (exponent == 0) {
			return 1;
		}
		if (exponent == 1) {
			return base;
		}
		double result = 1.0;
		for (int i = 1; i <= exponent; i++) {
			result = result * base;
		}
		return result;
	}

	/**
	 * 判断两个double数据是否相等
	 * @param num1
	 * @param num2
	 * @return
	 */
	private static boolean equal(double num1, double num2) {
		if ((num1 - num2 > -0.0000001) && num1 - num2 < 0.0000001) {
			return true;
		} else {
			return false;
		}

	}
    ```
12. 打印 1 到最大的 n 位数            
    输入数字n，按顺序打印出从1最大的的n位数十进制数。比如输入3，则打印出1，2，3一直到最大的3位数即999.           
    思路: 1位数就是`10-1`，两位数就是`10*10-1`三位数就是`10*10*10-1`              
    
    ```java
    public void print1ToMaxOfNDigits(int n) {
		int number = 1;
		int i = 0;
		while (i++ < n) {
			number *= 10;
		}
		for (int j = 1; j < number; ++j)
			System.out.println(j);
	}
    ```
    感觉挺简单，其实已经错了。因为没有规定n的值，如果很大的话，显然会超过int型的最大值。我们很自然的想到解决这个问题需要一个大数。最常用的也是最容易的用字符串或者数组表达大数。接下来我们用数组来解决大数问题。
    思路:每一位数都是0到9，这样弄一个数组，数组的长度就是n，每一位都是0-9，这样，循环去打印数组就可以了
    ```java
	public static void main(String[] args) {
		printToMaxOfNDigits(2);
	}

	public static void printToMaxOfNDigits(int n) {
		int[] array = new int[n];
		if (n <= 0)
			return;
		printArray(array, 0);
	}

	private static void printArray(int[] array, int n) {
		// 每一位都是0-9
		for (int i = 0; i < 10; i++) {
			if (n != array.length) {
				array[n] = i;
				// 递归弄另一位
				printArray(array, n + 1);
			} else {
				boolean isFirstNo0 = false;
				for (int j = 0; j < array.length; j++) {
					if (array[j] != 0) {
						System.out.print(array[j]);
						if (!isFirstNo0) {
							isFirstNo0 = true;
						}
					} else {
						if (isFirstNo0) {
							// 10 20 这种后位是0的
							System.out.print(array[j]);
						}
					}
				}
				System.out.println();
				// 打印完就return
				return;
			}
		}
	}
	```

13. 在O(1)时间删除链表节点            
    给定单向链表的头指针和一个节点指针，定义一个函数在O(1)时间删除该节点。          
    思路：在单向链表中删除一个节点，最常规的方法无疑是从链表的头结点开始，顺序遍历查找要删除的节点，并在链表中删除该节点。删除就是将这个要被删除的节点的前一节点设置成该要被删除节点的下一节点。- -！ 
    ```java
    public class DeleteListNodeTest {
		public static void main(String[] args) {
			ListNode head = new ListNode();
			ListNode second = new ListNode();
			ListNode third = new ListNode();
			head.nextNode = second;
			second.nextNode = third;
			head.data = 1;
			second.data = 2;
			third.data = 3;
			deleteNode(head, second);
			System.out.println(head.nextNode.data);
		}
	
		/**
		 * 
		 * @param head
		 *            头结点
		 * @param deListNode
		 *            将被删除的节点
		 */
		public static void deleteNode(ListNode head, ListNode deListNode) {
			if (deListNode == null || head == null) {
				return;
			}
	
			if (head == deListNode) {
				// 要删除的这个节点正好是头节点
				head = null;
			} else if (deListNode.nextNode == null) {
				// 要删除的这个节点正好是最后一个节点
				ListNode pointListNode = head;
				while (pointListNode.nextNode.nextNode != null) {
					pointListNode = pointListNode.nextNode;
				}
				pointListNode.nextNode = null;
			} else {
				// 要删除的节点是中间的节点，直接把该节点的值和next指向下一个节点就可以。
				deListNode.data = deListNode.nextNode.data;
				deListNode.nextNode = deListNode.nextNode.nextNode;
			}
		}
	}
	
	class ListNode {
		int data;
		ListNode nextNode;
	}
    ```
14. 调整数组顺序使奇数位于偶数前面              
    输入一个整数数组，实现一个函数来调整该函数数组中数字的顺序，使得
所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。
    思路: 维护两个指针，一个指向第一个元素，一个指向最后一个元素，然后通过指针的移动，来判断当前元素是奇数还是偶数，来交换位置。　　　
	```java
	public static void order(int[] array) {
		if (array == null || array.length == 0) {
			return;
		}
		int start = 0;
		int end = array.length - 1;
		while (start < end) {
			while (start < end && !isEven(array[start])) {
				// 如果当前位置是奇数，就下移指针
				start++;
			}
			while (start < end && isEven(array[end])) {
				// 第二个指针如果当前位置是偶数，就向前移动指针
				end--;
			}
			if (start < end) {
			    // 说明第一个指针指向的是偶数，第二个指针指向的是奇数，我们来更换他俩的位置。
				int temp = array[start];
				array[start] = array[end];
				array[end] = temp;
			}
		}
	}

	private static boolean isEven(int n) {
		return n % 2 == 0;
	}
	```
	
15. 链表中倒数第K个结点            
    输入一个链表，输出该链表中倒数第`k`个结点。   
    思路: 拿到倒数第k个节点，我们只需要知道该链表的总长度，然后我们从头开始遍历渠道第`totalLength-k`个就是了。如何拿到总长度，也简单就是遍历一遍就知道了。
	但是这样会牵扯到两次遍历，效率比较低。那怎么处理呢？也是使用两个指针，我们要保证第一个指针走到链表最后一个位置(totalLength)的时候，第二个指针正好指向倒数第`k`个节点(
	也就是从头开始第`totalLength-k+1个`)，那这两个指针之间差多少呢？`totalLength-(totalLength-k+1)`也就是`k-1`个位置，所以让第一个指针移动到第`k-1`个位置后，就让第二个指针
	开始移动，这样等第一个移动到最后一个元素的时候，第二个正好指向了倒数第`k`个元素。
	```java
	public class ListNodeTailText {
		public static void main(String[] args) {
			ListNode node1 = new ListNode();
			ListNode node2 = new ListNode();
			ListNode node3 = new ListNode();
			ListNode node4 = new ListNode();
			node1.nextNode = node2;
			node2.nextNode = node3;
			node3.nextNode = node4;
			node1.data = 1;
			node2.data = 2;
			node3.data = 3;
			node4.data = 4;
			ListNode resultListNode = findKToTail(node1, 3);
			System.out.println(resultListNode.data);
		}

		public static ListNode findKToTail(ListNode head, int k) {
			if (head == null || k == 0) {
				return null;
			}
			ListNode firstIndex = head;
			ListNode secondIndex = head;
			for (int i = 0; i < k; ++i) {
				if (firstIndex.nextNode != null) {
					firstIndex = firstIndex.nextNode;
				} else {
					return null;
				}
			}
			while (firstIndex != null) {
				secondIndex = secondIndex.nextNode;
				firstIndex = firstIndex.nextNode;
			}
			return secondIndex;
		}
	}

	class ListNode {
		int data;
		ListNode nextNode;
	}
	```
16. 反转链表           
    定义一个函数，输入一个链表的头结点，反转该链表并输出反转后链表的头结点。       
	思路:反转链表问的比较多，整体的思路就是从后往前来，这个问题我也是花了很长时间才弄明白，太笨了。
    也有两种方式：递归和普通的方式        

    ```java
    public class LinkedListDemo {
    	public static void main(String[] args) {
    		ListNode head = new ListNode();
    		ListNode second = new ListNode();
    		ListNode third = new ListNode();
    		ListNode forth = new ListNode();
    		head.nextNode = second;
    		second.nextNode = third;
    		third.nextNode = forth;
    		head.data = 1;
    		second.data = 2;
    		third.data = 3;
    		forth.data = 4;
    		ListNode resultListNode = reverse1(head);
    		System.out.println(resultListNode.data);
    	}
    
    	/**
    	 * 递归
    	 * 
    	 * @param head
    	 * @return
    	 */
    	public static ListNode reverse1(ListNode head) {
    		if (null == head || null == head.getNextNode()) {
    			return head;
    		}
    		// A B C -> A C B -> C B A
    		ListNode reversedHead = reverse1(head.getNextNode());
    		head.getNextNode().setNextNode(head);
    		head.setNextNode(null);
    		return reversedHead;
    	}
    
    	public static ListNode reverse2(ListNode head) {
    		if (null == head) {
    			return head;
    		}
    		// A B C
    		ListNode pre = head;  // A
    		ListNode cur = head.getNextNode();  // B
    		ListNode next;
    		while (cur != null) {
    			// next = C
    			next = cur.getNextNode();
    			// B -> A
    			cur.setNextNode(pre);
    			// pre = B
    			pre = cur;
    			// cur = C
    			cur = next;
    			// 第一轮下来就是 A B C -> A B A 
    			// 第二轮下来就是 C B A  pre = C cur = null
    			// 再继续就会跳出循环
    		}
    	
    		// 虽然已经是C B A 了，但是不要忘了此时A的next还是B，所以我们要将其设置为null
    		// 将原链表的头节点的下一个节点置为null，再将反转后的头节点赋给head
    		head.setNextNode(null);
    		head = pre;
    		// 到这就是返回C了。
    		return head;
    	}
    }
    
    class ListNode {
    	public ListNode nextNode;
    	public int data;
    
    	public ListNode getNextNode() {
    		return nextNode;
    	}
    
    	public void setNextNode(ListNode nextNode) {
    		this.nextNode = nextNode;
    	}
    
    	public int getData() {
    		return data;
    	}
    
    	public void setData(int data) {
    		this.data = data;
    	}
    }
    ```
17. 合并两个排序的链表
    输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按
    照递增排序的。	
	思路: 合并两个链表，按照递增顺序，那就是假设第一个链表是1 3 第二个链表是2 4 6那怎么去合并呢？
	先是比较两个链表的头结点，１和２比较，那合并后的新链表头肯定是１了，然后再拿2和3比较看谁是第二个结点，那可定是2了，到这里就确定了新链表的前两个结点，
	就是1 2 然后再用3和4比较确定谁是第三个，这是啥？这是递归。
	```java
	public class MergeListTest {
		public static void main(String[] args) {
			ListNode head1 = new ListNode();
			ListNode second1 = new ListNode();
			ListNode head2 = new ListNode();
			ListNode second2 = new ListNode();
			ListNode third2 = new ListNode();
			head1.nextNode = second1;
			head2.nextNode = second2;
			second2.nextNode = third2;
			head1.data = 1;
			second1.data = 3;
			head2.data = 2;
			second2.data = 2;
			third2.data = 2;
			MergeListTest test = new MergeListTest();
			ListNode result = test.mergeList(head1, head2);
			System.out.println(result.nextNode.nextNode.nextNode.nextNode.data);
		}

		public ListNode mergeList(ListNode head1, ListNode head2) {
			if (head1 == null) {
				return head2;
			} else if (head2 == null) {
				return head1;
			}
			ListNode mergeHead = null;
			if (head1.data < head2.data) {
				mergeHead = head1;
				mergeHead.nextNode = mergeList(head1.nextNode, head2);
			} else {
				mergeHead = head2;
				mergeHead.nextNode = mergeList(head1, head2.nextNode);
			}
			return mergeHead;
		}
	}

	class ListNode {
		public ListNode nextNode;
		public int data;

		public ListNode getNextNode() {
			return nextNode;
		}

		public void setNextNode(ListNode nextNode) {
			this.nextNode = nextNode;
		}

		public int getData() {
			return data;
		}

		public void setData(int data) {
			this.data = data;
		}
	}
	```
18. 树的子结构	               

    输入两颗二叉树 A 和 B，判断 B 是不是 A 的子结构。
	![image](https://raw.githubusercontent.com/CharonChui/Pictures/master/findchildbinarytree.png)	  
	思路:   想要判断B树是不是A的子树，那首先要先去在A树种遍历找到与B树根节点相同的结点，然后再从这个结点去遍历子结点
	看字节点与B树是否一样，如果不一样就继续往下遍历结点，找与B树根节点一直的结点，如此循环。
	以上图为例，我们先在A中找8的结点，发现A的根节点就是，然后继续看A的左子节点是8，而B的左子节点是9，锁着这
	肯定不是了，继续往下找为8的结点，发现在A树的第二层找打了，然后再进行判断该结点下的子节点，发现为9和2，正好与B的相同
	就是他了。
	```java
	public class Problem18 {
		public static void main(String args[]) {
			BinaryTreeNode root1 = new BinaryTreeNode();
			BinaryTreeNode node1 = new BinaryTreeNode();
			BinaryTreeNode node2 = new BinaryTreeNode();
			BinaryTreeNode node3 = new BinaryTreeNode();
			BinaryTreeNode node4 = new BinaryTreeNode();
			BinaryTreeNode node5 = new BinaryTreeNode();
			BinaryTreeNode node6 = new BinaryTreeNode();
			root1.leftNode = node1;
			root1.rightNode = node2;
			node1.leftNode = node3;
			node1.rightNode = node4;
			node4.leftNode = node5;
			node4.rightNode = node6;
			root1.value = 8;
			node1.value = 8;
			node2.value = 7;
			node3.value = 9;
			node4.value = 2;
			node5.value = 4;
			node6.value = 7;
			BinaryTreeNode root2 = new BinaryTreeNode();
			BinaryTreeNode a = new BinaryTreeNode();
			BinaryTreeNode b = new BinaryTreeNode();
			root2.leftNode = a;
			root2.rightNode = b;
			root2.value = 8;
			a.value = 9;
			b.value = 2;
			System.out.println(hasSubTree(root1, root2));
		}

		public static boolean hasSubTree(BinaryTreeNode root1, BinaryTreeNode root2) {
			boolean result = false;
			if (root1 != null && root2 != null) {
				if (root1.value == root2.value) {
					result = doesTree1HavaTree2(root1, root2);
					if (!result) {
						result = hasSubTree(root1.leftNode, root2);
					}
					if (!result) {
						result = hasSubTree(root1.rightNode, root2);
					}
				}
			}
			return result;
		}

		private static boolean doesTree1HavaTree2(BinaryTreeNode root1,
				BinaryTreeNode root2) {
			if (root2 == null) {
				return true;
			} else if (root1 == null)
				return false;
			if (root1.value != root2.value) {
				return false;
			}
			return doesTree1HavaTree2(root1.leftNode, root2.leftNode)
					&& doesTree1HavaTree2(root1.rightNode, root2.rightNode);
		}
	}

	class BinaryTreeNode {
		int value;
		BinaryTreeNode leftNode;
		BinaryTreeNode rightNode;
	}
	```
19. 二叉树的镜像
    请完成一个函数，输入一个二叉树，该函数输出它的镜像。	
    思路：什么事镜像？ 就像照镜子一样。打个比方现在的数是
    ```
            1                                                   1
        2         3      左图的二叉树的镜像就是             3          2
     4     5  6       7                              7       6  5        4
    ```
    就是从根节点开始，先前序遍历这棵树的每个结点,先转换它的两个子节点，然后再对这两个子节点下的子节点进行转换。
    ```java
    public class mirrorBinaryTreeTest {
        public static void main(String[] args) {
    		BinaryTreeNode root1 = new BinaryTreeNode();
    		BinaryTreeNode node1 = new BinaryTreeNode();
    		BinaryTreeNode node2 = new BinaryTreeNode();
    
    		BinaryTreeNode node3 = new BinaryTreeNode();
    		BinaryTreeNode node4 = new BinaryTreeNode();
    		BinaryTreeNode node5 = new BinaryTreeNode();
    		BinaryTreeNode node6 = new BinaryTreeNode();
    		root1.leftNode = node1;
    		root1.rightNode = node2;
    		node1.leftNode = node3;
    		node1.rightNode = node4;
    		node4.leftNode = node5;
    		node4.rightNode = node6;
    		root1.value = 8;
    		node1.value = 8;
    		node2.value = 7;
    		node3.value = 9;
    		node4.value = 2;
    		node5.value = 4;
    		node6.value = 7;
    		BinaryTreeNode rootBinaryTreeNode = mirrorBinaryTree(root1);
    	}
    
    	public static BinaryTreeNode mirrorBinaryTree(BinaryTreeNode root) {
    		if (root == null) {
    			return null;
    		}
    		if (root.leftNode == null && root.rightNode == null)
    			return null;
    		Stack<BinaryTreeNode> stack = new Stack<BinaryTreeNode>();
    		while (root != null || !stack.isEmpty()) {
    			while (root != null) {
    				BinaryTreeNode temp = root.leftNode;
    				root.leftNode = root.rightNode;
    				root.rightNode = temp;
    				stack.push(root);
    				root = root.leftNode;
    			}
    			root = stack.pop();
    			root = root.rightNode;
    		}
    		return root;
    	}
    }
    
    class BinaryTreeNode {
    	public int value;
    	public BinaryTreeNode leftNode;
    	public BinaryTreeNode rightNode;
    
    	public BinaryTreeNode() {
    
    	}
    
    	public BinaryTreeNode(int value) {
    		this.value = value;
    		this.leftNode = null;
    		this.rightNode = null;
    	}
    }
    ```
20. 顺时针打印矩阵
    输入一个矩阵,按照从外向里以顺时针的顺序依次打印出每一个数字。
    ```
    1   2   3    4  
    5   6   7    8
    9   10  11  12
    ```
    思路：顺时针打印也就是4步，从左往右、从上往下、从右向左、从下往上，然后继续循环如此，
    当然在每次循环的时候都要判断好，以免只有一行或者一列或者一个元素的情况。
    ```java
    public class printMatrixTest {
        public static void main(String[] args) {
    		int[][] arr = { { 1, 2, 3, 4 }, { 5, 6, 7, 8 }, { 9, 10, 11, 12 } };
    		printMatrixInCircle(arr);
    	}
    
    	public static void printMatrixInCircle(int[][] array) {
    		if (array == null) {
    			return;
    		}
    		int start = 0;
    		// 循环的次数就是维度要大于指针的2倍
    		while (array[0].length > start * 2 && array.length > start * 2) {
    			printOneCircle(array, start);
    			start++;
    		}
    	}
    
    	private static void printOneCircle(int[][] array, int start) {
    		int columns = array[0].length;
    		int rows = array.length;
    		int endX = columns - 1 - start;
    		int endY = rows - 1 - start;
    		// 从左到右打印一行
    		for (int i = start; i <= endX; i++) {
    			int number = array[start][i];
    			System.out.print(number + ",");
    		}
    		// 从上到下打印一列
    		if (start < endY) {
    			for (int i = start + 1; i <= endY; i++) {
    				int number = array[i][endX];
    				System.out.print(number + ",");
    			}
    		}
    		// 从右到左打印一行
    		if (start < endX && start < endY) {
    			for (int i = endX - 1; i >= start; i--) {
    				int number = array[endY][i];
    				System.out.print(number + ",");
    			}
    		}
    		// 从下到上打印一列
    		if (start < endY && start < endY - 1) {
    			for (int i = endY - 1; i >= start + 1; i--) {
    				int number = array[i][start];
    				System.out.print(number + ",");
    			}
    		}
    	}
    }
    ```
21. 包含`min`函数的栈        
    定义栈的数据结构,请在该类型中实现一个能够得到栈的最小元素的`min`函数。
    在该栈中,调用`min`、`push`及`pop`的时间复杂度都是O(1)          
    思路: 最先想到的就是用一个变量记录住最小的元素，但是如果这个最小的元素被取出了呢？ 
    怎么再返回剩下所有元素中最小的一个呢？很显然用一个变量记住是不行的，我们必须要用
    一个辅助栈来纪录这小小元素。
    ```java
    public class MinInStack {
        /**
    	 * 辅助栈，来纪录这些小的元素
    	 */
    	private MyStack<Integer> minStack = new MyStack<>();
    	private MyStack<Integer> dataStack = new MyStack<>();
    
    	public void push(Integer item) {
    		dataStack.push(item);
    		if (minStack.length == 0 || item <= minStack.head.data) {
    			minStack.push(item);
    		} else {
    			minStack.push(minStack.head.data);
    		}
    	}
    
    	public Integer pop() {
    		if (dataStack.length == 0 || minStack.length == 0) {
    			return null;
    		}
    		minStack.pop();
    		return dataStack.pop();
    	}
    
    	public Integer min() {
    		if (minStack.length == 0) {
    			return null;
    		}
    		return minStack.head.data;
    	}
    
    	public static void main(String[] args) {
    		MinInStack test = new MinInStack();
    		test.push(3);
    		test.push(2);
    		test.push(1);
    		System.out.println(test.pop());
    		System.out.println(test.min());
    	}
    }
    
    /**
     * 链表
     */
    class ListNode<K> {
    	K data;
    	ListNode<K> nextNode;
    }
    
    /**
     * 自定义栈的数据结构
     */
    class MyStack<K> {
    	public ListNode<K> head;
    	/**
    	 * 当前栈的大小
    	 */
    	public int length;
    
    	public void push(K item) {
    		ListNode<K> node = new ListNode<K>();
    		node.data = item;
    		node.nextNode = head;
    		head = node;
    		length++;
    	}
    
    	public K pop() {
    		if (head == null)
    			return null;
    		ListNode<K> temp = head;
    		head = head.nextNode;
    		length--;
    		return temp.data;
    	}
    
    }
    ```
22. 栈的压入、弹出序列         
    题目:输入两个整数序列,第一个序列表示栈的压入顺序,请判断第二个序列是 
    否为该栈的弹出序列。假设压入栈的所有数字均不相等。
    例如压栈序列为 1、2、3、4、5.序列 4、5、3、2、1 是压栈序列对应的一个
    弹出序列,但 4、3、5、1、2 却不是。        
    思路: 这道题我完全没看懂，看完后实在不理解是什么意思。 - -！搜了很久才弄明白。
    什么事对应的弹出序列呢？ 就是这个入栈后所有的可能弹出的方式。比如我可以完全按照1、2、3、4、5
    的方式去放入，然后取出就是5、4、3、2、1，也可以在1、2
    进入的时候先把2弹出，然后再继续添加3、4、5，这样的弹出顺序就是2、5、4、3、1。
    但是为什么4、3、5、1、2是不对的呢？ 这是因为1、2、3、4添加进入的时候，我们先弹出
    4然后再弹出3，因为以后要弹5，所以我们必须继续添加5，这样栈内就剩下了1、2、5，然后我们再
    往外弹出就只能是5、2、1，也就是他的顺序只能是4、3、5、2、1。        
    那怎么判断呢？ 如果下一个弹出的数字刚好是栈顶数字，那么直接弹出。
    如果下一个弹出的数字不在栈顶，我们把压栈序列中还没有入栈的数字压入辅助栈，
    直到把下一个需要弹出的数字压入栈顶为止。如果所有的数字都压入栈了仍没有
    找到下一个弹出的数字，那么该序列不可能是一个弹出序列。
    ```java
    public class PopOrderTest {
        public static void main(String[] args) {
    		int[] array1 = { 1, 2, 3, 4, 5 };
    		int[] array2 = { 4, 3, 5, 1, 2 };
    		System.out.println(isPopOrder(array1, array2));
    	}
    
    	public static boolean isPopOrder(int[] line1, int[] line2) {
    		if (line1 == null || line2 == null) {
    			return false;
    		}
    		int index = 0;
    		// 把line1中的元素都加入该栈
    		Stack<Integer> stack = new Stack<Integer>();
    		for (int i = 0; i < line2.length; i++) {
    			if (!stack.isEmpty() && stack.peek() == line2[i]) {
    				// 要取出的元素正好是栈顶的元素，就直接取出。
    				stack.pop();
    			} else {
    				if (index == line1.length) {
    					// line1的元素已经全部加入栈中，且要取出的元素仍然不是栈顶的元素，那就说明line1中不包含要取出的元素。直接返回false
    					return false;
    				} else {
    					// 只要要取出的元素不是栈顶的，就一直往栈里面加
    					do {
    						stack.push(line1[index++]);
    					} while (stack.peek() != line2[i] && index != line1.length);
    					
    					if (stack.peek() == line2[i]) {
    						stack.pop();
    					} else {
    						return false;
    					}
    				}
    			}
    		}
    		return true;
    	}
    }
    ```
23. 从上往下打印二叉树         
    从上往下打印二叉树的每个结点，同一层的结点按照从左到右的顺序打印。      
    思路: 每一次打印一个结点的时候，如果该结点有子节点，把该结点的子节点放到一个队列的尾。接下来到队列的头部取出最早进入队列的结点，重复前面打印操作，直到队列中所有的结点都被打印出为止。
    ```java
    public class Problem23 {
        public static void main(String args[]) {
    		BinaryTreeNode root1 = new BinaryTreeNode();
    		BinaryTreeNode node1 = new BinaryTreeNode();
    		BinaryTreeNode node2 = new BinaryTreeNode();
    		BinaryTreeNode node3 = new BinaryTreeNode();
    		BinaryTreeNode node4 = new BinaryTreeNode();
    		BinaryTreeNode node5 = new BinaryTreeNode();
    		BinaryTreeNode node6 = new BinaryTreeNode();
    
    		root1.leftNode = node1;
    		root1.rightNode = node2;
    		node1.leftNode = node3;
    		node1.rightNode = node4;
    		node4.leftNode = node5;
    		node4.rightNode = node6;
    		root1.value = 8;
    		node1.value = 8;
    		node2.value = 7;
    		node3.value = 9;
    		node4.value = 2;
    		node5.value = 4;
    		node6.value = 7;
    
    		Problem23 test = new Problem23();
    		test.printFromTopToBottom(root1);
    	}
    
    	public void printFromTopToBottom(BinaryTreeNode root) {
    		if (root == null)
    			return;
    		Queue<BinaryTreeNode> queue = new LinkedList<BinaryTreeNode>();
    		queue.add(root);
    		while (!queue.isEmpty()) {
    			BinaryTreeNode node = queue.poll();
    			System.out.print(node.value);
    			if (node.leftNode != null) {
    				queue.add(node.leftNode);
    			}
    			if (node.rightNode != null) {
    				queue.add(node.rightNode);
    			}
    		}
    	}
    }
    
    class BinaryTreeNode {
    	public int value;
    	public BinaryTreeNode leftNode;
    	public BinaryTreeNode rightNode;
    
    	public BinaryTreeNode() {
    
    	}
    
    	public BinaryTreeNode(int value) {
    		this.value = value;
    		this.leftNode = null;
    		this.rightNode = null;
    	}
    }
    ```


---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 


