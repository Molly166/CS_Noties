# HashMap的扩容机制

HashMap的底层用的也是数组，向HashMap里不停地添加元素，当数组无法装载更多元素时，就需要对数组进行扩容，以便装入更多的元素；除此以外，容量的提升也会相应地提高查询效率，因为“桶”更多了，原来需要通过链表存储的（查询的时候需要遍历），扩容后可能就有自己专属的位置了。

当然，数组无法自动扩容，所以如果要扩容的话，就需要新建一个大的数组，然后把之前的小的数组复制过去，并且要重新计算哈希值和重新分配桶（重新散列），这个过程也是挺耗时的。

## resize方法

HashMap的扩容是通过resize方法来实现的，JDK8中融入了红黑树（链表长度超过8的时候，会将链表转化为红黑树来提高查询效率）。

java7的resize方法源码：

```java
// newCapacity为新的容量
void resize(int newCapacity) {
    // 小数组，临时过度下
    Entry[] oldTable = table;
    // 扩容前的容量
    int oldCapacity = oldTable.length;
    // MAXIMUM_CAPACITY 为最大容量，2 的 30 次方 = 1<<30
    if (oldCapacity == MAXIMUM_CAPACITY) {
        // 容量调整为 Integer 的最大值 0x7fffffff（十六进制）=2 的 31 次方-1
        threshold = Integer.MAX_VALUE;
        return;
    }

    // 初始化一个新的数组（大容量）
    Entry[] newTable = new Entry[newCapacity];
    // 把小数组的元素转移到大数组中
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    // 引用新的大数组
    table = newTable;
    // 重新计算阈值
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

该方法接收一个新的容量 newCapacity，然后将 HashMap 的容量扩大到 newCapacity。

首先，方法获取当前 HashMap 的旧数组 oldTable 和旧容量 oldCapacity。如果旧容量已经达到 HashMap 支持的最大容量 MAXIMUM_CAPACITY（ 2 的 30 次方），就将新的阈值 threshold 调整为 Integer.MAX_VALUE（2 的 31 次方 - 1），这是因为 HashMap 的容量不能超过 MAXIMUM_CAPACITY。

因为 2,147,483,647（Integer.MAX_VALUE） - 1,073,741,824（MAXIMUM_CAPACITY） = 1,073,741,823，刚好相差一倍（HashMap 每次扩容都是之前的一倍）。

接着，方法创建一个新的数组 newTable，并将旧数组 oldTable 中的元素转移到新数组 newTable 中。转移过程是通过调用 transfer 方法来实现的。该方法遍历旧数组中的每个桶，并将每个桶中的键值对重新计算哈希值后，将其插入到新数组对应的桶中。

转移完成后，方法将 HashMap 内部的数组引用 table 指向新数组 newTable，并重新计算阈值 threshold。新的阈值是新容量 newCapacity 乘以负载因子 loadFactor 的结果，但如果计算结果超过了 HashMap 支持的最大容量 MAXIMUM_CAPACITY，则将阈值设置为 MAXIMUM_CAPACITY + 1，这是因为 HashMap 的元素数量不能超过 MAXIMUM_CAPACITY。

### 新容量newCapacity

```java
int newCapacity = oldCapacity * 2;
if (newCapacity < 0 || newCapacity >= MAXIMUM_CAPACITY) {
    newCapacity = MAXIMUM_CAPACITY;
} else if (newCapacity < DEFAULT_INITIAL_CAPACITY) {
    newCapacity = DEFAULT_INITIAL_CAPACITY;
}
```

新容量 newCapacity 被初始化为原容量 oldCapacity 的两倍。然后，如果 newCapacity 超过了 HashMap 的容量限制 MAXIMUM_CAPACITY（2^30），就将 newCapacity 设置为 MAXIMUM_CAPACITY。如果 newCapacity 小于默认初始容量 DEFAULT_INITIAL_CAPACITY（16），就将 newCapacity 设置为 DEFAULT_INITIAL_CAPACITY。这样可以避免新容量太小或太大导致哈希冲突过多或者浪费空间。

Java 8 的时候，newCapacity 的计算方式发生了一些细微的变化。

```java
int newCapacity = oldCapacity << 1;
if (newCapacity >= DEFAULT_INITIAL_CAPACITY && oldCapacity >= DEFAULT_INITIAL_CAPACITY) {
    if (newCapacity > MAXIMUM_CAPACITY)
        newCapacity = MAXIMUM_CAPACITY;
} else {
    if (newCapacity < DEFAULT_INITIAL_CAPACITY)
        newCapacity = DEFAULT_INITIAL_CAPACITY;
}
```

二进制数左移会变成原来的2倍、4倍8倍……

## transfer方法

该方法用来转移，将旧的小数组元素拷贝到新的大数组中。

```java
void transfer(Entry[] newTable, boolean rehash) {
    // 新的容量
    int newCapacity = newTable.length;
    // 遍历小数组
    for (Entry<K,V> e : table) {
        while(null != e) {
            // 拉链法，相同 key 上的不同值
            Entry<K,V> next = e.next;
            // 是否需要重新计算 hash
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            // 根据大数组的容量，和键的 hash 计算元素在数组中的下标
            int i = indexFor(e.hash, newCapacity);

            // 同一位置上的新元素被放在链表的头部
            e.next = newTable[i];

            // 放在新的数组上
            newTable[i] = e;

            // 链表上的下一个元素
            e = next;
        }
    }
}
```

该方法接受一个新的 Entry 数组 newTable 和一个布尔值 rehash 作为参数，其中 newTable 表示新的哈希表，rehash 表示是否需要重新计算键的哈希值。

在方法中，首先获取新哈希表（数组）的长度 newCapacity，然后遍历旧哈希表中的每个 Entry。对于每个 Entry，使用拉链法将相同 key 值的不同 value 值存储在同一个链表中。如果 rehash 为 true，则需要重新计算键的哈希值，并将新的哈希值存储在 Entry 的 hash 属性中。

接着，根据新哈希表的长度和键的哈希值，计算 Entry 在新数组中的位置 i，然后将该 Entry 添加到新数组的 i 位置上。由于新元素需要被放在链表的头部，因此将新元素的下一个元素设置为当前数组位置上的元素。

最后，遍历完旧哈希表中的所有元素后，转移工作完成，新的哈希表 newTable 已经包含了旧哈希表中的所有元素。

> 注意，`e.next=newTable[i]`，也就是使用了单链表的头插入方式，同一位置上新元素总会被放在链表的头部位置；这样先放在一个索引上的元素最终会被放到链表的尾部，这就会导致**在旧数组中同一个链表上的元素，通过重新计算索引位置后，有可能被放到了新数组的不同位置上**

### java8扩容

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table; // 获取原来的数组 table
    int oldCap = (oldTab == null) ? 0 : oldTab.length; // 获取数组长度 oldCap
    int oldThr = threshold; // 获取阈值 oldThr
    int newCap, newThr = 0;
    if (oldCap > 0) { // 如果原来的数组 table 不为空
        if (oldCap >= MAXIMUM_CAPACITY) { // 超过最大值就不再扩充了，就只好随你碰撞去吧
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && // 没超过最大值，就扩充为原来的2倍
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的 resize 上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr; // 将新阈值赋值给成员变量 threshold
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]; // 创建新数组 newTab
    table = newTab; // 将新数组 newTab 赋值给成员变量 table
    if (oldTab != null) { // 如果旧数组 oldTab 不为空
        for (int j = 0; j < oldCap; ++j) { // 遍历旧数组的每个元素
            Node<K,V> e;
            if ((e = oldTab[j]) != null) { // 如果该元素不为空
                oldTab[j] = null; // 将旧数组中该位置的元素置为 null，以便垃圾回收
                if (e.next == null) // 如果该元素没有冲突
                    newTab[e.hash & (newCap - 1)] = e; // 直接将该元素放入新数组
                else if (e instanceof TreeNode) // 如果该元素是树节点
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); // 将该树节点分裂成两个链表
                else { // 如果该元素是链表
                    Node<K,V> loHead = null, loTail = null; // 低位链表的头结点和尾结点
                    Node<K,V> hiHead = null, hiTail = null; // 高位链表的头结点和尾结点
                    Node<K,V> next;
                    do { // 遍历该链表
                        next = e.next;
                        if ((e.hash & oldCap) == 0) { // 如果该元素在低位链表中
                            if (loTail == null) // 如果低位链表还没有结点
                                loHead = e; // 将该元素作为低位链表的头结点
                            else
                                loTail.next = e; // 如果低位链表已经有结点，将该元素加入低位链表的尾部
                            loTail = e; // 更新低位链表的尾结点
                        }
                        else { // 如果该元素在高位链表中
                            if (hiTail == null) // 如果高位链表还没有结点
                                hiHead = e; // 将该元素作为高位链表的头结点
                            else
                                hiTail.next = e; // 如果高位链表已经有结点，将该元素加入高位链表的尾部
                            hiTail = e; // 更新高位链表的尾结点
                        }
                    } while ((e = next) != null); //
                    if (loTail != null) { // 如果低位链表不为空
                        loTail.next = null; // 将低位链表的尾结点指向 null，以便垃圾回收
                        newTab[j] = loHead; // 将低位链表作为新数组对应位置的元素
                    }
                    if (hiTail != null) { // 如果高位链表不为空
                        hiTail.next = null; // 将高位链表的尾结点指向 null，以便垃圾回收
                        newTab[j + oldCap] = hiHead; // 将高位链表作为新数组对应位置的元素
                    }
                }
            }
        }
    }
    return newTab; // 返回新数组
}
```

1、获取原来的数组 table、数组长度 oldCap 和阈值 oldThr。

2、如果原来的数组 table 不为空，则根据扩容规则计算新数组长度 newCap 和新阈值 newThr，然后将原数组中的元素复制到新数组中。

3、如果原来的数组 table 为空但阈值 oldThr 不为零，则说明是通过带参数构造方法创建的 HashMap，此时将阈值作为新数组长度 newCap。

4、如果原来的数组 table 和阈值 oldThr 都为零，则说明是通过无参数构造方法创建的 HashMap，此时将默认初始容量 DEFAULT_INITIAL_CAPACITY（16）和默认负载因子 DEFAULT_LOAD_FACTOR（0.75）计算出新数组长度 newCap 和新阈值 newThr。

5、计算新阈值 threshold，并将其赋值给成员变量 threshold。

6、创建新数组 newTab，并将其赋值给成员变量 table。

7、如果旧数组 oldTab 不为空，则遍历旧数组的每个元素，将其复制到新数组中。

8、返回新数组 newTab。

#### 小结

当我们往HashMap中不断添加元素时，HashMap会自动进行扩容操作（条件是元素数量达到负载因子（loadfactor）乘以数组长度时），以保证其存储的元素数量不会超出其容量限制。

在进行扩容操作时，HashMap会先将数组的长度扩大一倍，然后将原来的元素重新散列到新的数组中。

由于元素的位置是通过key和hash和数组长度进行与运算得到的，因此在数组长度扩大后，元素的位置也会发生一些改变。一部分索引不变，另一部分索引为“原索引+旧容量”。

