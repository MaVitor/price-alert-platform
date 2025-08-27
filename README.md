# Plataforma de Alerta de Preços - Arquitetura de Microsserviços

Este projeto é uma plataforma completa de monitoramento de preços da Amazon, construída utilizando uma arquitetura de microsserviços para demonstrar a aplicação de diferentes tecnologias para resolver problemas específicos.

O sistema é orquestrado utilizando Docker e Docker Compose, permitindo que todo o ambiente de desenvolvimento seja iniciado com um único comando.

## Arquitetura

A plataforma é composta pelos seguintes serviços independentes, cada um em seu próprio repositório e contêiner:

- **`data-api-php` (API de Dados):**
  - **Tecnologia:** PHP / Lumen
  - **Responsabilidade:** Único serviço que se comunica com o banco de dados. Gerencia todas as operações de CRUD (Criar, Ler, Atualizar, Deletar) para produtos e contatos. É a fonte central da verdade para os dados da aplicação.
  - **Porta:** `8000`

- **`scraper-service-rust` (Serviço de Scraping):**
  - **Tecnologia:** Rust / Actix Web
  - **Responsabilidade:** Serviço de alta performance focado em uma única tarefa: receber a URL de um produto e extrair (fazer o "scrape") do seu preço atual na página da Amazon.
  - **Porta:** `8082`

- **`notificador-service-go` (Serviço de Notificação):**
  - **Tecnologia:** Go (Golang)
  - **Responsabilidade:** Serviço concorrente e de baixo consumo de recursos, projetado para enviar um grande volume de notificações (atualmente via Telegram) de forma rápida e eficiente.
  - **Porta:** `8080`

- **`database` (Banco de Dados):**
  - **Tecnologia:** PostgreSQL (Imagem Oficial Docker)
  - **Responsabilidade:** Armazenar de forma persistente todos os dados da aplicação, como produtos, contatos e histórico de preços.
  - **Porta:** `5432`

## Pré-requisitos

Para executar este projeto, você precisa ter instalado na sua máquina:
- [Git](https://git-scm.com/)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (que já inclui o Docker Compose)

## Como Executar a Plataforma

Siga os passos abaixo para configurar e iniciar todo o ambiente.

### 1. Clone os Repositórios

Primeiro, clone este repositório principal e, em seguida, clone cada um dos microsserviços para dentro dele.

```bash
# Clone o repositório principal
git clone [https://github.com/MaVitor/price-alert-platform.git](https://github.com/MaVitor/price-alert-platform.git)
cd price-alert-platform

# Clone os microsserviços para dentro da pasta principal
git clone [https://github.com/MaVitor/data-api-php.git](https://github.com/MaVitor/data-api-php.git)
git clone [https://github.com/MaVitor/scraper-service-rust.git](https://github.com/MaVitor/scraper-service-rust.git)
git clone [https://github.com/MaVitor/notificador-service-go.git](https://github.com/MaVitor/notificador-service-go.git)
Ao final, sua estrutura de pastas deve ser:price-alert-platform/
├── data-api-php/
├── notificador-service-go/
├── scraper-service-rust/
└── docker-compose.yml
2. Configure as Variáveis de AmbienteO projeto utiliza um arquivo .env na raiz para gerenciar todas as chaves e configurações.Renomeie o arquivo env.example para .env.Abra o arquivo .env e preencha com suas chaves de API e credenciais.3. Inicie o AmbienteCom o Docker Desktop em execução, execute o seguinte comando na raiz do projeto price-alert-platform:# O --build garante que as imagens dos seus serviços sejam construídas na primeira vez
docker-compose up --build
Este comando irá baixar a imagem do PostgreSQL, construir as imagens de cada um dos seus microsserviços e iniciar todos os contêineres. Você verá os logs de todos os serviços no seu terminal.Comandos Úteis do Docker ComposeIniciar em segundo plano (detached mode):docker-compose up -d
Parar todos os serviços:docker-compose down
Ver os logs de um serviço específico:docker-compose logs -f api
