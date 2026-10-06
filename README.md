# AWS SSH-Chat Server

Esse projeto é uma implementação de um servidor de chat via terminal (SSH-CHAT) hospedado em uma instância EC2 na nuvem AWS. O foco principal desta infraestrutura é a aplicação de conceitos de segurança em nuvem, englobando configurações restritas de firewall, gerenciamento de chaves de autenticação e configuração de serviços no Linux para garantir alta disponibilidade.

## Arquitetura e Tecnologias

* **Cloud Provider:** AWS (Amazon Web Services)
* **Compute:** Instância EC2 (Tipo: t3.micro)
* **Sistema Operacional:** ubuntu-noble-24.04-amd64-server-20260610
* **Aplicação:** [ssh-chat](https://github.com/shazow/ssh-chat) (Desenvolvido em Go)

## Hardening e Segurança

A segurança do servidor foi priorizada desde o provisionamento, utilizando as seguintes práticas:

* **Security Groups (Firewall AWS):** Aplicação do Princípio do Menor Privilégio.
  * **Porta 2222 (SSH-Chat):** Aberta para comunicação dos usuários.
  * **Porta 22 (SSH Admin):** Acesso estritamente restrito ao IP do administrador, bloqueando tentativas de força bruta automatizadas globais.
* **Autenticação:** Login baseado exclusivamente em chaves criptográficas (Key Pairs `.pem`), com a desativação da autenticação por senha no serviço SSH padrão do Linux.

## Como Conectar

Para acessar o servidor de chat e testar a comunicação, utilize qualquer cliente SSH em seu terminal:

```bash
ssh visitante@15.229.111.31 -p 2222
```
## Guia de Implementação Manual

O provisionamento foi realizado manualmente via AWS Management Console seguindo estes passos:

1. **Provisionamento EC2:** Lançamento da instância t3.micro utilizando a AMI do Ubuntu 24.04 LTS e configuração do Key Pair (.pem) para acesso administrativo seguro.
2. **Configuração de Rede:** Criação de um Security Group customizado para gerenciar o tráfego de entrada (Inbound Rules) limitando o acesso administrativo à porta 22 e liberando o serviço na porta 2222.
3. **Deploy da Aplicação:**
   * Acesso à instância via SSH administrativo.
   * Download do binário compilado do `ssh-chat`.
   * Atribuição de permissões de execução (`chmod +x`).

## Alta Disponibilidade (Serviço Systemd)

Para garantir que o servidor de chat reinicie automaticamente em caso de falha ou reboot da instância EC2, a aplicação foi configurada como um serviço nativo no sistema operacional:

**Arquivo de configuração criado em:** `/etc/systemd/system/ssh-chat.service`

```ini
[Unit]
Description=SSH Chat Server
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/home/ubuntu/ssh-chat/ssh-chat --bind "0.0.0.0:2222"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```
