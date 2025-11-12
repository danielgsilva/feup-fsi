# CTF Semana 8 (SQL Injection)

Para este ctf a nossa primeira tarefa é encontrar os plugins usados pelo website e as suas versões.

Passado pouco tempo de entramos no site aparece uma notificação no canto inferior esquerdo "assinada" por um plugin chamado `NotificationX`.

Inspecionando o html do website com as developer tools conseguimos ver que a versão do plugin é `2.8.1`.
![](images/CTF8/versao_plugin_notificationX.png)

Procurando por CVEs deste plugin encontramos o [`CVE-2024-1698`](https://pentest-tools.com/vulnerabilities-exploits/notificationx-282-sql-injection_22574) que funciona em versões menores ou iguais a `2.8.2` e que usa uma vunerabilidade de `sql injection`, tambem conseguimos encontrar um [exploit](https://github.com/kamranhasan/CVE-2024-1698-Exploit) já feito sobre este cve que encontra o nome e pass do admin, só precisamos de alterar o domino do url para o site que queremos atacar e o delay(usado para perceber se a query produz um resultado certo ou não),o primeiro com que tivemos sucesso foi 4

Ao correr este exploit tivemos o seguinte output:
![](images/CTF8/process.png)
![](images/CTF8/process2.png)
![](images/CTF8/found_admin_pass.png)

Sabemos que está certo porque a hash começa com `$P$B` como é dito na dica e o nome do admin está correto porque este é o nome que aparece no primeiro post to site.

A partir daqui é necessário saber que tipo de encriptação é usada pelo wordpress após alguma pesquisa descobrimos que este usa a framework phpass(que é baseada no md5).

E agora temos  toda a informação necessária para fazer a desencriptação.

Para isso usamos um programa de "recuperação de passwords" chamado [hashcat](https://hashcat.net/hashcat/)


Usar o hascat é muito simples basta ir até à pasta onde foi instalado e executar o comando
```bash 
.\hashcat -a 0 -m 400 ctf8.hash cain-and-abel.txt
```
(no windows)
ou
```bash
hashcat -a 0 -m 400 ctf8.hash cain-and-abel.txt
``` 
(no linux)

Significado dos argumentos:

`-a 0`: attack mode 0(codigo para ataque com dicionário)

`-m 400`: é tipo de encriptação (400 simboliza a encriptação feita pela phpass)

`ctf8.hash`: ficheiro com a hash que encontramos

`cain-and-abel`: dicionario com possiveis passwods(foram tentados alguns dicionarios e este foi o primeiro que funcionou pode ser encontrado [aqui](https://github.com/david-palma/wordlists/blob/main/password-dictionaries/cain-and-abel.txt))

Depois de correr o comando temos o output:
![](images/CTF8/hash_cracked.png)
e assim descobrimos que a password é `heartbroken`



## Peguntas do Moodle

Como pode ser catalogada a vulnerabilidade?
Esta vulnerabilidade pode ser catalogada como uma time based sql injection pois quando uma query retorna um resultado positivo o exploit fica inativo durante um tempo expecifico e é atraves deste tempo que podemos perceber de a query retornou uma resposta positiva ou não

Será que armazenar uma hash da palavra-passe é seguro, no sentido em que torna impossível a recuperação da palavra-passe original?
Nesse sentido não porque existem various programas dedicados a reverter hashs.


