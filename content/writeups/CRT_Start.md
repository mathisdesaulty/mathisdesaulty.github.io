+++
date = '2026-02-01T18:26:35+01:00'
draft = false
title = 'CRT_Start'
categories = ["Forensic"]
tags = ["Intro"]
description = "L'un des premiers challenge de forensic sur la plateforme Hackropole"
+++
 
## Le chall
Ce challenge comporte 2 fichier:
- Un fichier `backup1999.bin`, apperement corrompu
- Et le binaire `crt_start`, qui permettrait de réparer ce binaire
![](/images/writeups/Pasted%20image%2020260124141320.png)
Nous avons la fonction start qui appelle une fonction prenant une autre en paramètre. 
Si nous regardons un peu dans les strings, nous en avons une particulièrement intéréssantes qui est appelé dans cette dernière fonction qui semblerait être le main:
![](/images/writeups/Pasted%20image%2020260124142001.png)
___
## Analyse du binaire
En lisant le code, on peut voir que ce binaire appelle un troisième fichier, `candidate.txt`:
![](/images/writeups/Pasted%20image%2020260124142309.png)
En soit, le code et assez simple, il prend deux fichiers, fais des opérations sur les deux et compare leurs résultats. Il est donc assez simple de comprendre que nous devons créer le `candidate.txt` et qu'il doit comporter les bonnes valeurs pour afficher à la fin le flag.
voici un résumé du binaire pour que le reste soit compréhensible:
```c
char* file_backup = read_file(argv[1], "rb");
/*
Vérification de sa taille et de ces valeurs.
Opération sur le file_backup (XOR, décalage de bits, rotations...).
*/
char *decrypted_values = new_file_backup; /* Après les opérations */
char* file_candidate = read_file("candidate.txt", "r");
/*
Vérification de sa taille et de ces valeurs.
Opération sur le file_candidate (XOR, décalage de bits, rotations...).
*/
# Comparaison des deux avec des opérations, encore...
do
{
if (file_candidate[i] != (((uint8_t)i * 0xd + 7)
                                            ^ decrypted_values[i]))
goto error;
    i += 1;
} while (i < len);
printf("HACKDAY{%s}\n",decrypted_values);
```
Sauf qu'il y a un problème. On se rend vite compte que les opérations faites sur le fichier `backup1999.bin` mais également sur `candidate.txt` prennent un temps beaucoup trop long à être déchiffré. 
Ayant commencer par comprendre ce que faisait le binaire, je me suis rapidement dis qu'il y avait un moyen de contourner toutes ces opérations. 
Le code nous affichera toujours `INVALID` tant que l'on a pas trouvé un candidate.txt parfait. 
MAIS, nous pouvons voir que ce qui est dans le flag correspond simplement à `decrypted_values` qui est simplement le fichier `backup1999.bin` avec toutes les modifications que l'on lui apporte. 
___
## Plan
Nous n'avons alors pas besoin de trouver un `candidate.txt`, mais simplement de lire ce qui se trouve dans `decrypted_values`.
Mon idée est donc la suivante:
- Lancer le binaire avec `gdb`
- Mettre un break juste après la dernière modification de `decrypted_values`
- Sauter avant le print du flag
- Continuer
On saute la vérification de notre `candidat.txt` et donc nous n'avons pas besoin de nous préoccuper de celui-ci. 
___
## Exploitation
```bash
gdb crt_start
```
On trouve l'adresse après la modification de `decrypted_values`:
```
(gdb) b *0x401929
Breakpoint 1 at 0x401929
(gdb) r
Starting program: ~/ctf/hackday/crt_start backup1999.bin
Breakpoint 1, 0x0000000000401929 in ?? ()
```
On trouve l'adresse avant le print du flag:
![](/images/writeups/Pasted%20image%2020260124145356.png)
En assembleur pour être plus précis:
![](/images/writeups/Pasted%20image%2020260124145449.png)
```
(gdb) set $pc = 0x00401ab6
(gdb) c
Continuing.
HACKDAY{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}
[Inferior 1 (process 797722) exited normally]
(gdb) 
```