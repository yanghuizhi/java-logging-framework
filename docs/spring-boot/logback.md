# Spring Boot + Logback

## Quick Start

在SpringBoot从1.4版本开始，内置的日志框架就是Logback,所以我们创建的是1.4+的Spring Boot项目无需引入Logback依赖，直接使用日志框架。



Link [官方文档](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#boot-features-logback-extensions)


## Logback介绍

Logback是由log4j创始人设计的一个开源日志组件。LogBack被分为3个组件，logback-core, logback-classic 和 logback-access。


```
1. logback-core:提供了LogBack的核心功能，是另外两个组件的基础。
2. logback-classic:实现了Slf4j的API，所以当想配合Slf4j使用时，需要引入logback-classic。
3. logback-access:为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。
```

一般来说我们使用Logback是基于Slf4j接口进行使用。

![slf4j-logger-factory](https://raw.githubusercontent.com/BoomManPro/java-logging-framework/master/docs/stastic/images/slf4j-logger-factory.png)


工具 | 官方网站
-|-|
SLF4J | 	[http://www.slf4j.org](http://www.slf4j.org) | 
logback | [http://logback.qos.ch](http://logback.qos.ch) | 


Slf4j：简单日志门面(Simple Logging Facade for Java)，不是具体的日志解决方案，它只服务于各种各样的日志系统。
在使用SLF4J的时候，不需要在代码中或配置文件中指定你打算使用那个具体的日志系统。


## 为什么使用logback

logback具有以下优点：

内核重写、测试充分、初始化内存加载更小，这一切让logback性能和log4j相比有诸多倍的提升
logback非常自然地直接实现了slf4j，这个严格来说算不上优点，只是这样，再理解slf4j的前提下会很容易理解logback，也同时很容易用其他日志框架替换logbac
logback有比较齐全的200多页的文档
logback当配置文件修改了，支持自动重新加载配置文件，扫描过程快且安全，它并不需要另外创建一个扫描线程
支持自动去除旧的日志文件，可以控制已经产生日志文件的最大数量
总而言之，如果大家的项目里面需要选择一个日志框架，那么我个人非常建议使用logback。


## 非Spring Boot 使用步骤

### 引入slf4j、logback相关依赖

```pom
    <!-- slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>

    <!-- logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>${logback.version}</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>${logback.version}</version>
    </dependency>

    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-access</artifactId>
        <version>${logback.version}</version>
    </dependency>
```

### 添加配置文件logback.xm

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- 定义日志文件的存储地址 -->
    <!--
        关于catalina.base解释如下：
            catalina.home指向公用信息的位置，就是bin和lib的父目录。
            catalina.base指向每个Tomcat目录私有信息的位置，就是conf、logs、temp、webapps和work的父目录。
    -->
    <property name="LOG_DIR" value="${catalina.base}/logs"/>

    <!--
        %p:输出优先级，即DEBUG,INFO,WARN,ERROR,FATAL
        %r:输出自应用启动到输出该日志讯息所耗费的毫秒数
        %t:输出产生该日志事件的线程名
        %f:输出日志讯息所属的类别的类别名
        %c:输出日志讯息所属的类的全名
        %d:输出日志时间点的日期或时间，指定格式的方式： %d{yyyy-MM-dd HH:mm:ss}
        %l:输出日志事件的发生位置，即输出日志讯息的语句在他所在类别的第几行。
        %m:输出代码中指定的讯息，如log(message)中的message
        %n:输出一个换行符号
    -->
    <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度 %msg：日志消息，%n是换行符-->
    <property name="pattern" value="%d{yyyyMMdd:HH:mm:ss.SSS} [%thread] %-5level  %msg%n"/>


    <!--
        Appender: 设置日志信息的去向,常用的有以下几个
            ch.qos.logback.core.ConsoleAppender (控制台)
            ch.qos.logback.core.rolling.RollingFileAppender (文件大小到达指定尺寸的时候产生一个新文件)
            ch.qos.logback.core.FileAppender (文件)
    -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 字符串System.out（默认）或者System.err -->
        <target>System.out</target>
        <!-- 对记录事件进行格式化 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
    </appender>

    <appender name="SQL_INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建 -->
        <file>${LOG_DIR}/sql_info.log</file>
        <!-- 当发生滚动时，决定RollingFileAppender的行为，涉及文件移动和重命名。属性class定义具体的滚动策略类 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 必要节点，包含文件名及"%d"转换符，"%d"可以包含一个java.text.SimpleDateFormat指定的时间格式，默认格式是 yyyy-MM-dd -->
            <fileNamePattern>${LOG_DIR}/sql_info_%d{yyyy-MM-dd}.log.%i.gz</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 可选节点，控制保留的归档文件的最大数量，超出数量就删除旧文件。假设设置每个月滚动，如果是6，则只保存最近6个月的文件，删除之前的旧文件 -->
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
        <!-- LevelFilter： 级别过滤器，根据日志级别进行过滤 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <!-- 用于配置符合过滤条件的操作 ACCEPT：日志会被立即处理，不再经过剩余过滤器 -->
            <onMatch>ACCEPT</onMatch>
            <!-- 用于配置不符合过滤条件的操作 DENY：日志将立即被抛弃不再经过其他过滤器 -->
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="SQL_ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_DIR}/sql_error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_DIR}/sql_error_%d{yyyy-MM-dd}.log.%i.gz</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>20MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>10</maxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>${pattern}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="APP_INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_DIR}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_DIR}/info.%d{yyyy-MM-dd}.log
            </FileNamePattern>
        </rollingPolicy>
        <encoder>
            <Pattern>[%date{yyyy-MM-dd HH:mm:ss}] [%-5level] [%thread] [%logger:%line]--%mdc{client} %msg%n</Pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="ch.qos.logback.classic.html.HTMLLayout">
                <property name="pattern" value="%d{yyyyMMdd:HH:mm:ss.SSS} [%thread] %-5level  %msg%n"/>
                <pattern>%d{yyyyMMdd:HH:mm:ss.SSS}%thread%-5level%F{32}%M%L%msg</pattern>
            </layout>
        </encoder>
        <file>${LOG_DIR}/test.html</file>
    </appender>


    <!--
        用来设置某一个包或者具体的某一个类的日志打印级别、以及指定<appender>。
        <loger>仅有一个name属性，一个可选的level和一个可选的addtivity属性
        name:
            用来指定受此logger约束的某一个包或者具体的某一个类。
        level:
            用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
            如果未设置此属性，那么当前logger将会继承上级的级别。
        additivity:
            是否向上级loger传递打印信息。默认是true。
        <logger>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger
    -->
    <logger name="java.sql" level="info" additivity="false">
        <level value="info" />
        <appender-ref ref="STDOUT"></appender-ref>
        <appender-ref ref="SQL_INFO"></appender-ref>
        <appender-ref ref="SQL_ERROR"></appender-ref>
    </logger>

    <logger name="com.souche.LogbackTest" additivity="false">
        <level value="info" />
        <appender-ref ref="STDOUT" />
        <appender-ref ref="APP_INFO" />
        <appender-ref ref="FILE"/>
    </logger>


    <!--
        也是<logger>元素，但是它是根logger。默认debug
        level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
        <root>可以包含零个或多个<appender-ref>元素，标识这个appender将会添加到这个logger。
    -->
    <root level="info">
        <level>info</level>
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="SQL_INFO"/>
        <appender-ref ref="SQL_ERROR"/>
        <appender-ref ref="FILE"/>
    </root>

</configuration>
```

### logback对配置文件的加载

```java
1. getSingleton()方法获取logback实例对象，说明在对象之前已经加载了相关的配置文件，跟进 StaticLoggerBinder
    static {
        // 初始化
        SINGLETON.init();
    }

    private boolean initialized = false;
    private LoggerContext defaultLoggerContext = new LoggerContext();
    private final ContextSelectorStaticBinder contextSelectorBinder = ContextSelectorStaticBinder.getSingleton();

    private StaticLoggerBinder() {
        defaultLoggerContext.setName(CoreConstants.DEFAULT_CONTEXT_NAME);
    }

    public static StaticLoggerBinder getSingleton() {
        return SINGLETON;
    }
    
2. 查看 init()
    /**
     * Package access for testing purposes.
     */
    void init() {
        try {
            try {
                // 上下文初始化环境 
                new ContextInitializer(defaultLoggerContext).autoConfig();
            } catch (JoranException je) {
                Util.report("Failed to auto configure default logger context", je);
            }
            // logback-292
            if (!StatusUtil.contextHasStatusListener(defaultLoggerContext)) {
                StatusPrinter.printInCaseOfErrorsOrWarnings(defaultLoggerContext);
            }
            contextSelectorBinder.init(defaultLoggerContext, KEY);
            initialized = true;
        } catch (Throwable t) {
            // we should never get here
            Util.report("Failed to instantiate [" + LoggerContext.class.getName() + "]", t);
        }
    }
    
3. 跟进autoConfig()
    public void autoConfig() throws JoranException {
        StatusListenerConfigHelper.installIfAsked(loggerContext);
        // 寻找默认配置文件
        URL url = findURLOfDefaultConfigurationFile(true);
        if (url != null) {
            configureByResource(url);
        } else {
            Configurator c = EnvUtil.loadFromServiceLoader(Configurator.class);
            if (c != null) {
                try {
                    c.setContext(loggerContext);
                    c.configure(loggerContext);
                } catch (Exception e) {
                    throw new LogbackException(String.format("Failed to initialize Configurator: %s using ServiceLoader", c != null ? c.getClass()
                                    .getCanonicalName() : "null"), e);
                }
            } else {
                // 没有找到配置文件，则使用默认的配置器，那么日志只会打印在控制台
                BasicConfigurator basicConfigurator = new BasicConfigurator();
                basicConfigurator.setContext(loggerContext);
                basicConfigurator.configure(loggerContext);
            }
        }
    }
    
4. findURLOfDefaultConfigurationFile() logback配置文件加载规则
    public URL findURLOfDefaultConfigurationFile(boolean updateStatus) {
        // 获取当前实例的类加载器，目的是在classpath下寻找配置文件
        ClassLoader myClassLoader = Loader.getClassLoaderOfObject(this);
        // 先找logback.configurationFile文件
        URL url = findConfigFileURLFromSystemProperties(myClassLoader, updateStatus);
        if (url != null) {
            return url;
        }
        // logback.configurationFile文件没找到，再找logback.groovy
        url = getResource(GROOVY_AUTOCONFIG_FILE, myClassLoader, updateStatus);
        if (url != null) {
            return url;
        }
        // logback.groovy没找到，再找logback-test.xml
        url = getResource(TEST_AUTOCONFIG_FILE, myClassLoader, updateStatus);
        if (url != null) {
            return url;
        }
        // logback-test.xml没找到，最后找logback.xml
        return getResource(AUTOCONFIG_FILE, myClassLoader, updateStatus);
    }
```

小结：
编译期间，完成slf4j的绑定已经logback配置文件的加载。slf4j会在classpath中寻找org/slf4j/impl/StaticLoggerBinder.class(会在具体的日志框架如log4j、logback等中存在)，找到并完成绑定；同时，logback也会在classpath中寻找配置文件，先找logback.configurationFile、没有则找logback.groovy，若logback.groovy也没有，则找logback-test.xml，若logback-test.xml还是没有，则找logback.xml，若连logback.xml也没有，那么说明没有配置logback的配置文件，那么logback则会启用默认的配置(日志信息只会打印在控制台)。



## Automatically reloading configuration file upon modification
[Automatically reloading configuration file upon modification](https://logback.qos.ch/manual/configuration.html#autoScan)

热部署  有点类似于自动编译吧  然后部署了以后不用在你修改过文件后重新加载所有的内容，包括必须重新加载和非必须重新加载的 如第三方的jar包

```xml
<configuration  scan="true" scanPeriod="30 seconds">  
....  
</configuration>   
```

### 相关链接

logback最佳实践 https://www.jianshu.com/p/b3dedb8fb61e