# TraceId与日志组件集成
通过Skywalking提供的工具包，我们可以将Skywalking在追踪过程中生成的TraceId唯一标识集成到我们业务应用中常见的日志组件logback、log4j。其中lo4j支持1.x和2.x版本。

本章节参考源码在[Github](https://github.com/DaoCloud-Labs/DMP-Demo/blob/master/skywalking/dmp-skywalking-agent-examples-master/dmp-skywalking-agent-integration-log4j-demo/README.md)

最终效果：

```shell
2018-07-25 17:44:58.842 [TID:N/A] [main] INFO  o.s.b.c.e.t.TomcatEmbeddedServletContainer -Tomcat started on port(s): 8001 (http)
2018-07-25 17:44:58.852 [TID:N/A] [main] INFO  i.d.n.s.l.AgentLog4jDemoApplication -Started AgentLog4jDemoApplication in 4.03 seconds (JVM running for 12.204)
2018-07-25 17:45:10.226 [TID:39.58.15325119102050001] [http-nio-8001-exec-1] INFO  o.a.c.c.C.[Tomcat].[localhost].[/] -Initializing Spring FrameworkServlet 'dispatcherServlet'
2018-07-25 17:45:10.226 [TID:39.58.15325119102050001] [http-nio-8001-exec-1] INFO  o.s.web.servlet.DispatcherServlet -FrameworkServlet 'dispatcherServlet': initialization started
2018-07-25 17:45:10.247 [TID:39.58.15325119102050001] [http-nio-8001-exec-1] INFO  o.s.web.servlet.DispatcherServlet -FrameworkServlet 'dispatcherServlet': initialization completed in 21 ms
2018-07-25 17:45:10.314 [TID:39.58.15325119102050001] [http-nio-8001-exec-1] INFO  i.d.n.sw.log4j.HelloWorldController -words:demo
```

## 步骤一
此处以**logback**工具包为例，在你的业务项目中，使用 Maven 或者 Gradle 导入相应的依赖工具包：

- Maven
在pom.xml文件中添加如下依赖。

```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>${skywalking.version}</version>
    <!-- <version>5.0.0-beta2</version>-->
</dependency>
```

- Gradle
在build.gradle中配置如下依赖：

```xml
dependencies {
	······
    compile 'org.apache.skywalking:apm-toolkit-logback-1.x:6.0.0-beta'
	······
}
```

## 步骤二
需要对你的日志组件配置文件中添加如下配置：

```xml
......
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
      <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
          <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
              <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%tid] [%thread] %-5level %logger{36} -%msg%n</Pattern>
          </layout>
      </encoder>
</appender>
......
```

## 最终运行你的应用/服务
使用**-javaagent**参数激活Skywalking的探针, 如果当前上下文中存在traceid，logback将输出traceId。否则将输出TID: N/A.

### -javaagent的使用

```shell
java -javaagent:/Your/path/to/skywalking/agent/incubator-skywalking/skywalking-agent/skywalking-agent.jar -jar -Dskywalking.agent.service_name=demo-project demo-project.jar
```

你也可以参考前面章节中的**Jar/War包接入Skywalking**。