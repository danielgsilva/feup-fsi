# CTF Semana #5 (Buffer Overflow)

Após instalar a biblioteca `pwntools` e o pacote `checksec` como sugerido no guião podemos correr o comando 
```bash
[10/30/24]seed@VM:~/.../semana5-desafio1$ checksec --file=program
```
que nos dá o seguinte output ![](images/CTF5/checksec_output.png)
daqui é importante realçar que os endereços de memória não são randomizadas `PIE : No PIE (0x8048000)` e à regiões de memória com premissões de leitura, escrita e execução (neste caso refereçe à stack) `RWX: Has RWX segments`.

Depois temos de analisar o código do `program`, que nos foi fornecido
![](images/CTF5/codigo_c_program.png)
como podemos ver existe um buffer que recebe input do utilizador, com esta informação mais a que foi descrita acima percebos que é possivel causar um buffer overflow.

Continuando a analisar o código vemos que o endereço da função `readtxt` é passada para o apontador `fun` e depois é invocada com o argumento `rules`, depois o endereço da função `echo` é colocado no apontador `fun`, o input do utilizador é colocado no `buffer` e `echo` é chamado com o que está no `buffer` atraves do apontador `fun`.

Se conseguíssemos alterar o endereço que está em `fun` antes de ser invocado com o buffer para a função `readtxt` obtemos controlo sobre o ficheiro que é lido.

Abrindo o gbd conseguimos ver que o endereço de `fun` está logo a depois da ultima posição do `buffer`.
![](images/CTF5/buffer_fun_positions.png)

Ou seja se echermos o buffer de "lixo" e depois colocarmos o endereço da função `readtxt` este vai ser dar override aquilo que estava do apontador `fun`. Como queremos ler o ficheiro `flag` basta por `flag\0` no inicio do `buffer`.

O endereço da funçao `readtxt` tambem é encontrado atraves do gbd e tem de ser colocado do payload "ao contrario" dado que a stack cresce de cima para baixo.
![](images/CTF5/readtxt_addr.png)

O nosso payload fica da seguinte forma:
![](images/CTF5/payload.png)

Com este payload quando o `program` é excutado o apontador `fun` vai ter o endereço da função `readtxt` em vez do da função `echo` e como o `buffer` tem `flag\0` no incio a função `readtxt` vai ler o ficheiro `falg` e assim conseguimos encontrar a falg.
![](images/CTF5/found_flag.png)