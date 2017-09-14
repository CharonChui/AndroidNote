剑指Offer(下)
===

剑指Offer(上)一共是23道题。       

24. 二叉搜索树的后序遍历序列     
    输入一个整数数组,判断该数组是不是某二叉搜索树的后序遍历的结果。 是则返回`true`,否则返回`false`。假设输入的数组的任意两个数字都互不相同              
    思路:  在后序遍历得到的序列中，最后一个数字是树的根节点的值。
    数组中前面的数字可以分为两部分：第一部分是左子树结点的值，
    它们都比根节点的值小；第二部分是右子树结点的值，他们都比根节点的值大。     
     
    ```
    public class Problem24 {
        public static void main(String[] args) {
    		int[] array = { 5, 7, 6, 9, 11, 10, 8 };
    		System.out.println(verfiySequence(array));
    	}
    
    	public static boolean verfiySequence(int[] array) {
    		if (array == null || array.length == 0) {
    			return false;
    		}
    		int length = array.length;
    		// 最后一个是根节点
    		int root = array[length - 1];
    		// 左右子树的分界点，因为左子树都是小于根节点的，右子树都是大于的。
    		int cut = 0;
    		for (int i = 0; i < length - 1; i++) {
    			if (array[i] > root) {
    				// 到了右子树的分界点
    				cut = i + 1;
    				break;
    			}
    		}
    		if (cut == 0) {
    			// 没有右子树，都是左子树，然后就继续判断除了上面根节点后的其他所有节点，这些都是左子树的。
    			verfiySequence(Arrays.copyOfRange(array, 0, length - 1));
    		} else {
    			// 有右子树，判断从分界点开始后的所有数是不是都大于根节点。
    			for (int j = cut; j < length - 1; j++) {
    				if (array[j] < root) {
    					// 有不大于根节点值的数，肯定是错误的。
    					return false;
    				}
    			}
    		}
    		boolean left = true;
    		if (cut > 0) {
    			// 判断左子数里面的元素是否都对
    			left = verfiySequence(Arrays.copyOfRange(array, 0, cut));
    		}
    		boolean right = true;
    		if (cut < length - 1) {
    			// 右子树
    			right = verfiySequence(Arrays.copyOfRange(array, cut, length - 1));
    		}
    		return (right && left);
    	}
    }
    ```
25. 二叉树中和为某一值的路径        
    输入一颗二叉树和一个整数，打印出二叉树中结点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶结点所经过的所有的结点形成一条路径。      
    思路: 
    
    ```
    public class Problem25 {
        public static void main(String args[]) {
    		BinaryTreeNode root1 = new BinaryTreeNode();
    		BinaryTreeNode node1 = new BinaryTreeNode();
    		BinaryTreeNode node2 = new BinaryTreeNode();
    		BinaryTreeNode node3 = new BinaryTreeNode();
    		BinaryTreeNode node4 = new BinaryTreeNode();
    		root1.leftNode = node1;
    		root1.rightNode = node2;
    		node1.leftNode = node3;
    		node1.rightNode = node4;
    		root1.value = 10;
    		node1.value = 5;
    		node2.value = 12;
    		node3.value = 4;
    		node4.value = 7;
    		Problem25 testFindPath = new Problem25();
    		testFindPath.findPath(root1, 22);
    	}
    
    	public void findPath(BinaryTreeNode root, int sum) {
    		if (root == null)
    			return;
    		Stack<Integer> stack = new Stack<Integer>();
    		int currentSum = 0;
    		findPath(root, sum, stack, currentSum);
    	}
    
    	private void findPath(BinaryTreeNode root, int sum, Stack<Integer> stack, int currentSum) {
    		currentSum += root.value;
    		stack.push(root.value);
    		if (root.leftNode == null && root.rightNode == null) {
    			// 到节点的尾部了
    			if (currentSum == sum) {
    				for (int path : stack) {
    					System.out.print(path + " ");
    				}
    				System.out.println();
    			}
    		}
    		if (root.leftNode != null) {
    			findPath(root.leftNode, sum, stack, currentSum);
    		}
    		if (root.rightNode != null) {
    			findPath(root.rightNode, sum, stack, currentSum);
    		}
    		stack.pop();
    	}
    }
    
    class BinaryTreeNode {
    	public static int value;
    	public BinaryTreeNode leftNode;
    	public BinaryTreeNode rightNode;
    }
    ```

26. 复杂链表的复制      
    实现函数复制一个复杂链表。在复杂链表中,每个结点除了有一个 next 指针指向下一个结点外,还有一个指向链表中任意结点或 null。      
    

---

- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 



