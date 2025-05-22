# Hack The Box - Two Million (Write-up)

Este write-up documenta o processo de exploração da máquina **“Two Million”** no Hack The Box. A máquina apresenta vulnerabilidades **web** comuns, incluindo:

- **Broken Access Control**
- **Injection**
- **Vulnerable and Outdated Components**

O objetivo é capturar as flags de **usuário** e **root**.

---

## 🛠️ Ferramentas Utilizadas

- [Wireshark](https://www.wireshark.org/)
- [Burp Suite](https://portswigger.net/burp)
- [GoBuster](https://github.com/OJ/gobuster)
- [Nmap](https://nmap.org/)

---

## 🔍 Enumeração e Descoberta

### 1. Acesso Inicial

A máquina foi atribuída com o IP: `10.10.11.221`. Ao tentar acessar `http://2million.htb`, era exibida a seguinte mensagem:

> *"We're having trouble finding that site"*

Um **TCP SYN Scan** com o **Nmap** foi realizado para identificar as portas ativas. No entanto, ao inspecionar o tráfego com o **Wireshark**, o servidor retornava pacotes **RST+ACK**, o que sugeria a presença de um **firewall** ou bloqueio de pacotes.

Além disso, o comando `nslookup` indicava falha na resolução DNS:

```bash

nslookup 2million.htb
Server:         127.0.0.53
Address:        127.0.0.53#53
server can't find 2million.htb: NXDOMAIN
```
### 2. DNS SPOONFING

Foi necessário adicionar uma entrada manual no arquivo localizado em /etc/hosts para simular uma resolução DNS:

> *10.10.11.221 2million.htb*

![etchosts](https://github.com/user-attachments/assets/238e65d1-4b02-49a5-9ec5-8877f7ce04b8)

Após isso, o site foi carregado corretamente, confirmando que o problema era de resolução local de nomes.


### 3 Análise do Front-End e descobertas de Endpoints

No código-fonte da aplicação, foi encontrado o script:

> */js/invite.min.api.js*

Esse script continha funções JavaScript utilizada pela aplicação, como makeInviteCode().


### 4 ROT13 DECODING

A execução da função makeInviteCode() no console do navegador retornou o seguinte texto, codificado em ROT13:

> *"Va beqre gb trarengr gur vaivgr pbqr, znxr n CBFG erdhrfg gb /ncv/i1/vaivgr/trarengr"*

Após decodificar, obtive: 

> *"In order to generate the invite code, make a POST request to /api/v1/invite/generate"*

### 5 Requisição POST para Geração do Código

Foi enviada uma requisição POST para o endpoint identificado: 

> *POST /api/v1/invite/generate
Host: 2million.htb*

Resposta da API:
```bash
{
  "0": 200,
  "success": 1,
  "data": {
    "code": "NlZVSkQtS0pDMVotRzFTMU8tNzlISkM=",
    "format": "encoded"
 }
}
```

### 6 Base64 Decoding

O valor do campo code estava codificado em Base64. Após decodificar, o resultado foi:

> *6VUJD-KJC1Z-G1S1O-79HJC*

Este é o invite code necessário para criação de um usuário na aplicação.


