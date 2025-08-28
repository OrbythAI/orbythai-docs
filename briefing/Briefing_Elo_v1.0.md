# **Briefing de Desenvolvimento: Agente “Órby Elo” v1.0**

**Referência:** Dossiê Mestre OrbythAI v4.1 (fonte única) \+ Contrato Master de Dados & API v4.1  
 **Para:** Equipe N8N/Supabase (Enzo) e Equipe SaaS  
 **De:** Suzy (Estrategista)  
 **Data:** 28 de agosto de 2025

## **1\. Visão Geral e Objetivos**

O **Órby Elo** é o **SDR digital centralizado** da plataforma. Ele recebe leads e oportunidades de várias origens (no MVP: **Garimpeiro** e **Guardião**; Fase 2: **Astro**) e conduz a **qualificação** até um dos desfechos-alvo:

* **Agendar Reunião** (handoff para equipe comercial).

* **Gerar Proposta** (handoff para **Órby Orçamentista**).

* **Reativação/Follow-up** (handoff para **Guardião**).

No MVP, o Elo opera em **automação parcial (human-in-the-loop)**: a IA sugere a abordagem e o usuário aprova/dispara. A automação completa em lote fica para a Fase 2\.

## **2\. Arquitetura da Funcionalidade (Como as Peças se Conectam)**

\[Interface do Cliente no SaaS\] ↔ \[API do SaaS\] ↔ \[Supabase (tabelas, views, RLS)\] ↔ \[Workflows N8N (webhooks/triggers)\]

### **2.1 Segurança e Multi-tenancy**

* Todas as tabelas têm `tenant_id` com **RLS** e **RBAC** (Admin / Gerente / Operador).

* O Elo segue o **Contrato Master de Dados & API v4.1** (schemas, endpoints, webhooks, políticas).

* **usage\_logs** registra consumo de IA com regra: **100 orbkens por consulta \+ 1 orbken/100 tokens**.

---

## **3\. Especificações de Desenvolvimento por Componente**

### **3.1 Painel de Configuração do Elo (Inputs)**

**Descrição:** Definir como o Elo aborda e qualifica leads por **origem** (garimpeiro, guardiao, astro-futuro).  
 **Requisitos (SaaS):**

* **Editor de Scripts** (CRUD): múltiplos scripts por origem (ex.: “Residencial”, “Comercial”).

* **Objetivo Final** (enum): `reuniao` | `proposta` | `follow_up`.

* **Tom de Voz**: `formal` | `consultivo` | `comercial`.

* **Canais**: e-mail / WhatsApp / telefone (no MVP registra canal, sem envio automatizado).

* **Regras por Origem**: mapear script e objetivo padrão para `garimpeiro`, `guardiao` (Astro na Fase 2).  
   **Requisitos (N8N/Supabase):**

* Tabela **`elo_scripts`** (id, tenant\_id, origem, nome, conteudo\_json, objetivo\_default, tom\_voz, canais, ativo, created\_at).

* View **`elo_active_script_v`** filtrando o script corrente por origem/tenant.

### **3.2 Fila & Execução de Qualificação (Motor)**

**Descrição:** Unificar os leads em uma **fila** por prioridade e estado.  
 **Requisitos (SaaS):**

* **Fila de Leads** (kanban/funil): `Novos` → `Em Qualificação` → `Qualificados` → `Desqualificados`.

* **Ações**: \[Assumir Conversa\], \[Enviar Mensagem Sugerida\], \[Desqualificar\], \[Handoff p/ Orçamentista\], \[Agendar Reunião\].

* **Transcrição** completa por lead (histórico).  
   **Requisitos (N8N/Supabase):**

* Usar tabelas centrais: `leads`, `opportunities`, `interactions`, `messages`.

* Nova tabela **`elo_sessions`** (id, tenant\_id, lead\_id, script\_id, objetivo, canal, estado, created\_at, updated\_at).

* View **`elo_queue_v`** (unifica leads \+ estado Elo por tenant).

* **Triggers** Supabase → **webhooks N8N**:

  * on insert em `leads` com `origem in (garimpeiro, guardiao)` e `mode='auto'` → iniciar sessão Elo.

  * on update `elo_sessions.estado`→ eventos de step (ex.: primeira resposta).

### **3.3 Prisma Essencial (Outputs do Elo)**

**Descrição:** Painel de métricas e funil.  
 **KPIs:** taxa de resposta, taxa de qualificação, tempo até 1ª resposta, conversão por canal, conversão por script.  
 **Views:**

* **`prisma_elo_kpis_v`** (agregados por período/script/canal).

* **`prisma_elo_funnel_v`** (estágios do funil por origem).  
   **Consulta Conversacional (limitada):** botão **\[✨ Analisar com IA\]** para perguntas sobre o funil/KPIs do Elo (registra `usage_logs`).

### **3.4 Handoffs & Sincronismos**

**Para Órby Orçamentista (Proposta):**

* Criar **opportunity** (lead\_id, objetivo=`proposta`, stage=`pre_proposta`) e acionar webhook do Orçamentista.  
   **Para Reunião (Comercial):**

* Criar **task** `meeting_request` com payload (contato, datas sugeridas, objetivo).  
   **Para Guardião (Reativação/Follow-up):**

* Criar **task** `reactivation` e logar interação.  
   **Sincronismo de Status (bidirecional):**

* Ao **qualificar/desqualificar** no Elo, atualizar `leads.status` e `flow_oportunidades_garimpeiro.status` (quando origem for garimpeiro), preservando `origem`.

---

## **4\. Parte A: Especificações para a Equipe de SaaS (Interface do Cliente)**

### **A1. Painel de Configuração**

**Requisitos de Interface:**

* CRUD de **Scripts** (lista, criar, duplicar, versionar, ativar por origem).

* Seletores de **Objetivo**, **Tom de Voz** e **Canais**.

* Permissões: **Admin/Gerente** editam scripts; **Operador** só usa.

### **A2. Fila e Conversas (Funil Visual)**

**Requisitos de Interface:**

* Colunas: **Novos**, **Em Qualificação**, **Qualificados**, **Desqualificados** (drag & drop permitido com restrições).

* Cartão do lead: nome/empresa, origem, última interação, prioridade.

* Drawer de lead: **Transcrição** (messages), **Histórico de interações**, **Ações** (assumir, enviar mensagem sugerida, desqualificar, handoff).

* **Botões rápidos:** \[Assumir Conversa\], \[Enviar Mensagem Sugerida\], \[Desqualificar\], \[Gerar Proposta\], \[Agendar Reunião\].

* **Prisma Essencial**: widgets de KPIs e botão **\[✨ Analisar com IA\]**.

### **A3. Endpoints do SaaS (mínimo)**

* `GET /api/elo/queue?status=novo|qualificacao|qualificado|desqualificado`

* `POST /api/elo/assumir` {lead\_id}

* `POST /api/elo/sugerir-mensagem` {lead\_id} → (IA gera; não envia)

* `POST /api/elo/enviar-mensagem` {lead\_id, message\_id?} (MVP: grava e marca como enviada; automação real fica p/ Fase 2\)

* `POST /api/elo/desqualificar` {lead\_id, motivo}

* `POST /api/elo/handoff-proposta` {lead\_id}

* `POST /api/elo/agendar-reuniao` {lead\_id, …} (cria task)

* `GET /api/elo/transcricao/{lead_id}`

---

## **5\. Parte B: Especificações para a Equipe N8N/Supabase (Motor de Automação)**

### **B1. Fontes / Ferramentas**

* **Leads de entrada:** `leads` (origem=`garimpeiro|guardiao` no MVP; `astro` na Fase 2).

* **Conversas/IA:** OpenAI (GPT-4o ou superior) para geração de perguntas e mensagens sugeridas.

* **Persistência:** Supabase (`interactions`, `messages`, `elo_sessions`, `usage_logs`).

### **B2. Lógica de Workflow (N8N)**

* **Gatilho:** webhook “elo\_start” (manual) **ou** trigger de insert em `leads` (auto).

* **Carregar Script:** buscar em `elo_active_script_v` por origem/tenant.

* **Qualificação:** executar ciclo pergunta→resposta (human-in-the-loop: gerar **sugestão** e aguardar aprovação via API).

* **Critérios de Saída:**

  * **Qualificado** → criar `opportunity` e acionar Orçamentista OU criar `task` de reunião.

  * **Desqualificado** → atualizar `leads.status` e (se origem garimpeiro) `flow_oportunidades_garimpeiro.status`.

* **Registro de Consumo:** gravar em `usage_logs` a regra **100 orbkens por consulta \+ 1/100 tokens** por mensagem IA (guardar `corr_id`, `lead_id`, `script_id`, `tokens_in/out`, `custo_estimado`).

* **Auditoria:** gravar cada troca em `interactions/messages`.

### **B3. Estrutura de Dados (Supabase)**

* **`elo_scripts`** (id, tenant\_id, origem enum, nome, conteudo\_json, objetivo\_default, tom\_voz, canais, ativo bool, version, created\_at).

* **`elo_sessions`** (id, tenant\_id, lead\_id, script\_id, objetivo, canal, estado enum: novo|qualificacao|qualificado|desqualificado, last\_interaction\_at, created\_at, updated\_at).

* **Views:** `elo_queue_v`, `prisma_elo_kpis_v`, `prisma_elo_funnel_v`.

* **Reuso de tabelas core:** `leads`, `opportunities`, `interactions`, `messages`, `tasks`, `usage_logs`.

### **B4. Webhooks / Eventos**

* **Entrada:** `/webhooks/elo/start` (start/resume com `{lead_id}`).

* **Handoff Proposta:** `/webhooks/orcamentista/start` (payload do lead).

* **Atualização de Status:** função RPC para sincronizar `leads` \+ `flow_oportunidades_garimpeiro` quando origem=garimpeiro.

---

## **6\. Parte C: Onboarding e Configuração Assistidos (MVP Enterprise)**

* **Setup Wizard:** guia inicial para selecionar script por origem, objetivo final e canal.

* **Configuração Contínua:** ajustes em scripts, objetivos e tom de voz.

* **Ciclo de Feedback (👍/👎):** registrar em `interactions/messages` com campo `feedback`.

* **Playground:** simulação de abordagem antes de ativar na fila real.

---

## **7\. Contrato de Dados do Agente (Supabase)**

* **Lê:** `leads`, `opportunities`, `interactions`, `messages`, `elo_scripts`, `elo_sessions`, views `elo_queue_v`, `prisma_elo_*`.

* **Escreve:** `elo_sessions`, `interactions`, `messages`, `opportunities`, `tasks`, atualiza `leads.status`, e quando origem=garimpeiro, atualiza `flow_oportunidades_garimpeiro.status`.

* **Eventos (triggers):** insert em `leads` (origem+auto) → webhook `elo/start`.

* **Rastreio:** `usage_logs` (regra 100 \+ 1/100), `audit_logs` (quem aprovou envio).

* **RLS:** todas as operações condicionadas por `tenant_id` \+ RBAC.

---

## **8\. Parte D: Entregáveis (Sprint 3 – Elo)**

**Equipe SaaS**

* Painel de Configuração (scripts, objetivo, tom de voz, canais) com permissões RBAC.

* Fila/Funil (Novos, Em Qualificação, Qualificados, Desqualificados) \+ cartões de lead.

* Drawer com **Transcrição** e **Histórico** \+ Ações (assumir, enviar sugestão, desqualificar, handoff proposta, agendar reunião).

* **Prisma Essencial do Elo** (widgets \+ botão **\[✨ Analisar com IA\]** com log de consumo).

* Endpoints mínimos da A3 implementados.

**Equipe N8N/Supabase (Enzo)**

* Tabelas/Views: `elo_scripts`, `elo_sessions`, `elo_queue_v`, `prisma_elo_kpis_v`, `prisma_elo_funnel_v`.

* Workflows: `elo_start` (manual/auto), qualificação human-in-the-loop, handoff → Orçamentista/Guardião, atualização de status.

* **usage\_logs** com regra **100 \+ 1/100 tokens** por mensagem IA; `audit_logs` com quem aprovou envios.

* Webhooks documentados \+ função RPC de sincronismo Garimpeiro↔Elo.

