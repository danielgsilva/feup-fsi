# CTF Semana #11 (Weak Encryption)

## Tarefa 1

```python
KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)
```

A função `gen` gera uma `chave` de `16 bytes`, sendo que os primeiros KEYLEN - offset = 16 - 3 = `13 bytes` são **fixos** com o valor `0x00`. Os últimos `3 bytes` (offset) são aleatórios, tratando-se de uma otimização... infeliz. A chave gerada torna-se previsível, porque assim um atacante só precisa de adivinhar os últimos 3 bytes, reduzindo significativamente o número de tentativas num ataque brute force: 2<sup>24</sup> = 16777216 combinações possíveis (viável).

```python
def enc(k, m, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph
```

A função `enc` recebe uma chave `k` (gerada como já vimos acima), a mensagem `m` e um `nonce` (que deverá ser único para cada operação de encriptação). Cria um objeto `Cipher` usando `AES-CTR` e assim encripta a mensagem com o `encryptor`.

```python
def dec(k, c, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg
```

A função `dec` é similar à `enc`, mas usa um `decryptor` para reverter a encriptação e recuperar a mensagem original.  

Para realizar o ataque de `brute force`, podemos desenvolver um script que teste todos os valores possíveis para os últimos 3 bytes da chave.

```python
def brute_force_decrypt(c, nonce):
    offset = 3
    prefix = b"\x00" * (KEYLEN-offset)  # Os primeiros 13 bytes da chave são fixos
    for b1 in range(256):
        for b2 in range(256):
            for b3 in range(256):
                k = prefix + bytes([b1, b2, b3])  # Gerar chave possível
                try:
                    msg = dec(k, c, nonce)
                    # Verificação: supondo que a mensagem seja códigos ASCII legíveis
                    if all(32 <= char <= 126 for char in msg):
                        print(f"Chave encontrada: {k.hex()}, Mensagem: {msg.decode()}")
                        return
                except Exception:
                    continue  # Ignora erros e continua a busca
    print("Chave não encontrada.")
```

Sabendo o `nonce` utilizado na cifração e o `criptograma` (em hexadecimal), é possível executar o ataque:

```python
nonce = unhexlify("af71cad5b078640aa6516b082f536705") 
c = unhexlify("01cdbc354f0942d2d2f7ba235ca47a73225d4a09c33e")

brute_force_decrypt(c, nonce)
```

```sh
Chave encontrada: 00000000000000000000000000101569, Mensagem: flag{iafiymnhqnhehtsa}
```

Assim os últimos 3 bytes da chave são `0x101569` e a mensagem/flag é `flag{iafiymnhqnhehtsa}`.

- Nota: na função `brute_force_decrypt` a verificação `if all(32 <= char <= 126 for char in msg)` é essencial para garantir que é possível imprimir a mensagem, isto é, só contém códigos ASCII legíveis. Na primeira execução sem esta condição, a mensagem não era legível.

## Tarefa 2

Para realizar esta experiência mediu-se o tempo necessário (`elapsed_time`) para testar `1 milhão` de chaves e calculou-se a taxa de tentativas/chaves por segundo (`keys_per_second`). Assim, convertendo 10 anos em segundos (`seconds_in_10_years`) podemos estimar o número total de chaves que podem ser testadas nesse período (`total_tests_in_10_years`) e determinar o `offset` necessário. 

```python
prefix = b"\x00" * 13
start_time = time.time()
keys_tested = 0
for b1 in range(256):
    for b2 in range(256):
        for b3 in range(256): 
            k = prefix + bytes([b1, b2, b3])
            try:
                dec(k, c, nonce)
            except:
                pass
            keys_tested += 1
            if keys_tested >= 10**6: 
                break
        if keys_tested >= 10**6:
            break
    if keys_tested >= 10**6:
        break

end_time = time.time()

# Calcular taxa de tentativas por segundo
elapsed_time = end_time - start_time
keys_per_second = keys_tested / elapsed_time

# Calcular tentativas possíveis em 10 anos
seconds_in_10_years = 10 * 365 * 24 * 3600
total_tests_in_10_years = keys_per_second * seconds_in_10_years

# Determinar offset necessário
offset = 0
while (256**offset) < total_tests_in_10_years:
    offset += 1

# Resultados
print(f"Chaves testadas: {keys_tested}")
print(f"Tempo decorrido: {elapsed_time:.2f} segundos")
print(f"Taxa de chaves por segundo: {keys_per_second:.2f}")
print(f"Total de chaves testadas em 10 anos: {total_tests_in_10_years:.2e}")
print(f"Tamanho mínimo de offset para inviabilidade: {offset}")
```

```sh
Chaves testadas: 1000000
Tempo decorrido: 12.49 segundos
Taxa de chaves por segundo: 80054.64
Total de chaves testadas em 10 anos: 2.52e+13
Tamanho mínimo de offset para inviabilidade: 6
```

O `offset` teria que ser `6 bytes` o que corresponde 2<sup>48</sup> combinações possíveis.

## Tarefa 3

O nonce deve ser um valor gerado aleatoriamente e é utilizado de modo a garantir que cada cifra gerada seja única, mesmo que a mesma chave seja reutilizada.  
Por um lado, usar o `nonce` com `1 byte` de tamanho corresponde a 2<sup>8</sup> possibilidades diferentes e já vimos que um espaço de busca destes é muito pequeno e completamente trivial, logo não adiciona complexidade ao ataque.  
Por outro lado, com um conjunto de possibilidades tão baixo, o mesmo nonce será inevitavelmente `reutilizado` após algumas mensagens. Quando o nonce é reutilizado com a mesma chave, `cifras de diferentes mensagens podem ser relacionadas`, o que leva a outro tipo de ataques.  
Como o nonce não é transmitido pela rede, até o `utilizador legítimo` terá que testar, no pior caso, 256 valores para decifrar a mensagem, o que `introduz um atraso` (apesar de pequeno) neste processo.
