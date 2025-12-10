---
title: "Santa's Secret Memes"
date: 2025-12-09T20:00:00+01:00
draft: false
categories: ["misc"]
tags: ["Easy"]
description: "Challenge du XMAS Root-Me"
---

# 1-Prendre les informations
Nous avons un fichier **ZIP** contenant **des images**. 

Lorsque l'on essaye de dezipper:
```bash
$>unzip santa-secret-memes.zip    
Archive:  santa-secret-memes.zip  
[santa-secret-memes.zip] dark_style.jpg password:
```
Le dossier est **protégé par un mot de passe**. 

On peut regarder l'algorithme de chiffrement du dossier avec `7z`:
```bash
$>7z l -slt santa-secret-memes.zip | grep "Method"  
   
Method = ZipCrypto Deflate  
Method = ZipCrypto Deflate  
Method = ZipCrypto Deflate  
Method = ZipCrypto Deflate  
Method = ZipCrypto Store  
Method = ZipCrypto Deflate  
Method = ZipCrypto Deflate
```
On peut voir que **ZipCrypto** est utilisé pour chiffrer **tous les fichiers**. 

Pour en être sûr et **avoir plus d'informations** on peut utiliser également `zipinfo`:
```bash
$>zipinfo santa-secret-memes.zip  
Archive:  santa-secret-memes.zip  
Zip file size: 605772 bytes, number of entries: 7  
-rw-r--r--  3.0 unx   103341 BX defN 25-Dec-09 02:05 dark_style.jpg  
-rw-r--r--  3.0 unx   124973 BX defN 25-Dec-09 02:05 green_bench.jpg  
-rw-r--r--  3.0 unx    98878 BX defN 25-Dec-09 02:05 just_a_dream.jpg  
-rw-r--r--  3.0 unx    85890 BX defN 25-Dec-09 02:05 mod_meme.jpg  
-rw-r--r--  3.0 unx     1221 BX stor 25-Dec-09 02:05 portrait.jpg  
-rw-r--r--  3.0 unx    81268 BX defN 25-Dec-09 02:05 raccoon.jpg  
-rw-r--r--  3.0 unx   109829 BX defN 25-Dec-09 02:05 rev_meme.jpg  
7 files, 605400 bytes uncompressed, 604474 bytes compressed:  0.2%
```

# 2-Déchiffrement 
On peut essayer autant que l'on veut de brute force avec zip2john ou zip2hash mais cela prend un temps infini.

Avec quelque recherches, on se rend compte que *ZipCrypto* à une grande faiblesse: si l'on peut **prédire le début d'un fichier**, on peut **trouver la clé de chiffrement** du zip. 

C'est vrai, dans le cas où les fichiers sont compressé après être chiffré, c'est à dire seulement les ***ZipCrypto Store***, donc **`portrait.jpg`**, et non pas les fichier ***ZipCrypto Deflate***.

D'après [wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures) on sait qu'un fichier JPG commence toujours par:
`FF D8 FF E0 00 10 4A 46 49 46 00 01`

On créer un faux fichier jpg:
```bash
$>printf '\xFF\xD8\xFF\xE0\x00\x10\x4A\x46\x49\x46\x00\x01' > plain.txt
```

On peut alors utiliser l'outil [bkcrack](https://github.com/kimci86/bkcrack):
```bash
$>bkcrack -C santa-secret-memes.zip -c portrait.jpg -p plain.txt

bkcrack 1.8.1 - 2025-10-25
[21:29:25] Z reduction using 5 bytes of known plaintext
100.0 % (5 / 5)
[21:29:25] Attack on 1127172 Z values at index 6
Keys: 4c0a34dd 9f68579b 9fd87f2f
11.5 % (129820 / 1127172)
Found a solution. Stopping.
You may resume the attack with the option: --continue-attack 129820
[21:31:41] Keys
xxxxxxxx xxxxxxxx xxxxxxxx
```

On a alors la clé que l'on peut utiliser pour déchiffrer le zip:
```bash
$>bkcrack -C santa-secret-memes.zip -k xxxxxxxx xxxxxxxx xxxxxxxx\n -D decrypted.zip
```
On dezippe:
```bash
$>unzip decrypted.zip            
Archive:  decrypted.zip  
 inflating: dark_style.jpg             
 inflating: green_bench.jpg            
 inflating: just_a_dream.jpg           
 inflating: mod_meme.jpg               
extracting: portrait.jpg               
 inflating: raccoon.jpg                
 inflating: rev_meme.jpg
```

# 3-flag
Rien dans les images, on utilise exiftool pour comprendre où se 
```bash
$>exiftool *.jpg | grep "Comment"    
Comment                         : RM{fake_flag}  
User Comment                    : Well you find a tool and a key, time to find the good image 🥸-  
Comment                         : tool: steghide | passphrase=magic_key  
Comment                         : RM{fake_flag}  
Comment                         : RM{fake_flag}  
Comment                         : RM{fake_flag}
```

On essaye steghide sur les images et on trouve:
```bash
$>steghide extract -sf green_bench.jpg  
Enter passphrase:    
wrote extracted data to "flag.txt".
```
