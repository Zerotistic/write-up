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
1) ``mmap()`` et utilisé sur une région et la rend écrivable, lisible et exécutable. La fonction ``mmap()`` a un peu le même effet que la commande ``chmod`` 
sous linux, à la différence près que cette fois-ci c'est une partie de la pile, et non un fichier dont on change les permissions d'accès.
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
Cette fonction n'est pas très complexe non plus, en sommes ce qu'elle fait c'est qu'elle cherche soit pour une chaine de caractère soit un caractère précis.
Si c'est trouvé la fonction va renvoyer une valeur différente de 0, et du coup nous empêcher d'exécuter notre shellcode.<br>
Mais quelles (chaines de) caractères sont bloqués exactement?<br>

<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033818189156585534/unknown.png">
</p>
<br>
Ok, maintenant on sait que ``/bin/bash``, ``/bin/sh``, ``/bin//sh`` et ``flag.txt`` sont des chaines interdites. 
<br>
<p align="center">
<img src="https://cdn.discordapp.com/attachments/693164567307616310/1033818189156585534/unknown.png">
</p>
<br>
Et maintenant on sait que les caractères 0x20 (un espace) et 0x0a (un retour à la ligne) sont aussi interdits!

## Création d'un exploit
