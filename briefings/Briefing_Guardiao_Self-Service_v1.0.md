# **Briefing: Agente Órby Guardião Essencial (Self-Service) v1.0**

**Referência:** Dossiê Mestre OrbythAI v4.1 (fonte única) \+ Contrato Master de Dados & API v4.1  
 **Para:** Equipe N8N/Supabase (Enzo) e Equipe SaaS (Byth one)  
 **De:** Suzy (Estrategista)  
 **Data:** 28 de agosto de 2025

**Alinhamento MVP:** Este agente será entregue no **MVP Enterprise (assistido)** no Sprint 4\. A **ativação Self-Service** (vitrine e checkout) fica para a **Fase 2 (pós-MVP)** — porém toda a **lógica do Guardião** e o **Gestor de Contatos Órby** já são usados no MVP.

## **1\. Missão do Agente**

Atuar como **inteligência e governança da carteira**. Detecta **inatividade/risco**, gera **alertas** e aciona **reengajamento** (mensagens sugeridas por IA e/ou handoff ao Elo), usando a base do cliente no nosso SaaS.

## **2\. Pilar**

**Órby Flow (Comercial).**

## **3\. Proposta de Valor**

Ideal para empresas **sem CRM** ou iniciando organização comercial. Entrega o **Gestor de Contatos Órby** (flexível, com campos customizáveis) \+ **políticas de carteira** \+ **IA de reengajamento**.

## **4\. Fonte de Dados (Conforme BaaS v4.1)**

* **Core:** `accounts`, `contacts (custom_fields JSON)`, `interactions`, `messages`, `tasks`.

* **Guardião:** `guardian_policies` (regras), `guardian_alerts` (alertas).

* **Importação:** `contacts_import_staging` (planilha) com **mapeamento assistido por IA**.

* O “Gestor de Contatos Órby” é a **UI** sobre `contacts` (com gerenciador de campos).

## **5\. Interação com o SaaS (Visão do Cliente)**

* **Gestor de Contatos Órby:** CRUD de contatos, **Gerenciador de Campos** (custom), importação assistida por IA com validação.

* **Painel de Políticas Comerciais:** regras (ex.: **60 dias sem contato ⇒ criar alerta em 24h**).

* **Inbox de Alertas (Guardião):** lista, filtros, ações (fechar, encaminhar ao Elo, criar tarefa).

## **6\. Lógica do Workflow (N8N)**

1. **Scheduler diário** \+ **execução manual** (webhook).

2. **Segmentação:** ler `guardian_policies` e aplicar sobre `contacts`/`interactions` (ex.: `dias_sem_contato >= N`).

3. **Gerar Alerta:** inserir em `guardian_alerts` (contato, motivo, SLA).

4. **(Opcional IA)**: gerar **mensagem sugerida** de reengajamento → **registrar `usage_logs`** (regra **100 orbkens/consulta \+ 1/100 tokens**).

5. **Handoff Elo (opcional):** criar `lead`/`task` e acionar webhook do **Órby Elo** para qualificação/reativação.

6. **Auditoria:** gravar em `interactions/messages` todas as ocorrências.

## **7\. Estrutura de Dados (Supabase)**

* **`guardian_policies`** (id, tenant\_id, regra\_json, sla\_horas, ativo, created\_at).

* **`guardian_alerts`** (id, tenant\_id, contact\_id, motivo, risco, status, due\_at, resolved\_at, created\_at).

* **`contacts_import_staging`** (id, tenant\_id, arquivo\_id, mapeamento\_json, status, created\_at).

* **Views:** `guardian_segments_v` (contatos elegíveis por política).

* **Reuso Core:** `accounts`, `contacts`, `interactions`, `messages`, `tasks`, `usage_logs`, `audit_logs`.

* **Segurança:** **RLS por `tenant_id`** e **RBAC** (Admin/Gerente/Operador).

## **8\. Prisma Essencial do Guardião (Outputs)**

* **KPIs:** clientes monitorados, inativos, alertas abertos/fechados, **tempo médio de resolução**, **taxa de reativação**.

* **Consulta conversacional (limitada):** botão **\[✨ Analisar com IA\]** sobre as tabelas do Guardião (registra `usage_logs`).

## **9\. Onboarding e Configuração Assistidos (MVP Enterprise)**

* **Setup Wizard:** criar 1ª política, definir parâmetros de inatividade e SLA.

* **Configuração Contínua:** ajustes de políticas, campos, times responsáveis.

* **Ciclo de Feedback (👍/👎):** registrar em `interactions/messages` com campo `feedback`.

* **Playground:** simular reengajamento antes de produção.

## **10\. Contrato de Dados do Agente (Supabase)**

* **Lê:** `accounts`, `contacts`, `interactions`, `messages`, `tasks`, `guardian_policies`, `guardian_alerts`, `guardian_segments_v`.

* **Escreve:** `guardian_alerts`, `tasks`, `interactions`, `messages` (+ `usage_logs` quando houver IA).

* **Eventos:** scheduler e webhook manual; opcional trigger ao salvar política.

* **Rastreio:** `usage_logs` (**100 \+ 1/100 tokens**), `audit_logs`.

* **Políticas:** RLS \+ RBAC. Segue **Contrato Master v4.1**.

## **11\. Entregáveis (Sprint 4 – Guardião Essencial)**

**Equipe SaaS**

* **Gestor de Contatos Órby** (CRUD, custom fields, importador assistido).

* **Painel de Políticas** (CRUD em `guardian_policies`).

* **Inbox de Alertas** (listar/filtrar/fechar/encaminhar ao Elo).

* **Prisma Essencial do Guardião** (widgets \+ **\[✨ Analisar com IA\]**).  
   **Equipe N8N/Supabase (Enzo)**

* Workflow: segmentação → alertas → (opcional IA) → handoff Elo → auditoria.

* SQL: `guardian_policies`, `guardian_alerts`, `guardian_segments_v`, `contacts_import_staging`.

* **usage\_logs** com regra **100 \+ 1/100 tokens**, **audit\_logs**.

* Webhooks/payloads conforme **Contrato Master v4.1**.

---

