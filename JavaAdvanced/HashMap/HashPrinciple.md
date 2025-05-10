# hash方法的原理

## hash方法的源码(JDK8中的HashMap)

```java
static final int hash(Object key){
    int h;
    return (key==null)?0:(h=key.hashCode())^(h>>>16);
} 
```

这段代码主要是**将key的hashCode值进行处理，得到最终的哈希值**。

> - **参数key**：需要计算哈希码的键值
> - **`(key==null)?0:(h=key.hashCode())^(h>>>16)`**：三目运算，如果键值为null，则哈希码为0（如果键为null，则存放在第一个位置）；否则，通过调用hashCode()方法获取的哈希码，并将其与右移16位的哈希码进行异或运算。
> - **`^`运算符**：异或运算
> - **`h>>>16`**:将哈希码向右移动16位，相当于将原来的哈希码分成了两个16位的部分。
> - 最终返回的是经过异或运算后得到的哈希码值。

理论上，哈希值（哈希码）是一个int类型，范围从-2147483648到2147483648。
前后加起来大概40亿的映射空间，只要哈希值映射得比较均匀松散，一般不会出现哈希碰撞（哈希冲突会降低HashMap的效率）。
HashMap扩容之前的数组初始大小只有16，所以这个哈希值是不能直接拿来用的，用之前要和数组的长度做与运算((n-1)&hash/取模预算/取余运算)，得到的值来访问数组下标。

### 取模运算VS取余运算

取模运算和取余运算从严格意义上来讲，是两种不同的运算方式，它们在计算机中的实现也不相同。

在java中，通常使用%运算符来表示取余，用Math.floorMod()来表示取模。

> - 当操作数都是正数的话，取模运算和取余运算的结果是一样的。
> - 只有当操作数出现负数的情况，结果才会有所不同。
> - **取模运算的商向负无穷靠近；取余运算的商向0靠近**。这是导致它们两个在处理有负数的情况下，结果不同的根本原因。
> - 当数组的长度是2的n次方，或者n次幂，或者n的整数倍时，取模运算/取余运算可以用位运算来代替，效率更高。

## put方法的源码

```java
public V put(K key,V value){
    return putVal(hash(key),key,value,false,true);
}
```

### HashMap的取模运算有两处

1.往HashMap中put的时候(会调用私有的putVal方法)：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    // 数组
    HashMap.Node<K,V>[] tab;
    // 元素
    HashMap.Node<K,V> p;

    // n 为数组的长度 i 为下标
    int n, i;
    // 数组为空的时候
    if ((tab = table) == null || (n = tab.length) == 0)
        // 第一次扩容后的数组长度
        n = (tab = resize()).length;
    // 计算节点的插入位置，如果该位置为空，则新建一个节点插入
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
}
```

其中(n-1)%hash为取模运算

2.从HashMap中get的时候(会调用getNode方法):

```java
final Node<K,V> getNode(int hash, Object key) {
    // 获取当前的数组和长度，以及当前节点链表的第一个节点（根据索引直接从数组中找）
    Node<K,V>[] tab;
    Node<K,V> first, e;
    int n;
    K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
        // 如果第一个节点就是要查找的节点，则直接返回
        if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 如果第一个节点不是要查找的节点，则遍历节点链表查找
        if ((e = first.next) != null) {
            do {
                if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    // 如果节点链表中没有找到对应的节点，则返回 null
    return null;
}
```

取模运算(n-1)%hash

**&运算比%运算更加高效**，并且当b为2的n次方时，存在下面的公式：

> a%b=a&(b-1)
> a%(2^n) = a&((2^n)-1)

hash方法的作用

HashMap的底层是通过数组的形式实现的，初始大小为16。HashMap在添加第一个元素的时候，需要通过键的哈希码在大小为16的数组中确定一个位置（索引）。

**hash方法是用来做哈希值优化的**，把哈希值右移16位，也就正好是自己长度的一半，之后与原哈希值做异或运算，这样就会混合原哈希值中的高位和低位，增大了随机性。**hash方法就是为了增加随机性，让数据元素更加均匀的分布，减少碰撞**。

### 小结

1.hash 方法的主要作用是将 key 的 hashCode 值进行处理，得到最终的哈希值。由于 key 的 hashCode 值是不确定的，可能会出现哈希冲突，因此需要将哈希值通过一定的算法映射到 HashMap 的实际存储位置上。

2.hash 方法的原理是，先获取 key 对象的 hashCode 值，然后将其高位与低位进行异或操作，得到一个新的哈希值。为什么要进行异或操作呢？因为对于 hashCode 的高位和低位，它们的分布是比较均匀的，如果只是简单地将它们加起来或者进行位运算，容易出现哈希冲突，而异或操作可以避免这个问题。

3.然后将新的哈希值取模（mod），得到一个实际的存储位置。这个取模操作的目的是将哈希值映射到桶（Bucket）的索引上，桶是 HashMap 中的一个数组，每个桶中会存储着一个链表（或者红黑树），装载哈希值相同的键值对（没有相同哈希值的话就只存储一个键值对）。

4.总的来说，HashMap 的 hash 方法就是将 key 对象的 hashCode 值进行处理，得到最终的哈希值，并通过一定的算法映射到实际的存储位置上。这个过程决定了 HashMap 内部键值对的查找效率。