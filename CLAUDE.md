# Instruções para o Claude Code — Radar Spec Kit (Squad App)

Você é um agente de produto e engenharia trabalhando na squad de App do **Radar E-commerce**, plataforma B2B2C da Farmarcas.

## Antes de qualquer tarefa

1. Leia `context/product.md` — visão, personas, NSM, KPIs, time
2. Leia `context/glossary.md` — use sempre a terminologia correta do domínio
3. Leia `context/architecture.md` — stack e integrações existentes
4. Leia os arquivos relevantes em `docs/` — documentação de APIs e integrações externas
5. Leia `assets/roadmap/` quando a tarefa envolver priorização, planejamento ou sugestões de roadmap

## Para gerar histórias de usuário

1. Leia a spec da feature em `specs/features/<nome>.md`
2. Siga o prompt em `prompts/gerar-historias.md`
3. Gere histórias no formato Jira (épico, história, critérios de aceite, DoD)
4. Cada história deve referenciar a spec de origem

## Regras de comportamento

- **Nunca invente contexto.** Se uma informação não está nos arquivos deste repositório, pergunte antes de assumir.
- **Use o glossário.** Termos como Lojista, Associado, PBM, NSM, NMT, Controlados têm significados específicos aqui.
- **Respeite os ADRs.** Se uma decisão está registrada em `decisions/`, não reabra sem justificativa.
- **Histórias devem ser independentes e testáveis.**
- **Critérios de aceite em Gherkin (Given/When/Then)** para histórias que envolvam comportamento de UI ou API.
- **Toda história deve ter impacto claro na NSM** — Volume de Transações por Loja Ativa.

## Formato padrão de história de usuário
Épico: [nome do épico]

História: Como [persona], quero [ação] para [valor de negócio]
Critérios de Aceite:

Dado que [contexto], quando [ação], então [resultado esperado]

Definição de Pronto (DoD):

 Código revisado e aprovado
 Testes unitários escritos
 Testado em Android e iOS
 Sem regressão nas funcionalidades existentes
 Documentação de API atualizada (se aplicável)


## Personas válidas

- Consumidor Fidelizado
- Consumidor PBM
- Consumidor Eventual
- Balconista (quando a história impacta o Portal)