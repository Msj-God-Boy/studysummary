# Appenders

> Logback将编写日志记录事件的任务委派给称为附加程序的组件

## Logback Core

### OutputStreamAppender

![image-20200526214546176](C:\Users\itachi\Desktop\阅读\logback\image-20200526214546176.png)

### ConsoleAppender

> 将日志打印在控制台上

```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

### FileAppender

> 将日志输出到文件上

```xml
View as .groovy
<configuration>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>testFile.log</file>
    <append>true</append>
    <!-- set immediateFlush to false for much higher logging throughput -->
    <immediateFlush>true</immediateFlush>
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

属性：

1. append：如果为true，则日志追加到尾部，否则原有的会被删除
2. encoder：和OutputStreamAppender一致
3. file：文件的名称路径

### RollingFileAppender

属性：

1. append：和FileAppender一致
2. file：和FileAppender一致
3. encoder：和OutputStreamAppender一致
4. rollingPolicy：滚动政策
5. triggeringPolicy：触发政策

####rollingPolicy

TimeBasedRollingPolicy：是最流行的一种滚动策略，按天或者按月

属性：

1. fileNamePattern：生成的日志文件格式
2. maxHistory：保存时间，会首先删除过期的日志文件，如果按月生成日志文件，则单位是月，按天生成日志文件，则单位是天，按小时生成日志文件，则单位是小时。
3. totalSizeGap：文件大小，超过时会首先删除最早的日志记录
4. cleanHistoryOnStart：默认为false，设置为true表示追加新的日志时清除历史日志文件

fileNamePattern属性详解：

| fileNamePattern                           | Rollover schedule                             | Example                                                      |
| ----------------------------------------- | --------------------------------------------- | ------------------------------------------------------------ |
| ./foo.%d                                  | 晚上0点生成一份新的日志文件格式为：yyyy-MM-dd | file属性没有设置，日志文件为foo.yyyy-MM-dd<br />file属性有设置时，到了晚上0点，旧的日志文件为foo.yyyy-MM-dd，新的日志文件为foo.txt |
| ./%d{yyyy/MM}/foo.txt                     | 每月生成新的日志文件                          | file属性没有设置，日志文件为/yyyy/MM/foo.txt<br />file属性有设置时，默认为./foo.txt，每个月最后一天的0点，会生成上个月的日志文件，格式为./yyyy/MM/foo.txt |
| ./foo.%d{yyyy-ww}                         | 按周生成日志文件                              |                                                              |
| ./foo%d{yyyy-MM-dd_HH}                    | 按小时生成日志文件                            |                                                              |
| ./foo%d{yyyy-MM-dd_HH_mm, UTC}            | 按分钟生成日志文件                            |                                                              |
| ./foo%d{yyyy-MM-dd_HH_mm}                 | 按分钟生成日志文件                            |                                                              |
| */ foo /％d {yyyy-MM，**aux** } /％d.log* | 每天滚动。存档位于包含年和月的文件夹下。      | 例如，在2006年11月，归档文件将全部放在/ foo / 2006-11 /文件夹下，例如*/foo/2006-11/2006-11-14.log*。 |

`TimeBasedRollingPolicy`支持自动文件压缩。如果fileNamePattern选项的值以*.gz* 或*.zip*结尾，则启用此功能。

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logFile.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>

      <!-- keep 30 days' worth of history capped at 3GB total size -->
      <maxHistory>30</maxHistory>
      <totalSizeCap>3GB</totalSizeCap>

    </rollingPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender> 

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!-- Support multiple-JVM writing to the same log file -->
    <prudent>true</prudent>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>
      <maxHistory>30</maxHistory> 
      <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>

    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender> 

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

```xml
<!-- 如果日志文件超过大小，则文件从0开始递增归档，%d和%i是必须的 -->
<configuration>
  <appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>mylog.txt</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>mylog-%d{yyyy-MM-dd}.%i.txt</fileNamePattern>
       <!-- each file should be at most 100MB, keep 60 days worth of history, but at most 20GB -->
       <maxFileSize>100MB</maxFileSize>    
       <maxHistory>60</maxHistory>
       <totalSizeCap>20GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>


  <root level="DEBUG">
    <appender-ref ref="ROLLING" />
  </root>

</configuration>
```

FixedWindowRollingPolicy

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>
	<!-- 表示文件的归档从索引1到3 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>tests.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

#### triggeringPolicy

SizeBasedTriggeringPolicy

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>test.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>
	<!-- 表示文件超过5M触发归档 -->
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

## Logback Classic

### SMTPAppender

属性

1. smtpHost
2. smtpPort
3. to
4. from
5. subject
6. discriminator
7. evaluator
8. cyclicBufferTracker
9. username
10. password
11. STARTTLS
12. SSL
13. charsetEncoding
14. localhost
15. asynchronousSending
16. includeCallerData
17. sessionViaJNDI
18. jndiLocation

```xml
<configuration>   
  <appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
    <smtpHost>ADDRESS-OF-YOUR-SMTP-HOST</smtpHost>
    <to>EMAIL-DESTINATION</to>
    <to>ANOTHER_EMAIL_DESTINATION</to> <!-- additional destinations are possible -->
    <from>SENDER-EMAIL</from>
    <subject>TESTING: %logger{20} - %m</subject>
    <layout class="ch.qos.logback.classic.PatternLayout">
      <pattern>%date %-5level %logger{35} - %message%n</pattern>
    </layout>       
  </appender>

  <root level="DEBUG">
    <appender-ref ref="EMAIL" />
  </root>  
</configuration>
```

```xml
<appender name="EMAIL" class="ch.qos.logback.classic.net.SMTPAppender">
  <smtpHost>${smtpHost}</smtpHost>
  <to>${to}</to>
  <from>${from}</from>
  <layout class="ch.qos.logback.classic.html.HTMLLayout"/>
</appender>
```

### DBAppender

首先需要创建三个表，如下：

![image-20200526230421026](C:\Users\itachi\Desktop\阅读\logback\image-20200526230421026.png)

![image-20200526230525663](C:\Users\itachi\Desktop\阅读\logback\image-20200526230525663.png)

```xml
<configuration>

  <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
    <connectionSource class="ch.qos.logback.core.db.DriverManagerConnectionSource">
      <driverClass>com.mysql.jdbc.Driver</driverClass>
      <url>jdbc:mysql://host_name:3306/datebase_name</url>
      <user>username</user>
      <password>password</password>
    </connectionSource>
  </appender>
  
  <root level="DEBUG" >
    <appender-ref ref="DB" />
  </root>
</configuration>
```

```xml
<configuration>

  <appender name="DB" class="ch.qos.logback.classic.db.DBAppender">
    <connectionSource
      class="ch.qos.logback.core.db.DataSourceConnectionSource">
      <dataSource
        class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <driverClass>com.mysql.jdbc.Driver</driverClass>
        <jdbcUrl>jdbc:mysql://${serverName}:${port}/${dbName}</jdbcUrl>
        <user>${user}</user>
        <password>${password}</password>
      </dataSource>
    </connectionSource>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="DB" />
  </root>
</configuration>
```

### AsyncAppender

属性：

1. queueSize：默认为256
2. discardingThreshold：当队列容量存在20%时，会丢弃TRACE，DEBUG，INFO级别日志，只打印WARN和ERROR级别日志，设置为0，表示所有级别日志都打印
3. includeCallerData
4. maxFlushTime
5. neverBlock：默认为false，设置为true表示队列满时，不存在异步情况（会阻塞应用程序）

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>myapp.log</file>
    <encoder>
      <pattern>%logger{35} - %msg%n</pattern>
    </encoder>
  </appender>

  <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <appender-ref ref="FILE" />
  </appender>

  <root level="DEBUG">
    <appender-ref ref="ASYNC" />
  </root>
</configuration>
```

