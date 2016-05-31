###Java应用中使用日志框架###

##使用SLF4J##
---
Java开发中我们常用log4j日志框架来管理日志操作。而开发中使用![SLF4J](http://www.slf4j.org)(Simple Logging Facade for Java)是更好的选择。

SLF4J允许我们在编码中使用统一的日志api接口，而不依赖于具体的日志框架(如log4j, logback, java.util.logging等)。Java应用部署时可以根据需要自由选择日志框架（在不变动Java应用的前提下）。反之如果我们在开发中使用log4j，那么在部署应用时也只能使用log4j了。

*开发中使用SLF4J*
使用SLF4J开发时，我们只需要依赖`slf4j-api.jar`即可。

```
User.java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class User {
    private Logger logger = LoggerFactory.getLogger(User.class);

    public void login(String username, String password) {
        logger.info("user attempt login:{}", username); // {} 为占位符
    }
}
```

*部署时绑定日志框架*
SLF4J不是日志框架，只是日志操作的`门面`。系统运行时需要绑定具体的日志框架才能正常运行。如果没有绑定具体的日志框架，则会在控制台打印提示信息：
```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder"
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://ww.slf4j.org/codes.html......
```

绑定日志框架很简单，比如我们使用log4j框架，需要把slf4j-log4j[*].jar加到classpath，同时还需要把log4j.jar也加到classpath(否则会抛出java.lang.NoClassDefFoundError: org/apache/log4j/Level)。

如果使用的是Log4j version 2，需要使用log4j-slf4j-impl(代替slf4j-log4j[*].jar)

*绑定好日志框架后，如果使用的是logj，还需要配置log4j.properties并放到classpath*

> SLF4J 提供的绑定jar
> slf4j-log4j12-*.jar       log4j version 1.2
> slf4j-jdk14-*.jar         java.util.logging
> slf4j-nop-*.jar           NOP
> slf4j-simple-*.jar        Simple
> slf4j-jcl-*.jar           Jakarta Commons Logging
> logback-classic-*.jar     logback


###使用Log4j###
---
这里我们主要了解下Log4j的配置文件log4j.properties。Log4j.properties里主要配置logger以及logger的相关信息。Log4j中顶层logger为`rootLogger`，默认的日志级别为 DEBUG，其他自定义logger均继承rootLogger。

*配置rootLogger*
每个logger可以有多个appender，appender负责实际的日志内容操作，如打印到控制台、输出到文件、保存到数据库等。
```
# 缺省日志级别和输出对象(rootLogger定义了两个appender)
log4j.rootLogger = INFO, console, logFile

# 控制台输出模式
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=[%d{MM-dd HH:mm:ss}] %c{1} %m%n

# 日志文件输出模式
log4j.appender.logFile=org.apache.log4j.FileAppender
log4j.appender.logFile.File=app.log
log4j.appender.logFile.layout=org.apache.log4j.PatternLayout
log4j.appender.logFile.layout.ConversionPattern=[%d{MM-dd HH:mm:ss}] %c{1} %m%n
```

*使用自定义logger*
我们还可以自定义logger
```
# 定义了一个名称为errorLogger的logger，使用该logger的日志内容会使用 errorLogFile(appender)
log4j.logger.errorLogger = ERROR, errorLogFile
log4j.additivity.errorLogger = false

log4j.appender.errorLogFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.errorLogFile.File=app_error.log
log4j.appender.errorLogFile.layout=org.apache.log4j.PatternLayout
log4j.appender.errorLogFile.layout.ConversionPattern=[%d{MM-dd HH:mm:ss}] %c{1} %m%n
```

使用自定义logger时需要对Java源代码进行调整：
```
User.java

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class User {
    private Logger logger = LoggerFactory.getLogger(User.class);
    // 使用自定义 logger
    private Logger errLogger = LoggerFactory.getLogger("errorLogger");

    public void login(String username, String password) {
        logger.info("user attempt login:{}", username); // {} 为占位符
        if(username == null) {
            // log content will print to errorLogFile
            errLogger.error("username is null");
        }
    }
}
```

这里我们注意到配置文件中有这么一行内容`log4j.additivity.errorLogger = false`。意思是不集成rootLogger的appender，如果属性值为`true`，则日志内容同样会应用到rootLogger的appender。

*使用package  logger*
我们还可以根据package使用logger
```
### package logger
log4j.logger.com.kopcoder=INFO, kopcoderLogFile
log4j.additivity.com.kopcoder = false

log4j.appender.kopcoderLogFile=org.apache.log4j.DailyRollingFileAppender
log4j.appender.kopcoderLogFile.File=app_kopcoder.log
log4j.appender.kopcoderLogFile.layout=org.apache.log4j.PatternLayout
log4j.appender.kopcoderLogFile.layout.ConversionPattern=[%d{MM-dd HH:mm:ss}] {1}
```

这个logger定义表示，包(com.kopcoder)中的日志操作会使用kopcoderLogFile(明确使用自定义logger的除外)记录日志。

*Log4j的appender*
```
ConsoleAppender           控制台打印日志
FileAppender              把日志写入文件         
    DailyRollingFileAppender  每隔一定时间生成新的日志文件(一般以天为单位，由datePattern设置)
    RollingFileAppender       当日志文件大小达到阈值时生成新的文件
SMTPAppender              把日志以邮件的形式发出
JmsLogAppender            序列化日志内容并发送到JMS Server的一个指定Topic中.
JDBCAppender              把日志写入数据库
AsyncAppender             异步appender。AsyncAppender内部有一个appender，当AsyncAppender中的缓存日志达到阈值时交由这个appender处理日志。
SocketAppender            序列化日志并发送到指定Host的port端口。
SocketHubAppender         类似，将日志内容发往日志服务器。与SocketAppender类似，将日志内容发往日志服务器。支持将日志发往多个日志服务器
...

还可以自定义Appender
```

*Log4j的conversionPattern*

```
%F  发出日志操作所在的文件名
%L  文件行号
%M  方法名
%l  文件名+行号

%m  输出原始信息（原始信息可能带格式化参数）
%n  换行符
%p  日志级别
%t  线程id

// 日期时间相关(%d)
%a  缩写英语星期几
%A  英语星期几
%b  缩写英语月份
%B  英语月份
%c  标准日期+时间格式
%d  当月中的几号
%H  小时(0-23)
%I  小时(1-12)
%j  一年中的第几天
%m  第几月
%M  分钟
%p  上午或下午
%q  毫秒
%Q  带小数毫秒
%S  秒
%U  当年第几周(周日为第一天)
%w  星期几（周日为0）
%W  当年第几周(周一为第一天)
%x  标准日期格式，如：16/31/05
%X  标准时间格式，如: 20:30:00
%y  两位数年份
%Y  四位数年份
%Z  时区

```
