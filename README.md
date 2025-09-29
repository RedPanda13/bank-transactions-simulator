# bank-transactions-simulator

## 📌 Visão Geral
Este projeto implementa um **simulador de transações PIX** baseado em **Saga com compensação** e no **Outbox Pattern**, garantindo consistência entre banco de dados e eventos publicados.  
O sistema processa transações de forma assíncrona (via **RabbitMQ**), suportando falhas, retries e observabilidade futura com Grafana.

---

## 🚀 Objetivos
- Simular transações PIX entre contas internas usando chaves válidas.  
- Garantir consistência entre o **Postgres** (fonte de verdade) e o sistema de eventos (**Outbox Pattern**).  
- Processamento assíncrono com suporte a retries, DLQ e compensação em caso de falha.  
- Permitir extensões futuras (tarifas, chargebacks, múltiplos métodos de pagamento).  

---

## 📂 Estrutura
- **API** (`/pix/transactions`) — criação de transações.
- **Outbox Publisher** — publica eventos `pix.transaction.created` no RabbitMQ.
- **Workers** — consomem eventos, processam débitos/créditos e atualizam status.
- **Postgres** — persistência de contas, chaves PIX, transações e outbox.
- **RabbitMQ** — transporte de eventos.
- **Grafana** — observabilidade (futuro: dashboards de logs e métricas).

---

## 🔄 Fluxo da Transação
1. Cliente chama `POST /pix/transactions`.  
2. Transação salva no banco com status `PENDING` e evento gravado na **outbox**.  
3. Outbox Publisher lê e publica evento `pix.transaction.created` no **RabbitMQ**.  
4. Worker consome evento, valida regras de negócio e:  
   - Atualiza saldos e marca `COMPLETED`.  
   - Ou marca `FAILED` (ex.: saldo insuficiente).  

---

## 📦 Modelo de Dados
As principais tabelas são:  
- **accounts** → contas e saldo.  
- **pix_keys** → chaves PIX.  
- **transactions** → transações com status (`PENDING`, `COMPLETED`, `FAILED`).  
- **outbox_events** → eventos pendentes de publicação.  

Veja [db/init.sql](./db/init.sql) para o DDL completo.  

---

## ⚡ Exemplos de API
### Criar Conta
```http
POST /api/v1/accounts
{
  "owner_name": "Alice",
  "initial_balance": 1000.00
}
Criar Transação PIX
http
Copiar código
POST /api/v1/pix/transactions
{
  "external_id": "ext-123",
  "from_account_id": "uuid-from",
  "to_pix_key": "alice@example.com",
  "amount": 150.75
}
Resposta:

json
Copiar código
{
  "id": "tx-uuid",
  "status": "PENDING",
  "amount": 150.75
}
🔐 Segurança
API autenticada (JWT ou API Key).

TLS obrigatório.

Rate limiting por conta/IP.

Validação rigorosa de entradas (especialmente valores e chaves PIX).

🏗️ Deploy Local
Com Docker Compose, os serviços são:

Postgres → banco principal.

RabbitMQ → broker de eventos (com UI em http://localhost:15672, usuário/pwd padrão: guest/guest).

API → exposta em http://localhost:8080.

Worker → processador de transações.

Outbox Publisher → publicador de eventos confiáveis.

Grafana → acessível em http://localhost:3000 (login: admin/admin).

Subir tudo:

sh
Copiar código
docker-compose up -d
✅ Checklist
 Criar schema DB e seeds

 Implementar API e endpoints principais

 Implementar Outbox Publisher (com RabbitMQ)

 Implementar Worker (consumer RabbitMQ)

 Criar dashboards no Grafana

 Criar testes unitários/integração/E2E
