#常用脚本示例

## zlib_decode
```
import zlib
import binascii
IDAT =("789C5D91011280400802BF04FFFF5C75294B5537738A21A27D1E49CFD17D
B3937A92E7E603880A6D485100901FB0410153350DE83112EA2D51C54CE2E585B15A
2FC78E8872F51C6FC1881882F93D372DEF78E665B0C36C529622A0A45588138833A1
70A2071DDCD18219DB8C0D465D8B6989719645ED9C11C36AE3ABDAEFCFC0ACF023E77
C17C7897667").decode("hex")
result = binascii.hexlify(zlib.decompress(IDAT))
print result
```

## hex to QR code  画二维码

```
import Image
MAX = 25
pic = Image.new("RGB",(MAX,MAX))
str = "11111110001000011011111111000001011100101101000001101110101000000000101110110
111010010000000010111011011101011101101001011101100000101010110110100000111111110101
010101011111110000000010111011100000000110100110000010100111011011110101010010000111
000000000001010000000010010011010001001110011110111001111000011101111100011001010001
100111000010101000110100011110101100000101000101100000110111011001000011100111001000
010111111101000000001101010010001111011111110111000011010110111000001000011001100011
110101110100011010011111000010111010110001110100111001011101001001110110110001100000
10110001101000110001111111011010110111011011"
i=0
for y in range (0,MAX):
    for x in range (0,MAX):
        if(str[i] == ‘1‘):
            pic.putpixel([x,y],(0,0,0))
        else:
            pic.putpixel([x,y],(255,255,255))
        i = i+1

pic.show()
pic.save("flag.png")

```
## 图片锐化
```
#coding:utf-8

import Image

img = Image.open(‘ifs.bmp‘)

X = img.size[0]
Y = img.size[1]

print X,Y

for i in range(X-2):
  for j in range(Y-2):
    a = img.getpixel((i,j))[0]+img.getpixel((i,j))[1]+img.getpixel((i,j))[2]
    b = img.getpixel((i,j+1))[0]+img.getpixel((i,j+1))[1]+img.getpixel((i,j+1))[2]
    c = img.getpixel((i,j+2))[0]+img.getpixel((i,j+2))[1]+img.getpixel((i,j+2))[2]
    if (a > b and c > b) or (a < b and c < b):
      pass
    else:
      img.putpixel((i,j),(255,255,255))

img.show()
```