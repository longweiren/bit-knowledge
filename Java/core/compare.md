###Java中的 Comparable, Comparator 以及比较操作###


**Comparable**
-----
`Comparable`接口定义了一个方法 `compareTo`，用于实现了该接口的同一个类不同实例对象之间进行比较。当我们使用`Collections.sort(), Arrays.sort()`对集合或数组中的元素进行排序时，都需要集合或数组中的对象有实现该接口。实现该接口的对象也可以作为`SortedMap`实例中的 key，或者作为 `SortedSet` 实例中的元素。

> `x.compareTo(y)`返回值(n为正数)：
> -n 当我们希望 x < y时
>  0 当我们希望 x == y时
>  n 当我们希望 x > y时

* Arrays.sort()

Arrays.sort(byte[] a), Arrays.sort(char[] a), Arrays.sort(short[] a), Arrays.sort(int[] a), Arrays.sort(long[] a), Arrays.sort(float[] a), Arrays.sort(double[] a)使用快速排序算法(`DualPivotQuicksort.sort()`)对数组进行(升序)排序。

######Arrays.sort(Object[] a)使用`LegacyMergeSort`(归并排序)或`ComparableTimSort`(二叉树排序)对数组进行排序，排序方法都会使用 compareTo 方法对数组元素进行比较。######

Arrays.sort(T[] a, Comparator<? super T> c)当comparator为null时退化到sort(Object[] a)，否则使用`LegacyMergeSort`(归并排序)或`TimSort`(二叉树排序)对数组进行排序，排序方法都会使用 comparator 方法对数组元素进行比较。

Arrays还有一个排序方法 parallelSort()，使用`DualPivotQuicksort.sort`或`ArraysParallelSortHelpers`对原始类型数组进行排序。使用`TimSort`或`ArraysParallelSortHelpers.FJObject.Sorter`对引用类型数组进行排序。  

如果parallelSort()方法是用Comparator参数（传入null值时默认使用NaturalOrder.INSTANCE），排序方法会使用`Comparator`对数组元素进行比较。

* Collections.sort(List list, [comparator])

参数是一个集合，集合元素需要实现`Comparable`接口。Collections.sort(List list)实际调用 list.sort([comparator])。List.sort([comparator])有一个参数`Comparator`，实际会调用Arrays.sort(list.toArray(), [comparator])对数组进行排序。

* SortedMap

SortedMap 有一个 comparator() 方法，如果该方法返回 comparator，则使用该 comparator 对 SortedMap 的key进行排序，否则使用 key 的Comparable 比较 key 并进行排序。如果 SortedMap 没有 comparator，同时 key 没有实现 Comparable接口，则会抛出 ClassCastException。

* SortedSet

与 SortedMap 类似，差别在于SortedMap关注的是 key， 而 SortedMap 关注的是集合元素。


需要注意`compareTo`和`equals`方法之间的区别，`x.compareTo(y) == 0` 不代表`x.equals(y)==true`。比如我们有两个`User`对象user1`{name:'David', age:19}`和user2`{name:'Lucy', age:19}`，我们希望集合中对user的年龄进行排序，这时`user1.compareTo(user2)==0`(两个user年龄相等)，而`user1.equals(user)`显然不能返回
true(完全两个user)。

```
User.java
public class User implements Comparable {
    public String name;
    public int    age;

    @Override
    public boolean equals(Object other) {
        if(this == other) {
            return true;
        }

        if(other instanceof User) {
            User anotherUser = (User) other;
            return this.name.equals(anotherUser.name)
                && (this.age == anotherUser.age)
        }

        return false;
    }
    @Override
    public int compareTo(User other) {
        return this.age - other.age;
    }
}
```

**Comparator**
-----
上面介绍 `Comparable` 时已经提到了 Comparator 的一些应用场景。`Comparator`接口有一个 `compare(T obj1, T obj2)` 方法，该方法与 `Comparable` 的 `compareTo(T obj)` 方法一致。

Comparator 与 Comparable 类似，主要用在排序算法中对两个元素进行比较。既然使用 Comparable 已经可以实现对集合对象进行排序，为什么还要出现 Comparator 呢？

```
初始数据
User[] users = [{
    name : 'David',
    age  : 16
},{
    name : 'Lucy',
    age  : 19
},{
    name : 'Amy',
    age  ：17
}];
```

场景1：按年龄对User对象集合排序
```
public void sort(User[] users) {
    Arrays.sort(users); // 使用 Comparable 对元素进行比较排序. 
    // 结果为[{
        name : 'David',
        age  : 16
    },{
        name : 'Amy',
        age  ：17
    },{
        name : 'Lucy',
        age  : 19
    }]
}
```

场景2：按姓名对User对象集合排序
> User 实现的 Comparable 接口仅使用 age 字段对 user 对象进行比较，这时就需要 Comparator 实现个性化的比较需求了

```
Comparator nameComparator = new Comparator<User>(){
    int compare(User u1, User u2){
        return u1.name.compareTo(u2.name);  // User 实现了 Comparable 接口
    }

    boolean equals(Object obj){
        // TODO
        return false;
    }
};
public void sort(User[] users, Comparator nameComparator) {
    Arrays.sort(users, nameComparator); // 使用 Comparator 对元素进行比较排序. 
    // 结果为[{
        name : 'Amy',
        age  ：17
    },{
        name : 'David',
        age  : 16
    },{
        name : 'Lucy',
        age  : 19
    }]
}
```

从示例代码我们可以看出，Comparable 接口已经固化了集合做排序时的比较方法，而Comparator接口赋予了更多的可能性，当我们需要对集合内部的元素做排序时，如果集合元素没有实现 Comparable 接口，或者集合元素实现 Comparable 接口的比较方法不满足需求时，就是 Comparator登场的时候了。

**Java中的 > < ==**
-----

Java 中可以使用 `>`, `>=`, `<`, `<=`, `==`对原始数据类型进行比较，也可以使用 `==` 对引用类型进行比较。使用 `==` 对引用类型进行比较时，比较的是引用的实际对象是否为同一个。

> 与比较操作符相关的JVM指令
> 
> int 类型比较(char, short, byte会转换为int再比较)
> `>`  转换为 if_icmple(小于等于)     
>   if(a > b){
>     ... code1
>   } else {
>     ... code2
>   }
>   //if_icmple 的意思是，如果 a <= b 则跳转到 code2 继续执行，否则继续执行 code1.
>   
> `>=` 转换为 if_icmplt(小于)     
> `==` 转换为 if_icmpne(不等于)     
> `<`  转换为 if_icmpge(大于等于)     
> `<=` 转换为 if_icmpgt(大于)     
> 
> float 类型比较
> `>`  转换为 fcmpl + ifle(小于等于)
>   if(a > b){
>     ... code1
>   } else {
>     ... code2
>   }
>   //fcmpl 的意思是比较a和b，
>   //ifle  的意思是如果 fcmpl 比较结果 <=0（也就是a<=b）,则跳转到 code2 继续执行，否则继续执行 code1.
> `>=` 转换为 fcmpl + iflt(小于)
> `==` 转换为 fcmpl + ifne(不等于)
> `<`  转换为 fcmpg + ifge(大于等于)
> `<=` 转换为 fcmpg + ifgt(大于)
> 
> double 类型比较（与 float 类似）
> `>`  转换为 dcmpl + ifle(小于等于)
> `>=` 转换为 dcmpl + iflt(小于) 
> `==` 转换为 dcmpl + ifne(不等于) 
> `<`  转换为 dcmpg + ifge(大于等于) 
> `<=` 转换为 dcmpg + ifgt(大于) 

> 引用类型比较
> if_acmpne(不等于)
