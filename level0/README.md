# level0

Binaire statique 32 bits, setuid vers level1.

Dans `main` (vu à l'objdump) : `atoi(argv[1])` est comparé à 0x1a7 (423). Si ça matche, il fait `setresuid`/`setresgid` vers l'uid de level1 puis `execv("/bin/sh")`.

Pas de bug mémoire ici, juste donner le bon argument :

    ./level0 423

Ça pop direct un shell avec les droits level1.
