## 前言

分析漏洞的本质是为了能让我们从中学习漏洞挖掘者的思路以及挖掘到新的漏洞，而CodeQL就是一款可以将我们对漏洞的理解快速转化为可实现的规则并挖掘漏洞的利器。根据网上的传言Log4j2的RCE漏洞就是作者通过CodeQL挖掘出的。虽然如何挖掘的我们不得而知，但我们现在站在事后的角度再去想想，可以推测一下作者如何通过CodeQL挖掘到漏洞的,并尝试基于作者的思路挖掘新漏洞。

## 分析过程

首先我们要构建Log4j的数据库，由于`lgtm.com`中构建的是新版本的Log4j数据库，所以只能手动构建数据库了。首先从github获取源码并切换到2.14.1版本。

1.  `git clone https://github.com/apache/logging-log4j2.git`
    
2.  `git checkout be881e5`
    

由于我们这次分析的主要是`log4j-core`和`log4j-api`中的内容，所以打开根目录的Pom.xml注释下面的内容。

1.  `<modules>`
    
2.   `<module>log4j-api-java9</module>`
    
3.   `<module>log4j-api</module>`
    
4.   `<module>log4j-core-java9</module>`
    
5.   `<module>log4j-core</module>`
    
6.   `<!-- <module>log4j-layout-template-json</module>`
    
7.   `<module>log4j-core-its</module>`
    
8.   `<module>log4j-1.2-api</module>`
    
9.   `<module>log4j-slf4j-impl</module>`
    
10.   `<module>log4j-slf4j18-impl</module>`
    
11.   `<module>log4j-to-slf4j</module>`
    
12.   `<module>log4j-jcl</module>`
    
13.   `<module>log4j-flume-ng</module>`
    
14.   `<module>log4j-taglib</module>`
    
15.   `<module>log4j-jmx-gui</module>`
    
16.   `<module>log4j-samples</module>`
    
17.   `<module>log4j-bom</module>`
    
18.   `<module>log4j-jdbc-dbcp2</module>`
    
19.   `<module>log4j-jpa</module>`
    
20.   `<module>log4j-couchdb</module>`
    
21.   `<module>log4j-mongodb3</module>`
    
22.   `<module>log4j-mongodb4</module>`
    
23.   `<module>log4j-cassandra</module>`
    
24.   `<module>log4j-web</module>`
    
25.   `<module>log4j-perf</module>`
    
26.   `<module>log4j-iostreams</module>`
    
27.   `<module>log4j-jul</module>`
    
28.   `<module>log4j-jpl</module>`
    
29.   `<module>log4j-liquibase</module>`
    
30.   `<module>log4j-appserver</module>`
    
31.   `<module>log4j-osgi</module>`
    
32.   `<module>log4j-docker</module>`
    
33.   `<module>log4j-kubernetes</module>`
    
34.   `<module>log4j-spring-boot</module>`
    
35.   `<module>log4j-spring-cloud-config</module> -->`
    
36.   `</modules>`
    

由于`log4j-api-java9`和`log4j-core- java9`需要依赖JDK9，所以要先下载JDK9并且在`C:\Users\用户名\.m2\toolchains.xml`中加上下面的内容。

1.  `<toolchains>`
    
2.  `<toolchain>` 
    
3.   `<type>jdk</type>` 
    
4.   `<provides>` 
    
5.   `<version>9</version>` 
    
6.   `<vendor>sun</vendor>` 
    
7.   `</provides>` 
    
8.   `<configuration>` 
    
9.   `<jdkHome>C:\Program Files\Java\jdk-9.0.4</jdkHome>` 
    
10.   `</configuration>` 
    
11.  `</toolchain>` 
    
12.  `</toolchains>`
    

通过下面的命令完成数据库构建

1.  `CodeQL database create Log4jDB --language=java --overwrite --command="mvn clean install -Dmaven.test.skip=true"`
    

构建好数据库后，我们要找JNDI注入的漏洞，首先要确定在这套系统中调用了InitialContext#lookup方法。在LookupInterface项目中已经集成了常见的发起JNDI请求的类,只要稍微改一下即可。

首先定义Context类型，这个类中综合了可能发起JNDI请求的类。

1.  `class Context extends RefType{`
    
2.   `Context(){`
    
3.   `this.hasQualifiedName("javax.naming", "Context")`
    
4.   `or`
    
5.   `this.hasQualifiedName("javax.naming", "InitialContext")`
    
6.   `or`
    
7.   `this.hasQualifiedName("org.springframework.jndi", "JndiCallback")`
    
8.   `or` 
    
9.   `this.hasQualifiedName("org.springframework.jndi", "JndiTemplate")`
    
10.   `or`
    
11.   `this.hasQualifiedName("org.springframework.jndi", "JndiLocatorDelegate")`
    
12.   `or`
    
13.   `this.hasQualifiedName("org.apache.shiro.jndi", "JndiCallback")`
    
14.   `or`
    
15.   `this.getQualifiedName().matches("%JndiCallback")`
    
16.   `or`
    
17.   `this.getQualifiedName().matches("%JndiLocatorDelegate")`
    
18.   `or`
    
19.   `this.getQualifiedName().matches("%JndiTemplate")`
    
20.   `}`
    
21.  `}`
    

下面寻找那里调用了`Context`的lookup方法。

1.  `from Call call,Callable parseExpression`
    
2.  `where`
    
3.   `call.getCallee() = parseExpression and` 
    
4.   `parseExpression.getDeclaringType() instanceof Context and`
    
5.   `parseExpression.hasName("lookup")`
    
6.  `select call`
    

![图片](_v_images/545082219214772.webp)

*   `DataSourceConnectionSource#createConnectionSource`
    

1.  `@PluginFactory`
    
2.   `public static DataSourceConnectionSource createConnectionSource(@PluginAttribute("jndiName") final String jndiName) {`
    
3.   `if (Strings.isEmpty(jndiName)) {`
    
4.   `LOGGER.error("No JNDI name provided.");`
    
5.   `return null;`
    
6.   `}`
    
7.   `try {`
    
8.   `final InitialContext context = new InitialContext();`
    
9.   `final DataSource dataSource = (DataSource) context.lookup(jndiName);`
    
10.   `if (dataSource == null) {`
    
11.   `LOGGER.error("No data source found with JNDI name [" + jndiName + "].");`
    
12.   `return null;`
    
13.   `}`
    
14.   `return new DataSourceConnectionSource(jndiName, dataSource);`
    
15.   `} catch (final NamingException e) {`
    
16.   `LOGGER.error(e.getMessage(), e);`
    
17.   `return null;`
    
18.   `}`
    
19.   `}`
    

*   `JndiManager#lookup`
    

1.  `@SuppressWarnings("unchecked")`
    
2.   `public <T> T lookup(final String name) throws NamingException {`
    
3.   `return (T) this.context.lookup(name);`
    
4.   `}`
    

找到sink后我们还需要找到source，虽然Codeql定义了`RemoteFlowSource`支持多种source，但是我们还是要根据实际的代码业务来分析可能作为source的点。

在Log4j作为日志记录的工具，除了从HTTP请求中获取输入点外，还可以在记录日志请求或者解析配置文件中来获取source。先不看解析配置文件获取source的点了，因为这需要分析Log4j解析配置文件的流程比较复杂。所以目前我们只考虑通过日志记录作为source的情况。稍微了解Log4j的同学都知道，Log4j会通过`error/fatal/info/debug/trace`等方法对不同级别的日志进行记录。通过分析我们可以看到我们输入的message都调用了`logIfEnabled`方法并作为第四个参数输入，所以可以将这里定义为source。

![图片](_v_images/542042219235617.webp)

下面使用全局污点追踪分析JNDI漏洞，还是套用LookupInterface项目中的代码，修改source部分即可。

1.  `/**`
    
2.   `*@name Tainttrack Context lookup`
    
3.   `*@kind path-problem`
    
4.   `*/`
    
5.  `import java`
    
6.  `import semmle.code.java.dataflow.FlowSources`
    
7.  `import DataFlow::PathGraph`
    
8.  `class Context extends RefType{`
    
9.   `Context(){`
    
10.   `this.hasQualifiedName("javax.naming", "Context")`
    
11.   `or`
    
12.   `this.hasQualifiedName("javax.naming", "InitialContext")`
    
13.   `or`
    
14.   `this.hasQualifiedName("org.springframework.jndi", "JndiCallback")`
    
15.   `or` 
    
16.   `this.hasQualifiedName("org.springframework.jndi", "JndiTemplate")`
    
17.   `or`
    
18.   `this.hasQualifiedName("org.springframework.jndi", "JndiLocatorDelegate")`
    
19.   `or`
    
20.   `this.hasQualifiedName("org.apache.shiro.jndi", "JndiCallback")`
    
21.   `or`
    
22.   `this.getQualifiedName().matches("%JndiCallback")`
    
23.   `or`
    
24.   `this.getQualifiedName().matches("%JndiLocatorDelegate")`
    
25.   `or`
    
26.   `this.getQualifiedName().matches("%JndiTemplate")`
    
27.   `}`
    
28.  `}`
    
29.  `class Logger extends RefType{`
    
30.   `Logger(){`
    
31.   `this.hasQualifiedName("org.apache.logging.log4j.spi", "AbstractLogger")`
    
32.   `}`
    
33.  `}`
    
34.  `predicate isLookup(Expr arg) {`
    
35.   `exists(MethodAccess ma |`
    
36.   `ma.getMethod().getName() = "lookup"`
    
37.   `and`
    
38.   `ma.getMethod().getDeclaringType() instanceof Context`
    
39.   `and`
    
40.   `arg = ma.getArgument(0)`
    
41.   `)`
    
42.  `}`
    
43.  `predicate isLogging(Expr arg) {`
    
44.   `exists(MethodAccess ma |`
    
45.   `ma.getMethod().getName() = "logIfEnabled"`
    
46.   `and`
    
47.   `ma.getMethod().getDeclaringType() instanceof Logger`
    
48.   `and`
    
49.   `arg = ma.getArgument(3)`
    
50.   `)`
    
51.  `}`
    
52.  `class TainttrackLookup extends TaintTracking::Configuration {`
    
53.   `TainttrackLookup() {` 
    
54.   `this = "TainttrackLookup"` 
    
55.   `}`
    
56.    
    
57.   `override predicate isSource(DataFlow::Node source) {`
    
58.   `exists(Expr exp |`
    
59.   `isLogging(exp)`
    
60.   `and`
    
61.   `source.asExpr() = exp`
    
62.   `)`
    
63.   `}`
    
64.    
    
65.   `override predicate isSink(DataFlow::Node sink) {`
    
66.   `exists(Expr arg |`
    
67.   `isLookup(arg)`
    
68.   `and`
    
69.   `sink.asExpr() = arg`
    
70.   `)`
    
71.   `}`
    
72.  `}` 
    
73.  `from TainttrackLookup config , DataFlow::PathNode source, DataFlow::PathNode sink`
    
74.  `where`
    
75.   `config.hasFlowPath(source, sink)`
    
76.  `select sink.getNode(), source, sink, "unsafe lookup", source.getNode(), "this is user input"`
    

虽然这些也得到了很多查询结果，但是在实际使用Log4j打印日志时可能不会带上Marker参数而是直接写入messge的内容。

![图片](_v_images/538992219242325.webp)

所以我们现在要追踪的source应该是带有一个参数的`error/fatal/info/debug/trace`等方法。我这里以error方法为例对source部分进行修改。

1.  `class LoggerInput extends Method {`
    
2.   `LoggerInput(){`
    
3.   `//限定调用的类名、方法名、以及方法只有一个参数`
    
4.   `this.getDeclaringType() instanceof Logger and`
    
5.   `this.hasName("error") and this.getNumberOfParameters() = 1`
    
6.   `}`
    
7.   `//将第一个参数作为source`
    
8.   `Parameter getAnUntrustedParameter() { result = this.getParameter(0) }`
    
9.  `}`
    
10.  `override predicate isSource(DataFlow::Node source) {`
    
11.   `exists(LoggerInput LoggerMethod |`
    
12.   `source.asParameter() = LoggerMethod.getAnUntrustedParameter())`
    
13.   `}`
    

这样我们就得到了多条链,现在我们要写个Demo验证这个链是否可行，比如最简单的`logger.error("xxxxx");`

1.  `1 message : Message AbstractLogger.java:709:23`
    
2.  `2 message : Message AbstractLogger.java:710:47`
    
3.  `3 message : Message AbstractLogger.java:1833:89`
    
4.  `4 message : Message AbstractLogger.java:1835:38`
    
5.  `5 message : Message Logger.java:262:70`
    
6.  `6 message : Message Logger.java:263:52`
    
7.  `7 msg : Message Logger.java:617:64`
    
8.  `8 msg : Message Logger.java:620:78`
    
9.  `9 msg : Message RegexFilter.java:73:87`
    
10.  `10 msg : Message RegexFilter.java:78:63`
    
11.  `...`
    
12.  `64 convertJndiName(...) : String JndiLookup.java:54:33`
    
13.  `65 jndiName : String JndiLookup.java:56:56`
    
14.  `66 name : String JndiManager.java:171:25`
    
15.  `67 name JndiManager.java:172:40`
    
16.  `Path`
    

但是这条链只有配置了Filter为`RegexFilter`才会继续执行，而默认没有配置则为空。

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

所以这种方式就稍微有些限制，所以我们再去看看其他链。这条链似乎不用配置Filter。

1.  `1 message : Message AbstractLogger.java:709:23`
    
2.  `2 message : Message AbstractLogger.java:710:47`
    
3.  `3 message : Message AbstractLogger.java:1833:89`
    
4.  `4 message : Message AbstractLogger.java:1836:51`
    
5.  `5 message : Message AbstractLogger.java:2139:94`
    
6.  `6 message : Message AbstractLogger.java:2142:59`
    
7.  `7 message : Message AbstractLogger.java:2155:43`
    
8.  `8 message : Message AbstractLogger.java:2159:67`
    
9.  `9 message : Message AbstractLogger.java:2202:32`
    
10.  `10 message : Message AbstractLogger.java:2205:48`
    
11.  `11 message : Message AbstractLogger.java:2116:9`
    
12.  `12 message : Message AbstractLogger.java:2117:41`
    
13.  `...`
    
14.  `78 var : String Interpolator.java:230:92`
    
15.  `79 key : String JndiLookup.java:50:48`
    
16.  `80 key : String JndiLookup.java:54:49`
    
17.  `81 jndiName : String JndiLookup.java:70:36`
    
18.  `82 jndiName : String JndiLookup.java:74:16`
    
19.  `83 convertJndiName(...) : String JndiLookup.java:54:33`
    
20.  `84 jndiName : String JndiLookup.java:56:56`
    
21.  `85 name : String JndiManager.java:171:25`
    
22.  `86 name JndiManager.java:172:40`
    

但是在`AbstractLogger#tryLogMessage`中Codeql会直接分析到`AbstractLogger#log`而实际请求时会解析到`Logger#log`方法。这是因为`Logger`是`AbstractLogger`的子类并且也实现了log方法，而且我们实例化的也是Logger对象，所以这里会调用到`Logger#log`。

实际请求

![图片](_v_images/535912219226230.webp)

CodeQL分析

![图片](_v_images/533862219226574.webp)

再看看下面这条链

1.  `1 message : Message AbstractLogger.java:709:23`
    
2.  `2 message : Message AbstractLogger.java:710:47`
    
3.  `3 message : Message AbstractLogger.java:1833:89`
    
4.  `4 message : Message AbstractLogger.java:1836:51`
    
5.  `5 message : Message AbstractLogger.java:2139:94`
    
6.  `6 message : Message AbstractLogger.java:2142:59`
    
7.  `7 message : Message AbstractLogger.java:2155:43`
    
8.  `8 message : Message AbstractLogger.java:2159:67`
    
9.  `9 message : Message AbstractLogger.java:2202:32`
    
10.  `10 message : Message AbstractLogger.java:2205:48`
    
11.  `11 message : Message Logger.java:158:9`
    
12.  `12 message : Message Logger.java:162:17`
    
13.  `13 data : Message AwaitCompletionReliabilityStrategy.java:78:83`
    
14.  `14 data : Message AwaitCompletionReliabilityStrategy.java:82:67`
    
15.  `15 data : Message LoggerConfig.java:430:28`
    
16.  `16 data : Message LoggerConfig.java:454:17`
    
17.  `17 message : Message ReusableLogEventFactory.java:78:86`
    
18.  `18 message : Message ReusableLogEventFactory.java:100:27`
    
19.  `19 msg : Message MutableLogEvent.java:209:28`
    
20.  `20 (...)... : Message MutableLogEvent.java:211:46`
    
21.  `21 reusable : Message MutableLogEvent.java:212:13`
    
22.  `22 parameter this : Message ReusableObjectMessage.java:47:17`
    
23.  `23 obj : Object ReusableObjectMessage.java:48:44`
    
24.  `...`
    
25.  `88 convertJndiName(...) : String JndiLookup.java:54:33`
    
26.  `89 jndiName : String JndiLookup.java:56:56`
    
27.  `90 name : String JndiManager.java:171:25`
    
28.  `91 name JndiManager.java:172:40`
    

这条链在执行到`MutableLogEvent#setMessage`时和CodeQL的分析结果略有不同。

![图片](_v_images/529812219235594.webp)

在CodeQL中`resusable.formatTo`会调用到`ReusableObjectMessage`中。

![图片](_v_images/527752219217953.webp)

但是实际运行过程中由于MessgeFactorty创建Message对象时默认创建的是`ResableSimpleMessage`对象，所以会执行到`ResableSimpleMessage#formatTo`方法。

![图片](_v_images/523692219227114.webp)

![图片](data:/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

所以似乎目前使用使用CodeQL的规则是发现不了Log4jShell那个漏洞的，既然我们已经知道了这个漏洞的触发链，可以分析下CodeQL为什么没有分析出来。

通过之前对CodeQL检测出的调用链分析，CodeQL已经分析到了createEvent方法。

![图片](_v_images/517472219235029.webp)

查看createEvent方法的调用，在`Log4jShell`的触发链中实际上是在对返回LogEvent的处理过程中触发的，所以这里CodeQL可能没有将返回的LogEvent对象再当作污点进行分析，所以导致没有分析成功。

![图片](_v_images/516402219238670.webp)

我们可以创建一个`isAdditionalTaintStep`函数，将`ReusableLogEventFactory#createEvent`的第六个参数Message和`LoggerConfig#log`第一个参数`logEvent`连接起来。

1.  `override predicate isAdditionalTaintStep(DataFlow::Node fromNode, DataFlow::Node toNode) {`
    
2.   `exists(MethodAccess ma,MethodAccess ma2 |`
    
3.   `ma.getMethod().getDeclaringType().hasQualifiedName("org.apache.logging.log4j.core.impl", "ReusableLogEventFactory")` 
    
4.   `and ma.getMethod().hasName("createEvent") and fromNode.asExpr()=ma.getArgument(5) and ma2.getMethod().getDeclaringType().hasQualifiedName("org.apache.logging.log4j.core.config", "LoggerConfig")` 
    
5.   `and ma2.getMethod().hasName("log") and ma2.getMethod().getNumberOfParameters() = 2 and toNode.asExpr()=ma2.getArgument(0)`
    
6.   `)`
    
7.   `}`
    

最后我们就可以通过CodeQL分析到Log4j shell漏洞的调用链。

1.  `1 message : Message AbstractLogger.java:709:23`
    
2.  `2 message : Message AbstractLogger.java:710:47`
    
3.  `3 message : Message AbstractLogger.java:1833:89`
    
4.  `4 message : Message AbstractLogger.java:1836:51`
    
5.  `5 message : Message AbstractLogger.java:2139:94`
    
6.  `6 message : Message AbstractLogger.java:2142:59`
    
7.  `7 message : Message AbstractLogger.java:2155:43`
    
8.  `8 message : Message AbstractLogger.java:2159:67`
    
9.  `9 message : Message AbstractLogger.java:2202:32`
    
10.  `10 message : Message AbstractLogger.java:2205:48`
    
11.  `11 message : Message Logger.java:158:9`
    
12.  `12 message : Message Logger.java:162:17`
    
13.  `13 data : Message DefaultReliabilityStrategy.java:61:83`
    
14.  `14 data : Message DefaultReliabilityStrategy.java:63:69`
    
15.  `15 data : Message LoggerConfig.java:430:28`
    
16.  `16 data : Message LoggerConfig.java:454:96`
    
17.  `17 message : Message ReusableLogEventFactory.java:58:47`
    
18.  `18 message : Message ReusableLogEventFactory.java:60:67`
    
19.  `19 event : LogEvent LoggerConfig.java:469:13`
    
20.  `20 event : LogEvent LoggerConfig.java:479:24`
    
21.  `21 event : LogEvent LoggerConfig.java:481:29`
    
22.  `22 event : LogEvent LoggerConfig.java:495:34`
    
23.  `23 event : LogEvent LoggerConfig.java:498:27`
    
24.  `24 event : LogEvent LoggerConfig.java:536:34`
    
25.  `25 event : LogEvent LoggerConfig.java:540:38`
    
26.  `26 event : LogEvent AppenderControl.java:80:30`
    
27.  `27 event : LogEvent AppenderControl.java:84:38`
    
28.  `28 event : LogEvent AppenderControl.java:117:47`
    
29.  `29 event : LogEvent AppenderControl.java:120:27`
    
30.  `30 event : LogEvent AppenderControl.java:126:32`
    
31.  `31 event : LogEvent AppenderControl.java:129:29`
    
32.  `32 event : LogEvent AppenderControl.java:154:34`
    
33.  `33 event : LogEvent AppenderControl.java:156:29`
    
34.  `34 event : LogEvent AbstractDatabaseAppender.java:107:30`
    
35.  `35 event : LogEvent AbstractDatabaseAppender.java:110:37`
    
36.  `36 event : LogEvent AbstractDatabaseManager.java:260:42`
    
37.  `37 event : LogEvent AbstractDatabaseManager.java:262:20`
    
38.  `38 event : LogEvent AbstractDatabaseManager.java:122:27`
    
39.  `39 event : LogEvent AbstractDatabaseManager.java:123:25`
    
40.  `40 parameter this : LogEvent Log4jLogEvent.java:530:26`
    
41.  `41 this : LogEvent Log4jLogEvent.java:534:16`
    
42.  `42 toImmutable(...) : LogEvent AbstractDatabaseManager.java:123:25`
    
43.  `43 this.buffer [post update] [<element>] : LogEvent AbstractDatabaseManager.java:123:9`
    
44.  `44 this [post update] [buffer, <element>] : LogEvent AbstractDatabaseManager.java:123:9`
    
45.  `45 this <.method> [post update] [buffer, <element>] : LogEvent AbstractDatabaseManager.java:262:13`
    
46.  `46 getManager(...) [post update] [buffer, <element>] : LogEvent AbstractDatabaseAppender.java:110:13`
    
47.  `47 this [post update] [manager, buffer, <element>] : LogEvent AbstractDatabaseAppender.java:110:13`
    
48.  `48 appender [post update] [manager, buffer, <element>] : LogEvent AppenderControl.java:156:13`
    
49.  `49 this <.field> [post update] [appender, manager, buffer, <element>] : LogEvent AppenderControl.java:156:13`
    
50.  `50 this <.method> [post update] [appender, manager, buffer, <element>] : LogEvent AppenderControl.java:129:13`
    
51.  `51 this <.method> [post update] [appender, manager, buffer, <element>] : LogEvent AppenderControl.java:120:13`
    
52.  `52 this <.method> [post update] [appender, manager, buffer, <element>] : LogEvent AppenderControl.java:84:9`
    
53.  `53 event : LogEvent AppenderControl.java:80:30`
    
54.  `54 event : LogEvent AppenderControl.java:84:38`
    
55.  `55 event : LogEvent AppenderControl.java:117:47`
    
56.  `56 event : LogEvent AppenderControl.java:120:27`
    
57.  `57 event : LogEvent AppenderControl.java:126:32`
    
58.  `58 event : LogEvent AppenderControl.java:129:29`
    
59.  `59 event : LogEvent AppenderControl.java:154:34`
    
60.  `60 event : LogEvent AppenderControl.java:156:29`
    
61.  `61 event : LogEvent AbstractOutputStreamAppender.java:179:24`
    
62.  `62 event : LogEvent AbstractOutputStreamAppender.java:181:23`
    
63.  `63 event : LogEvent AbstractOutputStreamAppender.java:188:28`
    
64.  `64 event : LogEvent AbstractOutputStreamAppender.java:190:31`
    
65.  `65 event : LogEvent AbstractOutputStreamAppender.java:196:38`
    
66.  `66 event : LogEvent AbstractOutputStreamAppender.java:197:28`
    
67.  `67 event : LogEvent GelfLayout.java:433:24`
    
68.  `68 event : LogEvent GelfLayout.java:438:43`
    
69.  `69 event : LogEvent GelfLayout.java:471:34`
    
70.  `70 event : LogEvent GelfLayout.java:496:46`
    
71.  `71 event : LogEvent StrSubstitutor.java:462:27`
    
72.  `72 event : LogEvent StrSubstitutor.java:467:25`
    
73.  `73 event : LogEvent StrSubstitutor.java:911:34`
    
74.  `74 event : LogEvent StrSubstitutor.java:912:27`
    
75.  `75 event : LogEvent StrSubstitutor.java:928:28`
    
76.  `76 event : LogEvent StrSubstitutor.java:978:44`
    
77.  `77 event : LogEvent StrSubstitutor.java:911:34`
    
78.  `78 event : LogEvent StrSubstitutor.java:912:27`
    
79.  `79 event : LogEvent StrSubstitutor.java:928:28`
    
80.  `80 event : LogEvent StrSubstitutor.java:1033:63`
    
81.  `81 event : LogEvent StrSubstitutor.java:1104:38`
    
82.  `82 event : LogEvent StrSubstitutor.java:1110:32`
    
83.  `83 event : LogEvent StructuredDataLookup.java:46:26`
    
84.  `84 event : LogEvent StructuredDataLookup.java:50:67`
    
85.  `85 parameter this : LogEvent RingBufferLogEvent.java:206:20`
    
86.  `86 message : Message RingBufferLogEvent.java:210:16`
    
87.  `87 getMessage(...) : Message StructuredDataLookup.java:50:67`
    
88.  `88 (...)... : Message StructuredDataLookup.java:50:43`
    
89.  `89 msg : Message StructuredDataLookup.java:54:20`
    
90.  `90 parameter this : Message StructuredDataMessage.java:239:19`
    
91.  `91 type : String StructuredDataMessage.java:240:16`
    
92.  `92 getType(...) : String StructuredDataLookup.java:54:20`
    
93.  `93 lookup(...) : String StrSubstitutor.java:1110:16`
    
94.  `94 resolveVariable(...) : String StrSubstitutor.java:1033:47`
    
95.  `95 varValue : String StrSubstitutor.java:1040:63`
    
96.  `96 buf [post update] : StringBuilder StrSubstitutor.java:1040:33`
    
97.  `97 buf [post update] : StringBuilder StrSubstitutor.java:912:34`
    
98.  `98 bufName [post update] : StringBuilder StrSubstitutor.java:978:51`
    
99.  `99 bufName : StringBuilder StrSubstitutor.java:979:47`
    
100.  `100 toString(...) : String StrSubstitutor.java:979:47`
    
101.  `101 varNameExpr : String StrSubstitutor.java:1010:55`
    
102.  `102 substring(...) : String StrSubstitutor.java:1010:55`
    
103.  `103 varName : String StrSubstitutor.java:1033:70`
    
104.  `104 variableName : String StrSubstitutor.java:1104:60`
    
105.  `105 variableName : String StrSubstitutor.java:1110:39`
    
106.  `106 key : String JndiLookup.java:50:48`
    
107.  `107 key : String JndiLookup.java:54:49`
    
108.  `108 jndiName : String JndiLookup.java:70:36`
    
109.  `109 ... + ... : String JndiLookup.java:72:20`
    
110.  `110 convertJndiName(...) : String JndiLookup.java:54:33`
    
111.  `111 jndiName : String JndiLookup.java:56:56`
    
112.  `112 name : String JndiManager.java:171:25`
    
113.  `113 name JndiManager.java:172:40`
    

## 漏洞挖掘尝试

通过上面的分析可以看到，挖掘到所有的链最终的触发点都是JndiManager，这个点目前的触发已经在新版本中修复了，但是在`DataSourceConnectionSource#createConnectionSource`中也直接调用了lookup方法，我们能否通过某种方式触发呢？

通过注释可以看到DataSource是Core类型插件，因此可以在XML中直接通过标签配置调用。

![图片](_v_images/511352219233872.webp)

1.  `<?xml version="1.0" encoding="UTF-8"?>`
    
2.  `<Configuration status="WARN">`
    
3.   `<Appenders>`
    
4.   `<Console name="Console" target="SYSTEM_OUT">`
    
5.   `<PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>`
    
6.   `</Console>`
    
7.   `</Appenders>`
    
8.   `<DataSource jndiName="ldap://9b89e78d.dns.1433.eu.org.">`
    
9.   `</DataSource>`
    
10.   `<Loggers>`
    
11.   `<Root level="ERROR">`
    
12.   `<AppenderRef ref="Console"/>`
    
13.   `</Root>`
    
14.   `</Loggers>`
    
15.  `</Configuration>`
    

配置后可以在插件加载的过程中触发漏洞，虽然这种方式也可以造成JNDI注入，但是需要在配置文件中修改参数才能触发，所以价值不大。

![图片](_v_images/508142219230488.webp)

最后给出整体的分析Log4j JNDI注入的CodeQL查询代码

1.  `/**`
    
2.   `*@name Tainttrack Context lookup`
    
3.   `*@kind path-problem`
    
4.   `*/`
    
5.  `import java`
    
6.  `import semmle.code.java.dataflow.FlowSources`
    
7.  `import DataFlow::PathGraph`
    
8.  `class Context extends RefType{`
    
9.   `Context(){`
    
10.   `this.hasQualifiedName("javax.naming", "Context")`
    
11.   `or`
    
12.   `this.hasQualifiedName("javax.naming", "InitialContext")`
    
13.   `or`
    
14.   `this.hasQualifiedName("org.springframework.jndi", "JndiCallback")`
    
15.   `or` 
    
16.   `this.hasQualifiedName("org.springframework.jndi", "JndiTemplate")`
    
17.   `or`
    
18.   `this.hasQualifiedName("org.springframework.jndi", "JndiLocatorDelegate")`
    
19.   `or`
    
20.   `this.hasQualifiedName("org.apache.shiro.jndi", "JndiCallback")`
    
21.   `or`
    
22.   `this.getQualifiedName().matches("%JndiCallback")`
    
23.   `or`
    
24.   `this.getQualifiedName().matches("%JndiLocatorDelegate")`
    
25.   `or`
    
26.   `this.getQualifiedName().matches("%JndiTemplate")`
    
27.   `}`
    
28.  `}`
    
29.  `class Logger extends RefType{`
    
30.   `Logger(){`
    
31.   `this.hasQualifiedName("org.apache.logging.log4j.spi", "AbstractLogger")`
    
32.   `}`
    
33.  `}`
    
34.  `class LoggerInput extends Method {`
    
35.   `LoggerInput(){`
    
36.   `this.getDeclaringType() instanceof Logger and`
    
37.   `this.hasName("error") and this.getNumberOfParameters() = 1`
    
38.   `}`
    
39.   `Parameter getAnUntrustedParameter() { result = this.getParameter(0) }`
    
40.  `}`
    
41.  `predicate isLookup(Expr arg) {`
    
42.   `exists(MethodAccess ma |`
    
43.   `ma.getMethod().getName() = "lookup"`
    
44.   `and`
    
45.   `ma.getMethod().getDeclaringType() instanceof Context`
    
46.   `and`
    
47.   `arg = ma.getArgument(0)`
    
48.   `)`
    
49.  `}`
    
50.  `class TainttrackLookup extends TaintTracking::Configuration {`
    
51.   `TainttrackLookup() {` 
    
52.   `this = "TainttrackLookup"` 
    
53.   `}`
    
54.    
    
55.   `override predicate isSource(DataFlow::Node source) {`
    
56.   `exists(LoggerInput LoggerMethod |`
    
57.   `source.asParameter() = LoggerMethod.getAnUntrustedParameter())`
    
58.   `}`
    
59.    
    
60.   `override predicate isAdditionalTaintStep(DataFlow::Node fromNode, DataFlow::Node toNode) {`
    
61.   `exists(MethodAccess ma,MethodAccess ma2 |`
    
62.   `ma.getMethod().getDeclaringType().hasQualifiedName("org.apache.logging.log4j.core.impl", "ReusableLogEventFactory")` 
    
63.   `and ma.getMethod().hasName("createEvent") and fromNode.asExpr()=ma.getArgument(5) and ma2.getMethod().getDeclaringType().hasQualifiedName("org.apache.logging.log4j.core.config", "LoggerConfig")` 
    
64.   `and ma2.getMethod().hasName("log") and ma2.getMethod().getNumberOfParameters() = 2 and toNode.asExpr()=ma2.getArgument(0)`
    
65.   `)`
    
66.   `}`
    
67.   `override predicate isSink(DataFlow::Node sink) {`
    
68.   `exists(Expr arg |`
    
69.   `isLookup(arg)`
    
70.   `and`
    
71.   `sink.asExpr() = arg`
    
72.   `)`
    
73.   `}`
    
74.  `}` 
    
75.  `from TainttrackLookup config , DataFlow::PathNode source, DataFlow::PathNode sink`
    
76.  `where`
    
77.   `config.hasFlowPath(source, sink)`
    
78.  `select sink.getNode(), source, sink, "unsafe lookup", source.getNode(), "this is user input"`
    

## 总结

通过CodeQL挖洞效率确实比较高，并且在官方也给出了针对很多类型漏洞的审计规则，确实可以高效的辅助挖洞，目前主要解决下面两个问题。

*   默认的Source应该只是针对HTTP请求，如何针对特定的框架去发现可能作为source的点
    
*   分析污点在何时会被打断并进行拼接