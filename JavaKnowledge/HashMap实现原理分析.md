HashMap实现原理分析
===

HashMap基于Map接口实现，元素以键值对的方式存储，并且允许使用null建和null值，因为key不允许重复，因此只能有一个键为null,另外HashMap不能保证放入元素的顺序，它是无序的，和放入的顺序并不能相同。



## 原理

其底层数据结构是数组称之为哈希桶，每个桶(bucket)里面放的是链表，链表中的每个节点，就是哈希表中的每个元素。

通过hash的方法，通过put和get存储和获取对象。存储对象时，我们将K / V传给put方法时，它调用hashCode计算hash从而得到bucket位置，进一步存储，HashMap会根据当前bucket的占用情况自动调整容量 (超过 Load  Facotr则resize为原来的2倍)。获取对象时，我们 K传给get，它调用hashCodeO()计算hash从而得到bucket位置，并进一步调用equals()方法确定键值对。如果发生碰撞的时候，Hashmap通过链表将产生碰撞冲突的元素组织起来，在JDK8中，如果一个bucket中碰撞冲突的元素超过8哥，则使用红黑树来替换链表，从而提高速度。

因其底层哈希桶的数据结构是数组，所以也会涉及到扩容的问题。当HashMap的容量达到threshold域值时，就会触发扩容。扩容前后，哈希桶的长度一定会是2的次方。这样在根据key的hash值寻找对应的哈希桶时，可以用位运算替代取余操作，更加高效。而key的hash值，并不仅仅只是key对象的hashCode()方法的返回值，还会经过扰动函数的扰动，以使hash值更加均衡。因为hashCode()是int类型，取值范围是40多亿，只要哈希函数映射的比较均匀松散，碰撞几率是很小的。 但就算原本的hashCode()取得很好，每个key的hashCode()不同，但是由于HashMap的哈希桶的长度远比hash取值范围小，默认是16，所以当对hash值以桶的长度取余，以找到存放该key的桶的下标时，由于取余是通过与操作完成的，会忽略hash值的高位。因此只有hashCode()的低位参加运算，发生不同的hash值，但是得到的index相同的情况的几率会大大增加，这种情况称之为hash碰撞。 即，碰撞率会增大。扰动函数就是为了解决hash碰撞的。它会综合hash值高位和低位的特征，并存放在低位，因此在与运算时，相当于高低位一起参与了运算，以减少hash碰撞的概率。（在JDK8之前，扰动函数会扰动四次，JDK8简化了这个操作）扩容操作时，会new一个新的Node数组作为哈希桶，然后将原哈希表中的所有数据(Node节点)移动到新的哈希桶中，相当于对原哈希表中所有的数据重新做了一个put操作。所以性能消耗很大，可想而知，在哈希表的容量越大时，性能消耗越明显。扩容时，如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位= low位+原哈希桶容量如果追加节点后，链表数量》=8，则转化为红黑树由迭代器的实现可以看出，遍历HashMap时，顺序是按照哈希桶从低到高，链表从前往后，依次遍历的。





## JDK1.7

HashMap在JDK1.8中发生了改变，下面的部分是基于JDK1.7的分析。HashMap主要是用数组来存储数据的，我们都知道它会对key进行哈希运算，哈希运算会有重复的哈希值，对于哈希值的冲突，HashMap采用链表来解决的。
在HashMap里有这样的一句属性声明:   

```java
transient Entry[] table;
```
可以看到Map是通过数组的方式来储存Entry那Entry是神马呢？就是HashMap存储数据所用的类，它拥有的属性如下:   
```java
static class Entry implements Map.Entry {
	final K key;
	V value;
	Entry next;
	final int hash;
	...//More code goes here
}  
```
看到next了吗？next就是为了哈希冲突而存在的。比如通过哈希运算，一个新元素应该在数组的第10个位置，但是第10个位置已经有Entry，那么好吧，将新加的元素也放到第10个位置，将第10个位置的原有Entry赋值给当前新加的Entry的next属性。数组存储的是链表，链表是为了解决哈希冲突的，这一点要注意。

好了，总结一下:       

- HashMap中有一个叫table的Entry数组。这个数组存储了Entry类的对象。Entry类包含了key-value作为实例变量。

- table的索引在逻辑上叫做“桶”(bucket)，它存储了链表的第一个元素。

- 每当往Hashmap里面存放key-value对的时候，都会为它们实例化一个Entry对象，这个Entry对象就会存储在前面提到的Entry数组table中。根据key的hashcode()方法计算出来的hash值来决定在Entry数组的索引(所在的桶)。

- 如果两个key有相同的hash值，他们会被放在table数组的同一个桶里面。

- key的equals()方法用来确保key的唯一性。

    

接下来看一下put方法:      

```java
/**
* Associates the specified value with the specified key in this map. If the
* map previously contained a mapping for the key, the old value is
* replaced.
*
* @param key
*            key with which the specified value is to be associated
* @param value
*            value to be associated with the specified key
* @return the previous value associated with <tt>key</tt>, or <tt>null</tt>
*         if there was no mapping for <tt>key</tt>. (A <tt>null</tt> return
*         can also indicate that the map previously associated
*         <tt>null</tt> with <tt>key</tt>.)
*/
public V put(K key, V value) {
    // 对key做null检查。如果key是null，会被存储到table[0]，因为null的hash值总是0。
	if (key == null)
		return putForNullKey(value);
	// 计算key的hash值,hash值用来找到存储Entry对象的数组的索引。有时候hash函数可能写的很不好，所以JDK的设计者添加了另一个叫做hash()的方法，它接收刚才计算的hash值作为参数	
	int hash = hash(key.hashCode());
	// indexFor(hash,table.length)用来计算在table数组中存储Entry对象的精确的索引
	int i = indexFor(hash, table.length);
	
	for (Entry<k , V> e = table[i]; e != null; e = e.next) {
	    // 如果这个位置已经有了(也就是hash值一样)就用链表来存了。 开始迭代链表
		Object k;
		// 直到Entry->next为null，就把当前的Entry对象变成链表的下一个节点。
		if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
		    // 如果我们再次放入同样的key会怎样呢？它会替换老的value。在迭代的过程中，会调用equals()方法来检查key的相等性(key.equals(k))，
			// 如果这个方法返回true，它就会用当前Entry的value来替换之前的value。
			V oldValue = e.value;
			e.value = value;
			e.recordAccess(this);
			return oldValue;
		}
	}
	// 如果计算出来的索引位置没有元素，就直接把Entry对象放到那个索引上。
	modCount++;
	addEntry(hash, key, value, i);
	return null;
}
```
再看一下get方法:　　　　
```java
/**
  * Returns the value to which the specified key is mapped, or {@code null}
  * if this map contains no mapping for the key.
  *
  * <p>
  * More formally, if this map contains a mapping from a key {@code k} to a
  * value {@code v} such that {@code (key==null ? k==null :
  * key.equals(k))}, then this method returns {@code v}; otherwise it returns
  * {@code null}. (There can be at most one such mapping.)
  *
  * </p><p>
  * A return value of {@code null} does not <i>necessarily</i> indicate that
  * the map contains no mapping for the key; it's also possible that the map
  * explicitly maps the key to {@code null}. The {@link #containsKey
  * containsKey} operation may be used to distinguish these two cases.
  *
  * @see #put(Object, Object)
  */
public V get(Object key) {
    // 如果key是null，table[0]这个位置的元素将被返回。
	if (key == null)
		return getForNullKey();
	// 计算hash值
	int hash = hash(key.hashCode());
	// indexFor(hash,table.length)用来计算要获取的Entry对象在table数组中的精确的位置，使用刚才计算的hash值。
	for (Entry<k , V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
		Object k;
		// 在获取了table数组的索引之后，会迭代链表，调用equals()方法检查key的相等性，如果equals()方法返回true，get方法返回Entry对象的value，否则，返回null。
		if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
    		return e.value;
		}
	return null;
}
```



## JDK1.8

在Jdk1.8中HashMap的实现方式做了一些改变，但是基本思想还是没有变得，只是在一些地方做了优化，下面来看一下这些改变的地方,数据结构的存储由数组+链表的方式，变化为数组+链表+红黑树的存储方式，当链表长度超过阈值（8）时，将链表转换为红黑树。利用红黑树快速增删改查的特点来提高HashMap的性能，其中会用到红黑树的插入、删除、查找等算法。HashMap中，如果key经过hash算法得出的数组索引位置全部不相同，即Hash算法非常好，那样的话，getKey方法的时间复杂度就是O(1)，如果Hash算法技术的结果碰撞非常多，假如Hash算极其差，所有的Hash算法结果得出的索引位置一样，那样所有的键值对都集中到一个桶中，或者在一个链表中，或者在一个红黑树中，时间复杂度分别为O(n)和O(lgn)。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/hashmap_data_structure.jpg)



既然发生了变化，那这里就以1.8的源码基础上再做一下分析:  

1. 首先看一下对应的两个Node类:  

    ```java
    // 单链表
    static class Node<K,V> implements Map.Entry<K,V> {
        // 用于定位数组的索引位置
        final int hash;
        final K key;
        V value;
        // 链表的下一个node
        Node<K,V> next;
    
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
    
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
    	// 每一个节点的hashcode值，是将key和value的hashCode值异或得到的
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
    
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
    
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
    ```

    ```java
    static class LinkedHashMapEntry<K,V> extends HashMap.Node<K,V> {
        LinkedHashMapEntry<K,V> before, after;
        LinkedHashMapEntry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
    ```

    ```java
    // 红黑树
    static final class TreeNode<K,V> extends LinkedHashMap.LinkedHashMapEntry<K,V> {
        TreeNode<K,V> parent;  // red-black tree links
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
        /**
         * Forms tree of the nodes linked from this node.
         * @return root of tree
         */
        final void treeify(Node<K,V>[] tab) {
       		...
        }
    
        /**
         * Returns a list of non-TreeNodes replacing those linked from
         * this node.
         */
        final Node<K,V> untreeify(HashMap<K,V> map) {
            ...
        }
    
        /**
    	 * 红黑树的插入
         * Tree version of putVal.
         */
        final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                       int h, K k, V v) {
            Class<?> kc = null;
            boolean searched = false;
            TreeNode<K,V> root = (parent != null) ? root() : this;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph; K pk;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((pk = p.key) == k || (k != null && k.equals(pk)))
                    return p;
                else if ((kc == null &&
                          (kc = comparableClassFor(k)) == null) ||
                         (dir = compareComparables(kc, k, pk)) == 0) {
                    if (!searched) {
                        TreeNode<K,V> q, ch;
                        searched = true;
                        if (((ch = p.left) != null &&
                             (q = ch.find(h, k, kc)) != null) ||
                            ((ch = p.right) != null &&
                             (q = ch.find(h, k, kc)) != null))
                            return q;
                    }
                    dir = tieBreakOrder(k, pk);
                }
    
                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    Node<K,V> xpn = xp.next;
                    TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    xp.next = x;
                    x.parent = x.prev = xp;
                    if (xpn != null)
                        ((TreeNode<K,V>)xpn).prev = x;
                    moveRootToFront(tab, balanceInsertion(root, x));
                    return null;
                }
            }
        }
    }
    ```

2. 再来看一下HashMap的实现类及构造函数

    ```java
    public class HashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V>, Cloneable, Serializable {
    	// 默认的初始化容量大小，必须是2的倍数，默认是16，为啥必须是2的倍数，后面会说
        static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
        // 在构造函数中未指定时使用的负载因子。
        static final float DEFAULT_LOAD_FACTOR = 0.75f;
        // 使用红黑树树而不链表的容器计数阈值
        static final int TREEIFY_THRESHOLD = 8;
        // 用于在执行过程中取消使用红黑树（拆分）箱调整大小操作的箱计数阈值，
        // 为啥这里转成红黑树是8，而从红黑树转成链表是6？ 在hash函数设计合理的情况下，发生hash碰撞8次的几率为百万分之6，概率说话。。因为8够用了，至于为什么转回来是6，因为如果hash碰撞次数在8附近徘徊，会一直发生链表和红黑树的转化，为了预防这种情况的发生。
        static final int UNTREEIFY_THRESHOLD = 6;
        // 哈希桶，存储数据的数组
        transient Node<K,V>[] table;
        // Node[] table的初始化长度length(默认值是16)，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node(键值对)个数。threshold = (capacity * load factor)，超过这个数量后就会进行重新resize(扩容)，扩容一后的HashMap容量是之前容量的两倍
        int threshold;
        
        
        public HashMap(int initialCapacity, float loadFactor) {
            if (initialCapacity < 0)
                throw new IllegalArgumentException("Illegal initial capacity: " +
                                                   initialCapacity);
            if (initialCapacity > MAXIMUM_CAPACITY)
                initialCapacity = MAXIMUM_CAPACITY;
            if (loadFactor <= 0 || Float.isNaN(loadFactor))
                throw new IllegalArgumentException("Illegal load factor: " +
                                                   loadFactor);
            this.loadFactor = loadFactor;
            this.threshold = tableSizeFor(initialCapacity);
        }
    
        public HashMap(int initialCapacity) {
            this(initialCapacity, DEFAULT_LOAD_FACTOR);
        }
    
        /**
         * Constructs an empty <tt>HashMap</tt> with the default initial capacity
         * (16) and the default load factor (0.75).
         */
        public HashMap() {
            this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
        }
    }
    ```

### 确定哈希桶数组索引位置

不管增加、删除、查找键值对，定位到哈希桶数组的位置都是很关键的第一步。前面说过HashMap的数据结构是数组和链表的结合，所以我们当然希望这个HashMap里面的元素位置尽量分布均匀些，尽量使得每个位置上的元素数量只有一个，那么当我们用hash算法求得这个位置的时候，马上就可以知道对应位置的元素就是我们要的，不用遍历链表，大大优化了查询的效率。HashMap定位数组索引位置，直接决定了hash方法的离散性能。先看看源码的实现：  

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

hash函数是先拿到通过key 的hashcode，是32位的int值，然后让hashcode的高16位和低16位进行异或操作。为什么要这样设计呢？ 主要有两个原因 ：  

1. 一定要尽可能降低hash碰撞，越分散越好。
2. 算法一定要尽可能高效，因为这是高频操作，因此采用位运算。

因为hashcode是32位的int值int值范围为**-2147483648~2147483647**，前后加起来大概40亿的映射空间。只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。你想，如果HashMap数组的初始大小才16，用之前需要对数组的长度取模运算，得到的余数才能用来访问数组下标。源码中模运算就是把散列值和数组长度-1做一个"与"操作，位运算比%运算要快。





对于任意给定的对象，只要它的hashCode()返回值相同，那么程序调用所计算得到的Hash码值总是相同的。我们首先想到的就是把hash值对数组长度取模运算，这样一来，元素的分布相对来说是比较均匀的。但是，模运算的消耗还是比较大的，在HashMap中是这样做的：调用 (table.length  -1) & hash来计算该对象应该保存在table数组的哪个索引处。这个方法非常巧妙，它通过 (table.length  -1) & hash来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，当length总是2的n次方时， (table.length  -1) & hash运算等价于对length取模，也就是h%length，但是&比%具有更高的效率。

当length总是2的倍数时，h & (length-1) 将是一个非常巧妙的设计：

- 假设h=5,length=16, 那么h & length - 1将得到 5；
- 如果h=6,length=16, 那么h & length - 1将得到 6
- 如果h=15,length=16, 那么h & length - 1将得到 15；
- 但是当h=16时 , length=16 时，那么h & length - 1将得到0了；
- 当h=17时, length=16时，那么h & length - 1将得到1了。

这样保证计算得到的索引值总是位于 table 数组的索引之内。



在JDK1.8的实现中，优化了高位运算的算法，通过hashCode()的高16位异或低16位实现的：(h = k.hashCode()) ^ (h >>>  16)，主要是从速度、功效、质量来考虑的，这么做可以在数组table的length比较小的时候，也能保证考虑到高低Bit都参与到Hash的计算中，同时不会有太大的开销。

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/hashmap_hash.bmp)



### put

```java
    public V put(K key, V value) {
        // 对key调用hash()方法获取hash值
        return putVal(hash(key), key, value, false, true);
    }


    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 1. 如果table还没初始化就初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 2. 根据键值key计算的hash值来得到插入的数组索引i，判断tab[i]是否为null，数组索引通过(n -1) & hash来获得
        if ((p = tab[i = (n - 1) & hash]) == null)
            // 3. 如果tab[索引]的值为null，那就说明这个索引没有数组桶，直接新建一个数组桶
            tab[i] = newNode(hash, key, value, null);
        else {
           	// 4. 否则就是目前已经存在该索引的数组桶了，要继续判断是链表还是红黑树。
            Node<K,V> e; K k;
            // 5. 判断table[索引]处的收个元素是否和key一样，如果首个就是那就直接覆盖value，不用再找了
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                // 6. 数组桶的首个元素就是，直接覆盖value的值
                e = p;
            else if (p instanceof TreeNode)
                // 7. 数组桶首个元素不是，并且链表是红黑树，红黑树直接插入键值对
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
            	// 8. 目前为链表，开始遍历链表准备插入
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 添加到目前的节点的next上
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // 判断链表长度是否大于8，如果大于直接转换成红黑树，插入键值对
                            treeifyBin(tab, hash);
                        break;
                    }
                    
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 判断当前存在的键值对数量size是否超过了最大容量threshold，如果超过就调用resize扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

上面主要有两个方法一个是红黑树的插入，一个是treeifyBin()也就是把链表转换成红黑树

```java
// MIN_TREEIFY_CAPACITY 的值为64，若当前table的length不够，则resize()
// 将桶内所有的 链表节点 替换成 红黑树节点
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 如果当前哈希表为空，或者哈希表中元素的个数小于树形化阈值(默认为 64)，就去新建(扩容)
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    // 如果哈希表中的元素个数超过了树形化阈值，则进行树形化
    // e 是哈希表中指定位置桶里的链表节点，从第一个开始
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 红黑树的头、尾节点
        TreeNode<K,V> hd = null, tl = null;
        do {
            // 新建一个树形节点，内容和当前链表节点 e 一致
            TreeNode<K,V> p = replacementTreeNode(e, null);
            // 确定树头节点
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        // 让桶的第一个元素指向新建的红黑树头结点，以后这个桶里的元素就是红黑树而不是链表了
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

### resize()扩容

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组，就像我们用一个小桶装水，如果想装更多的水，就得换大水桶。

resize()方法用于初始化数组或数组扩容，每次扩容后容量为原来的2倍，并进行数据迁移。 



例如我们从 16 扩展为 32 时，具体的变化如下所示：

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/resize1.bmp)




因此元素在重新计算hash之后，因为n变为 2 倍，那 n-1的mask范围在高位多1bit (红色)，因此新的 index就会发生这样的变化：

![](https://raw.githubusercontent.com/CharonChui/Pictures/master/resize2.bmp)


因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成 “原索引 + oldCap”。可以看看下图为16扩充为32的resize示意图：

![resize](https://raw.githubusercontent.com/CharonChui/Pictures/master/resize3.bmp)




这个设计确实非常的巧妙，既省去了重新计算 hash 值的时间，而且同时，**由于新增的 1bit 是 0 还是 1 可以认为是随机的，因此 resize 的过程，均匀的把之前的冲突的节点分散到新的 bucket 了**。

```java
final Node<K,V>[] resize() {
    //oldTab 为当前表的哈希桶
    Node<K,V>[] oldTab = table;
    //当前哈希桶的容量 length
    int oldCap = (oldTab == null) ? 0 : oldTab.length
    //当前的阈值
    int oldThr = threshold;
    //初始化新的容量和阈值为 0
    int newCap, newThr = 0;
    //如果当前容量大于 0
    if (oldCap > 0) {
        //超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        //没超过最大值，就扩充为原来的 2 倍 
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                oldCap >= DEFAULT_INITIAL_CAPACITY)
            //如果旧的容量大于等于默认初始容量 16, 那么新的阈值也等于旧的阈值的两倍
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;//那么新表的容量就等于旧的阈值
    else {// zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;//此时新表的容量为默认的容量 16
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);//新的阈值为默认容量 16 * 默认加载因子 0.75f = 12
    }
    if (newThr == 0) {//如果新的阈值是 0，对应的是当前表是空的，但是有阈值的情况
        float ft = (float)newCap * loadFactor;//根据新表容量和加载因子求出新的阈值
        //进行越界修复
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    //更新阈值
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    //根据新的容量构建新的哈希桶
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    //更新哈希桶引用
    table = newTab;
    //如果以前的哈希桶中有元素
    //下面开始将当前哈希桶中的所有节点转移到新的哈希桶中
    if (oldTab != null) {
        //把每个 bucket 都移动到新的 buckets 中
        for (int j = 0; j < oldCap; ++j) {
            //取出当前的节点 e
            Node<K,V> e;
            //如果当前桶中有元素,则将链表赋值给 e
            if ((e = oldTab[j]) != null) {
                //将原哈希桶置空以便 GC
                oldTab[j] = null;
                //如果当前链表中就一个元素，（没有发生哈希碰撞）
                if (e.next == null)
                    //直接将这个元素放置在新的哈希桶里。
                    //注意这里取下标是用哈希值与桶的长度-1。由于桶的长度是2的n次方，这么做其实是等于一个模运算。但是效率更高
                    newTab[e.hash & (newCap - 1)] = e;
                    //如果发生过哈希碰撞 ,而且是节点数超过8个，转化成了红黑树
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                //如果发生过哈希碰撞，节点数小于 8 个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
                else { // preserve order
                    //因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即 low 位， 或者扩容后的下标，即 high 位。 high 位 = low 位 + 原哈希桶容量
                    //低位链表的头结点、尾节点
                    Node<K,V> loHead = null, loTail = null;
                    //高位链表的头节点、尾节点
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;//临时节点 存放 e 的下一个节点
                    do {
                        next = e.next;
                        //这里又是一个利用位运算 代替常规运算的高效点：利用哈希值与旧的容量，可以得到哈希值去模后，是大于等于 oldCap 还是小于 oldCap，等于 0 代表小于 oldCap，应该存放在低位，否则存放在高位
                        if ((e.hash & oldCap) == 0) {
                            //给头尾节点指针赋值
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }//高位也是相同的逻辑
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }//循环直到链表结束
                    } while ((e = next) != null);
                    //将低位链表存放在原 index 处，
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    //将高位链表存放在新 index 处
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

再看一下往哈希表里插入一个节点的putVal函数,如果参数onlyIfAbsent是true，那么不会覆盖相同key的值value。如果evict是false。那么表示是在初始化时调用的

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    //tab存放 当前的哈希桶， p用作临时链表节点  
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    //如果当前哈希表是空的，代表是初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        //那么直接去扩容哈希表，并且将扩容后的哈希桶长度赋值给n
        n = (tab = resize()).length;
    //如果当前index的节点是空的，表示没有发生哈希碰撞。 直接构建一个新节点Node，挂载在index处即可。
    //这里再啰嗦一下，index 是利用 哈希值 & 哈希桶的长度-1，替代模运算
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {//否则 发生了哈希冲突。
        //e
        Node<K,V> e; K k;
        //如果哈希值相等，key也相等，则是覆盖value操作
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;//将当前节点引用赋值给e
        else if (p instanceof TreeNode)//红黑树暂且不谈
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {//不是覆盖操作，则插入一个普通链表节点
            //遍历链表
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {//遍历到尾部，追加新节点到尾部
                    p.next = newNode(hash, key, value, null);
                    //如果追加节点后，链表数量》=8，则转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                //如果找到了要覆盖的节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        //如果e不是null，说明有需要覆盖的节点，
        if (e != null) { // existing mapping for key
            //则覆盖节点值，并返回原oldValue
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            //这是一个空实现的函数，用作LinkedHashMap重写使用。
            afterNodeAccess(e);
            return oldValue;
        }
    }
    //如果执行到了这里，说明插入了一个新的节点，所以会修改modCount，以及返回null。

    //修改modCount
    ++modCount;
    //更新size，并判断是否需要扩容。
    if (++size > threshold)
        resize();
    //这是一个空实现的函数，用作LinkedHashMap重写使用。
    afterNodeInsertion(evict);
    return null;
}
```



扩容就是使用一个容量更大的数组来代替已有的容量小的数组，transfer()方法将原有Entry数组的元素拷贝到新的Entry数组里。

* 运算尽量都用位运算代替，更高效。
* 对于扩容导致需要新建数组存放更多元素时，除了要将老数组中的元素迁移过来，也记得将老数组中的引用置null，以便GC
* 取下标 是用 哈希值 与运算 （桶的长度-1） i = (n - 1) & hash。 由于桶的长度是2的n次方，这么做其实是等于 一个模运算。但是效率更高
* 扩容时，如果发生过哈希碰撞，节点数小于8个。则要根据链表上每个节点的哈希值，依次放入新哈希桶对应下标位置。
* 因为扩容是容量翻倍，所以原链表上的每个节点，现在可能存放在原来的下标，即low位， 或者扩容后的下标，即high位。 high位= low位+原哈希桶容量
* 利用哈希值 与运算 旧的容量 ，if ((e.hash & oldCap) == 0),可以得到哈希值去模后，是大于等于oldCap还是小于oldCap，等于0代表小于oldCap，应该存放在低位，否则存放在高位。这里又是一个利用位运算 代替常规运算的高效点
* 如果追加节点后，链表数量》=8，则转化为红黑树
* 插入节点操作时，有一些空实现的函数，用作LinkedHashMap重写使用。


### get

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 找到索引，判断数组桶中的第一个元素是不是
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // 第一个元素不是的话，就看是不是红黑树还是链表，然后一直去找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```





## JDK 7 与 JDK 8 中关于 HashMap的对比

1. JDK8为红黑树 + 链表 + 数组的形式，当桶内元素大于8时，便会树化。
2. hash值的计算方式不同 (jdk 8 简化)。
3. JDK7中table在创建hashmap时分配空间，而8中在put的时候分配。
4. 链表的插入方式从头插法改成了尾插法，简单说就是插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后；因为头插法会使链表发生反转，多线程环境下会产生环；
5. 在resize操作中，7 需要重新进行index的计算，而8不需要，通过判断相应的位是0还是1，要么依旧是原index，要么是oldCap + 原 index。
6. 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；





## 问题

1. 为什么capcity是2的幂？
     因为算index时用的是(n-1) & hash，这样就能保证n-1是全为1的二进制数，如果不全为1的话，存在某一位为 0，那么0，1与0与的结果都是0，这样便有可能将两个hash不同的值最终装入同一个桶中，造成冲突。所以必须是2的幂。这是为了服务key映射到index的Hash算法的，公式index=hashcode(key)&(length-1)，初始长度(16-1)，二进制为1111&hashcode结果为hashcode最后四位，能最大程度保持平均，二的幂数保证二进制为1，保持hashcode最后四位。这种算法在保持分布均匀之外，效率也非常高。

2. 为什么需要使用加载因子，为什么需要扩容呢？

    因为如果填充比很大，说明利用的空间很多，如果一直不进行扩容的话，链表就会越来越长，这样查找的效率很低，因为链表的长度很大（当然最新版本使用了红黑树后会改进很多），扩容之后，将原来链表数组的每一个链表分成奇偶两个子链表分别挂在新链表数组的散列位置，这样就减少了每个链表的长度，增加查找效率。HashMap 本来是以空间换时间，所以填充比没必要太大。但是填充比太小又会导致空间浪费。如果关注内存，填充比可以稍大，如果主要关注查找性能，填充比可以稍小。

3. 为什么 HashMap 是线程不安全的，实际会如何体现？

    - 如果多个线程同时使用put方法添加元素，假设正好存在两个put的key发生了碰撞 (hash值一样)，那么根据HashMap的实现，这两个key会添加到数组的同一个位置，这样最终就会发生其中一个线程的put的数据被覆盖。
- 如果多个线程同时检测到元素个数超过数组大小 * loadFactor。这样会发生多个线程同时对hash数组进行扩容，都在重新计算元素位置以及复制数据，但是最终只有一个线程扩容后的数组会赋给table，也就是说其他线程的都会丢失，并且各自线程put的数据也丢失。且会引起死循环的错误。
    
4. 与HashTable的区别

    Hashtable是遗留类，很多映射的常用功能与HashMap类似，不同的是它承自Dictionary类，并且是线程安全的，任一时间只有一个线程能写Hashtable，它的并发性不如ConcurrentHashMap，因为ConcurrentHashMap引入了分段锁。Hashtable不建议在新代码中使用，不需要线程安全的场合可以用HashMap替换，需要线程安全的场合可以用ConcurrentHashMap替换。

    - 与之相比HashTable是线程安全的，且不允许key、value是null。
    - HashTable默认容量是11。
    - HashTable是直接使用key的hashCode(key.hashCode())作为hash值，不像HashMap内部使用static final int hash(Object key)扰动函数对key的hashCode进行扰动后作为hash值。
    - HashTable取哈希桶下标是直接用模运算%.（因为其默认容量也不是2的n次方。所以也无法用位运算替代模运算）
    - 扩容时，新容量是原来的2倍+1。int newCapacity = (oldCapacity << 1) + 1;
    - Hashtable是Dictionary的子类同时也实现了Map接口，HashMap是Map接口的一个实现类
    
5. 扩容的时候为什么1.8 不用重新hash就可以直接定位原节点在新数据的位置呢?

    这是由于扩容是扩大为原数组大小的2倍，用于计算数组位置的掩码仅仅只是高位多了一个1，举个例子：

      扩容前长度为16，用于计算 (n-1) & hash 的二进制n - 1为0000 1111， 

       扩容后为32后的二进制就高位多了1，============>为0001 1111。因为是& 运算，1和任何数 & 都是它本身，那就分二种情况，如下图：原数据hashcode高位第4位为0和高位为1的情况；第四位高位为0，重新hash数值不变，第四位为1，重新hash数值比原来大16（旧数组的容量）。

    


参考:  

- [从 JDK7 与 JDK8 对比详细分析 HashMap 的原理与优化](https://allenmistake.top/2019/05/13/hashmap/)
- [面试必备：HashMap源码解析（JDK8）](https://blog.csdn.net/zxt0601/article/details/77413921)







---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	
