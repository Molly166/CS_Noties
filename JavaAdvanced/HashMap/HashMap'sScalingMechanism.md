# HashMap�����ݻ���

HashMap�ĵײ��õ�Ҳ�����飬��HashMap�ﲻͣ�����Ԫ�أ��������޷�װ�ظ���Ԫ��ʱ������Ҫ������������ݣ��Ա�װ������Ԫ�أ��������⣬����������Ҳ����Ӧ����߲�ѯЧ�ʣ���Ϊ��Ͱ�������ˣ�ԭ����Ҫͨ������洢�ģ���ѯ��ʱ����Ҫ�����������ݺ���ܾ����Լ�ר����λ���ˡ�

��Ȼ�������޷��Զ����ݣ��������Ҫ���ݵĻ�������Ҫ�½�һ��������飬Ȼ���֮ǰ��С�����鸴�ƹ�ȥ������Ҫ���¼����ϣֵ�����·���Ͱ������ɢ�У����������Ҳ��ͦ��ʱ�ġ�

## resize����

HashMap��������ͨ��resize������ʵ�ֵģ�JDK8�������˺�����������ȳ���8��ʱ�򣬻Ὣ����ת��Ϊ���������߲�ѯЧ�ʣ���

java7��resize����Դ�룺

```java
// newCapacityΪ�µ�����
void resize(int newCapacity) {
    // С���飬��ʱ������
    Entry[] oldTable = table;
    // ����ǰ������
    int oldCapacity = oldTable.length;
    // MAXIMUM_CAPACITY Ϊ���������2 �� 30 �η� = 1<<30
    if (oldCapacity == MAXIMUM_CAPACITY) {
        // ��������Ϊ Integer �����ֵ 0x7fffffff��ʮ�����ƣ�=2 �� 31 �η�-1
        threshold = Integer.MAX_VALUE;
        return;
    }

    // ��ʼ��һ���µ����飨��������
    Entry[] newTable = new Entry[newCapacity];
    // ��С�����Ԫ��ת�Ƶ���������
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    // �����µĴ�����
    table = newTable;
    // ���¼�����ֵ
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```

�÷�������һ���µ����� newCapacity��Ȼ�� HashMap ���������� newCapacity��

���ȣ�������ȡ��ǰ HashMap �ľ����� oldTable �;����� oldCapacity������������Ѿ��ﵽ HashMap ֧�ֵ�������� MAXIMUM_CAPACITY�� 2 �� 30 �η������ͽ��µ���ֵ threshold ����Ϊ Integer.MAX_VALUE��2 �� 31 �η� - 1����������Ϊ HashMap ���������ܳ��� MAXIMUM_CAPACITY��

��Ϊ 2,147,483,647��Integer.MAX_VALUE�� - 1,073,741,824��MAXIMUM_CAPACITY�� = 1,073,741,823���պ����һ����HashMap ÿ�����ݶ���֮ǰ��һ������

���ţ���������һ���µ����� newTable������������ oldTable �е�Ԫ��ת�Ƶ������� newTable �С�ת�ƹ�����ͨ������ transfer ������ʵ�ֵġ��÷��������������е�ÿ��Ͱ������ÿ��Ͱ�еļ�ֵ�����¼����ϣֵ�󣬽�����뵽�������Ӧ��Ͱ�С�

ת����ɺ󣬷����� HashMap �ڲ����������� table ָ�������� newTable�������¼�����ֵ threshold���µ���ֵ�������� newCapacity ���Ը������� loadFactor �Ľ��������������������� HashMap ֧�ֵ�������� MAXIMUM_CAPACITY������ֵ����Ϊ MAXIMUM_CAPACITY + 1��������Ϊ HashMap ��Ԫ���������ܳ��� MAXIMUM_CAPACITY��

### ������newCapacity

```java
int newCapacity = oldCapacity * 2;
if (newCapacity < 0 || newCapacity >= MAXIMUM_CAPACITY) {
    newCapacity = MAXIMUM_CAPACITY;
} else if (newCapacity < DEFAULT_INITIAL_CAPACITY) {
    newCapacity = DEFAULT_INITIAL_CAPACITY;
}
```

������ newCapacity ����ʼ��Ϊԭ���� oldCapacity ��������Ȼ����� newCapacity ������ HashMap ���������� MAXIMUM_CAPACITY��2^30�����ͽ� newCapacity ����Ϊ MAXIMUM_CAPACITY����� newCapacity С��Ĭ�ϳ�ʼ���� DEFAULT_INITIAL_CAPACITY��16�����ͽ� newCapacity ����Ϊ DEFAULT_INITIAL_CAPACITY���������Ա���������̫С��̫���¹�ϣ��ͻ��������˷ѿռ䡣

Java 8 ��ʱ��newCapacity �ļ��㷽ʽ������һЩϸ΢�ı仯��

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

�����������ƻ���ԭ����2����4��8������

## transfer����

�÷�������ת�ƣ����ɵ�С����Ԫ�ؿ������µĴ������С�

```java
void transfer(Entry[] newTable, boolean rehash) {
    // �µ�����
    int newCapacity = newTable.length;
    // ����С����
    for (Entry<K,V> e : table) {
        while(null != e) {
            // ����������ͬ key �ϵĲ�ֵͬ
            Entry<K,V> next = e.next;
            // �Ƿ���Ҫ���¼��� hash
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            // ���ݴ�������������ͼ��� hash ����Ԫ���������е��±�
            int i = indexFor(e.hash, newCapacity);

            // ͬһλ���ϵ���Ԫ�ر����������ͷ��
            e.next = newTable[i];

            // �����µ�������
            newTable[i] = e;

            // �����ϵ���һ��Ԫ��
            e = next;
        }
    }
}
```

�÷�������һ���µ� Entry ���� newTable ��һ������ֵ rehash ��Ϊ���������� newTable ��ʾ�µĹ�ϣ��rehash ��ʾ�Ƿ���Ҫ���¼�����Ĺ�ϣֵ��

�ڷ����У����Ȼ�ȡ�¹�ϣ�����飩�ĳ��� newCapacity��Ȼ������ɹ�ϣ���е�ÿ�� Entry������ÿ�� Entry��ʹ������������ͬ key ֵ�Ĳ�ͬ value ֵ�洢��ͬһ�������С���� rehash Ϊ true������Ҫ���¼�����Ĺ�ϣֵ�������µĹ�ϣֵ�洢�� Entry �� hash �����С�

���ţ������¹�ϣ��ĳ��Ⱥͼ��Ĺ�ϣֵ������ Entry ���������е�λ�� i��Ȼ�󽫸� Entry ��ӵ�������� i λ���ϡ�������Ԫ����Ҫ�����������ͷ������˽���Ԫ�ص���һ��Ԫ������Ϊ��ǰ����λ���ϵ�Ԫ�ء�

��󣬱�����ɹ�ϣ���е�����Ԫ�غ�ת�ƹ�����ɣ��µĹ�ϣ�� newTable �Ѿ������˾ɹ�ϣ���е�����Ԫ�ء�

> ע�⣬`e.next=newTable[i]`��Ҳ����ʹ���˵������ͷ���뷽ʽ��ͬһλ������Ԫ���ܻᱻ���������ͷ��λ�ã������ȷ���һ�������ϵ�Ԫ�����ջᱻ�ŵ������β������ͻᵼ��**�ھ�������ͬһ�������ϵ�Ԫ�أ�ͨ�����¼�������λ�ú��п��ܱ��ŵ���������Ĳ�ͬλ����**

### java8����

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table; // ��ȡԭ�������� table
    int oldCap = (oldTab == null) ? 0 : oldTab.length; // ��ȡ���鳤�� oldCap
    int oldThr = threshold; // ��ȡ��ֵ oldThr
    int newCap, newThr = 0;
    if (oldCap > 0) { // ���ԭ�������� table ��Ϊ��
        if (oldCap >= MAXIMUM_CAPACITY) { // �������ֵ�Ͳ��������ˣ���ֻ��������ײȥ��
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && // û�������ֵ��������Ϊԭ����2��
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // �����µ� resize ����
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr; // ������ֵ��ֵ����Ա���� threshold
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]; // ���������� newTab
    table = newTab; // �������� newTab ��ֵ����Ա���� table
    if (oldTab != null) { // ��������� oldTab ��Ϊ��
        for (int j = 0; j < oldCap; ++j) { // �����������ÿ��Ԫ��
            Node<K,V> e;
            if ((e = oldTab[j]) != null) { // �����Ԫ�ز�Ϊ��
                oldTab[j] = null; // ���������и�λ�õ�Ԫ����Ϊ null���Ա���������
                if (e.next == null) // �����Ԫ��û�г�ͻ
                    newTab[e.hash & (newCap - 1)] = e; // ֱ�ӽ���Ԫ�ط���������
                else if (e instanceof TreeNode) // �����Ԫ�������ڵ�
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap); // �������ڵ���ѳ���������
                else { // �����Ԫ��������
                    Node<K,V> loHead = null, loTail = null; // ��λ�����ͷ����β���
                    Node<K,V> hiHead = null, hiTail = null; // ��λ�����ͷ����β���
                    Node<K,V> next;
                    do { // ����������
                        next = e.next;
                        if ((e.hash & oldCap) == 0) { // �����Ԫ���ڵ�λ������
                            if (loTail == null) // �����λ����û�н��
                                loHead = e; // ����Ԫ����Ϊ��λ�����ͷ���
                            else
                                loTail.next = e; // �����λ�����Ѿ��н�㣬����Ԫ�ؼ����λ�����β��
                            loTail = e; // ���µ�λ�����β���
                        }
                        else { // �����Ԫ���ڸ�λ������
                            if (hiTail == null) // �����λ����û�н��
                                hiHead = e; // ����Ԫ����Ϊ��λ�����ͷ���
                            else
                                hiTail.next = e; // �����λ�����Ѿ��н�㣬����Ԫ�ؼ����λ�����β��
                            hiTail = e; // ���¸�λ�����β���
                        }
                    } while ((e = next) != null); //
                    if (loTail != null) { // �����λ����Ϊ��
                        loTail.next = null; // ����λ�����β���ָ�� null���Ա���������
                        newTab[j] = loHead; // ����λ������Ϊ�������Ӧλ�õ�Ԫ��
                    }
                    if (hiTail != null) { // �����λ����Ϊ��
                        hiTail.next = null; // ����λ�����β���ָ�� null���Ա���������
                        newTab[j + oldCap] = hiHead; // ����λ������Ϊ�������Ӧλ�õ�Ԫ��
                    }
                }
            }
        }
    }
    return newTab; // ����������
}
```

1����ȡԭ�������� table�����鳤�� oldCap ����ֵ oldThr��

2�����ԭ�������� table ��Ϊ�գ���������ݹ�����������鳤�� newCap ������ֵ newThr��Ȼ��ԭ�����е�Ԫ�ظ��Ƶ��������С�

3�����ԭ�������� table Ϊ�յ���ֵ oldThr ��Ϊ�㣬��˵����ͨ�����������췽�������� HashMap����ʱ����ֵ��Ϊ�����鳤�� newCap��

4�����ԭ�������� table ����ֵ oldThr ��Ϊ�㣬��˵����ͨ���޲������췽�������� HashMap����ʱ��Ĭ�ϳ�ʼ���� DEFAULT_INITIAL_CAPACITY��16����Ĭ�ϸ������� DEFAULT_LOAD_FACTOR��0.75������������鳤�� newCap ������ֵ newThr��

5����������ֵ threshold�������丳ֵ����Ա���� threshold��

6������������ newTab�������丳ֵ����Ա���� table��

7����������� oldTab ��Ϊ�գ�������������ÿ��Ԫ�أ����临�Ƶ��������С�

8������������ newTab��

#### С��

��������HashMap�в������Ԫ��ʱ��HashMap���Զ��������ݲ�����������Ԫ�������ﵽ�������ӣ�loadfactor���������鳤��ʱ�����Ա�֤��洢��Ԫ���������ᳬ�����������ơ�

�ڽ������ݲ���ʱ��HashMap���Ƚ�����ĳ�������һ����Ȼ��ԭ����Ԫ������ɢ�е��µ������С�

����Ԫ�ص�λ����ͨ��key��hash�����鳤�Ƚ���������õ��ģ���������鳤�������Ԫ�ص�λ��Ҳ�ᷢ��һЩ�ı䡣һ�����������䣬��һ��������Ϊ��ԭ����+����������

