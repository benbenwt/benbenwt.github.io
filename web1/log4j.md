[TOC]

# Log4j用法

>要控制输出什么信息(从哪个类)，什么级别（四个常用级别），输出到哪里（文件还是控制台）
>
>即输出级别，输出appender
>
>https://cloud.tencent.com/developer/article/1455713

## 组件

### rootLogger

```
#如下定义了一个rootLogger，rootLogger用于指定输出级别和appender组件。rootLogger表示适用于所有类的日志输出，如果需要指定特定类，可以使用全路径指定
#定义内容为，stdout和myout都要输出INFO级别的日志
log4j.rootLogger=INFO, stdout，myout

#为特定类指定appender输出规则,其中com.wt.sqlpractice.DataFrameLearn为类的全路径
log4j.logger.com.wt.sqlpractice.DataFrameLearn=INFO, stdout，myout
```

### appender

>appender用于指定appender的具体行为，确定输出什么，输出到哪里
>
>如下定义了名为stdout的appender的输出规则，其输出日志到控制台。

```
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.myout=org.apache.log4j.ConsoleAppender
log4j.appender.myout.layout=org.apache.log4j.PatternLayout
log4j.appender.myout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.MyConsole.Threshold=ERROR
```

#### 日志级别

>常用的四个级别 ERROR,WARN,INFO,DEBUG

#### appender类别

```
org.apache.log4j.ConsoleAppender（控制台） 
org.apache.log4j.FileAppender（文件）
org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）
org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件）
org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方）
```

#### layout布局

>org.apache.log4j.HTMLLayout（以HTML表格形式布局）
>
>org.apache.log4j.PatternLayout（可以灵活地指定布局模式）
>
>org.apache.log4j.SimpleLayout（包含日志信息的级别和字符串）
>
>org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等信息）

##### org.apache.log4j.PatternLayout（可以灵活地指定布局模式）

```
#PatternLayout可以使用正则表示如何输出日志
%m 输出代码中指定的消息
%p 输出优先级，即DEBUG，INFO，WARN，ERROR，FATAL  
%r 输出自应用启动到输出该log信息耗费的毫秒数  
%c 输出所属的类目，通常就是所在类的全名  
%t 输出产生该日志事件的线程名  
%n 输出一个回车换行符，Windows平台为“rn”，Unix平台为“n”  
%d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921  
%l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)

#示例，表示输出日志，优先级，类全路径，代码消息，换行
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
```

## 示例

```
log4j.rootLogger=INFO, stdout
log4j.rootLogger=WARN, stdout, logfile

log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n

log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n


#测试
import org.apache.log4j.Logger;

public class DataFrameLearn {
    public static void main(String[] args) {
        Logger logger=Logger.getLogger(DataFrameLearn.class);
        logger.info("haah");
    }
}
```

