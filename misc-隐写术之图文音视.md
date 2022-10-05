# 1. 隐写术之图片隐写
.png、.jpg、.bmp、.gif
1、附加式的图片隐写
2、基于文件结构的图片隐写
3、基于LSB原理的图片隐写
4、基于DCT域的JPG图片隐写
5、数字水印的隐写
6、图片容差的隐写

[深度残差网络隐写分析模型探究](https://www.anquanke.com/post/id/195323)
## 1.1. 附加式的图片隐写
在附加式的图片隐写术中，我们通常是用某种程序或者某种方法在载体文件中直接附加上需要被隐写的目标，然后将载体文件直接传输给接受者或者发布到网站上，然后接受者者根据方法提取出被隐写的消息，这一个过程就是我们这里想提到的附加式图片隐写。而在CTF赛事中，关于这种图片隐写的大概有两种经典方式，一是直接附加字符串，二是图种的形式出现。
### 1.1.1. 属性中直接附加字符
包括可更改的属性值，相机拍摄信息等。
### 1.1.2. 附加字符串隐写解答工具：
Strings    eg:  strings ctf.jpg
binwalk  eg： binwalk ctf.jpg / binwalk -e ctf.jpg
Winhex  打开查看或搜索
为什么字符串要附加在文件的后面呢?那是因为，如果图片附加在中间，有可能破坏了图片的信息，如果字符串附加在图片的头部位置，又破坏了文件头，可能导致图片无法识别。
### 1.1.3. 图种形式的隐写
图种是一种采用特殊方式将图片文件（如jpg格式）与rar文件结合起来的文件。该文件一般保存为jpg格式，可以正常显示图片，当有人获取该图片后，可以修改文件的后缀名，将图片改为rar、zip压缩文件，并得到其中的数据。
`copy /b C:\Users\DELL\Desktop\456789.png + C:\Users\DELL\Desktop\flag.zip output.png`
图种这是一种以图片文件为载体，通常为jpg格式的图片，然后将zip等压缩包文件附加在图片文件后面。因为操作系统识别的过程中是，从文件头标志，到文件的结束标志位，当系统识别到图片的结束标志位后，默认是不再继续识别的，所以我们在通常情况下只能看到它是只是一张图片。
工具 
1- binwalk cqzb.jpg   / binwalk -e cqzb.jpg
2- 使用winhex16进制编辑器提取ZIP文件
### 1.1.4. JPG图片的文件头和结束标志：
￼
FF D8 FF E1就是JPG图片的文件头，一般当我们看到文件开头是如此的格式，我们就能认为这是一个JPG图片了。以 03 FF D9为结束标志，这是JPG图片的结束标志位。
### 1.1.5. ZIP文件的文件头和结束标志：
50 4B 03 04就是ZIP文件的文件头，一般以PK表示。上文我们讲述了，JPG图片的结束标识是03 FF D9；ZIP文件的文件头是50 4B 03 04，我们只需要在winhex中找到ZIP文件的文件头即可，滑动滚条到最底下。上文讲了一般附加的位置是在原本文件的后面，所以我们果断滑动滚动条到最后，选中数据，右键单击，选择编辑，复制选块到新文件，保存新文件为zip格式命名规则，解压缩后就能得到flag

## 1.2. 基于文件结构的图片隐写
### 1.2.1. 背景知识
首先这里需要明确一下我这里所说的文件结构是什么意思。文件结构特指的是图片文件的文件结构。我们这里主要讲的是PNG图片的文件结构。
PNG，图像文件存储格式，其设计目的是试图替代GIF和TIFF文件格式，同时增加一些GIF文件格式所不具备的特性。PNG用来存储灰度图像时，灰度图像的深度可多到16位，存储彩色图像时，彩色图像的深度可多到48位，并且还可存储多到16位的α通道数据。对于一个正常的PNG图片来讲，其文件头总是由固定的字节来表示的，以16进制表示即位 89 50 4E 47 0D 0A 1A 0A，这一部分称作文件头。
#### 1.2.1.1. 标准的PNG文件结构应包括：
    + PNG文件标志
    + PNG数据块
    PNG图片是有两种数据块的，一个是叫关键数据块，另一种是辅助数据块。正常的关键数据块，定义了4种标准数据块，个PNG文件都必须包含它们。
它们分别是长度，数据块类型码，数据块数据，循环冗余检测即CRC。我们这里重点先了解一下，png图片**文件头数据块**以及png图片**IDAT块**，这次的隐写也是以这两个地方位基础的。png图片文件头数据块，即IHDR，这是PNG图片的第一个数据块，一张PNG图片仅有一个IHDR数据块，它包含了哪些信息呢？IHDR中，包括了图片的宽，高，图像深度，颜色类型，压缩方法等等。
![ihdr](_v_images/20191209155953580_15536.png)
如图中蓝色的部分即IHDR数据块。
#### 1.2.1.2. IDAT 数据块
它存储实际的数据，在数据流中可包含多个连续顺序的图像数据块。这是一个可以存在多个数据块类型的数据块。它的作用就是存储着图像真正的数据。
因为它是可以存在多个的，所以即使我们写入一个多余的IDAT也不会多大影响肉眼对图片的观察
### 1.2.2. 高度被修改引起的隐写
背景知识中，我们了解到，图片的高度，宽度的值存放于PNG图片的文件头数据块，那么我们就是可以通过修改PNG图片的高度值，来对部分信息进行隐藏的。
![010editor模板](_v_images/20191209164010845_10732.png)
使用winhex或者010Editor等编辑器打开图片。但我们推荐后者，因为它提供了不同文件的模板(需要事先准备模板)，通过加载png模板，我们可以直观的知道哪里是PNG的长度字段或宽度字段，它提供了hex字符串到字段名的映射，更便于我们进行修改。在修改文件后，需要利用CRC Calculator对CRC校验码进行重新计算赋值，以防图片被修改后，自身的CRC校验报错，导致图片不能正常打开。用010Editor打开图片，运行PNG模板(需要事先准备模板)
我们找到IHDR数据块，并翻到struct IHDR Ihdr位置，修改height的值到一个较大的值，如从700修改到800。

### 1.2.3. 隐写信息以IDAT块加入图片
在背景知识中，我们提到了一个重要的概念就是图片的IDAT块是可以存在多个的，这导致了我们可以将隐写西信息以IDAT块的形似加入图片。
工具pngcheck可以验证PNG图片的完整性
（通过检查内部CRC-32校验和&bra;比特&ket;)和解压缩图像数据；它能够转储几乎所有任选的块级别信息在该图像中的可读数据。
我们使用pngcheck -v hidden.png 如此的命令对图片进行检测
发现异常，并判断异常的原因
我们会发现，图片的的数据块形式是如下的
```
Type: IHDR
Size: 13
CRC ： 5412913F

Pos ： 33
Type: IDAT
Size: 10980
CRC ： 98F96EEB

Pos ： 11025
Type: IEND
Size: 0
CRC ： AE426082
```
我们会惊讶的发现pos为11025的size居然为0，这是一块有问题的地方，说明相应的块是无法用肉眼看到的，也即隐藏的内容。可以通过脚本对隐藏内容进行提取。
编写脚本并提取内容
```
#!/usr/bin/python

from struct import unpack
from binascii import hexlify, unhexlify
import sys, zlib

\# Returns [Position, Chunk Size, Chunk Type, Chunk Data, Chunk CRC]
def getChunk(buf, pos):
    a = []
    a.append(pos)
    size = unpack('!I', buf[pos:pos+4])[0]
    # Chunk Size
    a.append(buf[pos:pos+4])
    # Chunk Type
    a.append(buf[pos+4:pos+8])
    # Chunk Data
    a.append(buf[pos+8:pos+8+size])
    # Chunk CRC
    a.append(buf[pos+8+size:pos+12+size])
    return a

def printChunk(buf, pos):
    print 'Pos : '+str(pos)+''
    print 'Type: ' + str(buf[pos+4:pos+8])
    size = unpack('!I', buf[pos:pos+4])[0]
    print 'Size: ' + str(size)
    #print 'Cont: ' + str(hexlify(buf[pos+8:pos+8+size]))
    print 'CRC : ' + str(hexlify(buf[pos+size+8:pos+size+12]).upper())
    print

if len(sys.argv)!=2:
    print 'Usage: ./this Stegano_PNG'
    sys.exit(2)

buf = open(sys.argv[1]).read()
pos=0

print "PNG Signature: " + str(unpack('cccccccc', buf[pos:pos+8]))
pos+=8

chunks = []
for i in range(3):
    chunks.append(getChunk(buf, pos))
    printChunk(buf, pos)
    pos+=unpack('!I',chunks[i][1])[0]+12

decompressed = zlib.decompress(chunks[1][3])
\# Decompressed data length = height x (width * 3 + 1)
print "Data length in PNG file : ", len(chunks[1][3])
print "Decompressed data length: ", len(decompressed)

height = unpack('!I',(chunks[0][3][4:8]))[0]
width = unpack('!I',(chunks[0][3][:4]))[0]
blocksize = width * 3 + 1
filterbits = ''
for i in range(0,len(decompressed),blocksize):
    bit = unpack('2401c', decompressed[i:i+blocksize])[0]
    if bit == '\x00': filterbits+='0'
    elif bit == '\x01': filterbits+='1'
    else:
        print 'Bit is not 0 or 1... Default is 0 - MAGIC!'
        sys.exit(3)

s = filterbits
endianess_filterbits = [filterbits[i:i+8][::-1] for i in xrange(0, len(filterbits), 8)]

flag = ''
for x in endianess_filterbits:
    if x=='00000000': break
    flag += unhexlify('%x' % int('0b'+str(x), 2))

print 'Flag: ' + flag`
```
脚本如上，get flag

是否可以将一张二维码以IDAT块的形式写入图片呢？
试着将信息以IDAT块的形式写入图片，保存后，重新打开图片，我们就能看到被隐藏的内容

### 1.2.4. 文件格式缺失&GIF隐写
某些无法打开的图片，将其拖入winhex中查看。在CTF中有的时候会需要我们去修复图片，这对我们对于图片的文件结构要有了解。找到各类文件的默认格式，然后对照这个破损的文件对其进行修复，例如gif的文件格式。
![GIF头](_v_images/20191209162451885_7972.png)
gif和别的图片最大的区别就是gif是动态图，它是可以由多帧组成的可以顺序播放的，有的题就是把播放的时间弄得特别慢，几乎就不会动的，所以我们可以用工具一帧一帧的观察图片。 上面提到的Stegsolve就带有这种功能。或者使用GIF分帧工具。
111Stegsolve——Analyse——FrameBrower就可以看到是有8帧的图片，最终得到信息

## 1.3. 基于LSB原理的图片隐写
LSB，LSB也就是最低有效位 (Least Significant Bit)。原理就是图片中的像数一般是由三种颜色组成，即三原色，由这三种原色可以组成其他各种颜色，例如在PNG图片的储存中，每个颜色会有8bit，LSB隐写就是修改了像数中的最低的1bit，在人眼看来是看不出来区别的，也把信息隐藏起来了。譬如我们想把’A’隐藏进来的话，如下图，就可以把A转成16进制的0x61再转成二进制的01100001，再修改为红色通道的最低位为这些二进制串。
![lsb隐写原理](_v_images/20191209171008861_15689.png)

### 1.3.1. 简单的LSB隐写
Stegosolve介绍
CTF中，最常用来检测LSB隐写痕迹的工具是Stegsolve，这是一款可以对图片进行多种操作的工具，包括对图片进行xor,sub等操作，对图片不同通道进行查看等功能。用Stegsolve打开图片，在不同的通道中切换。

我们如何实现这种LSB隐写的？是否可以通过photoshop这样的工具实现？
查阅更多关于LSB隐写的资料。
### 1.3.2. 有一点难度的LSB隐写
我们从第一个部分可以知道，最简单的隐写我们只需要通过工具Stegsolve切换到不通通道，我们就可以直接看到隐写内容了，那么更复杂一点就不是这么直接了，而是只能这样工具来查看LSB的隐写痕迹，再通过工具或者脚本的方式提取隐写信息。
从Stegsolve中打开图片
首先 点击上方的FIle菜单，选择open，在所需要用的图片where.bmp然后 切换到不同通道，通过痕迹来判断是否是LSB隐写，分析是否有可能是LSB隐写，我们开始点击下面的方向按钮，切换到不同通道，我们逐渐对比不同通道我们所看到的图片是怎么样子的。我们发现在不同的同道中人出现了异常情况，我们这里基本可以断定就是LSB隐写了
![dsfd](_v_images/20191209161838844_11035.png)
编写代码提取信息
因为是LSB隐写，我们只按位提取RGB的最低位即可，代码如下：
```
from PIL import Image
im = Image.open("extracted.bmp")
pix = im.load()
width, height = im.size

extracted_bits = []
for y in range(height):
    for x in range(width):
        r, g, b = pix[(x,y)]
        extracted_bits.append(r & 1)
        extracted_bits.append(g & 1)
        extracted_bits.append(b & 1)

extracted_byte_bits = [extracted_bits[i:i+8] for i in range(0, len(extracted_bits), 8)]
with open("extracted2.bmp", "wb") as out:
    for byte_bits in extracted_byte_bits:
                byte_str = ''.join(str(x) for x in byte_bits)
        byte = chr(int(byte_str, 2))
        out.write(byte)
```
打开我们需要提取信息的的图片，y，x代表的是图片的高以及宽度，进行一个循环提取。
运行代码，extracted.py，打开图片即可。

我们这里用的LSB隐均对R,G,B，三种颜色都加以修改是否可以只修改一个颜色？
参考2016 HCTF的官方Writeup学习如何实现将一个文件以LSB的形式加以隐写。

## 1.4. 基于DCT域的JPG图片隐写
### 1.4.1. 背景知识
工具：
Stegdetect、Jphide、Outguess
JPEG图像格式使用离散余弦变换（Discrete Cosine Transform，DCT）函数来压缩图像，而这个图像压缩方法的核心是：通过识别每个8×8像素块中相邻像素中的重复像素来减少显示图像所需的位数，并使用近似估算法降低其冗余度。因此，我们可以把DCT看作一个用于执行压缩的近似计算方法。因为丢失了部分数据，所以DCT是一种有损压缩（Loss Compression）技术，但一般不会影响图像的视觉效果。
在这个隐写家族中，常见的隐写方法有JSteg、JPHide、Outguess、F5等等
### 1.4.2. Stegdetect
实现JPEG图像Jphide隐写算法工具有多个，比如由Neils Provos开发通过统计分析技术评估JPEG文件的DCT频率系数的隐写工具Stegdetect，它可以检测到通过JSteg、JPHide、OutGuess、Invisible Secrets、F5、appendX和Camouflage等这些隐写工具隐藏的信息，并且还具有基于字典暴力破解密码方法提取通过Jphide、outguess和jsteg-shell方式嵌入的隐藏信息。
### 1.4.3. JPHS
一款JPEG图像的信息隐藏软件JPHS，它是由Allan Latham开发设计实现在Windows和Linux系统平台针对有损压缩JPEG文件进行信息加密隐藏和探测提取的工具。软件里面主要包含了两个程序JPHIDE和JPSEEK， JPHIDE程序主要是实现将信息文件加密隐藏到JPEG图像功能，而JPSEEK程序主要实现从用JPHIDE程序加密隐藏得到的JPEG图像探测提取信息文件，Windows版本的JPHS里的JPHSWIN程序具有图形化操作界面且具备JPHIDE和JPSEEK的功能。
### 1.4.4. Outguess
Outgusee算法是Niels Provos针对Jsteg算法的缺陷提出的一种方法：
嵌入过程不修改ECT系数值为0，1的DCT系数，利用为随机数发生器产生间隔以决定下一个要嵌入的DCT系数的位置
纠正过程消除对效应的出现，对应的，也有针对该算法的隐写工具，名字也叫Outguess。

## 1.5. 数字水印隐写
### 1.5.1. 背景知识
数字水印（digital watermark）技术，是指在数字化的数据内容中嵌入不明显的记号。特征是，被嵌入的记号通常是不可见或不可察的，但是可以通过计算操作检测或者提取。
盲水印，是指人感知不到的水印，包括看不到或听不见（没错，数字盲水印也能够用于音频）。其主要应用于音像作品、数字图书等，目的是，在不破坏原始作品的情况下，实现版权的防护与追踪。
对图像进行傅里叶变换，起始是一个二维离散傅里叶变换，图像的频率是指图像灰度变换的强烈程度，将二维图像由空间域变为频域后，图像上的每个点的值都变成了复数，也就是所谓的复频域，通过复数的实部和虚部，可以计算出幅值和相位，计算幅值即对复数取模值，将取模值后的矩阵显示出来，即为其频谱图。但是问题来了，复数取模后，数字有可能变的很大，远大于255，如果数据超过255，则在显示图像的时候会都当做255来处理，图像就成了全白色。因此，一般会对模值再取对数，在在0~255的范围内进行归一化，这样才能够准确的反映到图像上，发现数据之间的差别，区分高频和低频分量，这也是进行傅里叶变换的意义。

### 1.5.2. 频域盲水印隐写
提取盲水印
`python decode.py --original <original  file> -- < file> --result <result file>`
original 是输入原图， 是之后跟的是加入了水印的图， result是保存水印图片。
```
\# coding=utf-8
import cv2
import numpy as np
import random
import os
from argparse import ArgumentParser
ALPHA = 5
def build_parser():
    parser = ArgumentParser()
    parser.add_argument('--original', dest='ori', required=True)
    parser.add_argument('--', dest='img', required=True)
    parser.add_argument('--result', dest='res', required=True)
    parser.add_argument('--alpha', dest='alpha', default=ALPHA)
    return parser
def main():
    parser = build_parser()
    options = parser.parse_args()
    ori = options.ori
    img = options.img
    res = options.res
    alpha = options.alpha
    if not os.path.isfile(ori):
        parser.error("original  %s does not exist." % ori)
    if not os.path.isfile(img):
        parser.error(" %s does not exist." % img)
    decode(ori, img, res, alpha)
def decode(ori_path, img_path, res_path, alpha):
    ori = cv2.imread(ori_path)
    img = cv2.imread(img_path)
    ori_f = np.fft.fft2(ori)
    img_f = np.fft.fft2(img)
    height, width = ori.shape[0], ori.shape[1]
    watermark = (ori_f - img_f) / alpha
    watermark = np.real(watermark)
    res = np.zeros(watermark.shape)
    random.seed(height + width)
    x = range(height / 2)
    y = range(width)
    random.shuffle(x)
    random.shuffle(y)
    for i in range(height / 2):
        for j in range(width):
            res[x[i]][y[j]] = watermark[i][j]
    cv2.imwrite(res_path, res, [int(cv2.IMWRITE_JPEG_QUALITY), 100])
if __name__ == '__main__':
    main()
```
如果是像HCTF那样的隐写题，只需要有mathlab这个强大的工具，再运用提取盲水印的代码，是不需要原图的,代码如下。
```
A = imread('3.bmp','bmp');
fftA = fft2(A);
imshow(fftshift(fftA))
imshow(fft(rgb2gray(imread('shimakaze.bmp'))), [1,2]);
```

请查阅关于空域盲水印的资料
试着对频域盲水印攻击，如截屏、倒转等操作，再进行提取水印，看看水印是否被破坏。

## 1.6. 图片容差隐写
工具：Beyond Compare
### 1.6.1. 背景知识
容差，在选取颜色时所设置的选取范围，容差越大，选取的范围也越大，其数值是在0-255之间。
### 1.6.2. 容差比较的隐写
beyond compare是一款很适合用来对两张图片进行比较的工具，就图片而言，它支持容差、范围、混合等模式。选择容差模式，并调整容差大小，
如果在CTF赛场中，就隐写这一部分，出题人给于两张或者多张图片，一般都是需要对图片的内容进行比较的。

Stegsolve 也有图片的比较的功能，是否能完成这个隐写？如果不可以为什么？
## 1.7. CMD-C彩图隐写方案
[未验证的隐写——cmd-c彩图隐写算法系列](https://www.anquanke.com/post/id/194988)
本文提出了当前经典的彩图隐写方案，它具有聚类修改方向的特点，是第一个公认的彩色图隐写方案，为后续彩图隐写术和隐写分析工作提供了清晰的思路，具有深远的意义。现代灰度图像隐写方案是在最小化失真函数框架下设计的，流行的隐写方案包括HUGO、WOW、S-UNIWARD、MG、HILL等。在彩色图像中，一些临时的隐写方案假定可以将每个颜色通道独立地视为灰度图像进行嵌入。由于这些方案未考虑颜色通道之间的相关性，因此可能会留下明显的隐写痕迹，而这些痕迹很容易被最新的彩色隐写分析模型捕获，例如CRM（color rich model，彩色富模型）和CCRM（channel correlation rich model，色彩丰富的通道相关模型）等。2015年，有学者提出了一个可应用于灰度图隐写领域的CMD（clustering modiﬁcation directions，聚类修改方向）隐写策略，它保证了嵌入秘密信息后局部区域修改的方向相同，从而使得隐写痕迹不易被检测到，这有效提高了灰度图像隐写术的安全性。

# 2. 隐写术之音频隐写
音频文件的隐写，现如今除了通过频谱以外，还有一种是在时域中进行的信息隐写，在PCTF中，或者汪神的OJ中有几个挺经典例子，大家可以尝试去做看看。
另一方面，音频还有高级随机化技术的LSB嵌入技术，但是这还是不是很成熟。
## 2.1. 众所周知的摩尔斯电码
摩尔斯电码（又译为摩斯密码，Morse code）是一种时通时断的信号代码，通过不同的排列顺序来表达不同的英文字母、数字和标点符号。在过去它以电报的形式来发送消息，如今当听到这样的电报你还能解出的他它的明文吗？
当我们确认是摩尔斯电报之后，我先用Adobe Auditon打开目标文件，并观察波形，以长的代表代表横线，短的代表点，大的间隔是空格，抄写下摩尔斯电码
当然，我们也可以利用一些！在线的工具翻译摩尔斯电码，也可以利用JPK这样的工具进行翻译。

## 2.2. 利用MP3stego进行的数据隐写
MP3stego是著名的音频数据隐写工具，支持常见的压缩音频文件格式如mp3的数据嵌入，它采用的是一种特殊的量化方法，并且将数据隐藏在MP3文件的奇偶校验块中。
使用方法
用MP3Stego进行加密解密：
加密：encode -E 加密文本 -P 密码  wav文件 mp3文件
解密：decode -X -P  密码  mp3文件
- 在工具文件中找到mp3stego，将目标文件拷贝到工具的目录下
- 尝试提取隐藏信息，最后的flag是flag{I_love_you}
首先先将目标文件复制到MP3stego这个工具的目录下
在CMD下使用Decode.exe程序进行提取
最后打开目录下的love.mp3.txt文件就能看到隐写内容了

## 2.3. 频谱图的音频隐写
频谱是频率谱密度的简称，是频率的分布曲线。复杂振荡分解为振幅不同和频率不同的谐振荡，这些谐振荡的幅值按频率排列的图形叫做频谱。
在CTF中，我们可以单独只对一个声道中，隐写进信息
直接用Adobe Aud ton打开目标文件
调整到频谱视图，调整大小，直到能清晰的看到隐写内容
尝试提取隐藏信息，flag
有必要只得一提的是，我们会发现只有一个音轨有隐写信息，那是因为我只对一个通道隐写进了信息，一般人也不会去分析这个wav文件的频谱吧。
# 3. 隐写术之视频隐写
目前在CTF赛事中较为常出现的视频隐写，一般都是将一场带有隐写信息的图片，嵌入视频中，我们所需要做的就是将这个图片从视频分离出来，然后在分析我们分离出来的文件是什么，之后的操作可能会涉及到密码编码，图片隐写等知识点。
另一方面，我们分离文件，如果单独对视频来说ffmpeg是一个很好的工具，这里我使用的是foremost ，一款linux下的命令行工具，当然我们也可以使用binwalk或者dd等工具，正如我们图片隐写中教大家分离图片所用的方法一样。
区别是，ffmpeg将视频分解成一张一张的图片，foremost是一个基于文件头和尾部信息以及文件的内建数据结构恢复文件的命令行工具

foremost powpow.mp4
steghide extract -sf thing.jpg -p password提取图片隐写内容

# 4. 隐写术之文本类 
.pdf、.txt、.doc、.docx
Office系列软件作为优秀的办公软件为我们提供了极大的便利，其中的Word、Excel、PowerPoint提供了许多在文档中隐藏数据的方法，比如批注、个人信息、水印、不可见内容、隐藏文字和定制的XML数据
## 4.1. 利用隐藏文本功能进行隐写
菜单栏中选择，并单击File（文件）->Tool（工具）->Option（选项） - 找到 隐藏文字 功能，选择这个功能，点击保存
## 4.2. word文档的xml转换
我们可以将word文档转换成xml格式，当然反过来我们也可以将xml转换成word文档，这导致了我们如果重新打包为word文档的过程中，有可能被隐藏进其他数据。
查看方法
file file.docx 
7z x file.docx -oout
我们会发现又flag.txt的文件被打包在file.docx中，
直接用7z等压缩包工具打开file.docx
## 4.3. PDF文件隐写
![PDF文件结构](_v_images/20191209171717934_22680.png)
工具wbStego4open 在工具目录中找到 wbStego4open，使用工具载入文档，
Step 1 是文件介绍
Step 2 中，我们选择Decode，
Step 3 我们选择目标文件
Step 4 输入加密密码，这里我是空密码，直接跳过
Step 5 为保存文件为 flag.txt
# 5. 文件头文件尾总结
JPEG(jpg)，文件头：FFD8FF文件尾：FFD9
PNG(png)，文件头：89504E47文件尾：AE426082
GIF(gif)，文件头：47494638文件尾：003B
ZIPArchive(zip)，文件头：504B0304文件尾：504B
TIFF(tif)，文件头：49492A00文件尾：
WindowsBitmap(bmp)，文件头：424D文件尾：
CAD(dwg)，文件头：41433130文件尾：
AdobePhotoshop(psd)，文件头：38425053文件尾：
RichTextFormat(rtf)，文件头：7B5C727466文件尾：
XML(xml)，文件头：3C3F786D6C文件尾：
HTML(html)，文件头：68746D6C3E
Email[thoroughonly](eml)，文件头：44656C69766572792D646174653A
OutlookExpress(dbx)，文件头：CFAD12FEC5FD746F
Outlook(pst)，文件头：2142444E
MSWord/Excel(xls.or.doc)，文件头：D0CF11E0
MSAccess(mdb)，文件头：5374616E64617264204A
WordPerfect(wpd)，文件头：FF575043
AdobeAcrobat(pdf)，文件头：255044462D312E
Quicken(qdf)，文件头：AC9EBD8F
WindowsPassword(pwl)，文件头：E3828596
RARArchive(rar)，文件头：52617221
Wave(wav)，文件头：57415645
AVI(avi)，文件头：41564920
RealAudio(ram)，文件头：2E7261FD
RealMedia(rm)，文件头：2E524D46
MPEG(mpg)，文件头：000001BA
MPEG(mpg)，文件头：000001B3
Quicktime(mov)，文件头：6D6F6F76
WindowsMedia(asf)，文件头：3026B2758E66CF11
MIDI(mid)，文件头：4D546864


# 6. 好用的隐写分析工具
## 6.1. zsteg
 检测 stegano & BMP中的隐藏数据 安装：gem install zsteg
可检测的隐写
 + PNG & BMP中的LSB
 + zlib压缩数据
 + OpenStego
 + 伪装 1.2.1版
 + 带 Eratosthenes set的 LSB。
简单 LSB
`zsteg flower_rgb3.png`
3b,rgb,lsb,xy. . text:"SuperSecretMessage"
多结果文件
` zsteg cats.png`
meta F. . ["Z" repeated 14999985 times]
meta C. . text:"Fourth and last cat is Luke"
wbStego甚至分布式
`zsteg wbstego/wbsteg_noenc_even.bmp 1b,lsb,bY -v`
## 6.2. Binwalk
是一个固件的分析工具，多用于逆向工程、取证、隐写分析。
$ binwalk firmware.bin  //最简单的操作
$ binwalk --enable-plugin=zlib firmware.bin  //有些签名无法识别，利用zlib插件扫描zlib压缩包可识别
$ binwalk -y filesystem firmware.bin  //指定字符串“filesystem”搜索(正则)，-Y 输出结果只包含文本字符串
$ binwalk -x filename firmware.bin  //排除搜索结果中指定'filename'字符串(正则)
$ binwalk -y filesystem -x jffs2 firmware.bin  //输出既包含'filesystem'又排除'jffs2'的字符串
$ binwalk --dd='zip archive:zip:unzip %e' firmware.bin  //<type>:<extension>[:<command>]. type 是签名中描述的小写字符串（支持正则表达式）;extension 是将数据保存到磁盘时使用的文件扩展名;command 是当数据已保存到磁盘后可选的命令执行语句
$ binwalk -e firmware.bin  //自动提取
$ binwalk --extract=./my_extract.conf firmware.bin  //自定义提取规则'my_extract.conf'
$ binwalk -Me firmware.bin  //递归提取(8层)
$ binwalk -A firmware.bin  //扫描与功能相关联的各种框架操作码
$ binwalk -W firmware1.bin firmware2.bin firmware3.bin  //比较, 在文件当中相同字节的是绿色显示，不同的是红色显示，蓝色表示只是有些文件当中的不同部分
$ binwalk -S firmware.bin  //字符串搜索
$ binwalk -E firmware.bin  //熵分析
$ binwalk -AE firmware.bin  //签名或字符串以及熵分析
$ binwalk --heuristic firmware.bin  //启发式扫描, 加密或压缩的高熵的数据块进行分类
$ binwalk --list-plugins  //插件列表
$ binwalk --enable-plugin=foo firmware.bin  //启用插件扫描
$ binwalk --disable-plugin=foo firmware.bin  //禁用插件扫描
$ binwalk -f binwalk.log firmware.bin  //日志记录功能
$ sudo binwalk -u  //升级binwalk
## 6.3. WinHex
Winhex是在Windows下执行的十六进制编辑软件，此软件功能很强大，有完好的分区管理功能和文件管理功能。能自己主动分析分区链和文件簇链。在CTF中一般用来查看文件头格式、直接修改16进制数据，等等。更多...

## 6.4. 010 Editor
010 Editor是一款非常强大的文本/十六进制编辑器，除了文本/十六进制编辑外，还包括文件解析、计算器、文件比较等功能，但它真正的强大之处还在于文件的解析功能。我们可以使用010Editor官方网站提供的解析脚本(Binary Template)对avi、bmp、png、exe等简单格式的文件进行解析，当然也可以根据需求来自己编写文件解析脚本。

## 6.5. Stegsolve
Stegsolve是一款图片分析工具，具体功能如下：
功能
Stegdetect
stegdetect是一种数字图像隐写分析工具,主要实现JPEG图像的隐秘信息的嵌入的检测。更多...
功能：
q – 仅显示可能包含隐藏内容的图像
n – 启用检查JPEG文件头功能，以降低误报率。如果启用，所有带有批注区域的文件将被视为没有被嵌入信息。
如果JPEG文件的JFIF标识符中的版本号不是1.1，则禁用OutGuess检测。
s – 修改检测算法的敏感度，该值的默认值为1。检测结果的匹配度与检测算法的敏感度成正比，
算法敏感度的值越大，检测出的可疑文件包含敏感信息的可能性越大。
d – 打印带行号的调试信息。
t – 设置要检测哪些隐写工具（默认检测jopi），可设置的选项如下：
j – 检测图像中的信息是否是用jsteg嵌入的。
o – 检测图像中的信息是否是用outguess嵌入的。
p – 检测图像中的信息是否是用jphide嵌入的。
i – 检测图像中的信息是否是用invisible secrets嵌入的。

## 6.6. ffmpeg
ffmpeg作为媒体文件处理软件，基本用法: ffmpeg -i INPUTfile [OPTIONS] OUTPUTfile
输入输出文件通常就是待处理的多媒体文件了。可以是纯粹的音频文件，纯粹的视频文件，或者混合的。ffmpeg支持绝大部分的常见音频,视频格式，像常见的mpeg,AVI封装的DIVX和Xvid等等，具体的格式支持列表可以使用ffmpeg -formats查看或直接查阅文档。更多...

## 6.7. MSU Stego
用于对 AVI 文件进行隐写\提取操作. 官方介绍如下：
MSU StegoVideo allows hiding any file in a video sequence.Different popular codecs were analyzed and an algorithm, providing the smallest data loss after compression, was chosen. Convolutional codes with Viterbi decoding are used to correct occurred errors. 更多...
Main features
Small video distortions after hiding info.
It is possible to extract info after video compression.
Information is protected with passcode.

## 6.8. QR Reader
二维码扫描工具可自定义参数，可以自动识别二维码反色，自动识别电脑屏幕二维码，识别率比手机扫码高，非常强大。

## 6.9. MP3Stego
用于对 MP3 音频文件进行隐写、提取等操作。更多...
用法：
encode -E hidden_text.txt -P pass svega.wav svega_stego.mp3
decode -X -P pass svega_stego.mp3
把数据文件和wav文件压缩成mp3 文件
文件要存放在安装目录下
encode -E 数据文件名称 载体名称 携密文件名称     需输入密钥
decode -X 携密文件名称

## 6.10. wbStego（软件）
安装，选择encode 加密，选择decode解密
首先创建文件，选择文件目录，选择文件载体图片，进行加密，生成目标文件
解密，选择目标文件，无加密密码，生成解密文件。

## 6.11. Hide and seek  利用像素LSB来隐藏  （命令行）
Hide hidden.txt ARDALA.GIF   执行命令生成OUTFILE.GIF 文件
seek OUTFILE.GIF out.txt 解密生成out.txt 和hidden.txt 内容相同

## 6.12. F5 针对 jpeg图像 （作为载体）（命令行）
运行ms_e.bat 把bmg图像 嵌入携密信息jpeg图像
运行ms_d.bat 提取秘密信息得到output.txt

###S-Tools 音频文件（作为载体） 采用LSB占用比特位（软件）
拖拽wav音频文件到窗口中，再拖拽要待隐藏文件，需要输入加密密码
完成后，右击，reveal显示秘密信息，

## 6.13. Exiftool
有时重要的东西隐藏在图像或文件的元数据中，exiftool可以非常有助于查看文件的元数据。
exiftool file ：显示给定文件的元数据
** Exiv2**
一种类似于exiftool的工具。
它可以安装，apt但源可以在github上找到。
exiv2 file ：显示给定文件的元数据

## 6.14. Wavsteg
WavSteg是一个python3工具，可以隐藏wav文件中的数据和文件，也可以从wav文件中提取数据。
你可以从github得到它
python3 WavSteg.py -r -s soundfile -o outputfile ：从wav声音文件中提取数据并将数据输出到新文件中

## 6.15. Fcrackzip
有时提取的数据是受密码保护的zip，此工具强制执行zip存档。它可以安装，apt源可以在github上找到。
fcrackzip -u -D -p wordlist.txt file.zip ：使用给定wordlist中的密码强制给定zip文件

## 6.16. StegCracker
使用steghide强制密码的工具

## 6.17. steghide暴力破解密码的脚本
```
# -*- coding: utf8 -*-
#author:pcat
#http://pcat.cnblogs.com
from subprocess import *
 
def foo():
    stegoFile='rose.jpg'
    extractFile='hide.txt'
    passFile='english.dic'
 
    errors=['could not extract','steghide --help','Syntax error']
    cmdFormat='steghide extract -sf "%s" -xf "%s" -p "%s"'
    f=open(passFile,'r')
 
    for line in f.readlines():
        cmd=cmdFormat %(stegoFile,extractFile,line.strip())
        p=Popen(cmd,shell=True,stdout=PIPE,stderr=STDOUT)
        content=unicode(p.stdout.read(),'gbk')
        for err in errors:
            if err in content:
                break
        else:
            print content,
            print 'the passphrase is %s' %(line.strip())
            f.close()
            return
 
if __name__ == '__main__':
    foo()
    print 'ok'
    pass
```