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
