🏗️ Ticket System Backend - Hub de Orquestração

Este repositório é o ponto central da arquitetura do Sistema de Gestão de Chamados. Ele atua como o orquestrador de um ecossistema distribuído, gerenciando microsserviços, mensageria assíncrona, isolamento multi-tenant e persistência relacional de alta integridade.

🌌 Arquitetura do Sistema

A solução foi desenhada sob os princípios de Clean Architecture e Event-Driven Design, garantindo escalabilidade e isolamento total entre empresas (tenants):

Segurança (Access Control Manager): Serviço de identidade responsável pela autenticação RBAC (Role-Based Access Control) e emissão de tokens JWT com claims de tenant_id.

Core Business (Ticket Service): Microsserviço responsável pelo ciclo de vida dos chamados, regras de transição de status e gestão de SLAs.

Mensageria (RabbitMQ): Broker utilizado para processamento assíncrono de tickets críticos e disparos de fluxos de aprovação, garantindo que a API tenha baixa latência.

Persistência (PostgreSQL): Bancos de dados isolados por serviço, utilizando Migrations automáticas (Flyway) para garantir a evolução controlada do schema.

📂 Estrutura de Pastas (Organizacional)

O docker-compose.yml espera que os repositórios sejam clonados em pastas irmãs para o correto funcionamento do contexto de build multi-stage:

Plaintext
 ticket-system-backend/        <-- (Repositório Principal / Você está aqui)
 ├── init-db/                  <-- Scripts de inicialização do PostgreSQL
 ├── docker-compose.yml        <-- Orquestrador Geral
 └── README.md

 access-control-manager/       <-- Microsserviço de Identidade e Acesso
 ticket-service/               <-- Microsserviço de Gestão de Chamados
 shared-contracts/             <-- Biblioteca Java de Contratos (DTOs/Events)

🏗️ Diferenciais de Nível Sênior

🛡️ Isolamento Multi-tenancy Real

Diferente de sistemas simples, este projeto implementa isolamento em nível de banco de dados via Hibernate Filters e ThreadLocal. Toda query executada pelo Ticket-Service é automaticamente filtrada pelo tenant_id extraído do Token JWT, impedindo qualquer vazamento de dados entre empresas.

⚡ Processamento Assíncrono e Resiliência

O upload de anexos e o cálculo de criticidade são processados fora do ciclo de requisição HTTP.

Se o volume de uploads for alto, o sistema não trava o usuário; as mensagens são enfileiradas no RabbitMQ e processadas conforme a disponibilidade.

Utilização de Virtual Threads (Java 21) para otimização de I/O em tarefas bloqueantes.

📂 Gestão Inteligente de Anexos

O sistema gerencia o armazenamento físico de arquivos no disco do servidor, organizando-os por pastas vinculadas ao UUID do tenant, facilitando backups e auditorias.

🚀 Como Executar o Ecossistema

1. Preparação
   Certifique-se de que todos os repositórios citados na Estrutura de Pastas foram clonados.

Bash
mvn clean install -DskipTests

2. Subida do Ambiente via Docker
   Na raiz deste repositório (ticket-system-backend), execute:

Bash
docker-compose up -d --build

O comando irá compilar os fontes, criar os bancos de dados access_control_db e ticket_db, configurar as filas no RabbitMQ e expor as APIs.

📡 Portas e Documentação (Swagger)

IAM / Access Control: http://localhost:8080/swagger-ui.html

Ticket Service: http://localhost:8081/swagger-ui.html

PgAdmin: http://localhost:5050 (admin@admin.com / admin)

RabbitMQ Console: http://localhost:15672 (guest / guest)

🧪 Cenários de Teste Obrigatórios

Para validar a robustez da solução, o sistema foi testado nos seguintes cenários:

Segregação de Dados: Tentativa de acesso a um Ticket do Tenant A usando Token do Tenant B (Retorno: 404/403).

Workflow de Aprovação: Criação de Ticket CRITICAL gerando pendência automática para MANAGER.

Integridade de Status: Tentativa de fechar (CLOSED) um ticket sem antes passar por atendimento (IN_PROGRESS).

🚀 Links do Ecossistema

Hub Principal (Orquestrador Docker): https://github.com/leafarortasac/ticket-system-backend

Segurança (Access Control Manager): https://github.com/leafarortasac/access-control-manager

Gerenciamento de tickets (Ticket Service): https://github.com/leafarortasac/ticket-service

Contratos (Shared Contracts): https://github.com/leafarortasac/shared-contracts