# TODO — Spec Kit Radar E-commerce

## Em andamento

- [ ] Reorganizar estrutura do repositório por squads (app, portal, core)
  - [ ] Atualizar `CLAUDE.md` de cada squad com contexto específico
  - [ ] Mover histórias existentes para `squads/app/specs/features/`
  - [ ] Criar estrutura inicial para squads Portal e Core

## Próximos passos

- [ ] Conectar via MCP para escrever histórias diretamente no Jira
  - [ ] Configurar MCP do Jira no Claude Code
  - [ ] Configurar MCP do GitHub no Claude Code
  - [ ] Testar fluxo completo: spec → Claude Code → história criada no Jira (GPEEDS)

## Backlog

- [ ] Preencher `context/architecture.md` com stack real do time
- [ ] Criar `squads/portal/specs/` com histórias do Portal (Q3: editar pedido OMS, comissão de vendas, estoque e preço)
- [ ] Criar spec da feature PBM em `squads/app/specs/features/pbm.md`
- [ ] Adicionar glossário de termos técnicos da API Interplayers em `docs/`
- [ ] Configurar `squads/core/specs/` com contexto da Squad Core

## Concluído

- [x] Criar estrutura inicial do repositório
- [x] Preencher `context/product.md` com informações reais do Radar
- [x] Preencher `context/glossary.md` com termos do domínio
- [x] Adicionar histórias reais como exemplos em `specs/examples/historias-exemplo.md`
- [x] Configurar `prompts/gerar-historias.md` com padrão real do time
- [x] Adicionar documentação de API do PBM em `docs/`
- [x] Configurar `.claude/settings.json`
- [x] Renomear repositório para `spec-kit-radar-ecommerce`
- [x] Configurar SSH no GitHub
- [x] Adicionar roadmap Q3 2026 do App em `assets/roadmap/`