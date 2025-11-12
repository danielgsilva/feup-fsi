# Secret-Key Encryption Lab

## Task 1: Frequency Analysis

Através da análise de `freq.py`, identificamos que as três letras mais comuns juntas (trigram) eram "ytn" e "vup" no arquivo `ciphertext.txt`, e as substituímos por "THE" e "AND", respectivamente, que são as duas palavras com três letras mais comumente usadas no inglês. Em seguida, em cada iteração (tentativa erro) procuramos palavras que estavam quase completas e substituímos as letras faltantes por letras que faziam sentido. 

```bash 
[11/29/24]seed@VM:~/.../Files$ tr 'ytnvup' 'THEAND'< ciphertext.txt > out.txt
[11/29/24]seed@VM:~/.../Files$ cat out.txt 
(linhas ocultas)
EkThA ixNr gEaAzqE THE xqaAhq lEhE cxfED Tx THE bmhqT lEEsEND mN cAhaH Tx
AfxmD axNbimaTmNr lmTH THE aixqmNr aEhEcxNd xb THE lmNTEh xidcemaq THANsq
edExNraHANr
```

Por exemplo, não é difícil adivinhar que `lEEsEND` deverá ser `WEEKEND` e assim tentar substituir "l" e "s" por "W" e "K" respetivamente.
No final, conseguimos decifrar o criptograma com o comando abaixo:

```bash
tr 'ytnvuplshqfdmrbxizaecgj' 'THEANDWKRSVYIGFOLUCPMBQ'< ciphertext.txt > out.txt
```

![](images/LOGBOOK9/image.png)

Podemos concluir que ao analisar as frequências das letras, conseguimos identificar padrões e substituir as letras de acordo com a frequência de ocorrência de cada uma. Isso é possível porque a frequência de ocorrência das letras em textos em inglês é conhecida e pode ser usada para decifrar textos cifrados. No entanto, esse método pode não ser eficaz para textos muito curtos ou com poucas palavras conhecidas.

## Task 2: Encryption using Different Ciphers and Modes

Primeiro, criamos o arquivo `plaintext.txt` e inserimos algum texto aleatório nele.

![](images/LOGBOOK9/image2.png)


#### Flags
Utilizamos as seguintes flags:

-d: indica que desejamos decifrar (decrypt).

-e: indica que desejamos cifrar (encrypt).

-in: especifica o arquivo de entrada (plaintext.txt).

-out: especifica o arquivo de saída (por exemplo, cipher_ecb.txt).

-K: define a chave em hexadecimal (00112233445566778889aabbccddeeff).

-iv: define o vetor de inicialização em hexadecimal (0102030405060708).

#### Cifrando
Usamos o -d para cifrar e -e para decifrar. O -k/-iv foi definido no seedlab.  

No modo `AES-128-ECB`, a cifragem é realizada sem a necessidade de um vetor de inicialização (IV). Para cifrar um arquivo nesse modo, utilizamos o seguinte comando:
```bash
openssl enc -aes-128-ecb -e -in plaintext.txt -out cipher_ecb.txt -K 00112233445566778889aabbccddeeff
```
No modo `AES-128-CBC`, é necessário fornecer o IV (ver explicação abaixo). A cifragem é feita com o comando:
```bash
openssl enc -aes-128-cbc -e -in plaintext.txt -out cipher_cbc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

No modo `AES-128-CTR`, numa primeira abordagem achou-se que não era necessário especificar a flag IV, no entanto: "We do need to ensure that the key streams used for encrypting different data are different; no key stream can be reused. Therefore, the counter value for each block is prepended with a randomly generated value called nonce. This nonce serves the same role as the IV does to the other encryption modes."

Fonte: SEED book (21.4.6 Counter (CTR) Mode)

```bash
[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-ctr -e -in plaintext.txt -out cipher_ctr.txt -K 00112233445566778889aabbccddeeff
iv undefined
```

No CTR, o IV é composto pelo `nonce` e pelo `contador`. O comando para cifrar nesse modo é:

```bash
openssl enc -aes-128-ctr -e -in plaintext.txt -out cipher_ctr.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

### Comparando cada uma das cifras
- `AES-128-ECB`: divide o plaintext em blocos independentes, o que permite que ao cifrar 2 blocos iguais o correspondente bloco cifrado (ciphertext) seja também igual. Isso pode ser um problema de segurança, pois um atacante pode identificar padrões no texto cifrado.

- `AES-128-CBC`: cada bloco do plaintext é combinado (via operação XOR) com o bloco "anterior" já cifrado (ciphertext). Desta forma, mesmo que 2 blocos do plaintext sejam iguais o input para o algoritmo de encriptação será diferente, resultando em outputs diferentes (mitigando o problema do ECB). O principal objetivo do IV é garantir que mesmo que 2 plaintexts sejam iguais os ciphertexts sejam diferentes (usando diferentes IVs).

- `AES-128-CTR`: cada bloco do plaintext é combinado (via operação XOR) com um `key stream` obtendo o cyphertext. O `key stream` é gerado cifrando um contador (e o nonce como já vimos acima) que varia para cada bloco (tipicamente +1).

#### Decifrando
Para decifrar os arquivos cifrados, utilizamos os comandos abaixo, mudamos a flag -e para -d para decifrar os arquivos e usamos as mesmas chaves e IVs que foram usados para cifrar os arquivos:

```bash
openssl enc -aes-128-ecb -d -in cipher_ecb.txt -out plaintext_ecb.txt -K 00112233445566778889aabbccddeeff
```
```bash
openssl enc -aes-128-cbc -d -in cipher_cbc.txt -out plaintext_cbc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```
```bash
openssl enc -aes-128-ctr -d -in cipher_ctr.txt -out plaintext_ctr.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Depois de correr esses comandos, conseguimos decifrar os arquivos cifrados. E todas elas tem o mesmo conteúdo que o arquivo plaintext.txt.  

A diferença principal entre o modo CTR e os outros dois é que a operação de `cifrar e decifrar pode ser paralelizada` (que no caso do CBC só se aplica ao decifrar), o que o torna mais eficiente. Isto é possível, uma vez que o computação do contador para um bloco não depende de cálculos anteriores. O CTR permite cifrar blocos de dados de qualquer tamanho, inclusive dados menores que o tamanho do bloco, ao contrário do CBC em que é necessário adicionar `padding` ao último bloco. Naturalmente, também é mais seguro que o ECB.


## Task 5: Error Propagation – Corrupted Cipher Text

Como somos o 5 grupo, alteraremos o 50 * 5 = 250 byte de cada arquivo cifrado. Para isso, utilizamos o Bless Hex Editor para alterar os bytes dos arquivos cifrados. 

Quando lidamos com arquivos em hexadecimal ou binário, cada byte é identificado por sua posição (ou offset) dentro do arquivo. A contagem dos offsets segue a lógica de indexação zero-based (baseada em zero), ou seja:
O primeiro byte do arquivo está no offset 0.
O segundo byte está no offset 1.

Assim, para acessar o 250º byte, precisamos nos deslocar até o offset 249 (já que o primeiro byte está no offset 0). Em hexadecimal, esse offset é representado por 0xF9.

#### cipher_ecb

Fizemos a alteração no arquivo cipher_ecb.bin primeiro:

![](images/LOGBOOK9/image5.png)

Como podemos ver o byte 0xF9 é o "45" em ASCII, que é a letra "E". Alteramos para "46" que é a letra "F".

![](images/LOGBOOK9/image6.png)

E depois corremos o comando para decifrar o arquivo alterado:
```bash
openssl enc -aes-128-ecb -d -in cipher_ecb.bin -out corrupted_decripted_ecb.txt -K 00112233445566778889aabbccddeeff
```
Algumas partes do arquivo decifrado estão corretas.

![](images/LOGBOOK9/image7.png)


Fizemos a alteração no arquivo cipher_cbc.bin e no arquivo cipher_ctr.bin da mesma forma que fizemos no arquivo cipher_ecb mudando.bin. E depois corremos os comandos
```bash
openssl enc -aes-128-cbc -d -in cipher_cbc.bin -out corrupted_decripted_cbc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
openssl enc -aes-128-ctr -d -in cipher_ctr.bin -out corrupted_decripted_ctr.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

#### cipher_cbc

Aqui está o arquivo cipher_cbc.bin:

![alt text](images/LOGBOOK9/image-1.png) 

E aqui está a alteração no arquivo cipher_cbc.bin:

![alt text](images/LOGBOOK9/image-4.png)

E aqui está o arquivo decifrado:

![alt text](images/LOGBOOK9/image-5.png)

#### cipher_ctr

Aqui está o arquivo cipher_ctr.bin:

![alt text](images/LOGBOOK9/image-2.png)

E aqui está a alteração no arquivo cipher_ctr.bin:

![alt text](images/LOGBOOK9/image-6.png)

E aqui está o arquivo decifrado:

![alt text](images/LOGBOOK9/image-7.png)


Para contar o número de bytes que se perderam, utilizamos o comando (ele compara os arquivos byte a byte e conta quantos bytes são diferentes):
```bash
diff corrupted_decripted_ecb.txt plaintext.txt | wc -l
diff corrupted_decripted_cbc.txt plaintext.txt | wc -l
diff corrupted_decripted_ctr.txt plaintext.txt | wc -l
```

E o resultado foi:

![](images/LOGBOOK9/image8.png)

- Modo `AES-128-ECB`: Neste modo, a corrupção de um único byte resultou na alteração de `16 bytes` no arquivo decifrado. No modo ECB, cada bloco é cifrado de forma independente, o que significa que a corrupção de um byte em um bloco afeta apenas esse bloco no arquivo decifrado. Apenas o bloco correspondente ao byte 250 será afetado. Em AES, o tamanho do bloco é 16 bytes.

- Modo `AES-128-CBC`: A corrupção de um byte causou a alteração de `17 bytes` no arquivo decifrado. No modo CBC, a cifra depende do bloco anterior (por meio de um vetor de inicialização - IV). Quando um byte é corrompido num bloco, ele afeta não apenas o bloco corrompido, mas também o próximo bloco. Assim, afeta a decifragem completa do bloco correspondente (16 bytes) e o bloco seguinte é parcialmente corrompido (apenas um byte correspondente à posição no bloco onde ocorreu a corrupção, neste caso, o byte 250 do bloco criptografado).

- Modo `AES-128-CTR`: Neste modo, a corrupção de um byte afetou apenas `1 byte` no arquivo decifrado. No modo CTR, o key stream é gerado de forma independente do plaintext ou criptograma. Como resultado, a corrupção de um único byte no criptograma afeta apenas o byte correspondente no plaintext, sem afetar outros blocos. Isso faz com que o modo CTR seja o `mais resistente` a erros induzidos na cifra.