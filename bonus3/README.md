# bonus3 : inconnu (pas de reverse fait)

    undefined4 main(int param_1,int param_2)
    {
        undefined4 uVar1;
        int iVar2;
        char *pcVar3;
        byte bVar4;
        char local_98 [65];
        undefined1 local_57;
        char local_56 [66];
        FILE *local_14;
        
        bVar4 = 0;
        local_14 = fopen("/home/user/end/.pass","r");
        pcVar3 = local_98;
        for (iVar2 = 0x21; iVar2 != 0; iVar2 = iVar2 + -1) {
            pcVar3[0] = '\0';
            pcVar3[1] = '\0';
            pcVar3[2] = '\0';
            pcVar3[3] = '\0';
            pcVar3 = pcVar3 + ((uint)bVar4 * -2 + 1) * 4;
        }
        if ((local_14 == (FILE *)0x0) || (param_1 != 2)) {
            uVar1 = 0xffffffff;
        }
        else {
            fread(local_98,1,0x42,local_14);
            local_57 = 0;
            iVar2 = atoi(*(char **)(param_2 + 4));
            local_98[iVar2] = '\0';
            fread(local_56,1,0x41,local_14);
            fclose(local_14);
            iVar2 = strcmp(local_98,*(char **)(param_2 + 4));
            if (iVar2 == 0) {
            execl("/bin/sh","sh",0);
            }
            else {
            puts(local_56);
            }
            uVar1 = 0;
        }
        return uVar1;
    }


version clair : 

    int main(int argc, char **argv) {
        char password[66];
        char buffer[66];
        FILE *file;

        if (argc != 2) return -1;

        file = fopen("/home/user/end/.pass", "r");
        if (!file) return -1;

        fread(password, 1, 0x42, file);
        // le password est lu ici 
        // exemple password = slt

        int index = atoi(argv[1]);
        // arg est converti en int
        // si argv[1] = "" (vide), alors atoi("") renvoie 0. Donc index = 0.
        
        password[index] = '\0';
        // le password est coupé à l'index 0
        // ce qui donne : password[0] = '\0' -> password devient une chaîne vide ("")
        
        fread(buffer, 1, 0x41, file);
        fclose(file);

        if (strcmp(password, argv[1]) == 0) {
            // donc la comparaison de zinzin : strcmp("", "") == 0
            execl("/bin/sh", "sh", 0);
        } else {
            puts(buffer);
        }

        return 0;
    }



