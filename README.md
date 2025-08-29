# Plataforma de Alerta de Preços - Arquitetura de Microsserviços

## Visão Geral

Este projeto é uma plataforma completa para monitoramento de preços de produtos na Amazon, desenvolvida com uma arquitetura de microsserviços. O objetivo principal é demonstrar a aplicação de diferentes tecnologias, cada uma escolhida para resolver um problema específico de forma otimizada, criando um sistema coeso, escalável e de fácil manutenção.

O sistema é totalmente orquestrado utilizando Docker e Docker Compose, o que permite que todo o ambiente de desenvolvimento e produção seja configurado e iniciado com um único comando.

## Fluxo de Funcionamento

1.  O **`worker-service-python`** (orquestrador) inicia seu ciclo.
2.  Ele solicita à **`data-api-php`** a lista de todos os produtos cadastrados para monitoramento.
3.  Para cada produto, o Worker envia a URL para o **`scraper-service-rust`**.
4.  O Scraper acessa a página da Amazon, extrai o preço atual do produto e o retorna ao Worker.
5.  O Worker compara o preço recebido com o preço armazenado.
6.  Se houver alteração de preço, o Worker atualiza o valor na **`data-api-php`**.
7.  Caso o novo preço seja igual ou menor que o preço desejado pelo usuário, o Worker aciona o **`notificador-service-go`**.
8.  O Notificador envia uma mensagem de alerta para o contato associado ao produto.
9.  O ciclo recomeça.

## Arquitetura

A plataforma é composta pelos seguintes serviços independentes, cada um executando em seu próprio contêiner Docker:

  - **`data-api-php` (API de Dados)**

      - **Tecnologia:** PHP / Lumen
      - **Responsabilidade:** É o único serviço que se comunica diretamente com o banco de dados. Gerencia todas as operações de CRUD para produtos e contatos, atuando como a fonte central da verdade para os dados da aplicação.
      - **Porta Interna:** `8000`

  - **`scraper-service-rust` (Serviço de Scraping)**

      - **Tecnologia:** Rust / Actix Web
      - **Responsabilidade:** Serviço de altíssima performance, focado em uma única tarefa: receber a URL de um produto e extrair seu preço atual na página da Amazon de forma rápida e eficiente.
      - **Porta Interna:** `8082`

  - **`notificador-service-go` (Serviço de Notificação)**

      - **Tecnologia:** Go (Golang)
      - **Responsabilidade:** Serviço concorrente e de baixo consumo de recursos, projetado para enviar um grande volume de notificações de alteração de preço para os usuários finais.
      - **Porta Interna:** `8080`

  - **`worker-service-python` (Serviço Orquestrador)**

      - **Tecnologia:** Python
      - **Responsabilidade:** Atua como o "maestro" do sistema. Em um ciclo contínuo, ele busca os produtos na API, coordena a raspagem de preços com o Scraper, comanda a atualização dos dados na API e, quando necessário, aciona o serviço de Notificação.

  - **`database` (Banco de Dados)**

      - **Tecnologia:** PostgreSQL (Imagem Oficial Docker)
      - **Responsabilidade:** Armazenar de forma persistente todos os dados da aplicação, como informações de produtos, preços e contatos.
      - **Porta Interna:** `5432`

## Pré-requisitos

Para executar este projeto, você precisa ter as seguintes ferramentas instaladas em sua máquina:

  - [Git](https://git-scm.com/)
  - [Docker Desktop](https://www.docker.com/products/docker-desktop/) (que já inclui o Docker Compose)

## Como Executar a Plataforma

Siga os passos abaixo para configurar e iniciar todo o ambiente de microsserviços.

### 1\. Clonar os Repositórios

Para que o `docker-compose.yml` funcione corretamente, os repositórios de cada microsserviço devem estar organizados na estrutura de pastas correta.

```bash
# Clone o repositório orquestrador principal
git clone https://github.com/MaVitor/price-alert-platform.git
cd price-alert-platform

# Clone todos os microsserviços necessários
git clone https://github.com/MaVitor/data-api-php.git
git clone https://github.com/MaVitor/scraper-service-rust.git
git clone https://github.com/MaVitor/notificador-service-go.git
git clone https://github.com/MaVitor/worker-service-python.git
```

Ao final, sua estrutura de pastas deve ser a seguinte:

```
.
└── price-alert-platform/     # Repositório principal (onde está o docker-compose.yml)
    ├── data-api-php/         # Repositório do serviço de dados
    ├── scraper-service-rust/ # Repositório do serviço de scraping
    ├── notificador-service-go/ # Repositório do serviço de notificação
    └── worker-service-python/  # Repositório do serviço do worker
```

### 2\. Configurar as Variáveis de Ambiente

O projeto utiliza um arquivo `.env` na raiz para gerenciar todas as chaves de API, credenciais de banco de dados e outras configurações.

```bash
# Na raiz do projeto 'price-alert-platform', copie o arquivo de exemplo
cp env.example .env
```

Após copiar, abra o arquivo `.env` e preencha com suas próprias chaves e credenciais.

### 3\. Iniciar o Ambiente

Com o Docker Desktop em execução, execute o seguinte comando a partir da pasta raiz (`price-alert-platform`):

```bash
# O argumento --build garante que as imagens de cada serviço sejam construídas do zero
docker-compose up --build
```

Este comando irá:

1.  Baixar a imagem oficial do PostgreSQL.
2.  Construir as imagens Docker para cada um dos microsserviços (`data-api-php`, `scraper-service-rust`, etc.).
3.  Iniciar todos os contêineres e conectá-los na mesma rede.

Você verá os logs de todos os serviços sendo exibidos em tempo real no seu terminal.

## Comandos Úteis do Docker Compose

Aqui estão alguns comandos úteis para gerenciar o ambiente:

  - **Iniciar em segundo plano (detached mode):**

    ```bash
    docker-compose up -d
    ```

  - **Parar e remover todos os contêineres, redes e volumes:**

    ```bash
    docker-compose down
    ```

  - **Ver os logs de um serviço específico em tempo real:**

    ```bash
    # Exemplo para ver os logs da API de dados
    docker-compose logs -f data-api-php
    ```

  - **Forçar a reconstrução de um serviço específico:**

    ```bash
    docker-compose up --build --no-deps -d scraper-service-rust
    ```

  - **Acessar o terminal de um contêiner em execução:**

    ```bash
    # Exemplo para acessar o shell do contêiner da API
    docker-compose exec data-api-php /bin/sh
    ```