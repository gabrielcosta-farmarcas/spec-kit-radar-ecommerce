# Plano de Implementação: Aplicativo de E-commerce — Radar E-commerce

| Campo | Valor |
|---|---|
| **Feature** | `001-app-ecommerce` |
| **Spec** | [`spec.md`](./spec.md) |
| **Status** | Entregue |
| **Atualizado** | 2026-06 |

> Descreve **como** os requisitos da spec foram realizados: contexto técnico, integrações, dependências de configuração e o sequenciamento por épico. Não substitui a documentação de arquitetura do time; serve de mapa entre requisitos e pontos de implementação.

---

## 1. Contexto técnico

- **Cliente:** aplicativo mobile (iOS e Android), com componentes do design system v1.
- **Backend/API:** serviços `api-edelivery` (merchandising, carrinho/checkout, pedidos).
- **Configuração de domínio:** **Portal Radar E-commerce** — fonte de verdade para catálogo, categorias, ofertas, fretes, horários (loja e delivery), tempos de retirada/entrega e formas de pagamento.
- **Integrações externas:** Google Maps (geocoding/autocomplete + geolocalização), Braspag/Cielo (cofre/tokenização de cartões, PIX e crédito), serviço de push notifications.
- **Conformidade:** RDC 144 (ANVISA) para medicamentos com retenção de receita.

## 2. Pontos de integração

| Capacidade | Endpoint / mecanismo | Observações |
|---|---|---|
| Flag de retenção (detalhe) | `GET /mobile/merchandising/v2/stock/products/{productId}/{skuId}` | `medicationLabel.requiresPrescriptionRetention` (boolean). Suporta FR-001..FR-010. |
| Flag de retenção (frete) | `GET /mobile/cart/shipping/strategies` | `shippingOptions[].requiresPrescriptionRetention`. Habilita regras de entrega/retirada. |
| Endereçamento | Google Maps Places/Geocoding | Autocomplete (até 5), PIN e geocodificação reversa. FR-011..FR-016. |
| Pagamento | Gateway Braspag/Cielo | Tokenização/cofre (FR-059/060), PIX 2h (FR-061), enriquecimento de CVV (FR-063). |
| Notificações | Serviço de push | Disparo por transição de status. FR-024/025. |

## 3. Dependências de configuração (Portal)

As regras abaixo são **dirigidas por dados** do Portal e devem refletir em carrinhos já montados (FR-035):

- Categoria "Medicamentos com Retenção de Receita" → controla FR-001..FR-010, FR-039, FR-044.
- Limitador de oferta → FR-036/037/042.
- Valor de frete grátis → FR-041.
- Horário da loja, horário do delivery, tempo de retirada e de entrega, raio → FR-046..FR-048.
- Parcelamento e formas de pagamento habilitadas → FR-033, FR-052..FR-057.
- Módulos ativos (vendas/ofertas/entrega) e grupo econômico → FR-012/013, FR-050.

## 4. Restrições e decisões

- **Fuso horário:** todos os cálculos de disponibilidade usam o horário local do consumidor, **ignorando o UTC+3 do servidor** (FR-049).
- **Roteamento de loja:** prioridade fixa entrega(≤30km) → vendas → ofertas (FR-012); fallback para tela de indisponibilidade (FR-013).
- **Segurança de pagamento:** dados de cartão não trafegam/armazenam em claro; apenas token + bandeira + 4 dígitos (FR-057/059); CVV usado apenas para enriquecer o payload da venda (FR-063).
- **Conformidade ANVISA:** ausência total de comunicação promocional em controlados é requisito legal, não estético (FR-007/008).

## 5. Fases de implementação (mapeadas aos épicos)

1. **Fase 1 — Catálogo e controlados (Épico 1):** exibição, supressão de promoção, restrição a retirada, flags de retenção. → FR-001..FR-010.
2. **Fase 2 — Onboarding e homepage (Épico 2):** entrada por GPS/endereço, roteamento, persistência do endereço, vitrines e cards. → FR-011..FR-023.
3. **Fase 3 — Notificações (Épico 3):** push por transição de status. → FR-024/025.
4. **Fase 4 — Endereços (Épico 4):** gestão, adicionar/editar/excluir. → FR-026..FR-030.
5. **Fase 5 — Checkout (Épico 5):** carrinho, entrega/retirada, troca de loja, pagamento, PIX, cartões, CVV. → FR-031..FR-063.

## 6. Estratégia de verificação

- Critérios de aceite por FR validados via casos de teste (QA) — ver `tasks.md`.
- Cenários sensíveis a horário (FR-047/048) exigem testes com relógio controlado e lojas 24h.
- Cenários de controlado (FR-004/044) validados com produto reclassificado no Portal.
- Pagamento validado em ambiente de staging do gateway antes de produção.

## 7. Riscos

| Risco | Mitigação |
|---|---|
| Divergência de horário (UTC) em mensagens de disponibilidade | Testes com fuso local e lojas 24h (FR-049). |
| Estoque/configuração defasados em carrinhos montados | Releitura de configuração do Portal no carrinho (FR-035). |
| Falha de tokenização/recusa do gateway | Mensagens de erro específicas e CVV para elevar aprovação (FR-059/063). |
| Não-conformidade ANVISA em controlados | Supressão total de comunicação promocional + revisão (FR-007/008). |
