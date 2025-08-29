#### **Briefing de Desenvolvimento: Agente "Órby Garimpeiro" v1.0** **Referência: Dossiê Mestre OrbythAI v4.1 (fonte única).**

* **Para:** Equipe N8N/Supabase (Enzo) **e** Equipe SaaS  
* **De:** Suzy (Estrategista)  
* **Data:** 28 de agosto de 2025

**1\. Visão Geral e Objetivos** O `Órby Garimpeiro` é o motor de prospecção ativa do nosso MVP. Sua missão é encontrar novas obras de construção civil, enriquecer os dados e entregá-los ao cliente de forma organizada, com opções de automação para a abordagem inicial.

**2\. Arquitetura da Funcionalidade (Como as Peças se Conectam)** O sistema funcionará da seguinte forma: `[Interface do Cliente no SaaS]` ↔ `[API do SaaS]` ↔ `[Banco de Dados Supabase]` ↔ `[Workflow N8N]`

**`2.1`** Segurança e Multi-tenancy: Todas as tabelas operam com tenant\_id sob RLS (Row Level Security) e RBAC (Admin/Gerente/Operador). O agente segue o Contrato Master de Dados & API v4.1 (schemas, endpoints, webhooks e políticas).

**3\. Especificações de Desenvolvimento por Componente**

* **3.1. Componente: Painel de Configuração do Agente**  
  * **Descrição Funcional:** O cliente precisa de uma interface para criar, salvar e agendar suas buscas.  
  * **Tarefas (Equipe SaaS):**  
    * Construir a interface do frontend para o cliente preencher os campos: `Nome do Filtro`, `Cidade Alvo`, `Tipo de Obra`, `Período do Edital`, `Área Mínima`, `Agendamento` e a seleção do `Modo de Abordagem` (Manual/Automático).  
  * **Tarefas (Equipe N8N/Supabase):**  
    * Criar a tabela `flow_garimpeiro_filtros` no Supabase para armazenar essas configurações, vinculadas ao `tenant_id`.

* **3.2. Componente: Motor de Execução do Agente**  
  * **Descrição Funcional:** O N8N precisa executar a lógica de busca e análise conforme as configurações do cliente.  
  * **Tarefas (Equipe SaaS):**  
    * Construir o endpoint de API (`POST /api/garimpeiro/executar`) que o frontend chamará para iniciar uma busca manual.  
  * **Tarefas (Equipe N8N/Supabase):**  
    * Desenvolver o workflow completo do N8N que lê os filtros do Supabase, executa o scraping, a extração com IA, o enriquecimento, o filtro, registra o consumo de "orbkens" e salva os resultados no Supabase, respeitando o `Modo de Abordagem`.

* **3.3. Componente: Painel de Resultados (Prisma Essencial)**  
  **Descrição Funcional: O cliente visualiza as oportunidades e toma ações. O Prisma Essencial já é conversacional (limitado) no Sprint 2 por meio do botão \[✨ Analisar com IA\], que responde apenas no contexto dos dados do Garimpeiro. Cada consulta registra consumo de orbkens e é auditada.**

  * **Tarefas (Equipe SaaS):**  
    * Construir a tabela de resultados no frontend com as colunas: `Nome da Oportunidade`, `Endereço`, `Contato`, `Status`.  
    * Implementar os botões de ação do usuário (`Iniciar Abordagem com Órby Elo`, `Descartar Oportunidade`).  
  * **Tarefas (Equipe N8N/Supabase):**  
    * Criar a tabela `flow_oportunidades_garimpeiro` no Supabase para armazenar os resultados.  
    * Criar a view ou função no Supabase que a API do SaaS usará para buscar os resultados de forma otimizada.  
* **3.4. Componente: Handoff para o Próximo Agente**  
  * **Descrição Funcional:** Quando o cliente ou o sistema (no modo automático) decidir abordar uma oportunidade, o `Órby Elo` deve ser acionado.  
  * **Tarefas (Equipe SaaS):**  
    * Implementar a lógica do botão `[Iniciar Abordagem com Órby Elo]` para chamar o endpoint de API correspondente.  
  * **Tarefas (Equipe N8N/Supabase):**  
    * No workflow do `Garimpeiro`, criar o passo final que aciona o webhook do workflow do `Órby Elo`.  
      No workflow do `Órby Elo`, criar o webhook que receberá os dados do `Garimpeiro`.

      **Sincronismo de Status:** Ao qualificar/desqualificar no Elo, atualizar o status correspondente em `flow_oportunidades_garimpeiro` e/ou `leads/opportunities` (Supabase), preservando a origem do lead.  
      

### **Parte A: Especificações para a Equipe de SaaS (A Interface do Cliente)**

#### **A1. Componente: Painel de Configuração do Agente**

* **Descrição Funcional:** O cliente precisa de uma interface para criar, salvar e agendar suas buscas.  
* **Requisitos da Interface:**  
  * **Filtros Salvos:** Permitir que o cliente crie e salve múltiplos "Filtros de Busca".  
  * **Campos do Filtro:** `Nome do Filtro`, `Cidade Alvo`, `Tipo de Obra` (Privada/Pública), `Período do Edital`, `Área Mínima`, `Agendamento` (Diário/Semanal/Manual).  
  * **Modo de Abordagem:** Seleção entre "Manual (Revisão Humana)" ou "Automático (Iniciar contato imediatamente)".

#### **A2. Componente: Painel de Resultados (Prisma Essencial)**

* **Descrição Funcional:** O cliente precisa de uma interface para visualizar as oportunidades encontradas e tomar ações.  
* **Requisitos da Interface:**  
  * **Tabela de Resultados:** Exibir os dados com as colunas: `Nome da Oportunidade`, `Endereço`, `Contato`, `Status`.  
  * **Ações do Usuário:** Implementar botões `[Executar Busca Manual]`, `[Iniciar Abordagem com Órby Elo]` e `[Descartar Oportunidade]`, além de checkboxes para seleção.

---

### **Parte B: Especificações para a Equipe N8N/Supabase (O Motor de Automação)**

#### **B1. Fontes de Dados e Ferramentas**

* **Fontes Primárias:** Diário Oficial de Cuiabá, Portal da Prefeitura, CREA-MT, CAU-MT, CNO.  
* **APIs de Enriquecimento:** ReceitaWS, cnpj.io, e outras que se mostrem necessárias.  
* **Ferramentas de Captura:** Playwright (para scraping complexo), requisições HTTP (para APIs e arquivos).  
* **Inteligência:** Modelos GPT-4o ou superiores para extração de dados de texto.

#### **B2. Lógica Detalhada do Workflow (N8N)**

O workflow seguirá os seguintes passos, acionado por agendamento ou por um comando manual do SaaS (via webhook):

1. **Gatilho e Leitura de Parâmetros:** O workflow inicia e busca no Supabase os parâmetros do "Filtro Salvo" que o cliente configurou.  
2. **Captura de Publicações:** Realiza o scraping ou a consulta às fontes de dados para obter as publicações recentes.  
3. **Extração com IA:** Usa um nó da OpenAI com um prompt otimizado para extrair os dados-chave (proprietário, endereço, responsável técnico, alvará, m²) e retornar em JSON.  
4. **Enriquecimento de Dados:**  
   * **CNPJ/CPF:** Consulta as APIs de enriquecimento para obter dados cadastrais e de contato.  
   * **Responsável Técnico:** Realiza scraping no site do CREA/CAU para validar o registro e buscar contatos adicionais.  
5. **Classificação e Filtragem:** Aplica as regras de relevância definidas pelo cliente (área mínima, tipo de obra, etc.).  
6. **Registro de Consumo:** Registrar em `usage_logs` a regra provisória: **100 orbkens por consulta \+ 1 orbken a cada 100 tokens processados** (incluindo chamadas de IA e APIs pagas). Persistir `tenant_id`, `agente`, `origem`, `tokens_in/out`, `custo_estimado`, `corr_id`.  
7. **Armazenamento:** Salva os resultados enriquecidos na tabela `flow_oportunidades_garimpeiro` no Supabase, definindo o `status` conforme o "Modo de Abordagem" (Manual ou Automático).  
8. **Handoff:** Se o modo for automático, aciona o webhook do `Órby Elo`. Se for manual, envia a notificação para o cliente.

#### **B3. Estrutura de Dados (Supabase)**

* **`flow_garimpeiro_filtros`:** Tabela para armazenar as configurações do cliente.  
* `flow_oportunidades_garimpeiro`: Tabela para armazenar os resultados. **Campos mínimos:**  
   `id, created_at, tenant_id, filtro_id, origem (enum: garimpeiro|guardiao|astro|import), modo_abordagem (manual|automatico), nome_empresa, cnpj_cpf, endereco_obra, responsavel_tecnico, telefone_contato, email_contato, area_m2, status, dados_brutos (JSON)`.  
* 

### **Parte C – Onboarding Assistido (MVP Enterprise)**

* **Setup Wizard:** guiado pela equipe OrbythAI para criar o 1º filtro, calibrar parâmetros e habilitar modo Manual/Automático.

* **Painel de Configuração:** ajustes contínuos (filtros, políticas).

* **Ciclo de Feedback (👍/👎):** o cliente treina respostas (gravar em `interactions`/`messages` com `feedback`).

* **Playground:** ambiente seguro para validar comportamento do agente antes de colocar em produção.

---

### **Parte D: Entregáveis (Sprint 2 – Garimpeiro)**

**5\. Entregáveis (Sprint 2 – Garimpeiro)**  
 **Equipe SaaS**

* Painel de Configuração (CRUD de filtros) integrado ao Supabase (RLS).

* Endpoints: `POST /api/garimpeiro/executar`, `GET /api/garimpeiro/filtros`, `GET /api/garimpeiro/resultados`.

* Tabela de Resultados \+ ações: \[Executar Busca Manual\], \[Iniciar Elo\], \[Descartar\].

* **Prisma Essencial com botão \[✨ Analisar com IA\]** (conversacional limitado \+ log de consumo).

* **Onboarding assistido** (Setup Wizard \+ Feedback \+ Playground).

**Equipe N8N/Supabase (Enzo)**

* Workflow N8N completo (scraping → extração IA → enriquecimento → filtros → persistência).

* `flow_garimpeiro_filtros`, `flow_oportunidades_garimpeiro`, views/funções de leitura.

* **usage\_logs** com a regra: 100/consulta \+ 1/100 tokens (campos de auditoria).

* Webhook de handoff → Elo \+ **sincronismo de status** de volta ao Garimpeiro.

* Documentação dos webhooks e payloads conforme **Contrato Master v4.1**.

## **Anexo A — Especificações de Ações & Fluxos de Uso (Órby Garimpeiro)**

Este anexo complementa o Briefing Garimpeiro v1.0. Os **inputs** (campos do filtro) e **outputs** (colunas da tabela, incluindo **Data da Descoberta**) estão especificados no corpo do Briefing; abaixo focamos **apenas** nos **elementos interativos** e **fluxos de uso** por modo.

### **1\) Ações do Usuário (Elementos Interativos) — \[MVP\]**

* **\[Executar Busca Manual\]**: aciona a execução imediata de um **Filtro Salvo**.

* **Checkbox por oportunidade**: permite seleção múltipla.

* **\[Iniciar Abordagem com Órby Elo\]**: habilita quando há 1+ oportunidades com status **“Aguardando Revisão”** selecionadas.

* **\[Descartar Oportunidade\]**: remove da lista oportunidades sem potencial.

### **2\) Experiência no Modo Manual (Usuário como Aprovador) — \[MVP\]**

**Objetivo:** nenhuma abordagem ocorre sem aprovação humana.

**Fluxo:**

1. **Descoberta:** oportunidades aparecem na tabela com status **“Aguardando Revisão”**.

2. **Análise:** o cliente revisa e seleciona (checkbox) as oportunidades promissoras.

3. **Ação decisiva:**

   * **\[Iniciar Abordagem com Órby Elo\]** → muda o status para **“Em Abordagem”** e aciona o Elo.

   * **\[Descartar Oportunidade\]** → remove da lista as oportunidades rejeitadas.

4. **Sob demanda:** **\[Executar Busca Manual\]** para rodar um Filtro Salvo sem esperar pelo agendamento.

### **3\) Experiência no Modo Automático (Usuário como Supervisor) — \[MVP\]**

**Objetivo:** a IA inicia rapidamente a abordagem, mantendo total possibilidade de intervenção humana.

**Fluxo:**

1. **Ação imediata:** quando o Garimpeiro encontra oportunidade válida, aciona o Elo e exibe o status **“Em Abordagem Automática”**.

2. **Monitoramento:** o cliente supervisiona o trabalho em andamento; ao abrir o item, pode **ver a transcrição** da conversa conduzida pelo Elo.

3. **Intervenção (refinamento humano):**

   * **\[Pausar Abordagem\]**: congela a automação **para aquele lead** até retomada manual.

   * **\[Assumir Conversa Manualmente\]**: transfere a conversa ativa para a equipe humana (o Elo finaliza e sinaliza a troca).

   * **\[Descartar Oportunidade\]**: continua disponível se, após a abordagem inicial, o lead se mostrar sem potencial.

Observação: no **MVP (Fase 1\)**, **\[Pausar Abordagem\]** e **\[Assumir Conversa Manualmente\]** já estão disponíveis para garantir controle humano imediato sobre abordagens automáticas.

#### **Briefing de Desenvolvimento: Agente "Órby Garimpeiro" v1.0 \
Referência: Dossiê Mestre OrbythAI v4.1 (fonte única).**



* **Para:** Equipe N8N/Supabase (Enzo) **e** Equipe SaaS
* **De:** Suzy (Estrategista)
* **Data:** 28 de agosto de 2025

**1. Visão Geral e Objetivos** O `Órby Garimpeiro` é o motor de prospecção ativa do nosso MVP. Sua missão é encontrar novas obras de construção civil, enriquecer os dados e entregá-los ao cliente de forma organizada, com opções de automação para a abordagem inicial.

**2. Arquitetura da Funcionalidade (Como as Peças se Conectam)** O sistema funcionará da seguinte forma: `[Interface do Cliente no SaaS]` ↔ `[API do SaaS]` ↔ `[Banco de Dados Supabase]` ↔ <code>[Workflow N8N] \
 \
<strong>2.1</strong> </code>Segurança e Multi-tenancy: Todas as tabelas operam com tenant_id sob RLS (Row Level Security) e RBAC (Admin/Gerente/Operador). O agente segue o Contrato Master de Dados & API v4.1 (schemas, endpoints, webhooks e políticas).

**3. Especificações de Desenvolvimento por Componente**



* **3.1. Componente: Painel de Configuração do Agente**
    * **Descrição Funcional:** O cliente precisa de uma interface para criar, salvar e agendar suas buscas.
    * **Tarefas (Equipe SaaS):**
        * Construir a interface do frontend para o cliente preencher os campos: `Nome do Filtro`, `Cidade Alvo`, `Tipo de Obra`, `Período do Edital`, `Área Mínima`, `Agendamento` e a seleção do `Modo de Abordagem` (Manual/Automático).
    * **Tarefas (Equipe N8N/Supabase):**
        * Criar a tabela `flow_garimpeiro_filtros` no Supabase para armazenar essas configurações, vinculadas ao `tenant_id`. \

* **3.2. Componente: Motor de Execução do Agente**
    * **Descrição Funcional:** O N8N precisa executar a lógica de busca e análise conforme as configurações do cliente.
    * **Tarefas (Equipe SaaS):**
        * Construir o endpoint de API (`POST /api/garimpeiro/executar`) que o frontend chamará para iniciar uma busca manual.
    * **Tarefas (Equipe N8N/Supabase):**
        * Desenvolver o workflow completo do N8N que lê os filtros do Supabase, executa o scraping, a extração com IA, o enriquecimento, o filtro, registra o consumo de "orbkens" e salva os resultados no Supabase, respeitando o `Modo de Abordagem`. \

* **3.3. Componente: Painel de Resultados (Prisma Essencial) \
Descrição Funcional: O cliente visualiza as oportunidades e toma ações. O Prisma Essencial já é conversacional (limitado) no Sprint 2 por meio do botão [✨ Analisar com IA], que responde apenas no contexto dos dados do Garimpeiro. Cada consulta registra consumo de orbkens e é auditada. \
 \
**
    * **Tarefas (Equipe SaaS):**
        * Construir a tabela de resultados no frontend com as colunas: `Nome da Oportunidade`, `Endereço`, `Contato`, `Status`.
        * Implementar os botões de ação do usuário (`Iniciar Abordagem com Órby Elo`, `Descartar Oportunidade`).
    * **Tarefas (Equipe N8N/Supabase):**
        * Criar a tabela `flow_oportunidades_garimpeiro` no Supabase para armazenar os resultados.
        * Criar a view ou função no Supabase que a API do SaaS usará para buscar os resultados de forma otimizada.
* **3.4. Componente: Handoff para o Próximo Agente**
    * **Descrição Funcional:** Quando o cliente ou o sistema (no modo automático) decidir abordar uma oportunidade, o `Órby Elo` deve ser acionado.
    * **Tarefas (Equipe SaaS):**
        * Implementar a lógica do botão `[Iniciar Abordagem com Órby Elo]` para chamar o endpoint de API correspondente.
    * **Tarefas (Equipe N8N/Supabase):**
        * No workflow do `Garimpeiro`, criar o passo final que aciona o webhook do workflow do `Órby Elo`. \
No workflow do `Órby Elo`, criar o webhook que receberá os dados do `Garimpeiro`. \
 \
**Sincronismo de Status:** Ao qualificar/desqualificar no Elo, atualizar o status correspondente em `flow_oportunidades_garimpeiro` e/ou `leads/opportunities` (Supabase), preservando a origem do lead.


### **Parte A: Especificações para a Equipe de SaaS (A Interface do Cliente)**


#### **A1. Componente: Painel de Configuração do Agente**



* **Descrição Funcional:** O cliente precisa de uma interface para criar, salvar e agendar suas buscas.
* **Requisitos da Interface:**
    * **Filtros Salvos:** Permitir que o cliente crie e salve múltiplos "Filtros de Busca".
    * **Campos do Filtro:** `Nome do Filtro`, `Cidade Alvo`, `Tipo de Obra` (Privada/Pública), `Período do Edital`, `Área Mínima`, `Agendamento` (Diário/Semanal/Manual).
    * **Modo de Abordagem:** Seleção entre "Manual (Revisão Humana)" ou "Automático (Iniciar contato imediatamente)".


#### **A2. Componente: Painel de Resultados (Prisma Essencial)**



* **Descrição Funcional:** O cliente precisa de uma interface para visualizar as oportunidades encontradas e tomar ações.
* **Requisitos da Interface:**
    * **Tabela de Resultados:** Exibir os dados com as colunas: `Nome da Oportunidade`, `Endereço`, `Contato`, `Status`.
    * **Ações do Usuário:** Implementar botões `[Executar Busca Manual]`, `[Iniciar Abordagem com Órby Elo]` e `[Descartar Oportunidade]`, além de checkboxes para seleção. \



---


### **Parte B: Especificações para a Equipe N8N/Supabase (O Motor de Automação)**


#### **B1. Fontes de Dados e Ferramentas**



* **Fontes Primárias:** Diário Oficial de Cuiabá, Portal da Prefeitura, CREA-MT, CAU-MT, CNO.
* **APIs de Enriquecimento:** ReceitaWS, cnpj.io, e outras que se mostrem necessárias.
* **Ferramentas de Captura:** Playwright (para scraping complexo), requisições HTTP (para APIs e arquivos).
* **Inteligência:** Modelos GPT-4o ou superiores para extração de dados de texto.


#### **B2. Lógica Detalhada do Workflow (N8N)**

O workflow seguirá os seguintes passos, acionado por agendamento ou por um comando manual do SaaS (via webhook):



1. **Gatilho e Leitura de Parâmetros:** O workflow inicia e busca no Supabase os parâmetros do "Filtro Salvo" que o cliente configurou.
2. **Captura de Publicações:** Realiza o scraping ou a consulta às fontes de dados para obter as publicações recentes.
3. **Extração com IA:** Usa um nó da OpenAI com um prompt otimizado para extrair os dados-chave (proprietário, endereço, responsável técnico, alvará, m²) e retornar em JSON.
4. **Enriquecimento de Dados:**
    * **CNPJ/CPF:** Consulta as APIs de enriquecimento para obter dados cadastrais e de contato.
    * **Responsável Técnico:** Realiza scraping no site do CREA/CAU para validar o registro e buscar contatos adicionais.
5. **Classificação e Filtragem:** Aplica as regras de relevância definidas pelo cliente (área mínima, tipo de obra, etc.).
6. **Registro de Consumo:** Registrar em `usage_logs` a regra provisória: **100 orbkens por consulta + 1 orbken a cada 100 tokens processados** (incluindo chamadas de IA e APIs pagas). Persistir `tenant_id`, `agente`, `origem`, `tokens_in/out`, `custo_estimado`, `corr_id`.
7. **Armazenamento:** Salva os resultados enriquecidos na tabela `flow_oportunidades_garimpeiro` no Supabase, definindo o `status` conforme o "Modo de Abordagem" (Manual ou Automático).
8. **Handoff:** Se o modo for automático, aciona o webhook do `Órby Elo`. Se for manual, envia a notificação para o cliente.


#### **B3. Estrutura de Dados (Supabase)**



* **<code>flow_garimpeiro_filtros</code>:** Tabela para armazenar as configurações do cliente.
* `flow_oportunidades_garimpeiro`: Tabela para armazenar os resultados. **Campos mínimos: \
** `id, created_at, tenant_id, filtro_id, origem (enum: garimpeiro|guardiao|astro|import), modo_abordagem (manual|automatico), nome_empresa, cnpj_cpf, endereco_obra, responsavel_tecnico, telefone_contato, email_contato, area_m2, status, dados_brutos (JSON)`.
* 


### **Parte C – Onboarding Assistido (MVP Enterprise)**



* **Setup Wizard:** guiado pela equipe OrbythAI para criar o 1º filtro, calibrar parâmetros e habilitar modo Manual/Automático. \

* **Painel de Configuração:** ajustes contínuos (filtros, políticas). \

* **Ciclo de Feedback (👍/👎):** o cliente treina respostas (gravar em `interactions`/`messages` com `feedback`). \

* **Playground:** ambiente seguro para validar comportamento do agente antes de colocar em produção. \



---


### **Parte D: Entregáveis (Sprint 2 – Garimpeiro)**

**5. Entregáveis (Sprint 2 – Garimpeiro) \
** **Equipe SaaS**



* Painel de Configuração (CRUD de filtros) integrado ao Supabase (RLS). \

* Endpoints: `POST /api/garimpeiro/executar`, `GET /api/garimpeiro/filtros`, `GET /api/garimpeiro/resultados`. \

* Tabela de Resultados + ações: [Executar Busca Manual], [Iniciar Elo], [Descartar]. \

* **Prisma Essencial com botão [✨ Analisar com IA]** (conversacional limitado + log de consumo). \

* **Onboarding assistido** (Setup Wizard + Feedback + Playground). \


**Equipe N8N/Supabase (Enzo)**



* Workflow N8N completo (scraping → extração IA → enriquecimento → filtros → persistência). \

* `flow_garimpeiro_filtros`, `flow_oportunidades_garimpeiro`, views/funções de leitura. \

* **usage_logs** com a regra: 100/consulta + 1/100 tokens (campos de auditoria). \

* Webhook de handoff → Elo + **sincronismo de status** de volta ao Garimpeiro. \

* Documentação dos webhooks e payloads conforme **Contrato Master v4.1**.


## **Anexo A — Especificações de Ações & Fluxos de Uso (Órby Garimpeiro)**


    Este anexo complementa o Briefing Garimpeiro v1.0. Os **inputs** (campos do filtro) e **outputs** (colunas da tabela, incluindo **Data da Descoberta**) estão especificados no corpo do Briefing; abaixo focamos **apenas** nos **elementos interativos** e **fluxos de uso** por modo.


### **1) Ações do Usuário (Elementos Interativos) — [MVP]**



* **[Executar Busca Manual]**: aciona a execução imediata de um **Filtro Salvo**. \

* **Checkbox por oportunidade**: permite seleção múltipla. \

* **[Iniciar Abordagem com Órby Elo]**: habilita quando há 1+ oportunidades com status **“Aguardando Revisão”** selecionadas. \

* **[Descartar Oportunidade]**: remove da lista oportunidades sem potencial. \



### **2) Experiência no Modo Manual (Usuário como Aprovador) — [MVP]**

**Objetivo:** nenhuma abordagem ocorre sem aprovação humana.

**Fluxo:**



1. **Descoberta:** oportunidades aparecem na tabela com status **“Aguardando Revisão”**. \

2. **Análise:** o cliente revisa e seleciona (checkbox) as oportunidades promissoras. \

3. **Ação decisiva: \
**
    * **[Iniciar Abordagem com Órby Elo]** → muda o status para **“Em Abordagem”** e aciona o Elo. \

    * **[Descartar Oportunidade]** → remove da lista as oportunidades rejeitadas. \

4. **Sob demanda:** **[Executar Busca Manual]** para rodar um Filtro Salvo sem esperar pelo agendamento. \



### **3) Experiência no Modo Automático (Usuário como Supervisor) — [MVP]**

**Objetivo:** a IA inicia rapidamente a abordagem, mantendo total possibilidade de intervenção humana.

**Fluxo:**



1. **Ação imediata:** quando o Garimpeiro encontra oportunidade válida, aciona o Elo e exibe o status **“Em Abordagem Automática”**. \

2. **Monitoramento:** o cliente supervisiona o trabalho em andamento; ao abrir o item, pode **ver a transcrição** da conversa conduzida pelo Elo. \

3. **Intervenção (refinamento humano): \
**
    * **[Pausar Abordagem]**: congela a automação **para aquele lead** até retomada manual. \

    * **[Assumir Conversa Manualmente]**: transfere a conversa ativa para a equipe humana (o Elo finaliza e sinaliza a troca). \

    * **[Descartar Oportunidade]**: continua disponível se, após a abordagem inicial, o lead se mostrar sem potencial. \


    Observação: no **MVP (Fase 1)**, **[Pausar Abordagem]** e **[Assumir Conversa Manualmente]** já estão disponíveis para garantir controle humano imediato sobre abordagens automáticas.



# **Prompt Sprint 2: Para a Equipe de Desenvolvimento do Agente Garimpeiro**

**TÍTULO:** Prompt de Execução – SPRINT 2 – Órby Garimpeiro (Primeiro Agente)

**PERSONA:** Você é um Engenheiro de Software especialista em automação de workflows com IA, integração de APIs e construção de painéis interativos em SaaS B2B.

**CONTEXTO:** Você tem acesso ao **Dossiê Mestre OrbythAI v4.1**, ao **Roadmap v4.1 (Sprint 2)** e ao **Briefing de Desenvolvimento: Órby Garimpeiro v1.0**. O objetivo é entregar o primeiro agente completo: o Órby Garimpeiro, responsável por encontrar novas obras em fontes públicas, enriquecer dados e entregá-los ao cliente via interface.

**OBJETIVO PRINCIPAL:** Entregar o Garimpeiro funcionando de ponta a ponta: configuração, execução, resultados e integração inicial com o Elo.


---


## **TAREFAS DETALHADAS**

**Equipe SaaS (Byth One + IA SaaS):**



* Construir **Painel de Configuração** com campos: \

    * Nome do Filtro \

    * Cidade \

    * Tipo de Obra (Privada/Pública) \

    * Período do Edital \

    * Área mínima \

    * Agendamento (manual/automático) \

    * Modo de Abordagem (manual/automático) \

* Criar endpoints: \

    * `POST /api/garimpeiro/executar \
`
    * `GET /api/garimpeiro/filtros \
`
    * `GET /api/garimpeiro/resultados \
`
* Construir **Painel de Resultados** com colunas: \

    * Nome da Oportunidade \

    * Endereço da Obra \

    * Contato Principal \

    * Data da Descoberta \

    * Status \

* Implementar botões: \

    * [Executar Busca Manual] \

    * [Iniciar Abordagem com Elo] \

    * [Descartar Oportunidade] \

* Implementar **Prisma Essencial** (tabela + botão [✨ Analisar com IA], limitado ao contexto do Garimpeiro, consumindo orbkens). \


**Equipe N8N (Enzo + IA N8N):**



* Criar tabelas `flow_garimpeiro_filtros` e `flow_oportunidades_garimpeiro` no Supabase com RLS por tenant. \

* Construir workflow completo: \

    * Captura de dados (scraping/HTTP). \

    * Extração com IA (OpenAI). \

    * Enriquecimento (ReceitaWS, CREA/CAU, etc.). \

    * Aplicação de filtros configurados pelo cliente. \

    * Registro de consumo de orbkens em `usage_logs` (regra: 100 por consulta + 1 por 100 tokens). \

    * Persistência de resultados em `flow_oportunidades_garimpeiro`. \

    * Handoff para Elo via webhook. \

* Garantir sincronismo de status entre Garimpeiro e Elo (ex.: qualificado, desqualificado). \



---


## **CRITÉRIOS DE ACEITAÇÃO**



* Painel de Configuração salva filtros corretamente no Supabase. \

* Workflow N8N executa busca, extrai, enriquece e salva no banco. \

* Painel de Resultados exibe oportunidades e permite ações do usuário. \

* Prisma Essencial responde consultas limitadas no contexto do Garimpeiro. \

* Consumo de orbkens registrado em `usage_logs`. \

* Handoff para Elo funcionando (modo automático ou manual). \



---


## **FORMATO DE SAÍDA**

Gere um relatório em formato **Markdown** contendo:



* Estrutura das tabelas criadas no Supabase. \

* Fluxo completo do workflow N8N exportado (.json). \

* Lista de filtros de teste e resultados obtidos. \

* Evidência de consumo de orbkens no `usage_logs`. \

* Prints ou links do Painel de Configuração e Painel de Resultados. \



---
