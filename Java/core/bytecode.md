### JVM Instruction Set CheetSheet ###

1. aaload

> 把数组元素的引用加载到操作数栈。

JVM Stack变化情况:  
before:  ..., arrayref, index  
after :  ..., valueref

> 如果 arrayref 为 null，则抛出 NullPointerException异常；  
> 如果 index 超出 arrayref 的范围，则抛出 ArrayIndexOutOfBoundsException

2. aastore

> 为数组元素赋值

JVM Stack变化情况:  
before:  ..., arrayref, index， valueref  
after :  ...  
arrayref[index] = valueref

```
 示例1：
 public void arrayR() {
   String[] arr = new String[10];
   String arr1, arr2, arr3, arr4, arr5 = null;
   arr[0] = "Hello";
   arr[1] = arr5;
 }
 
 字节码
 
 local_var[0] = this, local_var[1] = arr, local_var[2] = arr1, local_var[3] = arr2, local_var[4] = arr3, local_var[5] = arr4, local_var[6] = arr5
 
 byte值 10 入操作数栈.  stack[10]
 0: bipush        10
 创建数组引用（数组大小为栈顶值）.  stack[arr], arr.length=10
 2: anewarray     #2                  // class java/lang/String
 取栈顶元素存入本地变量表1. stack[], local_var[1] = arr
 5: astore_1
 
 null 值入操作数栈.  stack[null]
 6: aconst_null
 操作数栈顶元素赋值给本地变量6. stack[] arr5 = null
 7: astore        6
 
 本地变量1入操作数栈. stack[arr]
 9: aload_1
 常量0 入操作数栈.  stack[arr, 0]
 10: iconst_0
 从运行时常量池取值入操作数栈. stack[arr, 0, #3 ref of Hello]
 11: ldc           #3                  // String Hello
 从操作数栈顶取3个元素， 为数组元素赋值.  stack[] arr[0] = #3
 13: aastore
 
 本地变量1入操作数栈. stack[arr]
 14: aload_1
 常量 1 入操作数栈.   stack[arr, 1]
 15: iconst_1
 本地变量6入操作数栈. stack[arr, 1, arr5]
 16: aload           6
 从操作数栈顶取3个元素， 为数组元素赋值.  stack[] arr[1] = arr5 null
 18: aastore
 19: return
```

3. aconst_null

> 把 null 值入操作数栈

JVM Stack变化情况:  
before:  ...  
after :  ..., null

4. aload #index

> 从本地变量表加载引用到操作数栈

JVM Stack变化情况:  
before:  ...  
after :  ..., ref

5. aload_<n>

> 从本地变量表加载引用到操作数栈。 n 取值范围为 [0, 3]

JVM Stack变化情况:  
before:  ...  
after :  ..., ref  

6. anewarray #index

> 创建新数组，数组长度为栈顶元素，数组元素类型为 anewarray 指令后的参数指定的常量

JVM Stack变化情况:  
before:  ..., count  
after :  ..., arrayRef  

arrayRef[count]

7. areturn

> 返回栈顶元素值。当前栈帧销毁，当前方法调用结束。返回的值载入前一栈帧栈顶

JVM Stack变化情况(当前栈):  
before:  ref  
after :    

JVM Stack变化情况(pre栈):  
before:  ...  
after :  ..., ref  

8. arraylength

> 获得数组的长度

JVM Stack变化情况(当前栈):  
before:  ..., arrayref  
after :  ..., lenght  

9. astore   #index

> 把栈顶元素赋值给索引为 index 的本地变量

JVM Stack变化情况(当前栈):  
before:  ref  
after :    
local_var[#index] = ref

10. astore_<n>

> 把栈顶元素赋值给索引为 n 的本地变量

JVM Stack变化情况(当前栈):  
before:  ref  
after :    
local_var[n] = ref

11. athrow

> 抛出并返回异常，结束当前方法执行，销毁当前栈帧。

JVM Stack变化情况(当前栈):  
before: ..., exceptionref  
after : exceptionref   

```
示例代码
public int add10() {
  int x = 0;
  try {
    x = x + 10;
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    System.out.println("abc");
  }
  return x;
}
```

编译后的字节码

```
local_var[this, x, e(RuntimeException), otherException]
// int x = 0;
 0: iconst_0
 1: istore_1

// x = x + 10; 
 2: iload_1
 3: bipush        10
 5: iadd
 6: istore_1

// （正常语句块执行完后执行）finally 块。 System.out.println("abc"); 
 7: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
10: ldc           #10                 // String abc
12: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally 块结束 
15: goto          45

// RuntimeException 块。 e.printStackTrace()
18: astore_2    // 抛出 RuntimeException 异常，异常对象赋值给本地变量2
// stack[..., runtimeException]
19: aload_2
// stack[...]
20: invokevirtual #13                 // Method java/lang/RuntimeException.printStackTrace:()V

// RuntimeException 异常块结束 
// (RuntimeException执行完后执行)finally 块。 System.out.println("abc"); 
23: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
26: ldc           #10                 // String abc
28: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally 块结束 
31: goto          45

// (抛出非RuntimeException的Exception后执行)finally 块。 System.out.println("abc"); 
34: astore_3  // local_var[otherException] 赋值抛出的异常对象
35: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
38: ldc           #10                 // String abc
40: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally块结束。 stack[..., otherException]
43: aload_3
// stack[otherException]。 执行该指令不会继续执行 45行
44: athrow

// return x；
// stack[..., x]
45: iload_1
// stack[]
46: ireturn

Exception table:
   from    to  target type
     2     7    18   Class java/lang/RuntimeException
     2     7    34   any
    18    23    34   any
```

2~7行代码执行抛出RuntimeException时，先进入18行(参考 Exception table)开始的RuntimeException处理块；RuntimeException处理块结束后进入23行开始的finally块；finally块结束后，在15行跳转到45行；载入本地变量1（即变量x）到操作数栈；弹出操作数栈顶元素并返回。

2~7行代码执行抛出非RuntimeException时，进入34行(参考 Exception table)开始finally块处理；finally块处理结束后，把异常对象入操作数栈；随后抛出异常，结束当前方法执行。

RuntimeException处理块抛出异常时，进入34行(参考 Exception table)开始finally块处理；finally块处理结束后，把异常对象入操作数栈；随后抛出异常，结束当前方法执行。

12. baload

> 从数组中载入字节byte或boolean值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

13. bastore

> 为数组元素赋(byte or boolean)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

14. bipush

> 载入字节byte到操作数栈顶

JVM Stack变化情况(当前栈):  
before: ...,   
after : ..., value   

15. caload

> 从数组中载入字符char值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

16. castore

> 为数组元素赋(char)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

17. checkcast #index

> 检查对象是否为#index指定的类型。 

JVM Stack变化情况(当前栈):  
before: ..., objectref  
after : ..., objectref   

```
Object a; 
String x = (String) a;

Code:
   0: aconst_null
   1: astore_1

   2: aload_1
   3: checkcast     #14                 // class java/lang/String
   6: astore_2
   7: return
```

上述示例代码中，checkcast指令会检查a是否是String类型对象，如果不是系统抛出ClassCastException 异常并结束当前方法执行；如果是String类型对象，继续往下执行6行。

18. d2f

> double值转换为float值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   



19. d2i

> double值转换为int值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

>value is NaN => 0  
>value is infinity => Integer.MAX_VALUE  
>otherwith => IEEE754  

20. d2l

> double值转换为long值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

>value is NaN => 0  
>value is infinity => Long.MAX_VALUE  
>otherwith => IEEE754  

21. dadd

> 两个double值相加

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

value1 and value2都是double。
> value1 or value2 is NaN => NaN
> +infinity + -infinity => NaN
> +infinity + +infinity => +infinity
> -infinity + -infinity => -infinity
> infinity + value => infinity
> +0 + -0 => +0
> +0 + +0 => +0
> -0 + -0 => -0
> 0 + value => value
> +value + -value => +0
> others => IEEE 754

22. daload

> 从数组中载入double值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

23. dastore

> 为数组元素赋(double)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

24. dcmp<op>

> 比较两个 double。分为两个指令 dcmpg, dcmpl

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 > value2 => 1
> value1 < value2 => -1
> value1 = value2 => 0
> value1 or value2 is NaN=> dcmpg return 1, dcmpl return -1



25. dconst_<d>

> 把常量0.0 or 1.0如操作数栈。d取0 或1. dconst_0 => 压入 0.0; dconst_1 => 压入 1.0

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., <d>   

26. ddiv
27. dload
28. dload_<n>
29. dmul
30. dneg
31. drem
32. dreturn
33. dstore
34. dstore_<n>
35. dsub
36. dup
37. dup_x1
38. dup_x2
39. dup2
40. dup2_x1
41. dup2_x2
f2d
f2i
f2l
fadd
faload
fastore
fcmp<op>
fconst_<f>
fdiv
fload
fload_<n>
fmul
fneg
frem
freturn
fstore
fstore_<n>
fsub
getfield
getstatic
goto
goto_w
i2b
i2c
i2d
i2f
i2l
i2s
iadd
iaload
iand
iastore
if_acmp<cond>
if_icmp<cond>
if<cond>
ifnonnull
ifnull
iinc
iload
iload_<n>
imul
ineg
instanceof
invokedynamic
invokeinterface
invokespecial
invokestatic
invokevirtual
ior
irem
ireturn
ishl
ishr
istore
istore_<n>
isub
iushr
ixor
l2d
l2f
l2i
ladd
laload
land
lastore
lcmp
lconst_<l>
ldc
ldc_w
ldc2_w
ldiv
lload
lload_<n>
lmul
lneg
lookupswitch
lor
lrem
lreturn
lshl
lshr
lstore
lstore_<n>
lsub
lushr
lxor
monitorenter
monitorexit
multianewarray
new
newarray
nop
pop
pop2
putfield
putstatic
return
saload
sastore
sipush
swap
tableswitch
wide

来源 
http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html
