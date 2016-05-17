###Java中的原始类型###

*Java语言中有 8 种原始类型： char, boolean, byte, short, int, long, float, double。每一种原始类型都对应有一个包装类：Character, Boolean, Short, Integer, Long, Float, Double。我们都知道的是：原始类型值是常量值，不能在其上执行方法调用，而包装类对象是引用值；原始类型值和包装类对象在一定条件下可以执行自动装箱拆箱(autoboxing and unboxing)功能相互转换。除了这些每个人都知道的，本文总结了少部分人未关注到的知识点*

#####原始类型的存储空间
                   MIN_VALUE                                MAX_VALUE
* byte      1字节  (-2)^7  = -128(0x80 = 1000 0000)         (2)^7  - 1 = 127(0111 1111)
* short     2字节  (-2)^15 = -32768(0x 80 00)               (2)^15 - 1 = 32767(0x7f ff)
* int       4字节  (-2)^31 = 0x80 00 00 00                  (2)^31 - 1 = 0x7f ff ff ff
* long      8字节  (-2)^15 = 0x80 00 00 00 00 00 00 00L     (2)^15 - 1 = 0x7f ff ff ff ff ff ff ffL

* float     4字节  
* double    8字节  

原始类型的第一个 bit 是符号位(1代表负数，0代表正数)，剩下的位数表示值。当对原始类型的最大值 +1，或最小值 -1时，结果超出取值范围，会发生什么结果呢？

```
// local_var[args, bmin, bmax, bmin1, bmax1, bmin2, bmax2]
public static void main(String[] args) {
    byte bmin = -128;
    byte bmax = 127;

    byte bmin1 = (byte) (bmin - 1);     // (1) 127
    byte bmax1 = (byte) (bmax + 1);     // (2) -128

    int  bmin2 = bmin - 1;              // (3) -129
    int  bmax2 = bmax + 1;              // (4) 128
}
```

这里有几个点需要注意：
1. 对 byte 类型的值做算术运算时会先转换成 int 类型
2. 当对运算结果进行强制类型转换时，会对结果进行截取。代码行(1) `bmin - 1` 结果为 int 值(即代码行(3)的结果)，二进制表示为 (1111 1111) 1000 0000 - 0000 0000 0000 0001 =  1111 1111 0111 1111，结果用10进制表示即为 -129；当对结果强制转换为 byte 时，忽略高8位的值，剩下值为 0111 1111，结果用10进制表示即为 127.
3. 代码行(2) `bmax + 1` 结果为 int 值(即代码行(4)的结果)，二进制表示为 (0000 0000) 0111 1111 + 0000 0000 0000 0001 = 0000 0000 1000 0000，结果用10进制表示即为 128；当对结果强制转换为 byte 时，同样忽略高8位，剩下的值为 1000 0000，用10进制表示就成了负数 -128

编译后的代码：
```
    Code:
      stack=2, locals=7, args_size=1
         // 把 byte 常量 -128 压入操作数栈
         0: bipush        -128
         // 从栈顶取 int 元素值存入本地变量1(及变量 bmin).
         2: istore_1
         3: bipush        127
         5: istore_2

         // 提取 int 型本地变量1压入操作数栈
         6: iload_1
         7: iconst_1
         // 栈顶2个元素相减，结果存入栈顶
         8: isub
         // int 值强制转换为 byte值
         9: i2b
         // 把栈顶元素 int 型值存入本地变量3(bmin1)
        10: istore_3

        11: iload_2
        12: iconst_1
        13: iadd
        14: i2b
        15: istore        4

         // 提取 int 型本地变量1压入操作数栈
        17: iload_1
        18: iconst_1
        19: isub
         // 把栈顶元素 int 型值存入本地变量5(bmin2)
        20: istore        5

        22: iload_2
        23: iconst_1
        24: iadd
        25: istore        6
```

分析编译后的代码我们可以发现，虽然我们定义的变量类型为 byte，但是实际操作都是以 int 类型对待(如istore_1, iload_1等)；另外虽然压入操作数栈的元素是 byte(bipush)，但是取值时却看作是 int(istore)。

原因是，JVM操作数栈被组织成一个以字长为单位的数组，也就是说数组中的元素为一个字长(4 bytes，32 bits)，正好是 int 表示的长度。其实，在把 byte, short ,char类型的值压入到操作数栈都会被转换为 int。

所以两个 byte 值相加，结果赋值给 byte 时需要强制转换为 byte(因为两个 byte 相加的结果是 int)，这个在编译时就会验证。


##### 类型转换
原始类型之间进行类型转换时遵循如下规则：
* 长字节类型转换为短字节类型时，仅保留短字节类型长度的地位长字节类型值。如上述 int 转 byte 时，仅保留 int 值的低8位值。
* 短字节类型转换为长字节类型时，以短字节类型值的符号位补齐长字节类型的高位值。例如 byte 值 `0111 1111` 转换为 int 值时，结果是 `0000 0000 0111 1111`；而 `1000 0000` 转换为 int 值时，结果为 `1111 1111 1000 0000`。

这里还有一个值得注意的地方：对原始类型进行算术运算后，结果可能超出其能表现的范围，这个时候 JVM 会对结果进行截取。例如连个 int 值相乘后结果可能超出 32 位，结果会被截取，只保留低32位数据。 (Integer.MAX_VALUE + 1 = Integer.MIN_VALUE，suprise?)

##### 自动装箱拆箱(autoboxing unboxing)
这是一个 JDK5 之后的新功能。我们以(int - Integer)为例，我们可以把 int 值赋给 Integer 对象，也可以把 Integer 对象赋值给 int 变量。例如：
```
Integer count = 1;
int age = new Integer(19);
```

编译后代码为：
```
Code:

   0: iconst_1
   1: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
   4: astore_1

   5: new           #5                  // class java/lang/Integer
   8: dup
   9: bipush        19
  11: invokespecial #6                  // Method java/lang/Integer."<init>":(I)V
  14: invokevirtual #7                  // Method java/lang/Integer.intValue:()I
  17: istore_2
  18: return
```

分析编译后代码我们知道：  
* 把 int 值赋给 Integer 对象，其实是调用了 Integer 类的静态方法 valueOf()。查看 Integer.java 源代码，我们发现 valueOf 方法有两种方式得到 int 值对应的 Integer 对象：或者从 IntegerCache 中获取，或者调用 Integer(int) 构造函数新建一个对象。
* 而把 Integer 对象赋值给 int 变量的过程，是调用了 Integer 类的静态方法 intValue()。Integer 对象有一个 int 属性 `value`，表示的就是 Integer 对象的值。intValue() 方法即是返回 `value` 值；而 Integer(int) 构造函数则是为 value 赋值

```
//Integer.java

private final int value;
public Integer(int value) {
    this.value = value;
}
public int valueOf() {
    if(i >= -128 && i <= IntegerCache.high) {
        return IntegerCache.cache(i + 128);
    } else {
        return new Integer(i);
    }
}
```

##### Cache
上一小节的例子中我们看到引入了 IntegerCache(除了Double, Float外，原始类型都有对应的 Cache)。这是一个有趣的类，接下来我们分析下。

我们把 int 值赋给 Integer 对象，把 Integer 对象赋值给  int 变量后，两个值进行比较结果又会怎样呢？

```
Integer countObj = 19;
int countVar = new Integer(19);
System.out.println(countObj == countVar);  (1) true
System.out.println(countVar == countObj);  (2) true

Integer countObj1 = 19;
System.out.println(countVar == countObj1); (3) true
```

编译后的代码为：
```
    Code:
       0: bipush        19
       2: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1

       6: new           #5                  // class java/lang/Integer
       9: dup
      10: bipush        19
      12: invokespecial #6                  // Method java/lang/Integer."<init>":(I)V
      15: invokevirtual #7                  // Method java/lang/Integer.intValue:()I
      18: istore_2

      //countObj == countVar
      19: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      22: aload_1
      23: invokevirtual #7                  // Method java/lang/Integer.intValue:()I
      26: iload_2
      27: if_icmpne     34
      30: iconst_1
      31: goto          35
      34: iconst_0
      35: invokevirtual #8                  // Method java/io/PrintStream.println:(Z)V

      //countVar == countObj
      38: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      41: iload_2
      42: aload_1
      43: invokevirtual #7                  // Method java/lang/Integer.intValue:()I
      46: if_icmpne     53
      49: iconst_1
      50: goto          54
      53: iconst_0
      54: invokevirtual #8                  // Method java/io/PrintStream.println:(Z)V

      // countObj == countObj1
      57: bipush        19
      59: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      62: astore_3
      63: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      66: aload_1
      67: aload_3
      68: if_acmpne     75
      71: iconst_1
      72: goto          76
      75: iconst_0
      76: invokevirtual #8                  // Method java/io/PrintStream.printl
n:(Z)V
      79: return
```
分析编译后的代码我们可以发现：
* 代码(1)和(2)表达式都是 int 变量与 Integer 对象进行比较，编译后指令码也差不多（唯一差异在于 加载 int 值 或 Integer 对象到操作数栈的顺序相反，不影响比较结果），都需要调用 intValue() 方法先把 Integer 对象取 int 值后再比较。同样用 `if_icmpne` 指令对两个值进行比较。
* 代码(3)则与两个语句有差异。用的是`if_acmpne`指令对两个对象进行比较，也就是比较 countObj 和 countObj1 是否指向同一个对象（是比较引用，而不是比较值）。
* 这个地方我们同时可以发现，HostSpot虚拟机用常量1 表示 boolean 值 true，用常量0 表示 boolean 值 false。
* (1), (2)和(3)结果都是true

本文到这里就结束了吗？还没有，我们上面提到的Cache还没有说明。

上面代码中，我们把常量19 赋值给两个不同的对象引用，比较两个不同的对象引用发现指向同一个对象。那是不是操作所有的 int 值都能得到同样的结果？我们看 Integer.java 源代码中的 valueOf 方法时，看到一个判断条件是 `i >= -128 ` 时就从 Cache 里取值。那我们用小于 -128 的 int 值赋值给 Integer 对象时会有什么结果呢？

```
Integer countObj1 = -200;
Integer countObj2 = -200;
System.out.println(countObj1 == countObj2); (1)

Integer countObj3 = new Integer(1);
Integer countObj4 = new Integer(1);
System.out.println(countObj3 == countObj4); (2)

```

编译后的代码为：
```
    Code:
       0: sipush        -200
       3: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       6: astore_1

       7: sipush        -200
      10: invokestatic  #4                  // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      13: astore_2

      // countObj1 == countObj2
      14: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      17: aload_1
      18: aload_2
      19: if_acmpne     26
      22: iconst_1
      23: goto          27
      26: iconst_0
      27: invokevirtual #8                  // Method java/io/PrintStream.println:(Z)V


      30: new           #5                  // class java/lang/Integer
      33: dup
      34: iconst_1
      35: invokespecial #6                  // Method java/lang/Integer."<init>":(I)V
      38: astore_3

      39: new           #5                  // class java/lang/Integer
      42: dup
      43: iconst_1
      44: invokespecial #6                  // Method java/lang/Integer."<init>":(I)V
      47: astore        4

      // countObj3 == countObj4
      49: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
      52: aload_3
      53: aload         4
      55: if_acmpne     62
      58: iconst_1
      59: goto          63
      62: iconst_0
      63: invokevirtual #8                  // Method java/io/PrintStream.println:(Z)V
      66: return
```

分析编译后的代码我们可以发现，代码(1)和(2)都是用`if_acmpne`指令比较两个对象的引用，运行代码后结果均为 `false`，也就是说两个引用并未指向相同的对象，虽然两个对象的表示值是一样的(前两个均是 -200，后两个均是 1)。为什么会这样呢？

代码(2)比较容易理解，两个都是用 `new` 指令新创建的对象实例的引用，分别指向不同的对象，用 `==` 比较结果为 false 也比较容易理解。

但是代码(1)可能会给部分人带来困惑，因为我们在上一节用`Integer countObj = 19; Integer countObj1 = 19;`时，比较`countObj == countObj1`结果是 true。这里仅仅把 19 改为 -200，比较结果就反转了。Cache 就是造成这个结果的原因。

前面说过把常量值赋值给 Integer 对象时会调用 valueOf() 方法。里面有个两个返回 Integer 对象的分支，一个是新创建对象，一个是从 Cache 里取。如果是新创建对象，则每次返回一个新对象(`Integer countObj3 = -200; Integer countObj4 = -200;`就是这种情况，所以比较 `countObj3 == countObj4`时返回了 false)；而如果从 Cache里取，则除第一次外每次返回从Cache里取得对应的的第一次创建的对象(`Integer countObj = 19; Integer countObj1 = 19;`就是这种情况，所以比较 `countObj == countObj1`时返回了 true)。

IntegerCache 会缓存 [-128, high]范围 int 值对应的 Integer对象(其中 high 可配置，默认为 127)，这也正是 valueOf() 方法里的判断条件。而缓存值是存放在 IntegerCache 的一个 Integer 数组里。
```
public int valueOf() {
    if(i >= -128 && i <= IntegerCache.high) {
        return IntegerCache.cache(i + 128);
    } else {
        return new Integer(i);
    }
}
```

boolean 在Java中比较简单，前面提到过用 1 表示 true，用 0 表示 false。而char, float, double 有部分要点与其他几个原始类型不太一样，后面继续总结。
