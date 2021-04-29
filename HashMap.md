[1.7和1.8区别](#JDK1.7和1.8区别)

[ConcurrentHashMap](#ConcurrentHashMap)

[处理hash冲突](#处理Hash冲突)

[HashMap哈希函数设计](#HashMap哈希函数设计)

[在解决hash冲突的时候，为什么HashMap不直接用红黑树？而是先用链表再用红黑树？](#在解决hash冲突的时候，为什么HashMap不直接用红黑树？而是先用链表再用红黑树？)

[扩容](#扩容)

[关于头插尾插](#关于头插尾插)

## Base1.7

<img src="E:\笔记\Java面试笔记\images\111.jpg" alt="img" style="zoom:50%;" />

https://blog.csdn.net/weixin_44460333/article/details/86770169

这是 HashMap 中比较核心的几个成员变量；看看分别是什么意思？

1. 初始化桶大小，因为底层是数组，所以这是数组默认的大小。
2. 桶最大值。----MAXIMUM CAPACITY
3. 默认的负载因子（0.75）----DEFALUT LOAD FACTOR
4. `table` 真正存放数据的数组。---Entry<K,V>
5. `Map` 存放数量的大小。---size
6. 桶大小，可在初始化时显式指定。---threshold
7. 负载因子，可在初始化时显式指定。----loadFactor

重点解释下负载因子：

加载因子 是**衡量哈希表密集程度**的一个参数，如果加载因子越大，说明哈希表被装在的越多，出现hash冲突的可能性越大，繁殖，被装载的越少，出现hash冲突的可能性越小，如果过小，内存使用率不高，该值取值应该考虑到**内存使用率和hash冲突概率的平衡。**

由于给定的 HashMap 的容量大小是固定的，比如默认初始化：

```java
 1    public HashMap() {
 2        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
 3    }
 4
 5    public HashMap(int initialCapacity, float loadFactor) {
 6        if (initialCapacity < 0)
 7            throw new IllegalArgumentException("Illegal initial capacity: " +
 8                                               initialCapacity);
 9        if (initialCapacity > MAXIMUM_CAPACITY)
10            initialCapacity = MAXIMUM_CAPACITY;
11        if (loadFactor <= 0 || Float.isNaN(loadFactor))
12            throw new IllegalArgumentException("Illegal load factor: " +
13                                               loadFactor);
14
15        this.loadFactor = loadFactor;
16        threshold = initialCapacity;
17        init();
18    }
```

给定的默认容量为 16，负载因子为 0.75。Map 在使用过程中不断的往里面存放数据，当数量达到了 `16 * 0.75 = 12` 就需要将当前 16 的容量进行扩容，而扩容这个过程涉及到 rehash、复制数据等操作，所以非常消耗性能。

因此通常建议能提前预估 HashMap 的大小最好，尽量的减少扩容带来的性能损耗

根据代码可以看到其实真正存放数据的是

```java
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;
```

Entry 是 HashMap 中的一个内部类，从他的成员变量很容易看出：

![img](E:\笔记\Java面试笔记\images\2.png)

- key 就是写入时的键。

- value 自然就是值。

- 开始的时候就提到 HashMap 是由数组和链表组成，所以这个 next 就是用于实现链表结构。

- hash 存放的是当前 key 的 hashcode。

- put方法

  - 判断当前数组是否需要初始化。

  - 如果 key 为空，则 put 一个空值进去。

  - 根据 key 计算出 hashcode。

  - 根据计算出的 hashcode 定位出所在桶。

  - 如果桶是一个链表则需要遍历判断里面的 hashcode、key 是否和传入 key 相等，如果相等则进行覆盖，并返回原来的值。

  - 如果桶是空的，说明当前位置没有数据存入；新增一个 Entry 对象写入当前位置。

  - 当调用 addEntry 写入 Entry 时需要判断是否需要扩容。

    如果需要就进行两倍扩充，并将当前的 key 重新 hash 并定位。

    而在 `createEntry` 中会将当前位置的桶传入到新建的桶中，如果当前桶有值就会在位置形成链表。

## Base1.8

1.7可优化的点：

> 当 Hash 冲突严重时，在桶上形成的链表会变的越来越长，这样在查询时的效率就会越来越低；时间复杂度为 `O(N)`。

![640?wx_fmt=png](E:\笔记\Java面试笔记\images\3.png)

和 1.7 大体上都差不多，还是有几个重要的区别：

- `TREEIFY_THRESHOLD` 用于判断是否需要将链表转换为红黑树的阈值。

- HashEntry 修改为 Node。

- put方法：

  ![640?wx_fmt=png](E:\笔记\Java面试笔记\images\444.png)

  1. 判断当前桶是否为空，空的就需要初始化（resize 中会判断是否进行初始化）。
  2. 根据当前 key 的 hashcode 定位到具体的桶中并判断是否为空，为空表明没有 Hash 冲突就直接在当前位置创建一个新桶即可。
  3. 如果当前桶有值（ Hash 冲突），那么就要比较当前桶中的 `key、key 的 hashcode` 与写入的 key 是否相等，相等就赋值给 `e`,在第 8 步的时候会统一进行赋值及返回。
  4. 如果当前桶为红黑树，那就要按照红黑树的方式写入数据。
  5. 如果是个链表，就需要将当前的 key、value 封装成一个新节点写入到当前桶的后面（形成链表）。
  6. 接着判断当前链表的大小是否大于预设的阈值，大于时就要转换为红黑树。
  7. 如果在遍历过程中找到 key 相同时直接退出遍历。
  8. 如果 `e != null` 就相当于存在相同的 key,那就需要将值覆盖。
  9. 最后判断是否需要进行扩容。

**从两个核心方法（get/put）可以看出 1.8 中对大链表做了优化，修改为红黑树之后查询效率直接提高到了 `O(logn)`。**

但是 HashMap 原有的问题也都存在，比如在并发场景下使用时容易出现死循环。

原因：

> HashMap 扩容的时候会调用 `resize()` 方法，就是这里的并发操作容易在一个桶上形成环形链表；这样当获取一个不存在的 key 时，计算出的 index 正好是环形链表的下标就会出现死循环。

> 简单总结下 HashMap：无论是 1.7 还是 1.8 其实都能看出 JDK 没有对它做任何的同步操作，所以并发会出问题，甚至出现死循环导致系统不可用。

因此 JDK 推出了专项专用的 [ConcurrentHashMap](#ConcurrentHashMap) ，该类位于 `java.util.concurrent` 包下，专门用于解决并发问题。

## 处理Hash冲突

`链地址法`

`开放地址法`

> 从发生冲突的单元起，按照一定的顺序从哈希表中找出一个空白单元，然后把冲突元素存入该单元的方法；所需长度>=元素个数；**开放地址中解决冲突的方法：线性探测法（ThreadLocal），平方探测法，双散列函数探测法**

`再哈希`：同时构造多个不同的哈希函数，第一个哈希函数冲突，使用第二个，以此类推。

`建立公共溢出区`：将哈希表分为公共表和溢出表，发生溢出时，将溢出数据存入溢出区。

## 在解决hash冲突的时候，为什么HashMap不直接用红黑树？而是先用链表再用红黑树？

HashMap 解决 hash 冲突的时候，先用链表，再转红黑树，是为了时间和空间的平衡。
TreeNodes 占用的空间大小大约是普通 Nodes 的两倍，只有在容器中包含足够的节点保证使用才用它，在节点数比较小的时候，对于红黑树来说，内存上的劣势会超过查找等操作的优势，使用链表更加好。
节点数比较多的时候，综合考虑时间和空间，红黑树比链表要好。

HashMap 与红黑树

红黑树需要进行左旋右旋变色等操作来保持平衡，而单链表则不需要。当元素小于8个的时候，做查询操作，链表结构已经能保证查询性能。

当元素大于 8 个的时候，此时需要红黑树来加快查询速度，但是新增节点的效率变慢了。

对 HashMap 的相应位置进行查询的时候，就回去循环遍历这个超级大的链表，性能不好。 java8使用红黑树来代替 8 个节点数的链表后，查询方式性能得到了很好的提升，从原来的 O(n) 到了 O(log n)

## 扩容

扩容会发生在两种情况下（满足任意一种条件就可以发生扩容）：

> 当前存入的数据个数大于扩容阈值（例如 16* 0.75）即发生扩容
> 存入数据到某一条链表上，此时长度大于 8，且数组长度小于64 即发生扩容

扩容：每次扩容的容量都是之前容量的2倍。 HashMap 的容量是有上限的，必须小于 1 << 30 即 2^30。如果容量超出了这个数，则不再增长，且阈值会被设置为 Integer.MAX_VALUE。 HashMap 1.8版本的扩容相对于1.7版本，性能方面做了优化（见 14 问），1.8版本的HashMap扩容后，节点在新数组的位置只有两种，原下标位置或者原下标+旧数组的长度

相关源码：put()->putVal() treeifyBin() 其中有对是否要resize()做判断

`HashMap 的初始容量是 2 的 n 次幂，扩容也是 2 倍的形式进行扩容，可以使得添加的元素均匀分布在 HashMap 中的数组上，减少 hash 碰撞，充分利用内存空间（所有下标都能用上）。`

## JDK1.7和1.8区别

**数据结构**的差异 1.7 数组+链表 ；1.8 数组+链表或红黑树
**链表的插入方式**优化 1.7头插法； 1.8尾插法
**扩容的改进** 1.7全部rehash ；1.8简单判断（判断新增位是0还是1），要么原位置，要么【原位置+旧容量】的位置
**插入数据的差别** 1.7先判断是否要扩容再插入； 1.8先插入，插入完成再判断是否需要扩容

**底层数据结构不一样：**

- jdk7： 数组+单链表     

- jdk8 ：数组+(单链表+红黑树)  [链表长度大于8转为红黑树]

  > 这里需要区分一下，JDK1.7和 JDK1.8之后的 HashMap 存储结构。在JDK1.7及之前，是用数组加链表的方式存储的。
  >
  > 但是，众所周知，当链表的长度特别长的时候，查询效率将直线下降，查询的时间复杂度为 O(n)。因此，JDK1.8 把它设计为达到一个特定的阈值之后，就将链表转化为红黑树。由于红黑树，是一个自平衡的二叉搜索树，因此可以使查询的时间复杂度降为O(logn)。

jdk7 ：新增节点链表头插     jdk8 ：链表尾插    

​	头插: resize后transfer数据时不需要遍历链表到尾部再插入    

​	头插: 最近put的可能等下就被get，头插遍历到链表头就匹配到了    

​	头插: resize后链表可能倒序; 并发resize可能产生循环链

**resize()扩容：**

- jdk7： 先扩容再put插入数据       

- jdk8 ：先put再扩容  (why?有什么区别吗?)

jdk7 计算hash运算多 jdk8 计算hash运算少

jdk7： 受rehash影响，有可能改变链表的顺序(头插法导致)    

jdk8 ：调整后是(原位置)or(原位置+旧容量) 保证原链表的顺序

JDK8增加了Spliterator进行遍历，而JDK1.7没有

https://blog.csdn.net/weixin_43093006/article/details/111469521?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242

## 关于头插尾插

![在这里插入图片描述](E:\笔记\Java面试笔记\images\6.jpg)

**HashMap1.7版本用的是头插法**，多线程场景下，1.7版本的头插法存在死循环的风险，**1.8版本用的是尾插法**，改进了存在死循环的缺陷。HashMap1.7在并发情况下，发生扩容时，可能会产生循环链表，在执行get的时候，会触发死循环，引起CPU的100%占用问题，所以一定要避免在并发环境下使用HashMap。

HashMap1.8版本在同样的前提下并不会引起死循环，原因是扩容转移后前后链表顺序不变，保持之前节点的应用关系。并发环境下，优先考虑使用ConcurrentHashMap

## ConcurrentHashMap

并发容器，1.7 1.8都有，实现略有不同

**Base 1.7**

和 HashMap 非常类似，唯一的区别就是其中的核心数据如 value ，以及链表都是 volatile 修饰的，保证了获取时的可见性。

原理上来说：ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get  操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment  数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

**Base 1.8**

<img src="E:\笔记\Java面试笔记\images\5.png" alt="640?wx_fmt=png" style="zoom:50%;" />

看起来是不是和 1.8 HashMap 结构类似？

其中抛弃了原有的 Segment 分段锁，而采用了 `CAS + synchronized` 来保证并发安全性

其中的 `val next` 都用了 volatile 修饰，保证了可见性。

## HashMap哈希函数设计

Hash函数设计： hash 函数是先拿到 key 的 hashcode， 它是一个 32 为的 int 值，然后让 hashcode 的高16位于低16位进行异或操作。

hash函数作用： HashMap 采用 hash 算法来决定每个元素的存储位置（真正找到数组中下标则是通过寻址算法 (n-1) & hash 这也是取模运算的变体）

```java
  //Java8的散列值优化函数,也叫扰动函数
  static final int hash(Object key) {
        int h;
        //右位移16位，正好是32bit的一半，自己的高半区和低半区做异或，
        //就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }   
```

32位哈希码的空间范围大，[-2^31 ~ 2^31-1]，前后加起来有42亿多的映射空间，哈希码右移16位（32bit的一半），高半区和低半区做异或，混合了原始哈希码的高位和低位，加大了低位的随机性，减少哈希碰撞。
混合后的低位掺杂了高位的部分特征，高位的信息也被变相保留了下来。

## HashMap、LinkedHashMap、TreeMap有什么区别？

共同点： 线程不安全

不同点：数据无序、数据有序、数据有序还可以对数据进行排序

HashMap 中 key 的值没有顺序，相对于 LinkedHashMap 和 TreeMap，使用更广泛。
LinkedHashMap 内部有一个双向链表（head、tail、双向），保持Key插入的顺序。访问顺序和插入顺序是一致的。
TreeMap 的顺序是 key 的自然顺序（如整数从小到大），也可以指定比较函数。访问顺序和插入顺序不一定是一致的。

