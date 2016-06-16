Author Pymac(Maciej Pytel)

# Task: CRC

CTF: Backdoorctf16  
Author: IamLupo  
Points: 250  
Category: Misc  
Description:
Backdoor user IamLupo wanted to submit his challenge for BackdoorCTF16 but he was struck by lightening and his challenge file shattered into 26 pieces. We want you to recover it for us (and maybe get the flag while you do it). Thanks in advance! Here is the zipped file:
http://hack.bckdr.in/CRC/challenge.zip

###TL;DR

27 encrypted zip archives, each contains 5 character txt file. All files
together make a php program. CRC32 checksum can be retrieved from zip archives.
Brute force is reasonable, but takes a long time (too long).
Use knowledge of php and already decrypted text to guess some characters in
encrypted archives to speed up brute force attack.

###Solution

In this task we get a zip file (challenge.zip) and we need to somehow get a
flag out of it. The archive extracts to 27 separate .zip files, named 0.zip
to 26.zip. Obviously we need to get the content of those archives, but they're
password protected...

The name of the challenge suggests us what to do next. A zip archive contains
a CRC32 checksum for each archived file. Surprisingly most popular zip tools
leave this checksum unencrypted for password protected archive. In fact all
the metadata is easily available. We can just use python zipfile module to list
it all:
```shell
pymac@liskamm:~/ctf/backdoor/CRC/ext$ python list_crc.py
0 (1 files)
    name=0.txt
    comment=
    compressed_size=17
    file_size=5
    crc=a36bb2ae

...

25 (1 files)
    name=25.txt
    comment=
    compressed_size=17
    file_size=5
    crc=4d76cfce
26 (1 files)
    name=26.txt
    comment=
    compressed_size=14
    file_size=2
    crc=d8662568
```

Huh, so each archive contains a single text file with 5 characters only?
Except the last one is only 2 characters long? That one should be easy to
brute-force, right? Python binascii module provides crc32() method that we
can use for that. We just loop over 2-character printable strings (it's txt after
all) and look for the matching one:

```python
import binascii
import itertools
import string

for chars in itertools.product(string.printable, repeat=2):
    text = ''.join(chars)
    if (binascii.crc32(text) & 0xffffffff) == int('d8662568', 16):
        print text
        break
```
Ok, that took 0.016 second to run and returned '?>' as a result. Let's try the
same approach with other files.

Unfortunately brute forcing 5 character strings turns out to be waaay too slow on my laptop.

Maybe we can be a bit smarter about that? That string we got from 26.zip looks
awfully familiar - I wonder if the message we're trying to extract starts with
'<?php '? Turns out it does. Also turns out, that if we can guess just one symbol
in each archive (or just guess a reasonably small set of possible characters for
it) we can brute the rest in just a few minutes.

For example we find out that one of the files contains " 0x19". There is only a limited
number of things that make sense before number literal in php, right? I would expect
arithmetic and bit operators, =, <, > to be most likely candidates. Let's just assume
the last character of previous archive is one of those things and see if we're right.

So I updated the initial script a bit to accept "hints" and we (Rodbert and michals helped
me there) spend a few hours guessing and waiting for the script to brute force
full php code based on out guesses. Finally we get the full php code.
Running it returns the flag.

---

## The final script
I've removed most hints, to let you try yourself :)
```python
import binascii
import itertools
import os
import string
import sys
import zipfile

chars = string.ascii_lowercase + string.digits + '\n $(){};[]=+-'
#chars = string.printable

hints = {
    5: {'start': 'cho'},
    6: [{'start': 'fla'}, {'start': 'FLA'}, {'start': "Fla"}, {'start': 'the'}, {'start': 'The'}, {'start': 'THE'}],
    24: [{'end': 'fun'}],
    25: [{'end': ' '}],
}

m = 24
attack_chars = string.ascii_lowercase + string.digits + string.punctuation + ' '

def attack(fname, hints=None, charset=None):
    if charset is None:
        charset = chars
    if not hints:
        hints = [{}]
    with zipfile.ZipFile(fname, 'r') as zf:
        info = zf.infolist()[0]
        length = info.file_size
        crc = info.CRC
        if isinstance(hints, dict):
            hints = [hints]
        for hint in hints:
            start_hint = hint.get('start', '')
            end_hint = hint.get('end', '')
            eff_len = length - len(start_hint) - len(end_hint)
            combs = len(charset) ** eff_len
            print "Attacking file {} (length={} (minus hint={}), crc={}, combinations={})".format(fname, length, eff_len, crc, combs)
            if start_hint or end_hint:
                print "Hint: '{}' -- '{}'".format(start_hint, end_hint)
            print "Using chars: {}".format(charset)
            last_p = 0
            for i, guess in enumerate(itertools.product(charset, repeat=eff_len)):
                if (i * 100) / combs > last_p:
                    last_p = (i * 100) / combs
                    print '{}% '.format(last_p),
                    sys.stdout.flush()
                g = ''.join(guess)
                g = start_hint + g + end_hint
                c = binascii.crc32(g) & 0xffffffff
                if c == crc:
                    print
                    print "---{}".format(g)
                    #return # turns out this is not a good idea, too easy to stop on wrong string
                            # that just happens to produce CRC32 collision
            print
        print "Attack done"

attack('{}.zip'.format(m), hints.get(m), attack_chars)
```
