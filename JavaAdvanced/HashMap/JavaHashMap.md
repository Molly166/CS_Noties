# Java HashMap

HashMap是一个散列表，它存储的内容是键值对（key-value）映射。
HashMap实现了Map接口，根据键的HashCode值存储数据，具有很快的访问速度，最多允许一条记录的键为null，不支持线程同步。
HashMap是无序的，即不会记录插入的顺序。
HashMap继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。

![HashMap](/JavaAdvanced/images/Screenshot%202025-05-10%20110612.png)

**HashMap的key与value类型可以相同也可以不同**，可以是字符串(String)类型的key和value，也可以是整型(Integer)的key和字符串(String)类型的value。

```java
Map<String,String> map=Map.of("google","google.com","runoob","runoob.com");
```

|key|value|
|---|---|
|google|google.com|
|runoob|runoob.com|

```java
Map<Integer,String> map=Map.of(1,"google",2,"runoob");
```

|key|value|
|---|---|
|1|google|
|2|runoob|

> HashMap中的元素实际上是对象，一些常见的基本类型可以使用它的包装类。

|基本类型|引用类型|
|---|---|
|boolean|Boolean|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|

HashMap类位于java.util包中，使用前需要引入它，语法格式如下：

```java
import java.util.HashMap;//引入HashMap类
```

以下实例我们创建一个HashMap对象Sites，整型(Integer)的key和字符串(String)类型的value：

```java
HashMap<Integer,String> Sites = new HashMap<Integer,String>();
```

HashMap类提供有很多**添加元素**的方法，添加键值对(key-value)可以使用put()方法:

```java
import java.util.HashMap;
public class RunoobTest{
    public static void main(String[] args){
        HashMap<Integer,String>Sites=new HashMap<Integer,String>();
        Sites.put(1,"google");
        Sites.put(2."runoob");
        System.out.println(Sites);
    }
}
```

我们可以使用get(key)方法来获取key对应的value,来**访问元素**：

```java
import java.util.HashMap;
public class RunoobTest{
    public static void main(String[] args){
        HashMap<Integer,String>Sites=new HashMap<Integer,String>();
        Sites.put(1,"google");
        Sites.put(2."runoob");
        System.out.println(Sites.get(1));
    }
}
```

我们可以使用remove(key)方法来**删除**key对应的键值对(key-value):

```java
import java.util.HashMap;
public class RunoobTest{
    public static void main(String[] args){
        HashMap<Integer,String>Sites=new HashMap<Integer,String>();
        Sites.put(1,"google");
        Sites.put(2."runoob");
        Sites.remove(2);
        System.out.println(Sites);
    }
}
```

**删除所有键值对**可以使用clear()方法

```java
import java.util.HashMap;
public class RunoobTest{
    public static void main(String[] args){
        HashMap<Integer,String>Sites=new HashMap<Integer,String>();
        Sites.put(1,"google");
        Sites.put(2."runoob");
        Sites.clear();
        System.out.println(Sites);
    }
}
```

**计算大小**:如果要计算HashMap中的元素的数量可以使用size()方法：

```java
import java.util.HashMap;
public class RunoobTest{
    public static void main(String[] args){
        HashMap<Integer,String>Sites=new HashMap<Integer,String>();
        Sites.put(1,"google");
        Sites.put(2."runoob");
        System.out.println(Sites.size());
    }
}
```

**迭代HashMap**：可以使用for-each来迭代HashMap中的元素。
如果只是想获取key，可以使用keySet()方法，然后通过get(key)获取对应的value。
如果只想获取value,可以使用values()方法

```java
// 引入 HashMap 类      
import java.util.HashMap;

public class RunoobTest {
    public static void main(String[] args) {
        // 创建 HashMap 对象 Sites
        HashMap<Integer, String> Sites = new HashMap<Integer, String>();
        // 添加键值对
        Sites.put(1, "Google");
        Sites.put(2, "Runoob");
        Sites.put(3, "Taobao");
        Sites.put(4, "Zhihu");
        // 输出 key 和 value
        for (Integer i : Sites.keySet()) {
            System.out.println("key: " + i + " value: " + Sites.get(i));
        }
        // 返回所有 value 值
        for(String value: Sites.values()) {
          // 输出每一个value
          System.out.print(value + ", ");
        }
    }
}
```

## HashMap方法

|方法|描述|
|---|---|
|clear()|删除hashMap中的所有键值对|
|clone()|复制一份hashMap|
|size()|计算hashMap中键值对的数量|
|isEmpty()|判断hashMap是否为空|
|put()|将键值对添加到hashMap中|
|putAll()|将所有的键值对添加到hashMap中|
|putlfAbsent()|如果hashMap中不存在指定的键，则将指定的键值对插入到hashMap中|
|remove()|删除hashMap中指定键key的映射关系|
|containsKey|检查hashMap中是否存在指定的key对应的映射关系|
|containsValue|检查hashMap中是否存在指定的values对应的映射关系|
|replace()|替换hashMap中是指定的key对应的value|
|replaceAll()|将hashMap中的所有映射关系替换成给定的函数所执行的结果|
|get()|获取指定key对应的value|
|getOrDefault()|获取指定的key对应的value，如果找不到key，则返回设置的默认值|
|forEach()|对hashMap中的每个映射指定的操作|
|entrySet()|返回hashMap中所有映射项的集合集合视图|
|keySet()|返回hashMap中所有key组成的集合视图|
|values()|返回hashMap中存在的所有value值|
|merge()|添加键值对到hashMap中|
|compute()|对hashMap中指定的key的值进行重新计算|
|computelfAbsent()|对hashMap中指定key的值进行重新计算，如果不存在这个key，则添加到hashMap中|
|computelfPresent()|对hashMap中指定key的值进行重新计算，前提是该key存在于hashMap中|