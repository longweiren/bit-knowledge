###Java中的 `+`###

`+` 是 Java 中再寻常不过的操作符了，按理没什么可说的。但仔细想想还是有些耐人寻味的东西，简单总结一下。

**局部变量中的 `+`**

> （代码1）
> public void process() {
>   int a = 0;
>   a = a + 1;  // (1)
>   a += 1;     // (2)
>   a++;        // (3)
> }

代码1中，(1), (2), (3)都实现对变量 a 的加1操作，但是细节方面有细微差异。我们看下编译后的代码：

```
  public void process();
    Code:
       // int a = 0;
       0: iconst_0
       1: istore_1

       // a = a + 1;
       2: iload_1
       3: iconst_1
       4: iadd
       5: istore_1

       // a += 1;
       6: iinc          1, 1

       // a++;
       9: iinc          1, 1
      12: return
```

分析反编译代码我们可以发现， `a += 1` 和 `a++` 表达式效果完全一致，在这里都是一条指令实现对本地变量1（也就是变量a，本地变量0为this）实现加1操作。

而 `a = a + 1` 与另外两个表达式有较大差异，是通过4条指令实现相同功能。先把本地变量1加载到操作数栈；再加载常量1到操作数栈；取出栈顶两个元素相加，结果压入操作数栈；取出栈顶元素存入本地变量1。

所以实际编码中用 `a += 1` 要优于 `a = a + 1`，更少的指令可以减少程序执行时间。

经常有文章说 `i++` 不是安全操作，需要先把i加入本地缓存，再 +1，最后把结果赋值给i，需要多步操作。而这里只有1条指令，问题出在？我们看下面例子就明白了。

**实例变量中的`+`**

> （代码2）
> public class Plus {
>   private int a = 0;
>   public void process() {
>       a = a + 1;  // (1)
>       a += 1;     // (2)
>       a++;        // (3)
>   }
> }

编译后的字节码

```
  public void process();
    Code:
        // a = a + 1;
       // stack[this]
       0: aload_0
       // stack[this, this]
       1: aload_0
       // stack[this, a]
       2: getfield      #2                  // Field a:I
       // stack[this, a, 1]
       5: iconst_1
       // stack[this, a + 1]
       6: iadd
       // stack[]  this.a = a + 1
       7: putfield      #2                  // Field a:I

       // a += 1;
      // stack[this]
      10: aload_0
      // stack[this, this]
      11: dup
      // stack[this, a]
      12: getfield      #2                  // Field a:I
      // stack[this, a, 1]
      15: iconst_1
      // stack[this, a + 1]
      16: iadd
      // stack[] this.a = a + 1
      17: putfield      #2                  // Field a:I

      // a++;
      20: aload_0
      21: dup
      22: getfield      #2                  // Field a:I
      25: iconst_1
      26: iadd
      27: putfield      #2                  // Field a:I
      30: return
```

变量`a`更改为实例变量后，同样的3个加法操作，实际底层操作完全不一样。这里，源代码(1), (2) 和 (3)编译后的代码基本一致。

细微差别是一处指令是 `aload_0`， 其他两处是 `dup`。`aload_0` 指令把本地变量0(也就是this)载入操作数栈， `dup`是复制栈顶元素再载入操作数栈顶（因为这里 `dup`指令之前已执行 `aload_0`指令把 this 压入操作数栈顶，所以 `dup`指令会复制 this 并压入操作数栈顶），在这里效果完全一样。

这里也可以发现实例变量的 a++ 操作确实是使用了6条指令完成功能，也就回答了第一小节中的疑问。实际上，局部变量只会在线程独有的JVM栈中操作，不会和其他线程共享，所以操作是安全的（反映在字节码中就是一条指令）；而实例变量是多线程共享的，值存储在实例所在的堆里，对堆里实例变量的操作，需要先从堆加载到线程内部的JVM栈，借助JVM栈完成操作后，再回写到堆中（反映到字节码中就是多条指令）。

所以对实例变量执行 ++ 操作时需要注意是否需要加锁同步操作。

可以发现，实例变量的加法表达式，上面三种方式效果均一样，可按个人口味使用。

**字符串连接操作**

> public void process() {
>   String str = "abc" + "123"; (1)
>   str = str + "!";            (2)
> }

简单的两行代码，编译后的代码如下：

```
  public void process2();
    Code:
       0: ldc           #3                  // String abc123
       2: astore_1

       3: new           #4                  // class java/lang/StringBuilder
       6: dup
       7: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
      10: aload_1
      11: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      14: ldc           #7                  // String !
      16: invokevirtual #6                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      19: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      22: astore_1
      23: return
```

代码（1）编译后成为两条指令，`ldc` 指令 从运行时常量池加载字符串 abc123 到操作数栈，`astore_1` 指令把栈顶元素存入本地变量1（str）。我们可以发现源代码中的 `+` 运算符在这里并没有反映，其实这是被javac编译时优化了。

而代码（2）编译后成了9条指令，内容丰富。  
1. new 指令创建了一个 StringBuilder 实例对象。 stack[stringBuilderRef]
2. dup 指令复制一个对象。stack[stringBuilderRef, stringBuilderRef]
3. invokespecial 指令调用 StringBuilder 实例对象的初始化方法 init （默认构造函数）。stack[stringBuilderRef]
4. aload_1 指令载入本地变量1（str）。stack[stringBuilderRef, abc123]
5. invokevirtual 指令调用 StringBuilder 实例对象的实例方法 append。 stack[], stringBuilderRef.append(str)
6. ldc 指令 载入字符串常量 `~`
7. invokevirtual 指令调用 StringBuilder 实例对象的实例方法 append。 stack[], stringBuilderRef.append(abc123).append(!)
8. invokevirtual 指令调用 StringBuilder 实例对象的实例方法 toString，返回值压入操作数栈。 stack[stringBuilderRef.append(abc123).append(!).toString] => stack[abc123!]
9. astore_1 指令把栈顶元素值存入本地变量1(str)。 str  = 'abc123!'


