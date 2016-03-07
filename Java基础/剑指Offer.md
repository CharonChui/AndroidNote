剑指Offer
===

最近面试，遇到一些笔试题，写不上来，内心是崩溃的，该好好复习下了，所以决定仔细做一遍，随便也整理下，方便大家学习。

1. 我没找到第一题是什么- -!,谁知道的给补充下吧

2. 实现单例模式
    单例的实现分为好几种:    
	- 饿汉式
	- 懒汉式
	- 枚举
	
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
	
	我这里写一种自我感觉是单例最完美的实现方式:　　
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
	1   6   11
	5   9   15
	7   13   20
	
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
		}
		System.out.println(headNode.data);
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
    用两个栈实现一个队列，实现对了的两个函数`appendTail`和
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

	

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 