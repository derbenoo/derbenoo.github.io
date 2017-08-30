---
category: ctf
layout: ctfpost
title: WannaFly
author: Benjamin Assadsolimani
event: SHA2017 CTF
ctftime-task-id: 4418
tags: [Forensic]
points: 100
solves: 187
description: "My daughter Kimberly her computer got hacked. Now she lost all her favorite images. Can you please help me recover those images?"
---

## Challenge Description

My daughter Kimberly her computer got hacked. Now she lost all her favorite images. Can you please help me recover those images? 

`wannafly.tgz  7509faed92d67a242068de6605659ca1`

## Overview

The provided archive file contains an image file of about 100MB. We use 7z to extract the image file: `7z x kimberly.img`. We get a quick overview of the extracted files with the `ls -laR` command. The pictures directory contains 17 PNG images of ponies, which are encrypted by the "WannaFly" ransomware, as the embedded disclaimer tells us:

![PNG encrypted by the WannaFly ransomware]({{site.url}}/public/images/ctf/sha2017_ctf_wannafly_encrypted_img.png) 

We quickly check whether the PNG files have any unusual business going on by using the `exiftool` and `zsteg` utilities. Indeed, each PNG file seems to have additional base64 encoded data appended to the normal image data:

```
$ zsteg Pictures/78d14f37ae413511b119776c2294c414.png 
[?] 812169 bytes of extra data after image end (IEND), offset = 0x37a3d
```

Looks like the data of the original image was encrypted and appended to the watermarked version of it. Now we just need the key and the correct encryption algorithm to recover the images! We look a little further and notice 2 interesting files in the home directory: `...` and `.bash_history`. Let's check out the bash history first:

```
$ cat .bash_history 
unset HISTFIL
ls -la
pwd
chmod +x ...
./... Hb8jnSKzaNQr5f7p
ls -Rla
```

So, somebody tried to not leave any traces but failed, classic ransomware author! The `unset HISTFILE` command turns off the automatic saving of the bash history for that session, however the attacker misspelled the command so that the bash history still got saved. Apparently the `...` script was made executable and called with the parameter `Hb8jnSKzaNQr5f7p`. This could very well be our key as the script was probably used to encrypt the image files and needed a key to do so. All we have to do now is to understand how the script encrypted the images and revert the process.

## Decrypt All The Images!

We analyze the Python script and figure out the steps that are performed to encrypt the images:
* The passphrase is provided as the first command line argument
* The current directory is searched for files with the ending ".png" and each file is edited in the following way:
    * A blurred version of the image is created with the WannaFly ransomware logo alongside the instructions on how to get the original images back and is saved under `/tmp/sha.png`
    * The original image data is encrypted using AES with the following parameters:
        * Cipher feedback mode (CFB) 
        * The passphrase taken from the command line arguments
        * Initialization vector (IV) is a 16 bytes long random string of letters and digits with the random function being seeded with the current time 
    * The image file is completely overwritten with null bytes, then opened again and the blurred image data is written followed by the output of the AES encryption routine

We make use of the existing code and modify it in the following way such that it performs the decryption for us:
* We recover the IV used for the encryption of each image by taking its last modified timestamp as the seed value for the random function. While this might not be 100% accurate, it should suffice for most of the images as they are probably written to the file around the same time as they were encrypted. Luckily the current time is converted to an integer before being used as a seed, which means that the time resolution is reduced to seconds.
* We read the last line of each encrypted file and pass it to the already existing decrypt function.
* We create a new directory `Decrypted` and use it to save the output of the decryption function for each image using its original file name.

```
#!/usr/bin/env python

import random, string, sys, os
from time import time
from Crypto.Cipher import AES
import base64

def get_iv(img):
    img_seed= os.path.getmtime(img)
    iv = ""
    random.seed(int(img_seed))
    for i in range(0,16):
        iv += random.choice(string.letters + string.digits)
    return iv

def decrypt(m, p, i):
    aes = AES.new(p, AES.MODE_CFB, i)
    return aes.decrypt(base64.b64decode(m))

def find_images():
    i = []
    for r, d, f in os.walk("."):
        for g in f:
            if g.endswith(".png"):
                i.append((os.path.join(r, g)))
    return i

def decrypt_images():
    for i in find_images():
        decrypt_image(i)

def decrypt_image(img):
    data = open(img, 'r').readlines()[-1]
    decrypted_img = decrypt(data, sys.argv[1], get_iv(img))
    dec_img= img.replace("Pictures", "Decrypted")
    open(dec_img, 'w').write(decrypted_img)

if __name__ == '__main__':
    if len(sys.argv) != 2:
        print "Usage: %s <pass>" % sys.argv[0]
    else:
        decrypt_images()
```

Now, all that we have to do is call the script with the passphrase we found in the bash history:

`$ python decrypt.py Hb8jnSKzaNQr5f7p`

We are able to decrypt all but one image. However, while flipping through the decrypted images we notice that one of them already contains the flag:

![Decrypted pony image containing the flag]({{site.url}}/public/images/ctf/sha2017_ctf_wannafly_decrypted_img.png) 

The flag is: `flag{ed70550afe72e2a8fed444c5850d6f9b}`