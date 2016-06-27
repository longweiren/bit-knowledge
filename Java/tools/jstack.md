###使用jstack分析Java线程运行情况###

*jstack用来分析JVM运行实例的线程情况，包括线程的锁信息*

> jstat [option] vmid
> 
> option可选值:  
> -l  打印附加信息，比如锁的信息。  
> -m  打印JVM和native frames的信息(一般不包含native信息)。  
> -F  强制thread dump.当使用 jstack vmid没有响应时使用-F
