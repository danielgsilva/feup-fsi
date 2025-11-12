# Hash Length Extension

## Task 1: Send Request to List Files

Mudamos o pedido para esse, com o nome de um dos participantes do grupo e usamos esse par:
"1001:123456"
    
```bash
http://www.seedlab-hashlen.com/?myname=123456:myname=LucasGreco&uid=1001&lstcmd=1&mac=<need-to-calculate>
```

Para gerar o mac, é nessário esse comando:
```bash
echo -n "123456:myname=LucasGreco&uid=1001&lstcmd=1" | sha256sum
```

Que nos deu esse output : `589572123e62cd198a011f9ca586cc96cd046dc6064dff8e88716fd9c4cbd01b`

Logo o pedido final será:
```bash
http://www.seedlab-hashlen.com/?myname=LucasGreco&uid=1001&lstcmd=1&mac=589572123e62cd198a011f9ca586cc96cd046dc6064dff8e88716fd9c4cbd01b
```

Apos colocar o pedido no navegador, conseguimos acessar o conteudo do site:

![alt text](images/LOGBOOK10/image.png)

Para gerar o mac com o pedido de dowload, é nessário esse comando:
```bash
echo -n "123456:myname=LucasGreco&uid=1001&lstcmd=1&download=secret.txt" | sha256sum
```

Que nos deu esse output : `994bef4a990093192d3a8566fa67451547813f31265d5c75a7443df7cdda8232`

Logo o pedido final será:
```bash
http://www.seedlab-hashlen.com/?myname=LucasGreco&uid=1001&lstcmd=1&download=secret.txt&mac=994bef4a990093192d3a8566fa67451547813f31265d5c75a7443df7cdda8232
```

Depois de acessar esse link, o servidor nos deu esse output:

![alt text](images/LOGBOOK10/image-1.png)


## Task 2: Create Padding

Para criar o padding da mensagem `123456:myname=LucasGreco&uid=1001&lstcmd=1`, precisamos fazer esses passos:

1. Calculamos o tamanho da mensagem em bytes (42 bytes)
2. Calcular o tamanho do padding (64 - 42 = 22 bytes)
3. Multiplicar por 8 o tamanho da mensagem (42 * 8 = 336 bits = 0x150)
4. Logo o padding será 22 bytes, começando com 0x80 e terminando com 0x150 em Big-Endian. (0x01 0x50)

A imagem abaixo mostra os passo:

![alt text](images/LOGBOOK10/image-2.png)

## Task 3: The Length Extension Attack

Usando o MAC que criamos na task 1 (`589572123e62cd198a011f9ca586cc96cd046dc6064dff8e88716fd9c4cbd01b`),criamos um programa em C chamado task3.c para simular o último estágio da função de compressão do SHA-256. No código, ajustamos:

Estado Interno Inicial: Definimos o estado interno do SHA-256 com base no MAC original, separando-o em 8 palavras de 32 bits (Big-Endian).  
Mensagem Adicional: Especificamos a "Extra message" que desejamos adicionar ao comando original, neste caso, `&download=secret.txt`.


![alt text](images/LOGBOOK10/image-3.png)

Depois compilamos e deu esse novo MAC (`539b3b6588403daba26e70a20d1282005526b07e119bf5beaf974204ca3ebbf3`):

![alt text](images/LOGBOOK10/image-4.png)


Usamos esse MAC para acessar o arquivo secret.txt, com o seguinte url (utilizamos o padding que criamos na task 2):

Montamos o URL final combinando:

- Pedido original: `myname=LucasGreco&uid=1001&lstcmd=1`
- Padding calculado: `%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%50` 
- Novo comando: `&download=secret.txt`
- Novo MAC: `539b3b6588403daba26e70a20d1282005526b07e119bf5beaf974204ca3ebbf3`

O URL completo ficou:
```bash
http://www.seedlab-hashlen.com/?myname=LucasGreco&uid=1001&lstcmd=1%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%01%50&download=secret.txt&mac=539b3b6588403daba26e70a20d1282005526b07e119bf5beaf974204ca3ebbf3
```

Ao acessar o URL, o servidor retornou o conteúdo do arquivo `secret.txt`:

![alt text](images/LOGBOOK10/image-5.png)



