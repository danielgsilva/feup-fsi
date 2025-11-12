# CTF Semana #4 (Environment Variables)

## Desafio 1

Numa primeira fase de reconhecimento, recolheu-se a informação abaixo ao navegar pela aplicação web, que pode ser importante para encontrar e explorar vunerabilidades.

Começamos por verificar apt list --installed para verificar os pacotes instalados no sistema. E logo nas primeiras linhas encontramos o apache2, que é um servidor web open-source.

```
    apache2/trusty-updates,trusty-security,now 2.4.7-1ubuntu4.22 amd64 [installed]
    apache2-bin/trusty-updates,trusty-security,now 2.4.7-1ubuntu4.22 amd64 [installed,automatic]
    apache2-data/trusty-updates,trusty-security,now 2.4.7-1ubuntu4.22 all [installed,automatic]
```	

Pesquisamos por vulnerabilidades conhecidas do apache2 mas faltava saber a versão do GNU bash, que é um interpretador de comandos que é executado no terminal.

```
    bash/trusty-updates,trusty-security,now 4.3-7ubuntu1.14 amd64 [installed]
```
Com essas duas informações e também que o nome do ctf é sobre variáveis de ambiente, pesquisamos por vulnerabilidades conhecidas do GNU bash e encontramos a vulnerabilidade Shellshock. Que é uma vulnerabilidade de segurança que afeta o GNU Bash e que permite a execução de comandos arbitrários em sistemas afetados. E tanbém tem haver com o nome do site que é "shell". 

[CVE-2014-6271](https://www.cve.org/CVERecord?id=CVE-2014-6271)
Possui essa descrição: "GNU Bash through 4.3 processes trailing strings after function definitions in the values of environment variables, which allows remote attackers to execute arbitrary code via a crafted environment, as demonstrated by vectors involving the ForceCommand feature in OpenSSH sshd, the mod_cgi and mod_cgid modules in the Apache HTTP Server, scripts executed by unspecified DHCP clients, and other situations in which setting the environment occurs across a privilege boundary from Bash execution, aka "ShellShock." NOTE: the original fix for this issue was incorrect; CVE-2014-7169 has been assigned to cover the vulnerability that is still present after the incorrect fix." 



## Desafio 2
Após ler atentamente a descrição da vulnerabilidade Shellshock, percebemos que a vulnerabilidade está relacionada com a forma como o GNU Bash processa strings após definições de funções nos valores das variáveis de ambiente. Primeiros procuramos os ficheiros que estão no site escrevendo simplesmente /, porque percebemos que ele sempre faz esse comando "ls -al" .


![alt text](/images/CTF4/image-2.png)

Como a vulnerabilidade Shellshock está relacionada à manipulação de variáveis de ambiente, navegamos até o diretório /var, onde encontramos o ficheiro flag.txt dentro da pasta flag. Decidimos então explorar a vulnerabilidade para abrir esse ficheiro, usando o operador || para separar comandos. O comando executado foi:
"|| /var/flag/flag.txt"

![alt text](/images/CTF4/image-1.png)


Como o site é vulnerável a [Shellshock](https://www.amen.pt/assistencia/shellshock-a-nova-vulnerabilidade-do-shell-bash/), A exploração consiste em definir uma variável de ambiente com o nome de uma função e um valor que contenha o comando a ser executado. Baseando-nos no exemplo clássico da vulnerabilidade Shellshock, utilizamos o seguinte comando:

```bash
    env x='() { :;}; cat /var/flag/flag.txt' 
```

Tendo esse output:  

![alt text](/images/CTF4/image-3.png)