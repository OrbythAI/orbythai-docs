# **Briefing: Órby Guardião Enterprise (API Integrado) v1.0**

**Referência:** Dossiê Mestre OrbythAI v4.1 (fonte única) \+ Contrato Master de Dados & API v4.1  
 **Para:** Equipe N8N/Supabase (Enzo) e Equipe SaaS (Byth one)  
 **De:** Suzy (Estrategista)  
 **Data:** 28 de agosto de 2025

## **1\. Missão do Agente**

Ser a **camada de inteligência** sobre o **CRM/ERP** do cliente: conectar em tempo real, aplicar políticas de carteira e **reativar clientes** com **IA preditiva** e/ou handoff ao Elo.

## **2\. Pilar**

**Órby Flow (Comercial).**

## **3\. Proposta de Valor**

Para empresas com sistemas estabelecidos: o Guardião **potencializa** o CRM/ERP existente, adicionando **alertas proativos**, **playbooks** e **reengajamento inteligente**, sem mudar o sistema de trabalho da equipe.

## **4\. Fonte de Dados**

* **API do CRM/ERP** do cliente (pull via `HTTP Request` **e/ou** push via **webhooks**).

* **Mapeamento/normalização** para nosso modelo (`accounts`, `contacts`, `interactions`).

## **5\. Interação com o SaaS (Visão do Cliente)**

* O time do cliente permanece no **CRM/ERP**.

* No nosso SaaS: **Painel de Políticas**, **Inbox de Alertas** e **Prisma Essencial**.

* **Configurações de Integração** (chaves, endpoints, mapeamento de campos).

## **6\. Lógica do Workflow (N8N)**

1. **Entrada por API**: Pull (agendado) e/ou **webhooks** (event-driven).

2. **Nó Adaptador (Mapeamento):** converter payload externo para nosso modelo.

3. **Motor comum do Guardião:** políticas → alertas → IA opcional → handoff Elo.

4. **Sincronismo:** opcionalmente escrever **status/tags** de volta no CRM/ERP.

5. **Auditoria e Consumo:** `interactions/messages`, `usage_logs` (IA).

## **7\. Estrutura de Dados (Supabase)**

* **Reuso Core:** `accounts`, `contacts`, `interactions`, `messages`, `tasks`.

* **Guardião:** `guardian_policies`, `guardian_alerts`, `guardian_segments_v`.

* **Integração:** `integration_accounts_map`, `integration_contacts_map` (chaves CRM↔SaaS).

* **Segurança:** **RLS por `tenant_id`** e **RBAC** (Admin/Gerente/Operador).

## **8\. Prisma Essencial (Enterprise)**

* Dashboards com **KPIs** (alertas, resolução, reativação, SLAs) e **drill-down por origem** (CRM/ERP).

* **Consulta conversacional (limitada):** **\[✨ Analisar com IA\]** sobre dados sincronizados (com `usage_logs`).

## **9\. Onboarding e Configuração Assistidos (MVP Enterprise)**

* **Setup Wizard de Integração:** credenciais, endpoints, mapeamento de campos.

* **Políticas:** criar política inicial (ex.: 60 dias sem contato) e SLA.

* **Playground/Validação:** simular alertas com amostra real.

## **10\. Contrato de Dados do Agente (Supabase)**

* **Lê:** `guardian_policies`, `guardian_alerts`, `guardian_segments_v`, `integration_*`, além do core (`accounts`, `contacts`, `interactions`, `messages`, `tasks`).

* **Escreve:** `guardian_alerts`, `tasks`, `interactions`, `messages`, **usage\_logs**.

* **Eventos:** pull agendado \+ webhooks do CRM/ERP; RPC de sincronismo reverso (opcional).

* **Rastreio:** `usage_logs` (**100 \+ 1/100 tokens**), `audit_logs`.

* **Políticas:** RLS \+ RBAC. Segue **Contrato Master v4.1**.

## **11\. Entregáveis (Sprint 4 – Guardião Enterprise)**

**Equipe SaaS**

* **Painel de Políticas** (CRUD \+ versionamento/redeploy).

* **Inbox de Alertas** (filtros, bulk actions, encaminhar Elo).

* **Prisma Essencial** (widgets \+ **\[✨ Analisar com IA\]**).

* **Configurações de Integração** (chaves/API, mapeamentos).  
   

  **Equipe N8N/Supabase (Enzo)**

* Workflows: **pull** por API \+ **webhook** (push), adaptador, motor comum.

* SQL/Views/RPC: `guardian_policies`, `guardian_alerts`, `guardian_segments_v`, `integration_*`.

* Documentação de **endpoints/payloads** exigidos do CRM/ERP do cliente.

* **usage\_logs** e **audit\_logs** nos pontos de IA/ações.

