##浅析StackOverFlow##

`StackOverFlow` 在 Java 里是一个Error，当发生 StackOverFlow 错误时说明系统出现了严重问题。Java 应用中出现 Error 时，应用会直接退出（不能 catch Error）。

Java程序员日常开发中很少遇到该类错误。 `StackOverflowError.java` 源代码中明确说明：由于应用的递归调用太深导致栈溢出，从而发生该错误。而Java程序员一般在开发中很少使用递归（JavaScript等函数式语言程序员或更注重代码设计结构高级程序员可能会更喜欢使用递归），一般使用递归的场景用循环也能解决（实际上使用循环也是解决　StackOverflowError 的一种方法）。

我们先回忆下什么是`栈`。`栈`是Java虚拟机中的一种内存结构，用来存放一组`栈帧`，随着线程创建而创建，随着线程的中止而被回收，为创建线程所私有。`栈帧`则是在方法调用时创建，方法执行完毕或异常终止时销毁 `栈帧`。`栈帧`存放本地变量，中间计算结果等。

我们可以使用-Xss设置了JVM的栈大小(stack size)，JDK5以后每个JVM栈大小为1M，以前每个JVM栈大小为256K。当一个线程创建时，创建`栈`；当线程中有方法调用时，创建 `栈帧` 并压入栈；当在方法调用内部有调用其他方法时，创建新的`栈帧`并压入栈。如果有递归方法调用，每递归调用一次就创建一个`栈帧`并压入`栈`，当`栈`的大小超过 -Xss设置的大小时，就会抛出 StackOverFlow 错误。

> 当然我们可以使用可扩展的`栈`，这样就不会出现 StackOverFlow 错误，而是可能出现 OutOfMemory错误。

接下来，我们会分析Java中怎样出现 StackOverFlow 以及避免出现StackOverFlow；还会分析Scala中的尾递归为什么不会出现StackOverFlow，而Scala中的首递归同样会出现StackOverFlow的原因。show me the code:

**Java 递归出现 StackOverFlow Error**

```
public class Rec1 {
    public static void main(String[] args) {
        process(0);
    }

    private static void process(int n) {
        System.out.println(n);
        process(n + 1);
    }
}
```

n大概加到7000左右时会出现 StackOverFlow Error，并退出应用。

分析反编译后的代码：

```
  private static void process(int);
    descriptor: (I)V
    flags: ACC_PRIVATE, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: iload_0
         4: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V

         7: iload_0
         8: iconst_1
         9: iadd

        10: invokestatic  #2                  // Method process:(I)V
        13: return
```
从0～4行完成打印操作后，7～8行实现本地变量0(静态方法的本地变量0即是参数1)与常量1相加，结果替换本地变量0；第10行调用其他方法（递归调用本方法），这个地方会创建`栈帧`。当递归调用一直往下调用（递归调用一般要设置退出递归的条件，防止无限递归），就会出现栈溢出错误。

**Java 循环解决 StackOverFlow Error**

```
public class Rec2 {
    public static void main(String[] args) {
        process(0);
    }

    private static void process(int n) {
        for(;;) {
            System.out.println(n);
            n += 1;
        }
    }
}
```

分析反编译后的代码：

```
  private static void process(int);
    descriptor: (I)V
    flags: ACC_PRIVATE, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: iload_0
         4: invokevirtual #4                  // Method java/io/PrintStream.println:(I)V
         7: iinc          0, 1
        10: goto          0
```

0～4行依然是打印操作；第7行实现本地变量0自增1（安全操作，与i++不同），第10行的地方我们看到使用的是 goto指令，并不是invoke*指令。这个地方没有方法调用，不会创建栈帧，会使用当前栈帧继续执行指令。

**Scala 中的尾递归没有  StackOverFlow Error**

``` Hello.scala
object Hello {
    def main(args: Array[String]): Unit = {
        process(1);
    }

    def process(line: Integer): Integer = {
        println(line);
        if(line > 100000) {
          return line;
        }
        process(line + 1);

    }
}

```

以下是反编译后的代码：

```
Hello.class
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #16                 // Field Hello$.MODULE$:LHello$;
         3: aload_0
         4: invokevirtual #22                 // Method Hello$.main:([Ljava/lang/String;)V
         7: return

Hello$.class
  public void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=2
         0: aload_0
         1: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
         4: iconst_1
         5: invokevirtual #23                 // Method scala/Predef$.int2Integer:(I)Ljava/lang/Integer;
         8: invokevirtual #27                 // Method process:(Ljava/lang/Integer;)Ljava/lang/Integer;
        11: pop
        12: return
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      13     0  this   LHello$;
            0      13     1  args   [Ljava/lang/String;
      LineNumberTable:
        line 3: 0

  public java.lang.Integer process(java.lang.Integer);
    descriptor: (Ljava/lang/Integer;)Ljava/lang/Integer;
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=2
         0: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
         3: aload_1
         4: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V

         7: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        10: aload_1
        11: invokevirtual #38                 // Method scala/Predef$.Integer2int:(Ljava/lang/Integer;)I
        14: ldc           #39                 // int 100000
        16: if_icmple     21

        19: aload_1
        20: areturn

        21: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        24: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        27: aload_1
        28: invokevirtual #38                 // Method scala/Predef$.Integer2int:(Ljava/lang/Integer;)I
        31: iconst_1
        32: iadd
        33: invokevirtual #23                 // Method scala/Predef$.int2Integer:(I)Ljava/lang/Integer;
        36: astore_1

        37: goto          0
```

分析反编译后的代码我们可以发现，`Hello.scala`编译后生成了两个class文件（Hello.class, Hello$.class）。

Hello.class中main方法是入口，持有一个Hello$类型的对象MODULE$。main方法调用MODULE$的process方法。

而Hello$.class中的代码基本反映了Hello.scala源代码。main方法反映了Hello.scala中的main方法（只是此处不是静态方法）。首先会调用Predef$.println打印本地变量1（也就是方法参数1，实例方法的参数0默认是this）；接着if_icmple判断条件；然后把本地变量1和常量1相加，结果替换本地变量1；最后37行的goto指令跳转到0行继续执行（没有调用方法，所以不会创建栈帧，和上面的for循环一样）。

**Scala 中的非尾递归也会有 StackOverFlow Error**

```
object Hello1 {
    def main(args: Array[String]): Unit = {
        process(1);
    }

    def process(line: Integer): Integer = {
        println(line);
        if(line > 100000) {
          return line;
        }
        process(line + 1) + 1;

    }
}

```

默认情况下line加到3000多时出现栈溢出错误。

分析反编译代码：

```
  public java.lang.Integer process(java.lang.Integer);
    descriptor: (Ljava/lang/Integer;)Ljava/lang/Integer;
    flags: ACC_PUBLIC
    Code:
      stack=6, locals=2, args_size=2
         0: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
         3: aload_1
         4: invokevirtual #34                 // Method scala/Predef$.println:(Ljava/lang/Object;)V

         7: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        10: aload_1
        11: invokevirtual #38                 // Method scala/Predef$.Integer2int:(Ljava/lang/Integer;)I
        14: ldc           #39                 // int 100000
        16: if_icmple     21

        19: aload_1
        20: areturn

        21: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        24: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        27: aload_0
        28: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        31: getstatic     #19                 // Field scala/Predef$.MODULE$:Lscala/Predef$;
        34: aload_1
        35: invokevirtual #38                 // Method scala/Predef$.Integer2int:(Ljava/lang/Integer;)I
        38: iconst_1
        39: iadd
        40: invokevirtual #23                 // Method scala/Predef$.int2Integer:(I)Ljava/lang/Integer;
        43: invokevirtual #27                 // Method process:(Ljava/lang/Integer;)Ljava/lang/Integer;

        46: invokevirtual #38                 // Method scala/Predef$.Integer2int:(Ljava/lang/Integer;)I
        49: iconst_1
        50: iadd
        51: invokevirtual #23                 // Method scala/Predef$.int2Integer:(I)Ljava/lang/Integer;
        54: areturn
```        

我们主要关注43行的process递归调用，这里是发生栈溢出的关键。
