# Tarefas: Aplicativo de E-commerce — Radar E-commerce

| Campo | Valor |
|---|---|
| **Feature** | `001-app-ecommerce` |
| **Spec / Plan** | [`spec.md`](./spec.md) · [`plan.md`](./plan.md) |
| **Atualizado** | 2026-06 |

> Decomposição por épico, rastreada a tickets `ECA-###` e requisitos `FR-###`.
> Legenda de status: ✅ concluído · 🧪 em testes. `[P]` = paralelizável (sem dependência entre si).

---

## Fase 1 — Medicamentos controlados (Épico 1)

- **T001** ✅ `[P]` Liberar exibição de controlados em busca/departamentos/detalhe, sem rankeamento em vitrines — *ECA-96* — FR-001, FR-002, FR-003
- **T002** ✅ Restringir pedido a retirada na loja quando houver controlado na cesta; texto e tag de receita — *ECA-95* — FR-004, FR-005, FR-006
- **T003** ✅ `[P]` Suprimir toda comunicação promocional de controlados (card e detalhe) — *ECA-200* — FR-007, FR-008
- **T004** ✅ `[P]` Remover componente de entrega no endereço no detalhe de controlados — *ECA-238* — FR-009
- **T005** ✅ `[P]` Ocultar progress bar de frete quando houver controlado na cesta — *ECA-234* — FR-010

## Fase 2 — Onboarding e homepage (Épico 2)

- **T006** ✅ Tela de localização (GPS) + loading + tela de "lojas não encontradas" — *ECA-35* — FR-011, FR-013, FR-023
- **T007** ✅ Tela de endereço digitado com autocomplete (até 5 sugestões) — *ECA-133* — FR-011, FR-012
- **T008** ✅ Implementar roteamento para a melhor loja (entrega ≤30km → vendas → ofertas) — *ECA-35, ECA-133* — FR-012
- **T009** ✅ Mapa com PIN + modal de confirmação/complemento (validações de campos) — *ECA-274* — FR-014, FR-015, FR-016
- **T010** ✅ Persistir endereço do onboarding (header, usuário logado, carrinho; substituição) — *ECA-307* — FR-017, FR-018
- **T011** ✅ `[P]` Botão "Alterar endereço" na tela de indisponibilidade — *ECA-519* — FR-019
- **T012** ✅ `[P]` Vitrines da homepage (ordenação, estoque, regra "Mais vendidos") — *ECA-58* — FR-020, FR-021
- **T013** ✅ `[P]` Relayout da homepage no design system — *ECA-34* — FR-022
- **T014** ✅ `[P]` Refatoração do card de produto (todas as telas; CTA "Ativar") — *ECA-59* — FR-022

## Fase 3 — Notificações (Épico 3)

- **T015** ✅ Push por transição de status com mensagens da matriz (uma por transição) — *ECA-194* — FR-024, FR-025

## Fase 4 — Endereços do consumidor (Épico 4)

- **T016** ✅ Tela de gestão (seções, tag "Selecionado", estados vazio/único, legados) — *ECA-311* — FR-026, FR-027
- **T017** ✅ Adicionar endereço reutilizando o fluxo do onboarding (vira selecionado) — *ECA-373* — FR-028
- **T018** ✅ `[P]` Editar endereço (tela preenchida; reflete na gestão) — *ECA-370* — FR-029
- **T019** ✅ `[P]` Excluir endereço (bloqueio quando só há um) — *ECA-414* — FR-030

## Fase 5 — Checkout (Épico 5)

### 5a. Carrinho
- **T020** ✅ Lista de produtos, preço unitário, subtotal, parcelamento condicional, limite de estoque, navegação — *ECA-300* — FR-031..FR-035
- **T021** ✅ Limitação de oferta: card dividido + banner dinâmico + regra de redução — *ECA-515* — FR-036, FR-037
- **T022** ✅ `[P]` Carrinho vazio (mensagem + "Explorar produtos") — *ECA-301* — FR-038
- **T023** ✅ `[P]` Tag "Exige receita" no card do carrinho — *ECA-520* — FR-039
- **T024** ✅ `[P]` Tag de oferta restrita a ofertas do Portal — *ECA-517* — FR-040
- **T025** ✅ `[P]` Tag de frete grátis (contabilização, e-mails internos, controlados) — *ECA-566* — FR-041
- **T026** ✅ `[P]` Layout do card na cesta (design system) — *ECA-516* — FR-031
- **T027** ✅ `[P]` Tag "Preço normal" apenas no limite de oferta — *ECA-595* — FR-042

### 5b. Entrega e retirada
- **T028** ✅ Tela de entrega/retirada (endereço, retirada grátis, mensagens de horário, controlados) — *ECA-309* — FR-043..FR-049
- **T029** ✅ Troca de farmácia (seções, raio 100km, ordenação, modal de alterações, até 15) — *ECA-579* — FR-050, FR-051

### 5c. Pagamento
- **T030** ✅ Pagamento na entrega/retirada (modalidade dinâmica, ordem, dinheiro/troco, resumo) — *ECA-567* — FR-052..FR-056
- **T031** ✅ Pagamento no app (PIX e crédito; estados de cartão salvo/ausente) — *ECA-568* — FR-053, FR-057
- **T032** ✅ Adicionar cartão (cartão ilustrativo, BIN, máscaras, validações, tokenização, erros) — *ECA-569* — FR-058, FR-059
- **T033** ✅ Gerenciar cartões (excluir do app/checkout/cofre; mensagens; retorno) — *ECA-577* — FR-060
- **T034** ✅ Fluxo PIX (timer 2h persistente, copiar código, QR, estados de status) — *ECA-578* — FR-061, FR-062
- **T035** 🧪 CVV ao finalizar no app (modal, validação, Amex 4 dígitos, payload) — *ECA-665* — FR-063

---

## Baseline (fora do escopo de tarefas dos épicos)

Os requisitos **FR-064 a FR-116** descrevem comportamento **pré-existente em produção** (acesso/login, ofertas, gateway/antifraude, histórico de pedidos e complementos de homepage, troca de loja, PIX e tokenização). Não correspondem a tickets ECA destes épicos; estão na `spec.md` para completar a documentação do produto. Caso sejam revisados no futuro, gerar tarefas específicas aqui.

## Pendências

- **T035 (ECA-665)** em testes — concluir validação e marcar FR-063 como verificado na `spec.md`.
- **Migração de gateway** (Cielo → Braspag) — confirmar lojas migradas (FR-109..112).
