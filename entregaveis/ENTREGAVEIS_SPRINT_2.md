# Entregáveis Sprint 2 – Órby Garimpeiro

## Equipe SaaS
- [ ] Painel de Configuração com filtros salvo no Supabase.
- [ ] Endpoints criados:
  - POST /api/garimpeiro/executar
  - GET /api/garimpeiro/filtros
  - GET /api/garimpeiro/resultados
- [ ] Painel de Resultados (Nome, Endereço, Contato, Data, Status).
- [ ] Botões implementados: [Executar Busca Manual], [Iniciar Elo], [Descartar].
- [ ] Prisma Essencial com botão [✨ Analisar com IA].

## Equipe N8N
- [ ] Tabela `flow_garimpeiro_filtros` criada.
- [ ] Tabela `flow_oportunidades_garimpeiro` criada.
- [ ] Workflow completo implementado (scraping → IA → enriquecimento → filtros → persistência).
- [ ] Registro de consumo em `usage_logs` funcionando.
- [ ] Handoff para Elo via webhook configurado.

## Estratégia
- [ ] Testes ponta a ponta validados.
- [ ] Checklist de consumo de orbkens confirmado.
- [ ] Validação do sincronismo Garimpeiro ↔ Elo concluída.
