Author: Rodbert (Robert Tomkowski)

Lossless
========

CTF: Backdoorctf16  
Link: https://backdoor.sdslabs.co/challenges/LOSSLESS  
Author: Arpit Singla  
Points: 100  
Category: stegano, image


Description
-----------

>   d4rth used his dirty methods to hide a secret in a png file. He is
>   cleverly trying to divert your focus from challenge, but the force
>   is strong with you. Now extract the flag from these images, my young
>   padawan.
>
>   http://hack.bckdr.in/LOSSLESS/original.png
>
>   http://hack.bckdr.in/LOSSLESS/encrypted.png


tl;dr
-----

Subtract both images. LSB is changed by 1 in `encrypted.png`.


Solution
--------

We have been given two images, seemingly identical:

![original](img/original.png)
and
![encrypted](img/encrypted.png)

When compared with ImageMagick, by:
```shell
compare original.png encrypted.png difference.png
```
an encrypted message can be spotted in the upper left corner.

![The difference between the given images](img/difference.png)

With a bit of work in GIMP we end up with the following image:

![The solution](img/solution.png)

The data is a 49x7 binary matrix, which looks like vertically written,
binary encoded, ascii letters. The first one is a capital letter,
as it begins with `10`, while the others are lower case letters
(begin with `11`). Also we can see spaces (`0100000`).

Let's decode the hidden message. We've rotated the image by 90 degrees
counter-clockwise and exported it in BMP format. Then, treating the image
as a data hexdump, we've replaced all `00 00 00` with `1` and all
`ff ff ff` with 0. Lastly, we've added leading zeros to every group
of seven bits.

Utilitarian script to transform the data:
```python
data = 'some hexes from image'
data = data.replace('00 00 00', '1').replace('ff ff ff', '0').replace(' ', '')
data = ''.join['0' + data[i:i+7] for i in xrange(0, len(data)/7)]
import binascii
print binascii.unhexlify('%x' % int(data,2))
```

The decoded data is printable ASCII string, which happens to be the flag.
