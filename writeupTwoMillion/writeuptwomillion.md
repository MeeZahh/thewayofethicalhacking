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
- [Ffuf](https://github.com/ffuf/ffuf)
- [Nmap](https://nmap.org/)

---

## 🔍 Enumeração e Descoberta

### 1. Acesso Inicial

Após instalação da VPN fornecido pela plataforma `https://www.hackthebox.com/`, foi exibido um endereço IP: `10.10.11.221`. Ao utlizar esse IP no navegador, ele levava para um site com o endereço  `http://2million.htb`, e era exibida a seguinte mensagem:

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

### 7 Fuzzing com Ffuf

Após login, foi realizado fuzzing para procurar outros endpoints, onde foi encontrado um endpoint padrão /api/

![image](https://github.com/user-attachments/assets/3a6c5e93-48a3-4b7f-bc96-abb0b7a3adfa)

Nesse endpoint foi encontrado subendpoints. Porém o que mais chamou atenção foi o /api/v1, que mostrava um JSON com vários outros endpoints de alterações administrativas. Permitindo escalonamento de privilégio.

![image](https://github.com/user-attachments/assets/084f5cf1-0e3f-435f-b7bc-55cf6fec474c)

### Escalando privilégio

Através do repeater no Burp Suite, foi feito uma requisição PUT em formato JSON, onde foi possível alterar a conta de usuário padrão para administrativo.

> */api/v1/admin/vpn/generate*

![image](https://github.com/user-attachments/assets/70c2dade-e406-4064-bb3a-e286649798ca)
