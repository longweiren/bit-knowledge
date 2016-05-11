### Java ByteCode Instruction CheetSheet ###

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

6. anewarray
areturn
arraylength
astore
astore_<n>
athrow
baload
bastore
bipush
caload
castore
checkcast
d2f
d2i
d2l
dadd
daload
dastore
dcmp<op>
dconst_<d>
ddiv
dload
dload_<n>
dmul
dneg
drem
dreturn
dstore
dstore_<n>
dsub
dup
dup_x1
dup_x2
dup2
dup2_x1
dup2_x2
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
