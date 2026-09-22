# 🛡️ AWS Cloud Security & Network Analysis Lab

> **Projeto Prático de Segurança em Nuvem, Análise de Rede e Troubleshooting**  
> *Demonstração de competências técnicas em fundamentos de rede, hardening de segurança na AWS, captura de pacotes Linux e análise de tráfego.*

---

## 📌 Visão Geral do Projeto

Este laboratório demonstra a aplicação prática de **Network Security** em um ambiente **AWS Cloud**. O objetivo foi provisionar uma infraestrutura segura, configurar um servidor web Nginx protegido por HTTPS/TLS, atuar como **Reverse Proxy** e utilizar ferramentas avançadas de diagnóstico de rede no Linux (`tcpdump`, `ss`, `nc`, `dig`, `openssl`) para análise de pacotes com o **Wireshark**.

---

## 🏗️ Arquitetura do Laboratório

```
[ Cliente / Internet ]
       │
       ▼ (Portas 22, 80, 443)
[ Security Group (Stateful Firewall) ]
       │
       ▼
[ AWS EC2 - Ubuntu Server ]
       ├── Nginx (Reverse Proxy - Public 80/443)
       └── Python HTTP App (Internal Only - 127.0.0.1:8080)
       │
       ▼
[ Captura de Dados: tcpdump ] ──► [ Análise: Wireshark (.pcap) ]
```

---

## ⚙️ Tecnologias & Conceitos Aplicados

- **Nuvem:** AWS (VPC, Subnet, Route Tables, Internet Gateway, EC2, Security Groups).
- **Sistemas & Redes:** Linux (Ubuntu), TCP/IP, CIDR, Roteamento (`ip route`), Portas/Sockets (`ss`).
- **Segurança & Servidor:** Nginx (Reverse Proxy), TLS/HTTPS, OpenSSL, Firewall Stateful.
- **Análise & Diagnóstico:** `tcpdump`, `Wireshark`, `nc` (Netcat), `dig`, `curl`.

---

## 🚀 Demonstração Prática (Execução de Comandos)

> *Abaixo estão demonstradas as validações operacionais diretamente no terminal Ubuntu da instância EC2, comprovando o domínio prático das ferramentas através dos vídeos de demonstração.*

---

### 1. Diagnóstico Inicial de Rede e Interfaces

Validação do endereçamento IPv4 interno, tabela de roteamento padrão e verificação de sockets/portas em escuta na máquina.

```bash
# Exibe interfaces e endereços IP
ip a

# Verifica a tabela de roteamento da máquina
ip route

# Exibe sockets TCP/UDP ativados e ouvindo conexões
ss -tuln
```



---

### 2. Configuração e Teste do Servidor Nginx (Reverse Proxy)

Subida de um serviço HTTP interno rodando isoladamente na porta `8080` (não exposto à internet) e roteamento de requisições através do Nginx como Reverse Proxy.

```bash
# Inicia e habilita o serviço do Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Teste da aplicação interna isolada em Loopback
curl http://127.0.0.1:8080

# Validação da resposta HTTP via Proxy na porta 80
curl -I http://localhost
```
![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração/2.gif)

---

### 3. Validação de Certificado TLS/HTTPS com OpenSSL

Investigação da camada de segurança TLS, validação de handshake e inspeção do certificado digital apresentado na porta `443`.

```bash
# Requisição do cabeçalho HTTPS
curl -I https://SEU_DOMINIO

# Inspeção detalhada do Handshake TLS e Certificado
openssl s_client -connect SEU_DOMINIO:443 -servername SEU_DOMINIO
```

![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração/3.gif)

---

### 4. Teste de Acessibilidade de Portas (Regras de Firewall/SG)

Uso do `nc` (Netcat) para provar a eficácia das regras do Security Group. A porta `8080` interna deve ser bloqueada para acesso externo, enquanto a `443` deve responder.

```bash
# Teste de porta HTTPS pública (Deve conectar)
nc -vz IP_PUBLICO 443

# Teste da porta 8080 interna (Deve falhar/dar timeout externamente)
nc -vz IP_PUBLICO 8080
```

![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração/4.gif)

---

### 5. Captura de Tráfego de Rede com `tcpdump`

Captura em tempo real dos pacotes recebidos na porta `443` durante uma requisição e salvamento em arquivo binário `.pcap` para auditoria.

```bash
# Captura tráfego HTTPS em tempo real no terminal
sudo tcpdump -i any -nn port 443

# Grava a captura de pacotes em arquivo para análise externa
sudo tcpdump -i any -nn port 443 -w captures/https-analysis.pcap
```

![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração/5.gif)

---

### 6. Análise de Pacotes via Wireshark

Abertura do arquivo `https-analysis.pcap` extraído da EC2 no Wireshark para auditoria profunda do fluxo TCP/TLS.

- **3-Way Handshake TCP:** Identificação dos pacotes `SYN`, `SYN-ACK`, `ACK`.
- **TLS Handshake:** Visualização das mensagens `Client Hello` e `Server Hello`.
- **Criptografia:** Comprovação de que o payload transmitido está cifrado (Application Data).

![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração/6.gif)

---

### 7. Simulação e Troubleshooting de Incidente de Segurança

**Cenário Real:** A aplicação HTTPS parou de responder repentinamente.  
**Metodologia Aplicada:** Sintoma ➔ Hipótese ➔ Diagnóstico ➔ Evidência ➔ Correção ➔ Re-teste.

1. **Validação do Serviço:** `sudo systemctl status nginx` *(Resultado: Nginx ativo/OK)*.
2. **Verificação de Sockets:** `ss -tuln` *(Resultado: Porta 443 ouvindo localmente/OK)*.
3. **Teste de Conectividade:** `nc -vz SEU_DOMINIO 443` *(Resultado: Connection Timed Out)*.
4. **Causa Raiz Identificada:** Regra de Inbound para a porta TCP `443` removida do Security Group da AWS.
5. **Resolução:** Reinstalação da regra no Security Group liberando tráfego `0.0.0.0/0:443`.
6. **Validação Final:** `curl -I https://SEU_DOMINIO` *(Resultado: HTTP/1.1 200 OK)*.

![Demonstração do Nginx e Reverse Proxy](./Videos%20Demonstração)

---

## ✍️ Autora

**Isadora Vanderlan**  
- **LinkedIn:** [(https://www.linkedin.com/in/isadoravanderlan/)]  


## 🤝 Agradecimentos

Agradecimento especial ao **Edson Bezerra** (_Manager, LATAM Cyber Security Infrastructure Services - DXC Technology_) pela mentoria, orientações estratégicas e incentivo na estruturação deste plano de estudos.
