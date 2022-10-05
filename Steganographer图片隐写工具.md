# Steganographer图片隐写工具

## Steganographer

Steganographer是一款功能强大的隐写工具，该工具基于Python编程语言开发，能够帮助广大研究人员在一张图片中实现数据或文件的隐写。这个Python模块可以将文件隐藏在一张图片之中（当前版本仅支持PNG文件），并将包含了隐写数据的文件导出至磁盘中存储。可隐写的最大文件大小取决于图片的尺寸。计算方式如下：

max\_file\_size = ( height\_of\_ * width\_of\_ * 6 / 8 ) bytes

比如说，我们打算将“100k words.txt”这个文件隐藏在一个名为“original\_.png”的图片文件内，然后修改后的图片导出为“\_with\_100k words.png”。导出完成之后，你可以看一看这两张图片在修改前后的区别。当然了，你也可以使用Steganographer来从“\_with_100k words.png”中提取出我们隐藏的“100k words.txt”文件。

## 工具下载&使用

广大研究人员可以使用下列命令将该项目源码克隆至本地：

git clone https://github.com/priyansh-anand/steganographer.git

下载完成之后，我们会看到项目目录内已经给大家提供了一些测试用的文件、图片和数据了，我们可以直接调用项目内的Steganographer.py文件来使用Steganographer。

## 工具效果展示

### 原始图片：![](_v_images/20200812113520405_3532.png)

### 修改后的图片：

### ![](_v_images/20200812113520193_22082.png)

没错，我们的文件已经成功隐藏在了这张图片里面，大家能看得出区别吗?

## 工作机制

![](_v_images/20200812113519981_27751.jpg)

该工具的实现原理非常简单，如果我们改变每一个像素的LSB（最低有效位算法），那么这个修改变化在图片上是不会产生很大区别的，人类肉眼也是无法察觉的。

因此，Steganographer将从要隐藏的文件中提取2位数据，用这2位数据替换一个像素的最后2位数据，然后再去操作下一个像素。像素的最大变化单位可以是4个单位，并且在PNG图像中值得变化范围是（0, 255），所以这种变化在图片上并不显著。

在PNG图像中，每个像素有3个通道，即红、绿、蓝。（有些PNG图像有1个或4个通道，但此程序会将它们转换为3个通道）PNG图像中的典型像素结构如下所示：

a_pixel = (17,32,11)     # (RED, GREEN, BLUE)

因此，我们可以在一个像素中保存3个2位长度得数据，也就是每个像素存储和6位数据。这样一来，我们就可以计算出能够隐藏在图像中的最大文件大小了：

max\_file\_size = ( height\_of\_ * width\_of\_ * 6 / 8 ) bytes

为了方便大家更好理解，我们举个例子。比如说，我们需要隐藏的数据如下：

binary_data = 0b100111

然后我们取前两位数据，用我们图片像素“a_pixel”的红色通道替换它们：

a\_pixel = (0b10001, 0b100000, 0b1011)   # binary representation of a\_pixel values

\# Let's change a_pixel's RED Channel

\# 0b10001 -> 0b10010  ( First 2 bits are 10 )

a_pixel = (0b10010, 0b100000, 0b1011)  # modified pixel

接下来，再从binary\_data中提取两位数据，用我们图片像素“a\_pixel”的绿色通道替换它们：

a_pixel = (0b10010, 0b100000, 0b1011)

\# Let's change a_pixel's GREEN Channel

\# 0b100000 -> 0b100001  ( The 2 bits are 01 )

a_pixel = (0b10010, 0b100001, 0b1011)  # modified pixel

然后再用蓝色通道进行一次数据替换操作：

a_pixel = (0b10010, 0b100001, 0b1011)

\# Let's change a_pixel's BLUE Channel

\# 0b1011 -> 0b1011  ( The 2 bits are 11 ) ; Notice that the value wasn't changed this time

a_pixel = (0b10010, 0b100001, 0b1011)  # pixel wasn't modified this time

这样一来，我们就成功地在一个像素中隐藏了六位数据了，我们看看像素内的数据变化情况：

a_pixel             = (0b10001, 0b100000, 0b1011) = (17, 32, 11)

a\_pixel\_with_data   = (0b10010, 0b100001, 0b1011) = (18, 33, 11)

我们可以看到，这种变化在像素级情况下变化根本就不明显。而Steganographer将不断重复这种操作，直到我们的所有数据都隐藏在图像之中。

 ![](_v_images/20200812113519670_8136.jpg)

## 注意事项

数据隐写操作完成之后，导出的图片噪声会增加很多，如果我们使用任何照片编辑软件并将其与原始图像进行比较的话，就会发现导出后图像的噪声将比原始图像大得多。那么解决这个问题的方法就是，永远不要上传/分享原始图像到互联网上！

## 许可证协议

Steganographer项目的开发与发布遵循MIT开源许可证协议。

## 项目地址

Steganographer：【[GitHub传送门](https://github.com/priyansh-anand/steganographer)】