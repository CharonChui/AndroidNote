HashMap实现原理分析
===

`HashMap`主要是用数组来存储数据的，我们都知道它会对`key`进行哈希运算，哈系运算会有重复的哈希值，对于哈希值的冲突，`HashMap`采用链表来解决的。
`在HashMap`里有这样的一句属性声明:   
```java
transient Entry[] table;
```
可以看到`Map`是通过数组的方式来储存`Entry`那`Entry`是神马呢？就是`HashMap`存储数据所用的类，它拥有的属性如下:   
```java
static class Entry implements Map.Entry {
	final K key;
	V value;
	Entry next;
	final int hash;
	...//More code goes here
}  
```
看到`next`了吗？`next`就是为了哈希冲突而存在的。比如通过哈希运算，一个新元素应该在数组的第10个位置，但是第10个位置已经有Entry，那么好吧，
将新加的元素也放到第10个位置，将第10个位置的原有`Entry`赋值给当前新加的`Entry`的`next`属性。数组存储的是链表，链表是为了解决哈希冲突的，这一点要注意。

好了，总结一下:       

- `HashMap`中有一个叫`table`的`Entry`数组。
- 这个数组存储了`Entry`类的对象。`HashMap`类有一个叫做`Entry`的内部类。这个`Entry`类包含了`key-value`作为实例变量。
- 每当往`Hashmap`里面存放`key-value`对的时候，都会为它们实例化一个`Entry`对象，这个`Entry`对象就会存储在前面提到的`Entry`数组`table`中。
现在你一定很想知道，上面创建的`Entry`对象将会存放在具体哪个位置(在`table`中的精确位置)。答案就是，根据`key`的`hashcode()`方法计算出来的`hash`值来决定。
`hash`值用来计算`key`在`Entry`数组的索引。
- 我们往`hashmap`放了4个`key-value`对，但是有时候看上去好像只有2个元素！！！这是因为，如果两个元素有相同的`hashcode`，它们会被放在同一个索引上。
问题出现了，该怎么放呢？原来它是以链表`(LinkedList)`的形式来存储的。

接下来看一下`put`方法:      
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
再看一下`get`方法:　　　　
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

总结:     

- `HashMap`有一个叫做`Entry`的内部类，它用来存储`key-value`对。
- 上面的`Entry`对象是存储在一个叫做`table`的`Entry`数组中。
- `table`的索引在逻辑上叫做“桶”`(bucket)`，它存储了链表的第一个元素。
- `key`的`hashcode()`方法用来找到`Entry`对象所在的桶。
- 如果两个`key`有相同的`hash`值，他们会被放在`table`数组的同一个桶里面。
- `key`的`equals()`方法用来确保`key`的唯一性。
- `value`对象的`equals()`和`hashcode()`方法根本一点用也没有。

---
- 邮箱 ：charon.chui@gmail.com  
- Good Luck! 

	
