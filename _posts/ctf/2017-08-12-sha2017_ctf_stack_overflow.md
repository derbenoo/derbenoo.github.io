---
category: ctf
layout: ctfpost
title: Stack Overflow
author: Benjamin Assadsolimani
event: SHA2017 CTF
ctftime-task-id: 4432
tags: [Crypto]
points: 100
solves: 155
description: "I had some issues implementing strong encryption, luckily I was able to find a nice example on stackoverflow that showed me how to do it."
---


## Challenge Description

I had some issues implementing strong encryption, luckily I was able to find a nice example on stackoverflow that showed me how to do it.

`stackoverflow.tgz  b245e83f0fb65763b3c1b346813cb2d3`

## Overview

The archive contains a file named `flag.pdf.enc` which is just composed of random bytes and a python script called `encrypt.py`: 

{% highlight python %}
import os, sys
from Crypto.Cipher import AES

fn = sys.argv[1]
data = open(fn,'rb').read()

# Secure CTR mode encryption using random key and random IV, taken from
# http://stackoverflow.com/questions/3154998/pycrypto-problem-using-aesctr
secret = os.urandom(16)
crypto = AES.new(os.urandom(32), AES.MODE_CTR, counter=lambda: secret) 

encrypted = crypto.encrypt(data)
open(fn+'.enc','wb').write(encrypted)
{% endhighlight %}

So we specify an input file (in our case `flag.pdf`) which is encrypted by the python script using AES. We look up the `AES.new` function [here](http://pythonhosted.org/pycrypto/Crypto.Cipher.AES-module.html#new) to understand the initialization:
* We choose 32 bytes of random for the AES key
* We use AES in counter mode
* We use a lambda expression to create a function that returns a series of _counter blocks_ based on a block of 16 randomly chosen bytes

Wait, the last point seems pretty odd. The function is supposed to implement a *counter*, instead the same 16 bytes are repeated infinitely. To explain why this is a problem and how we can leverage this behavior to reconstruct the encrypted PDF file, we first have to explain how the counter mode works.

## Counter Mode

Encryption using counter mode basically works like this:

![Counter Mode]({{site.url}}/public/images/ctf/sha2017_ctf_stack_overflow_counter_mode.png)

* E<sub>k</sub>: Block cipher (e.g., AES) with key _k_
* m<sub>1</sub>, m<sub>2</sub>, ... : Plaintext message bits
* c<sub>1</sub>, c<sub>2</sub>, ... : Resulting ciphertext bits
* IV: Initialisation vector used for the counter

So we turn a block cipher into a stream cipher by using the block cipher to encrypt a counter (IV+1, IV+2, ...) and performing an XOR operation of the resulting ciphertext with the corresponding plaintext bit. 

## Guessing some plaintext