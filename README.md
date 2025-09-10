# Projeto MQTT - Exploração e Correção de Vulnerabilidades

Este projeto tem como objetivo identificar e corrigir vulnerabilidades
de segurança em um **broker MQTT Mosquitto**, aplicando **boas práticas
de autenticação, controle de acesso e criptografia TLS**.

------------------------------------------------------------------------

## Estrutura do Projeto

-   **mosquitto/config/mosquitto.conf** → Arquivo principal de
    configuração do broker.\
-   **mosquitto/config/passwordfile** → Arquivo contendo usuários e
    senhas (hash) para autenticação.\
-   **mosquitto/config/mosquitto.acl** → Lista de controle de acesso
    (ACL) que define permissões de publicação/assinatura.\
-   **mosquitto/certs/** → Diretório contendo certificados e chaves TLS
    (.crt e .key).

------------------------------------------------------------------------

## Vulnerabilidades Identificadas

1.  **Acesso anônimo liberado**
    -   O broker permitia conexões sem autenticação.
2.  **Falta de autenticação de usuário**
    -   Não havia usuários definidos para controlar o acesso ao broker.
3.  **Ausência de criptografia TLS**
    -   As conexões eram realizadas sem proteção contra interceptação.

------------------------------------------------------------------------

## Correções Implementadas

1.  **Bloqueio de acesso anônimo**
    -   Configurado `allow_anonymous false` no `mosquitto.conf`.
2.  **Criação de usuário e senha**
    -   Foi criado o usuário `USUARIO` no arquivo `passwordfile`, com
        senha armazenada em hash.
3.  **Configuração de ACL (Access Control List)**
    -   Criado o arquivo `mosquitto.acl`, permitindo que apenas o
        usuário `USUARIO` publique mensagens.
4.  **Implementação de TLS**
    -   Foram gerados certificados e chave de criptografia (`.crt` e
        `.key`), configurados no `mosquitto.conf` para habilitar
        conexões seguras na porta **8883**.

------------------------------------------------------------------------

## Configuração de Portas

-   **Porta 1883** → Comunicação interna (sem TLS, para testes e
    sensores locais).\
-   **Porta 8883** → Comunicação externa segura, com **TLS e
    autenticação obrigatória**.

------------------------------------------------------------------------

## Como Executar o Projeto

1.  **Criar diretórios necessários**

    ``` bash
    mkdir -p mosquitto/{config,data,log,certs}
    ```

2.  **Gerar certificados TLS**

    ``` bash
    cd mosquitto/certs

    # Gerar chave privada da CA
    openssl genrsa -out ca.key 4096

    # Gerar certificado da CA
    openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -subj "/CN=MyTestCA" -out ca.crt

    # Gerar chave e CSR do servidor
    openssl genrsa -out server.key 4096
    openssl req -new -key server.key -subj "/CN=mqtt-broker" -out server.csr

    # Assinar certificado do servidor
    openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 365 -sha256
    ```

3.  **Criar usuário no broker**

    ``` bash
    mosquitto_passwd -c mosquitto/config/passwordfile USUARIO
    ```

4.  **Definir ACL** (mosquitto/config/mosquitto.acl)

        user USUARIO
        topic readwrite sensor/#

5.  **Subir o broker com Docker Compose**

    ``` bash
    docker compose up -d --build
    ```

------------------------------------------------------------------------

## Testes

-   **Publicar mensagem (sem TLS, porta 1883):**

    ``` bash
    mosquitto_pub -h localhost -p 1883 -t 'sensor/temperature' -m '27.5'
    ```

-   **Assinar tópico (com TLS, porta 8883):**

    ``` bash
    mosquitto_sub -h localhost -p 8883 --cafile ./mosquitto/certs/ca.crt   -t 'sensor/#' -v --tls-version tlsv1.2 -u USUARIO -P <SENHA>
    ```

------------------------------------------------------------------------

## Resumo Técnico

-   O acesso anônimo foi desabilitado.\
-   Usuário `USUARIO` criado com autenticação por senha.\
-   ACL implementada para restringir permissões.\
-   TLS habilitado para proteger a comunicação externa.\
-   Segregação de portas configurada (1883 → interno / 8883 → externo).

Este projeto demonstra como corrigir falhas comuns de segurança em
brokers MQTT, garantindo **autenticação, criptografia e controle de
acesso** de forma prática e funcional.
