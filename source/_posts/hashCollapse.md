title: 哈希碰撞
date: 2014-12-14 19:46:07
categories: Java
tags: [hash, coding]
description: 当多个关键字映射到同一槽位时，便发生哈希碰撞。主要解决方案有两种，一种是基于链表的链地址法，另一种则是开放寻址法。
---

##链地址法
Java.util.Hashtable类实现了JAVA中的哈希表，内部采用Entry[]数组，每个Entry均可作为链表的头，用来解决哈希碰撞。
###**Hashtable特点**

* 线程安全，能用于多线程环境中。
* Key、Value均不能为null。
* Hashtable同样实现了Serializable接口，它支持序列化，实现了Cloneable接口，能被克隆。
* 包含了一个Entry[]数组，而Entry又是一个链表，用来处理冲突。
* 每个Key对应了Entry数组中固定的位置（记为index），称为槽位（Slot）。槽位计算公式为：
             (key.hashCode() & 0x7FFFFFFF) % Entry[].length() 
* 当Entry[]的实际元素数量（Count）超过了分配容量（Capacity）的75%时，新建一个Entry[]是原先的2倍，重新Hash（rehash）。
* rehash的核心思路是，将旧Entry[]数组的元素重新计算槽位，散列到新Entry[]中。
<br/>
***

###**Hashtable源码分析**
**Entry类**
```java
class Entry<K,V> // Entry<K,V>是槽中的元素，可做链表，解决散列冲突。
{
     int hash; // 即key.hashCode()
     K key;
     V value;
     Entry<K,V> next; // 用来实现链表结构。同一链表中的key的hash是相同的。
     protected Entry(int hash, K key, V value, Entry<K,V> next) {
          this.hash=hash;this.key=key;this.value=value;this.next=next;
     }
}
```
<br/>
**构造函数**
```java
public Hashtable(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                initialCapacity);
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal Load: "+loadFactor);
 
    if (initialCapacity==0)
        initialCapacity = 1;
    this.loadFactor = loadFactor;
    table = new Entry<?,?>[initialCapacity];
    threshold = (int)Math.min(initialCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
}
 
/**
 * Constructs a new, empty hashtable with the specified initial capacity
 * and default load factor (0.75).
 * @param     initialCapacity   the initial capacity of the hashtable.
 * @exception IllegalArgumentException if the initial capacity is less
 *              than zero.
 */
public Hashtable(int initialCapacity) {
    this(initialCapacity, 0.75f);
}
 
/**
 * Constructs a new, empty hashtable with a default initial capacity (11)
 * and load factor (0.75).
 */
public Hashtable() {
    this(11, 0.75f);
}
```
 * threshold：Hashtable的阈值，用于判断是否需要调整Hashtable的容量。threshold的值="容量*加载因子"
 * loadFactor：加载因子，通常设为**`0.75`**
<br/>

**hashcode**
考虑一种情况，当向集合中插入对象时，如何判别在集合中是否已经存在该对象了？（注意：集合中不允许重复的元素存在）。

也许大多数人都会想到调用equals方法来逐个进行比较，这个方法确实可行。但是如果集合中已经存在一万条数据或者更多的数据，如果采用equals方法去逐一比较，效率必然是一个问题。
```java
public synchronized boolean containsKey(Object key) {
    Entry<?,?> tab[] = table;
    int hash = key.hashCode();
    int index = (hash & 0x7FFFFFFF) % tab.length;
    for (Entry<?,?> e = tab[index] ; e != null ; e = e.next) {
        if ((e.hash == hash) && e.key.equals(key)) {
            return true;
        }
    }
    return false;
}
```
当集合要添加新的对象时，先调用这个对象的hashCode方法，得到对应的hashcode值(实际上在HashMap的具体实现中会用一个table保存已经存进去的对象的hashcode值)；
* if（table中没有该hashcode值），then → 直接存进去
* if（存在该hashcode值），then → 调用equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址
所以这里存在一个冲突解决的问题，这样一来实际调用equals方法的次数就大大降低了，说通俗一点：**Java中的hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为散列值。**
<br/>

**addEntry**
addEntry方法实现向Hashtable中添加k-v值对
```java
private void addEntry(int hash, K key, V value, int index) {
    modCount++;
 
    Entry<?,?> tab[] = table;
    if (count >= threshold) {
        // Rehash the table if the threshold is exceeded
        rehash();
 
        tab = table;
        hash = key.hashCode();
        index = (hash & 0x7FFFFFFF) % tab.length;
    }
 
    // Creates the new entry.
    @SuppressWarnings("unchecked")
    Entry<K,V> e = (Entry<K,V>) tab[index]; // 旧有Entry
    tab[index] = new Entry<>(hash, key, value, e); /* 旧有Entry成为了新增Entry的next */
    count++;
}
```
当count大于阀值时，要重新进行hash，重新确定Hashtable的大小
<br/>

**rehash**
rehash方法即再次hash，当Entry[]的实际存储数量占分配容量的约75%时，扩容并且重新计算各个对象的槽位
``` java
protected void rehash() {
          int oldCapacity = table.length;
          Entry[] oldMap = table;
          int newCapacity = (oldCapacity << 1) + 1; // 2倍+1
          Entry[] newMap = new Entry[newCapacity];
          threshold = (int)(newCapacity * loadFactor);
          table = newMap;
 
          for( int i=oldCapacity; i-- >0;){ //  i的取值范围为 [oldCapacity-1,0]
               for (Entry<K,V> old = oldMap[i]; old!=null;){ // 遍历旧Entry[]
                    Entry<K,V> e = old;
                    //重新计算各个元素在新Entry[]中的槽位index。
                    int index = (e.hash & 0x7FFFFFFF) % newCapacity;     
                    e.next = newMap[index]; // 已经存在槽位中的Entry成为了新增Entry的next 
                    newMap[index]=e;     // 放到槽位中
                    old = old.next;
               }
          }
 
     }
```

<br/>
***

###Hashtable源码总结

**1**. **关于hashCode()和equals()方法**
下面这段话摘自Effective Java一书：
>* 在程序执行期间，只要equals方法的比较操作用到的信息没有被修改，那么对这同一个对象调用多次，hashCode方法必须始终如一地返回同一个整数。
* 如果两个对象根据equals方法比较是相等的，那么调用两个对象的hashCode方法必须返回相同的整数结果。
* 如果两个对象根据equals方法比较是不等的，则hashCode方法不一定得返回不同的整数。

对于第二条和第三条很好理解，但是第一条，很多时候就会忽略。在《Java编程思想》一书中的P495页也有同第一条类似的一段话：
> 设计hashCode()时最重要的因素就是：无论何时，对同一个对象调用hashCode()都应该产生同样的值。如果在讲一个对象用put()添加进HashMap时产生一个hashCdoe值，而用get()取出时却产生了另一个hashCode值，那么就无法获取该对象了。

**所以如果你的hashCode方法依赖于对象中易变的数据，用户就要当心了，因为此数据发生变化时，hashCode()方法就会生成一个不同的散列码。**

<br/>
**2**. **Hashtable中key和value都不允许为null**
而HashMap中key和value都允许为null（key只能有一个为null，而value则可以有多个为null）。但是如果在Hashtable中有类似put(null,null)的操作，编译同样可以通过，因为key和value都是Object类型，但运行时会抛出NullPointerException异常，这是JDK的规范规定的。
<br/>
**3**. **Hashtable计算hash值**
直接用key的hashCode()，而HashMap重新计算了key的hash值，Hashtable在求hash值对应的位置索引时，用取模运算，而HashMap在求位置索引时，则用与运算，且这里一般先用hash&0x7FFFFFFF后，再对length取模，&0x7FFFFFFF的目的是为了将负的hash值转化为正值，因为hash值有可能为负数，而&0x7FFFFFFF后，**只有符号外改变，而后面的位都不变** 。

<br/>
***

###链地址法性能分析
给定一个能存放n个元素，具有m个槽位的散列表T，定义T的装载因子$a=n/m$，即一个链中的平均元素数。假定可以在`O(1)`的时间内计算出散列值h(k)，即计算散列函数和寻址槽h(k)为`O(1)`时间。现查找元素k：

* 一次不成功情况
在简单一致散列的假设下。一次不成功查找的`期望`时间为`0(1+a)`
一次不成功即查找遍整个散列表`T[h(k)]`，平均每个链表查找元素为$a=n/m$，加上计算哈希h(k)的时间，总时间为`O(1+a)`

* 一次成功情况
在简单一致散列的假设下。一次成功查找的期望时间为`0(1+a)`
<center> ![](http://7rf27k.com1.z0.glb.clouddn.com/QQ图片20141215121340.png) <center/>

由于链表中新元素是在表头插入的，所以一次成功的查找中，所检查的元素数比x所在链表中出现在x前面的元素数多1
在hash值为index的槽位，一次成功的查找中，所检查元素的期望为：

$E\ [1/n·\sum\_{i=1}^n(1+\sum\_{j=i+1}^n X\_{ij})]$

$=1/n·\sum\_{i=1}^n(1+\sum\_{j=i+1}^n E[X\_{ij}])=1/n·\sum\_{i=1}^n(1+\sum\_{j=i+1}^n1/m)$

$=1+1/{mn}(\sum\_{i=1}^nn-\sum\_{i=1}^ni)=1+1/{mn}(n^2-n(n+1)/2)$

$=1+(n-1_/2m)=1+a/2-a/2n$

于是，一次成功的查找所需全部时间为`O(2+a/2-a/2n)`=`O(1+a)`.

**Conclusion**：如果散列表中槽位数至少与表中的元素数成正比，则有n=O(m)，从而 $a=n/m=O(m)/m=O(1)$
所以平均查找操作需要常数量的时间，且已经知道插入操作最坏情况需要`O(1)`，删除操作最坏需要`O(1)`，因而全部字典操作可以在`O(1)`时间完成。

##开放寻址法
前面讲的方法主要基于用链表的方法解决哈希冲突，但元素数n大于槽位数m时，必然会导致至少一个槽位被多个元素hash到，因此链地址法的查询性能平均为`O(1+a)`，不能达到理想的`O(1)`。
开放寻址法中，所有的元素都存放在散列表里，即每个表项是动态集合中的一个元素，或者为NIL。当要插入一个元素时，可以连续的检查散列表的各项,直到找个一个空槽来存放待插入元素，整个数组必须支持`动态扩容`：当数组空闲节点低于一个阀值时，将扩展数组容量为原来的一倍。这个阀值通常是`0.72`。当然检查的顺序不一定是0,1,...,m-1，主要分为下面三种探查方法：

###线性探查
将散列表T[0,1,...,m-1]看成是一个循环向量，若初始探查的地址为d(即h(key)=d)，则最长的探查序列为：
<center> $d，d+l，d+2，…，m-1，0，1，…，d-1$ <center/>

散列函数为
<center> $h(k,i)=(h^\*(k)+i) \ mod \ m \ , \ i=0,1,...,m-1$ <center/>


探查过程终止于三种情况：
 1. 若当前探查的单元为空，则表示查找失败（若是插入则将key写入其中）；
 2. 若当前探查的单元中含有key，则查找成功，但对于插入意味着失败；
 3. 若探查到T[d-1]时仍未发现空单元也未找到key，则无论是查找还是插入均意味着失败(此时表满)。

<center> ![](http://7rf27k.com1.z0.glb.clouddn.com/直接寻址.jpg) <center/> 

用线性探测法处理冲突，思路清晰，算法简单，但存在下列缺点：

 - 处理溢出需另编程序。一般可另外设立一个溢出表，专门存放哈希表中放不下的记录。此溢出表最简单的结构是顺序表，查找方法可用顺序查找。
 - 按上述算法建立起来的哈希表，删除工作非常困难。假如要从哈希表中删除一个记录，按理应将这个记录所在位置置为空，但我们不能这样做，而只能标上已被删除的标记`DELETE`，否则，将会影响以后的查找。
 - 线性探测法很容易产生堆聚现象。所谓堆聚现象，就是存入哈希表的记录在表中连成一片。按照线性探测法处理冲突，如果生成哈希地址的连续序列愈长（即不同关键字值的哈希地址相邻在一起愈长），则当新的记录加入该表时，与这个序列发生冲突的可能性愈大。因此，哈希地址的较长连续序列比较短连续序列生长得快，这就意味着，一旦出现堆聚 ( 伴随着冲突 ) ，就将引起进一步的堆聚。
 
**线性补偿探测法** 
线性补偿探测法的基本思想是：将线性探测的步长从 1 改为 Q ，即将上述算法中的 

$h(k,i)=(h^"(k)+1) \ mod \ m$ 改为 $h(k,i)=(h^"(k)+Q) \ mod \ m$ 

而且要求 Q 与 m 是`互质`的，以便能探测到哈希表中的所有单元。
<br/>
***

###二次探查
二次探查采用如下形式的散列函数：

<center> $h(k,i)=(h^\*(k)+c\_1i+c\_2i^2) \ mod \ m \ , \ i=0,1,...,m-1, \ c\_2\not=0$ <center/>

其中$h^\*(k)$是一个辅助散列函数，$c_1$和 $c_2$为辅助常数。初始的探查位置为$T[h^\*(k)]$，后续的探查位置要加上一个偏移量，该偏移量以二次的方式依赖于探查序号i。这种探查方法的效果要比线性探查好很多，但是，为了能够充分利用散列表，$c_1$，$c_2$和 $m$的值要受到限制。此外，如果两个关键字的初始探查位置相同，那么它们的探查序列也是相同的。这一性质可导致一种轻度的群集，称为二次群集。
<br/>
***

###双重散列
双重散列是用于开放寻址最好的方法之一，它采用如下形式的散列函数：

<center> $h(k,i)=(h\_1(k)+ih\_2(k)) \ mod \ m$ <center/>

为了能查找整个散列表，值$h\_2(k)$要与表的大小m`互质`，确保这一条件成立的方法有两种

 1. 取m为2的幂，并设计一个总产生奇数的$h_2$
 2. 取m为质数，并设计一个总是产生较m小的正整数的$h_2$

例如下图所示，散列表大小为 $m=13$，$h\_1(k)=k \ mod \ 13$, $h\_2(k)=1+(k \ mod \ 11)$，因为$14\equiv1\ (mod\ 13)$，$14\equiv3\ (mod\ 11)$，故在探查了槽1和槽5并发现被占用后，关键字14被插入到空槽9中。

<center> ![](http://7rf27k.com1.z0.glb.clouddn.com/QQ图片20141215204708.png) <center/>

双重散列法中用了$O(m^2)$种探查序列，而线性探查或二次探查中用了$O(m)$种，所以是一种改进。
<br/>
***

###开放寻址散列性能分析
在开放寻址法中，每个槽位至多只有一个元素，因而 $n\leq m$，这意味着 $n \leq 1$
下面分析在`一致散列`假设下，用开放寻址法进行散列时预期的探查数

 - 给定一个装载因子为$a=n/m<1$的开放寻址散列表，在`一次不成功`的查找中，期望的探查数至多为：$\frac{1}{(1-a)}$
 - 给定一个装载因子为$a<1$的开放寻址散列表，`一次成功`查找中的期望探查数至多为：$\frac{1}{a}·ln\frac{1}{1-a}$
 
##Java代码实现
###Hashtable链地址法
```java
package Practice;

import java.util.Objects;

public class Hashtable<K, V> {
	private Entry<K, V>[] table;
	private int count;
	private int threhold;
	private float loadFactor;
	private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

	private static class Entry<K, V> {
		int hash;
		K key;
		V value;
		Entry next;

		public Entry() {
		}

		public Entry(int hash, K key, V value, Entry next) {
			this.hash = hash;
			this.key = key;
			this.value = value;
			this.next = next;
		}

		public K getKey() {
			return key;
		}

		public V getValue() {
			return value;
		}

		public V setValue(V value) {
			if (value == null) {
				throw new NullPointerException();
			}
			V old = this.value;
			this.value = value;
			return old;
		}

		public boolean equals(Object o) {
			if (!(o instanceof Entry)) {
				return false;
			}
			Entry<K, V> e = (Entry<K, V>) o;
			return (key == null ? e.getKey() == null : key.equals(e.getKey()))
					&& (value == null ? e.getValue() == null : value.equals(e
							.getValue()));
		}

		public int hashcode() {
			return hash ^ Objects.hashCode(value);
		}

		public String toString() {
			return key.toString() + "=" + value.toString();
		}
	}

	public Hashtable(int initCapacity, float loadFactor) {
		if (initCapacity < 0)
			throw new IllegalArgumentException("Illegal Capacity"
					+ initCapacity);
		if (loadFactor <= 0 || Float.isNaN(loadFactor))
			throw new IllegalArgumentException("Illegal Load" + loadFactor);

		if (initCapacity == 0)
			initCapacity = 1;
		this.loadFactor = loadFactor;
		table = new Entry[initCapacity];
		threhold = (int) Math
				.min(initCapacity * loadFactor, MAX_ARRAY_SIZE + 1);
	}

	public Hashtable(int initCapacity) {
		this(initCapacity, 0.75f);
	}

	public Hashtable() {
		this(11, 0.75f);
	}

	public synchronized int size() {
		return count;
	}

	public synchronized V put(K key, V value) {
		if (value == null) {
			throw new NullPointerException();
		}

		Entry<?, ?> tab[] = table;
		int hash = key.hashCode();
		int index = (hash & 0x7FFFFFFF) % tab.length;
		for (Entry<K, V> entry = (Entry<K, V>) tab[index]; entry != null; entry = entry.next) {
			if ((entry.hash == hash) && entry.key.equals(key)) {
				V old = entry.value;
				entry.value = value;
				return old;
			}
		}
		if (count > threhold) {
			rehash();
			tab = table;
			hash = key.hashCode();
			index = (hash & 0x7FFFFFFF) % tab.length;
		}
		
		@SuppressWarnings("unchecked")
		Entry<K, V> entry = (Entry<K, V>) tab[index];
		tab[index] = new Entry<K, V>(hash, key, value, entry);
		count ++;
		return null;
	}
	
	public synchronized V get(Object key){
		Entry<?, ?> tab[] = table;
		int hash = key.hashCode();
		int index = (hash & 0x7FFFFFFF) % tab.length;
		for (Entry<K, V> e = (Entry<K, V>) tab[index]; e != null ; e = e.next) {
			if (e.hash == hash && e.key.equals(key)) {
				return e.value;
			}
		}
		return null;
	}
	
	protected void rehash(){
		int oldCapacity = table.length;
		Entry<?, ?>[] oldEntry = table;
		
		int newCapacity = (oldCapacity << 1) + 1;
		
		Entry[]	newMap = new Entry[newCapacity];
		for (int i = oldCapacity; i > 0; i--) {
			for (Entry<K, V> e = (Entry<K, V>) oldEntry[i];  e != null; e = e.next) {
				int index = (e.hash & 0x7FFFFFFF) % newCapacity;
				e.next = newMap[index];
				newMap[index] = e;
			}
		}
	}

	public static void main(String arg[]) {
        Hashtable<String, String> hashtable = new Hashtable<String, String>();
        for (int i = 0; i < 9; i++) {
            hashtable.put("lyx" + i, "lyx" + i);
        }
 
        hashtable.put("lyx0", "aaaaaaaaaaaaaaaaaaaaa");
        hashtable.put("lyx0", "aaaaaaaaaaaaaaaaaaaaa");
        hashtable.put("lyx0", "aaaaaaaaaaaaaaaaaaaaa");
        hashtable.put("lyx0", "aabbccdd");
 
        System.out.println(hashtable.get("lyx0"));
        System.out.println(hashtable.get("lyx1"));
        System.out.println(hashtable.get("lyx2"));
        System.out.println(hashtable.get("lyx3"));
    }
}
```
 
###线性探查
```java
package Practice;

import java.nio.BufferOverflowException;

public class LinearProbingHash<T> {
	private T[] table;
	private int tableCapacity;
	private int size;
	
	public LinearProbingHash(int tableSize){
		table = (T[]) new Object[tableSize];
		this.tableCapacity = tableSize;
		size = 0;
	}
	
	public void add(T item){
		int index, originIndex;
		index = (item.hashCode() & 0x7FFFFFFF) % tableCapacity;
		originIndex = index;
		do {
			if (table[index] == null || table[index].equals("DELETE")) {
				table[index] = item;
				size++;
				return;
			}else {
				index = (index + 1) % tableCapacity;
			}
		} while (index != originIndex);
		throw new BufferOverflowException();
	}
	
	@SuppressWarnings("unchecked")
	public int delete(T item){
		int index, originIndex;
		index = (item.hashCode() & 0x7FFFFFFF) % tableCapacity;
		originIndex = index;
		do {
			if (table[index].equals(item)) {
				table[index] = (T) "DELETE";
				size--;
				return index;
			}else {
				index = (index & 0x7FFFFFFF) % tableCapacity;
			}
		} while (index != originIndex);
		return -1;
	}
	
	public boolean contains(T item){
		return (find(item) < 0) ? false : true;
	}
	
	public int size(){
		return size;
	}
	
	public int find(T item){
		int index, originIndex;
		index = (item.hashCode() & 0x7FFFFFFF) % tableCapacity;
		originIndex = index;
		do {
			if (table[index].equals(item)) {
				return index;
			}else {
				index = (index + 1) % tableCapacity;
			}
		} while (index != originIndex);
		return -1;
	}
	
	public String toString() {  
        int max = tableCapacity - 1;  
        StringBuffer buf = new StringBuffer();  
        buf.append("[");  
        for(int i = 0; i < tableCapacity; i++) {  
            if(table[i] != null) {  
                buf.append(table[i]);  
                if(i < max)  
                    buf.append(", ");  
            }  
        }  
        buf.append("]");  
        return buf.toString();  
    }  
	
	public static void main(String[] args) {  
        LinearProbingHash<Integer> lp = new LinearProbingHash<Integer>(10);  
        lp.add(new Integer(54));  
        lp.add(new Integer(77));  
        lp.add(new Integer(94));  
        lp.add(new Integer(89));  
        lp.add(new Integer(14));  
        lp.add(new Integer(45));  
        lp.add(new Integer(35));  
        lp.add(new Integer(76));  
          
        System.out.println(lp); // [35, 76, 54, 94, 14, 77, 45, 89]  
        System.out.println(lp.contains(new Integer(45))); // true  
        System.out.println(lp.size()); // 8  
        
        lp.delete(new Integer(77));
        System.out.println(lp); // [35, 76, 54, 94, 14, DELETE, 45, 89]
        System.out.println(lp.size()); // 7
        lp.add(new Integer(100));
        System.out.println(lp); // [35, 76, 100, 54, 94, 14, DELETE, 45, 89]
        System.out.println(lp.size()); // 8
    }  
}
```