Author Rodbert(Robert Tomkowski)

# Task: Lossless
CTF: Backdoorctf16  
Author: Arpit Singla  
Points: 100  
Category: Stegano  

Description:  
> d4rth used his dirty methods to hide a secret in a png file. He is cleverly trying to divert your focus from challenge, but the force is strong with you. Now extract the flag from these images, my young padawan.
http://hack.bckdr.in/LOSSLESS/original.png
http://hack.bckdr.in/LOSSLESS/encrypted.png

### TL;DR
Subtract both images. LSB is changed by 1 in encrypted.png.

### Solution
We are given two images seemingly identical.

original.png:  
![original.png](original.png)

encrypted.png:  
![encrypted.png](encrypted.png)

But when we compare them with imagemagick…
```shell
compare original.png encrypted.png difference.png
```
difference.png:  
![difference.png](difference.png)

In upper left corner we will get encrypted flag.

After cleaning flag i got image like this:  
![solution.png](solution.png)

Picture is size 49x7 and has black pixels on top, for me it looks like vertical written, binary encoded ascii letters. (First is capital because starts with 10, next are lower, also we can spot spaces 0100000)

Let’s try that idea.  
In this point we could decrypt it by hand in like 5 min, but we are lazy, aren’t we?  
I turned our encrypted cleaned flag by 90 counter-clockwise and exported it as bmp encoded with 3 bytes.  
Next we open this image in hex editor, copy image body, paste it to your favorite shell, replace all ‘00 00 00’ by ‘1‘ and ‘ff ff ff’ by ‘0’ and at the end remove spaces. After that if we want to use some built in magick functions that changes binary to ascii we need to add leading zeros to every group of seven.

Handy implementation in python 2.7:
```python
data = 'some hexes from image'
data = data.replace('00 00 00', '1').replace('ff ff ff', '0').replace(' ', '')
data = ''.join['0' + data[i:i+7] for i in xrange(0, len(data)/7)]
import binascii
binascii.unhexlify('%x' % int(data,2))
```
And the flag is ours, don’t forget to SHA256 it.
