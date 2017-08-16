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

Block ciphers (such as AES) are able to encrypt or decrypt a single block of fixed length. Therefore, we need to specify modes of operation that dictate how inputs that are longer than a block size are being processed. Every mode of operation chains together multiple applications of the block cipher transformation to process larger inputs. The **counter mode** does this by successively encrypting a *counter* value and using the resulting ciphertext blocks as a keystream, which essentially turns the block cipher into a stream cipher. The plaintext is then simply XORed with the keystream for encryption. Decryption works analogously as we only need to reproduce the same keystream and can perform an XOR on the ciphertext to recover the plaintext.

The counter is typically implemented by using a publicly known initialization vector and incrementing it:

![Counter Mode]({{site.url}}/public/images/ctf/sha2017_ctf_stack_overflow_counter_mode.png)

* E<sub>k</sub>: Block cipher (e.g., AES) with key _k_
* m<sub>1</sub>, m<sub>2</sub>, ... : Plaintext message blocks
* c<sub>1</sub>, c<sub>2</sub>, ... : Resulting ciphertext blocks
* IV: Initialisation vector used for the counter

Note that by turning the block cipher into a stream cipher we have to make sure that the security requirements for a stream cipher are met. The most important requirement is that the keystream is not allowed to ever repeat itself. Otherwise, an attacker can XOR the ciphertext parts where the key repeats itself and obtain the XOR of the two plaintexts:

{% highlight null %}
C1= P1 XOR K
C2= P1 XOR K

C1 XOR C2= (P1 XOR K) XOR (P2 XOR K)= P1 XOR P2

{% endhighlight %}

The Python package `Crypto` allows us to specify a custom counter function. Unfortunately, this freedom can be abused to implement very insecure counter functions such as `counter=lambda: secret`. With `secret` being only 16 bytes long, we basically subverted the whole security of AES and reduced it to a repeating 16 byte long stream cipher key. All we have to do now is to XOR the ciphertext parts that were encrypted under the same key and guess some plaintext. While guessing plaintext is pretty pointless against a properly used stream cipher, it makes sense in this scenario since we have a method of validating our guess. Let's use the example from above and pretend we are guessing what P1 could be. Since we have access to `P1 XOR P2`, we can check our guess (let's call it P1') by calculating `P1' XOR (P1 XOR P2)` and see whether P2 makes sense or not. If so, we probably guessed the correct plaintext for P1. Once we successfully did that, we can extract the key by simple calculating `C1 XOR P1= K`

## Guessing some plaintext

	