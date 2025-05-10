# Java HashMap

HashMap��һ��ɢ�б����洢�������Ǽ�ֵ�ԣ�key-value��ӳ�䡣
HashMapʵ����Map�ӿڣ����ݼ���HashCodeֵ�洢���ݣ����кܿ�ķ����ٶȣ��������һ����¼�ļ�Ϊnull����֧���߳�ͬ����
HashMap������ģ��������¼�����˳��
HashMap�̳���AbstractMap��ʵ����Map��Cloneable��java.io.Serializable�ӿڡ�

![HashMap](/JavaAdvanced/images/Screenshot%202025-05-10%20110612.png)

**HashMap��key��value���Ϳ�����ͬҲ���Բ�ͬ**���������ַ���(String)���͵�key��value��Ҳ����������(Integer)��key���ַ���(String)���͵�value��

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

> HashMap�е�Ԫ��ʵ�����Ƕ���һЩ�����Ļ������Ϳ���ʹ�����İ�װ�ࡣ

|��������|��������|
|---|---|
|boolean|Boolean|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|float|Float|
|double|Double|
|char|Character|

HashMap��λ��java.util���У�ʹ��ǰ��Ҫ���������﷨��ʽ���£�

```java
import java.util.HashMap;//����HashMap��
```

����ʵ�����Ǵ���һ��HashMap����Sites������(Integer)��key���ַ���(String)���͵�value��

```java
HashMap<Integer,String> Sites = new HashMap<Integer,String>();
```

HashMap���ṩ�кܶ�**���Ԫ��**�ķ�������Ӽ�ֵ��(key-value)����ʹ��put()����:

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

���ǿ���ʹ��get(key)��������ȡkey��Ӧ��value,��**����Ԫ��**��

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

���ǿ���ʹ��remove(key)������**ɾ��**key��Ӧ�ļ�ֵ��(key-value):

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

**ɾ�����м�ֵ��**����ʹ��clear()����

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

**�����С**:���Ҫ����HashMap�е�Ԫ�ص���������ʹ��size()������

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

**����HashMap**������ʹ��for-each������HashMap�е�Ԫ�ء�
���ֻ�����ȡkey������ʹ��keySet()������Ȼ��ͨ��get(key)��ȡ��Ӧ��value��
���ֻ���ȡvalue,����ʹ��values()����

```java
// ���� HashMap ��      
import java.util.HashMap;

public class RunoobTest {
    public static void main(String[] args) {
        // ���� HashMap ���� Sites
        HashMap<Integer, String> Sites = new HashMap<Integer, String>();
        // ��Ӽ�ֵ��
        Sites.put(1, "Google");
        Sites.put(2, "Runoob");
        Sites.put(3, "Taobao");
        Sites.put(4, "Zhihu");
        // ��� key �� value
        for (Integer i : Sites.keySet()) {
            System.out.println("key: " + i + " value: " + Sites.get(i));
        }
        // �������� value ֵ
        for(String value: Sites.values()) {
          // ���ÿһ��value
          System.out.print(value + ", ");
        }
    }
}
```

## HashMap����

|����|����|
|---|---|
|clear()|ɾ��hashMap�е����м�ֵ��|
|clone()|����һ��hashMap|
|size()|����hashMap�м�ֵ�Ե�����|
|isEmpty()|�ж�hashMap�Ƿ�Ϊ��|
|put()|����ֵ����ӵ�hashMap��|
|putAll()|�����еļ�ֵ����ӵ�hashMap��|
|putlfAbsent()|���hashMap�в�����ָ���ļ�����ָ���ļ�ֵ�Բ��뵽hashMap��|
|remove()|ɾ��hashMap��ָ����key��ӳ���ϵ|
|containsKey|���hashMap���Ƿ����ָ����key��Ӧ��ӳ���ϵ|
|containsValue|���hashMap���Ƿ����ָ����values��Ӧ��ӳ���ϵ|
|replace()|�滻hashMap����ָ����key��Ӧ��value|
|replaceAll()|��hashMap�е�����ӳ���ϵ�滻�ɸ����ĺ�����ִ�еĽ��|
|get()|��ȡָ��key��Ӧ��value|
|getOrDefault()|��ȡָ����key��Ӧ��value������Ҳ���key���򷵻����õ�Ĭ��ֵ|
|forEach()|��hashMap�е�ÿ��ӳ��ָ���Ĳ���|
|entrySet()|����hashMap������ӳ����ļ��ϼ�����ͼ|
|keySet()|����hashMap������key��ɵļ�����ͼ|
|values()|����hashMap�д��ڵ�����valueֵ|
|merge()|��Ӽ�ֵ�Ե�hashMap��|
|compute()|��hashMap��ָ����key��ֵ�������¼���|
|computelfAbsent()|��hashMap��ָ��key��ֵ�������¼��㣬������������key������ӵ�hashMap��|
|computelfPresent()|��hashMap��ָ��key��ֵ�������¼��㣬ǰ���Ǹ�key������hashMap��|