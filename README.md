# Rainfall

Binary exploitation wargame (42). A chain of setuid binaries on a 32-bit
system; each level is exploited to recover the next user's password and
escalate privileges.

Each `levelXX/` and `bonusX/` directory holds the analysis and a writeup of the
technique used.

## Levels

| Level | Technique |
|-------|-----------|
| 0 | Argument check |
| 1 | Stack buffer overflow |
| 2 | Overflow + shellcode on the heap |
| 3 | Format string |
| 4 | Format string |
| 5 | Format string |
| 6 | Heap overflow |
| 7 | Heap overflow |
| 8 | Heap abuse |
| 9 | Overflow + shellcode |
| bonus0 | NOP sled + overflow |
| bonus1 | Integer overflow |
| bonus2 | NOP sled + overflow |

## Concepts

Stack and heap overflows, format-string exploitation, integer overflows,
NOP sleds and shellcoding on 32-bit ELF. Tooling: Ghidra, gdb/gef,
objdump/readelf.

> Educational material. The exploited binaries are intentionally vulnerable 42
> coursework.
