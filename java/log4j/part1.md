### 1、 Log4j是什么?

Log4j是Apache的一个开源项目，通过使用Log4j，我们可以控制日志信息输送的目的地是控制台、文件、GUI组件，甚至是套接口服务器、NT的事件记录器、UNIX Syslog守护进程等；我们也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。要下载和了解更详细的内容，还是访问其官方网站吧:http://jakarta.apache.org/log4j

### 2、Log4j的概念

Log4j由三个重要的组成构成：日志记录器(Loggers)，输出端(Appenders)和日志格式化器(Layout)。

1.Logger：控制要启用或禁用哪些日志记录语句，并对日志信息进行级别限制

2.Appenders : 指定了日志将打印到控制台还是文件中

3.Layout : 控制日志信息的显示格式

Log4j中将要输出的Log信息定义了5种级别，依次为DEBUG、INFO、WARN、ERROR和FATAL，当输出时，只有级别高过配置中规定的 级别的信息才能真正的输出，这样就很方便的来配置不同情况下要输出的内容，而不需要更改代码，这点实在是方便啊。

#### A）.Logger对象的获得或创建

Logger被指定为实体，由一个String类的名字识别。Logger的名字是大小写敏感的，且名字之间具有继承关系，子名用父名作为前缀，用点“.”分隔，例如x.y是x.y.z的父亲。

root Logger(根Logger)是所有Logger的祖先，它有如下属性：


1.它总是存在的。
2.它不可以通过名字获得。

root Logger可以通过以下语句获得：
```
public static Logger Logger.getRootLogger();
或
public static Logger Logger.getLogger(Class clazz)
```
其中调用Logger.getLogger(Class clazz)是目前Logger对象最理想的方法。

#### B）日志级别

每个Logger都被了一个日志级别（log level），用来控制日志信息的输出。日志级别从高到低分为：

- A：off         最高等级，用于关闭所有日志记录。
- B：fatal       指出每个严重的错误事件将会导致应用程序的退出。
- C：error      指出虽然发生错误事件，但仍然不影响系统的继续运行。
- D：warm     表明会出现潜在的错误情形。
- E：info         一般和在粗粒度级别上，强调应用程序的运行全程。
- F：debug     一般用于细粒度级别上，对调试应用程序非常有帮助。
- G：all           最低等级，用于打开所有日志记录。

上面这些级别是定义在org.apache.log4j.Level类中,Log4j只建议使用4个级别，优先级从高到低分别是:

- 1.error
- 2.warn
- 3.info
- 4.debug

通过使用日志级别，可以控制应用程序中相应级别日志信息的输出。例如，如果使用了info级别，则应用程序中所有低于info级别的日志信息(如debug)将不会被打印出来。
```
package log4j;
 
import org.apache.log4j.BasicConfigurator;
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
 
public class Log4jTest {
 
    public static void main(String[] args) {
        
        Logger logger = Logger.getLogger(Log4jTest.class);
        
        //使用默认的配置信息，不需要写log4j.properties
        BasicConfigurator.configure();
        //设置日志输出级别为info，这将覆盖配置文件中设置的级别
        logger.setLevel(Level.INFO);
        //下面的消息将被输出
        logger.info("this is an info");
        logger.warn("this is a warn");
        logger.error("this is an error");
        logger.fatal("this is a fatal");
    } 
}

```

#### C）输出端Appender

Appender用来指定日志信息输出到哪个地方，可以同时指定多个输出目的地。Log4j允许将信息输出到许多不同的输出设备中，一个log信息输出目的地就叫做一个Appender。


每个Logger都可以拥有一个或多个Appender，每个Appender表示一个日志的输出目的地。可以使用Logger.addAppender(Appender app)为Logger增加一个Appender，也可以使用Logger.removeAppender(Appender app)为Logger删除一个Appender。


以下为Log4j几种常用的输出目的地。

- a：org.apache.log4j.ConsoleAppender：将日志信息输出到控制台。
- b：org.apache.log4j.FileAppender：将日志信息输出到一个文件。
- c：org.apache.log4j.DailyRollingFileAppender：将日志信息输出到一个日志文件，并且每天输出到一个新的日志文件。
- d：org.apache.log4j.RollingFileAppender：将日志信息输出到一个日志文件，并且指定文件的尺寸，当文件大小达到指定尺寸时，会自动把文件改名，同时产生一个新的文件。
- e：org.apache.log4j.WriteAppender：将日志信息以流格式发送到任意指定地方。
- f：org.apache.log4j.jdbc.JDBCAppender：通过JDBC把日志信息输出到数据库中。

#### D）日志格式化器Layout

有四种：

1.HTMLLayout : 格式化日志输出为HTML表格形式如下

2.SimpleLayout : 以一种非常简单的方式格式化日志输出，它打印三项内容：级别-信息

例：INFO - info

3.PatternLayout : 根据指定的转换模式格式化日志输出，或者如果没有指定任何转换模式，就使用默认的转化模式格式。

下面的代码实现了SimpleLayout和FileAppender的程序
```
public static void main(String[] args) {
        
        Logger logger = Logger.getLogger(Log4jTest.class);        
        SimpleLayout layout = new SimpleLayout();
        //HTMLLayout  layout = new HTMLLayout();
        FileAppender appender = null;
        try
        {
            //把输出端配置到out.txt
            appender = new FileAppender(layout,"out.txt",false);
        }catch(Exception e)
        {            
        }
        logger.addAppender(appender);//添加输出端
        logger.setLevel((Level)Level.DEBUG);//覆盖配置文件中的级别
        logger.debug("debug");
        logger.info("info");
        logger.warn("warn");
        logger.error("error");
        logger.fatal("fatal");
    }

```

4.TTCCLayout : 包含日志产生的时间、线程、类别等等信息

### 3、Log4j的配置

配置Log4j环境就是指配置root Logger，包括把Logger为哪个级别，为它增加哪些Appender,以及为这些Appender设置Layout，等等。因为所有其他的Logger都是root Logger的后代，所以它们都继承了root Logger的性质。这些可以通过设置系统属性的方法来隐式地完成，也可以在程序中调用XXXConfigurator.configure()方法来显式地完成。有以下几种方式来配置Log4j。

- A：配置放在文件里，通过环境变量传递文件名等信息，利用Log4j默认的初始化过程解析并配置。
- B：配置放在文件里，通过应用服务器配置传递文件甸等信息，利用一个特定的Servlet来完成配置。
- C：在程序中调用BasicConfigor.configure()方法。
- D：配置放在文件里，通过命令行PropertyConfigurator.configure(args[])解析log4j.properties文件并配置Log4j。

下面对BasicConfigurator.configure()方法和PropertyConfigurator.config()方法分别进行介绍。

BasicConfigurator.configure()方法：

它使用简单的方法配置Log4j环境。这个方法完成的任务是：

1：用默认的方式创建PatternLayout对象p:

> PatternLayout p = new PatternLayout("%-4r[%t]%-5p%c%x-%m%n");

2：用p创建ConsoleAppender对象a，目标是System.out，标准输出设备：

> ConsoleAppender a = new CpnsoleAppender(p,ConsoleAppender.SYSTEM_OUT);

3：为root Logger增加一个ConsoleAppender p;

> rootLogger.addAppender(p);

4：把rootLogger的log level设置为DUBUG级别;

> rootLogger.setLevel(Level.DEBUG);
PropertyConfigurator.configure()方法：

当使用以下语句生成Logger对象时：

```
static Logger logger = Logger.getLogger(mycalss.class);
```

如果没有调用BasicConfigurator.configure()，PropertyConfigurator.configure()或DOMConfigurator.configure()方法，Log4j会自动加载CLASSPATH下名为log4j.properties的配置文件。如果把此配置文件改为其他名字，例如my.properties,程序虽然仍能运行，但会报出不能正确初始化Log4j系统的提示。这时可以在程序中加上：

> PropertyConfigurator.configure("classes/my.properties");

### 4、Log4j的log4j.properties配置详解

虽然可以不用配置文件，而在程序中实现配置，但这种方法在如今的系统开发中显然是不可取的，能采用配置文件的地方一定一定要用配置文件。Log4j支持两 种格式的配置文件：XML格式和Java的property格式。

```
################################################################################ 
#①配置根Logger，其语法为： 
# 
#log4j.rootLogger = [level],appenderName,appenderName2,... 
#level是日志记录的优先级，分为OFF,TRACE,DEBUG,INFO,WARN,ERROR,FATAL,ALL 
##Log4j建议只使用四个级别，优先级从低到高分别是DEBUG,INFO,WARN,ERROR 
#通过在这里定义的级别，您可以控制到应用程序中相应级别的日志信息的开关 
#比如在这里定义了INFO级别，则应用程序中所有DEBUG级别的日志信息将不被打印出来 
#appenderName就是指定日志信息输出到哪个地方。可同时指定多个输出目的 
################################################################################ 
################################################################################ 
#②配置日志信息输出目的地Appender，其语法为： 
# 
#log4j.appender.appenderName = fully.qualified.name.of.appender.class 
#log4j.appender.appenderName.optionN = valueN 
# 
#Log4j提供的appender有以下几种： 
#1)org.apache.log4j.ConsoleAppender(输出到控制台) 
#2)org.apache.log4j.FileAppender(输出到文件) 
#3)org.apache.log4j.DailyRollingFileAppender(每天产生一个日志文件) 
#4)org.apache.log4j.RollingFileAppender(文件大小到达指定尺寸的时候产生一个新的文件) 
#5)org.apache.log4j.WriterAppender(将日志信息以流格式发送到任意指定的地方) 
# 
#1)ConsoleAppender选项属性 
# -Threshold = DEBUG:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -Target = System.err:默认值System.out,输出到控制台(err为红色,out为黑色) 
# 
#2)FileAppender选项属性 
# -Threshold = INFO:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:\log4j.log:指定消息输出到C:\log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -Encoding = UTF-8:可以指定文件编码格式 
# 
#3)DailyRollingFileAppender选项属性 
# -Threshold = WARN:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:\log4j.log:指定消息输出到C:\log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -DatePattern='.'yyyy-ww:每周滚动一次文件,即每周产生一个新的文件。还可以按用以下参数: 
#              '.'yyyy-MM:每月 
#              '.'yyyy-ww:每周 
#              '.'yyyy-MM-dd:每天 
#              '.'yyyy-MM-dd-a:每天两次 
#              '.'yyyy-MM-dd-HH:每小时 
#              '.'yyyy-MM-dd-HH-mm:每分钟 
# -Encoding = UTF-8:可以指定文件编码格式 
# 
#4)RollingFileAppender选项属性 
# -Threshold = ERROR:指定日志消息的输出最低层次 
# -ImmediateFlush = TRUE:默认值是true,所有的消息都会被立即输出 
# -File = C:/log4j.log:指定消息输出到C:/log4j.log文件 
# -Append = FALSE:默认值true,将消息追加到指定文件中，false指将消息覆盖指定的文件内容 
# -MaxFileSize = 100KB:后缀可以是KB,MB,GB.在日志文件到达该大小时,将会自动滚动.如:log4j.log.1 
# -MaxBackupIndex = 2:指定可以产生的滚动文件的最大数 
# -Encoding = UTF-8:可以指定文件编码格式 
################################################################################ 
################################################################################ 
#③配置日志信息的格式(布局)，其语法为： 
# 
#log4j.appender.appenderName.layout = fully.qualified.name.of.layout.class 
#log4j.appender.appenderName.layout.optionN = valueN 
# 
#Log4j提供的layout有以下几种： 
#5)org.apache.log4j.HTMLLayout(以HTML表格形式布局) 
#6)org.apache.log4j.PatternLayout(可以灵活地指定布局模式) 
#7)org.apache.log4j.SimpleLayout(包含日志信息的级别和信息字符串) 
#8)org.apache.log4j.TTCCLayout(包含日志产生的时间、线程、类别等等信息) 
#9)org.apache.log4j.xml.XMLLayout(以XML形式布局) 
# 
#5)HTMLLayout选项属性 
# -LocationInfo = TRUE:默认值false,输出java文件名称和行号 
# -Title=Struts Log Message:默认值 Log4J Log Messages 
# 
#6)PatternLayout选项属性 
# -ConversionPattern = %m%n:格式化指定的消息(参数意思下面有) 
# 
#9)XMLLayout选项属性 
# -LocationInfo = TRUE:默认值false,输出java文件名称和行号 
# 
#Log4J采用类似C语言中的printf函数的打印格式格式化日志信息，打印参数如下： 
# %m 输出代码中指定的消息 
# %p 输出优先级，即DEBUG,INFO,WARN,ERROR,FATAL 
# %r 输出自应用启动到输出该log信息耗费的毫秒数 
# %c 输出所属的类目,通常就是所在类的全名 
# %t 输出产生该日志事件的线程名 
# %n 输出一个回车换行符，Windows平台为“\r\n”，Unix平台为“\n” 
# %d 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式 
#    如：%d{yyyy年MM月dd日 HH:mm:ss,SSS}，输出类似：2012年01月05日 22:10:28,921 
# %l 输出日志事件的发生位置，包括类目名、发生的线程，以及在代码中的行数 
#    如：Testlog.main(TestLog.java:10) 
# %F 输出日志消息产生时所在的文件名称 
# %L 输出代码中的行号 
# %x 输出和当前线程相关联的NDC(嵌套诊断环境),像java servlets多客户多线程的应用中 
# %% 输出一个"%"字符 
# 
# 可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如： 
#  %5c: 输出category名称，最小宽度是5，category<5，默认的情况下右对齐 
#  %-5c:输出category名称，最小宽度是5，category<5，"-"号指定左对齐,会有空格 
#  %.5c:输出category名称，最大宽度是5，category>5，就会将左边多出的字符截掉，<5不会有空格 
#  %20.30c:category名称<20补空格，并且右对齐，>30字符，就从左边交远销出的字符截掉 
################################################################################ 
################################################################################ 
#④指定特定包的输出特定的级别 
#log4j.logger.org.springframework=DEBUG 
################################################################################ 
 
#OFF,systemOut,logFile,logDailyFile,logRollingFile,logMail,logDB,ALL 
log4j.rootLogger =ALL,systemOut,logFile,logDailyFile,logRollingFile,logMail,logDB 
 
#输出到控制台 
log4j.appender.systemOut = org.apache.log4j.ConsoleAppender 
log4j.appender.systemOut.layout = org.apache.log4j.PatternLayout 
log4j.appender.systemOut.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.systemOut.Threshold = DEBUG 
log4j.appender.systemOut.ImmediateFlush = TRUE 
log4j.appender.systemOut.Target = System.out 
 
#输出到文件 
log4j.appender.logFile = org.apache.log4j.FileAppender 
log4j.appender.logFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logFile.Threshold = DEBUG 
log4j.appender.logFile.ImmediateFlush = TRUE 
log4j.appender.logFile.Append = TRUE 
log4j.appender.logFile.File = ../Struts2/WebRoot/log/File/log4j_Struts.log 
log4j.appender.logFile.Encoding = UTF-8 
 
#按DatePattern输出到文件 
log4j.appender.logDailyFile = org.apache.log4j.DailyRollingFileAppender 
log4j.appender.logDailyFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logDailyFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logDailyFile.Threshold = DEBUG 
log4j.appender.logDailyFile.ImmediateFlush = TRUE 
log4j.appender.logDailyFile.Append = TRUE 
log4j.appender.logDailyFile.File = ../Struts2/WebRoot/log/DailyFile/log4j_Struts 
log4j.appender.logDailyFile.DatePattern = '.'yyyy-MM-dd-HH-mm'.log' 
log4j.appender.logDailyFile.Encoding = UTF-8 
 
#设定文件大小输出到文件 
log4j.appender.logRollingFile = org.apache.log4j.RollingFileAppender 
log4j.appender.logRollingFile.layout = org.apache.log4j.PatternLayout 
log4j.appender.logRollingFile.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
log4j.appender.logRollingFile.Threshold = DEBUG 
log4j.appender.logRollingFile.ImmediateFlush = TRUE 
log4j.appender.logRollingFile.Append = TRUE 
log4j.appender.logRollingFile.File = ../Struts2/WebRoot/log/RollingFile/log4j_Struts.log 
log4j.appender.logRollingFile.MaxFileSize = 1MB 
log4j.appender.logRollingFile.MaxBackupIndex = 10 
log4j.appender.logRollingFile.Encoding = UTF-8 
 
#用Email发送日志 
log4j.appender.logMail = org.apache.log4j.net.SMTPAppender 
log4j.appender.logMail.layout = org.apache.log4j.HTMLLayout 
log4j.appender.logMail.layout.LocationInfo = TRUE 
log4j.appender.logMail.layout.Title = Struts2 Mail LogFile 
log4j.appender.logMail.Threshold = DEBUG 
log4j.appender.logMail.SMTPDebug = FALSE 
log4j.appender.logMail.SMTPHost = SMTP.163.com 
log4j.appender.logMail.From = xly3000@163.com 
log4j.appender.logMail.To = xly3000@gmail.com 
#log4j.appender.logMail.Cc = xly3000@gmail.com 
#log4j.appender.logMail.Bcc = xly3000@gmail.com 
log4j.appender.logMail.SMTPUsername = xly3000 
log4j.appender.logMail.SMTPPassword = 1234567 
log4j.appender.logMail.Subject = Log4j Log Messages 
#log4j.appender.logMail.BufferSize = 1024 
#log4j.appender.logMail.SMTPAuth = TRUE 
 
#将日志登录到MySQL数据库 
log4j.appender.logDB = org.apache.log4j.jdbc.JDBCAppender 
log4j.appender.logDB.layout = org.apache.log4j.PatternLayout 
log4j.appender.logDB.Driver = com.mysql.jdbc.Driver 
log4j.appender.logDB.URL = jdbc:mysql://127.0.0.1:3306/xly 
log4j.appender.logDB.User = root 
log4j.appender.logDB.Password = 123456 
log4j.appender.logDB.Sql = INSERT INTOT_log4j(project_name,create_date,level,category,file_name,thread_name,line,all_category,message)values('Struts2','%d{yyyy-MM-ddHH:mm:ss}','%p','%c','%F','%t','%L','%l','%m')

```
















