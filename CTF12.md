# CTF Semana 12 (RSA)


Tal como foi sugerido fizemos uma implementação basica do **RSA**, começamos por implementar o algoritmo de **Miller Rabin** para testar a primalidade de números, de seguida criamos uma função que procura um número primo `p` a partir de um offset e um número primo `q` tal que `p*q=n`, sendo que ``n`` é o nosso modulus.

```python
def Miller_Rabin(n, k=10):
   
    # n numero a ser testado, k numero de testes, se n for primo retorna True, se não retorna False
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # Witness loop
    for _ in range(k):
        a = random.randint(2, n - 2)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def find_primes_near(n, offset):
    guess = 2 ** (500+offset)
    
    i = 0
    while True:
        p = guess + i
        if Miller_Rabin(p):
            q = n // p
            if n % p == 0 and Miller_Rabin(q):
                return p, q
        i += 1
```


Agora que conseguimos encontrar os primos, temos de encontrar um par de exponente público e exponente privado (e,d) tal que e*d % (p-1)*(q-1) = 1, ou seja $$e \cdot d \equiv 1 \mod (p-1)(q-1)$$. Para isso podemos usar o **extended euclidean algorithm**



```python
def extendedEuclidean(e,phi):
    if e == 0: 
        return phi,0,1
             
    gcd,x1,y1 = extendedEuclidean(phi%e, e) 
    
    x = y1 - (phi//e) * x1 
    y = x1 
     
    return gcd,x,y 

```

Depois de perceber como gerar todos os componentes do **RSA** fica mais simples perceber como desencriptar a mensagem que nos foi dada.

Primeiro definimos variaveis com o nosso modulus e com o nosso exponente publico.

```python
n = 424468290327933077899172321362270868313591827166170171796412396789941591485700577725168502600236268509330570717307730733594960173226091561598610885712519647558932306381206042342177985947260131851886120104005529686484292686219621613682788254653269936014522307877975610701270998268801359060991451104490198003859394611611638853495589 

e = 65537
```

depois calculamos o offset da forma que nos foi dito no guião e usamos a função ``get_primes_near`` para descobrir os primos ``p`` e ``q``

```python
t = 10
g = 5
offset = ((t-1)*10+g) // 2

p,q=find_primes_near(n,offset)
```

Por fim definimos uma variavel com a mensagem e damos print ao resultado da função decrypt. 
```python
m = "40e16cb5deaf3697f859296dd592799326d8826405ff60be777e5f8046f6505e122377b5c1abb9fffd75d01a24fd12814abbeb3f7206278a2740e44f2164270a98b0ad0e73b48ae946d79da40a2c8a3c8e5d3b6064c0b634487c5caafb2cf5ddc1f2edf4f065ed28bc6ff40b0388536e2a8c23b34a55d6a78df6d14ac12dda36b280f322e39d25c74f0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"

print(decrpyt(m, e, n, p, q))  
```


A função decrypt recebe como argumentos a mensagem, o exponente publico, o modulus e os dois primos previamnete calculados, começa por calcular o phi usando p e q, chama a função ``extendedEuclidian``, que recebe o exponente publico e o phi, para obter o exponente privado ``d``, passa a mensagem para bytes e depois para um inteiro, calcula a potencia desse inteiro elevado a ``d`` com modulus iqual a ``n``, por fim passa o inteiro desencriptado para uma representação em bytes e usa o metodo ``decode`` para converter bytes para texto e retorna esse valor.

```python
def decrypt(message,e,n,p,q):
    phi = (p-1)*(q-1)
    (_,d,_)=extendedEuclidean(e, phi)
    
    messageInt = int.from_bytes(binascii.unhexlify(message), 'little')
    
    decipherMessage = pow(messageInt,d,n)
    
    decipherMessageBytes = decipherMessage.to_bytes((decipherMessage.bit_length() + 7) // 8, 'little')
    
    return decipherMessageBytes.decode()
```

No final o nosso output é
![](images/CTF12/flag.png)


## Perguntas do Moodle

Como consigo descobrir se a minha inferência está correta?

No nosso caso sabemos que a inferencia está correta pois ``q`` é gerado a partir de ``p`` e ``n``


## Script completo

```python
import random
import binascii


def Miller_Rabin(n, k=10):
   
    # n numero a ser testado, k numero de testes, se n for primo retorna True, se não retorna False
    r, d = 0, n - 1
    while d % 2 == 0:
        r += 1
        d //= 2
    
    # Witness loop
    for _ in range(k):
        a = random.randint(2, n - 2)
        x = pow(a, d, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

def find_primes_near(n, offset):
    guess = 2 ** (500+offset)
    
    i = 0
    while True:
        p = guess + i
        if Miller_Rabin(p):
            q = n // p
            if n % p == 0 and Miller_Rabin(q):
                return p, q
        i += 1
    
    

def extendedEuclidean(e,phi):
    if e == 0: 
        return phi,0,1
             
    gcd,x1,y1 = extendedEuclidean(phi%e, e) 
    
    x = y1 - (phi//e) * x1 
    y = x1 
     
    return gcd,x,y 

def decrpyt(message,e,n,p,q):
    phi = (p-1)*(q-1)
    (_,d,_)=extendedEuclidean(e, phi)
    
    messageInt = int.from_bytes(binascii.unhexlify(message), 'little')
    
    decipherMessage = pow(messageInt,d,n)
    
    decipherMessageBytes = decipherMessage.to_bytes((decipherMessage.bit_length() + 7) // 8, 'little')
    
    return decipherMessageBytes.decode()

t = 10
g = 5
offset = ((t-1)*10+g) // 2




n = 424468290327933077899172321362270868313591827166170171796412396789941591485700577725168502600236268509330570717307730733594960173226091561598610885712519647558932306381206042342177985947260131851886120104005529686484292686219621613682788254653269936014522307877975610701270998268801359060991451104490198003859394611611638853495589 
e = 65537

p,q=find_primes_near(n,offset)

m = "40e16cb5deaf3697f859296dd592799326d8826405ff60be777e5f8046f6505e122377b5c1abb9fffd75d01a24fd12814abbeb3f7206278a2740e44f2164270a98b0ad0e73b48ae946d79da40a2c8a3c8e5d3b6064c0b634487c5caafb2cf5ddc1f2edf4f065ed28bc6ff40b0388536e2a8c23b34a55d6a78df6d14ac12dda36b280f322e39d25c74f0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"

print(decrpyt(m, e, n, p, q))   
```
