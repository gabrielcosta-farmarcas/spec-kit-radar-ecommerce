# Produto: Radar E-commerce — App (Consumidor Final)

## Visão

> "Oferecer uma jornada de compra tão simples quanto pedir uma pizza, altamente personalizada, com operação estável e capaz de impulsionar as vendas dos associados."

## Posicionamento

O Radar E-commerce é uma plataforma B2B2C da Farmarcas que conecta farmácias associadas a consumidores finais por meio de um portal web (Portal) e um aplicativo mobile (App). Não é um marketplace genérico — cada farmácia associada opera sua própria loja digital, mantendo o relacionamento direto com seu consumidor.

## Modelo de Negócio

**B2B2C:** Farmarcas (plataforma) → Lojistas/Associados (farmácias franqueadas) → Consumidores finais

- A Farmarcas fornece e evolui a plataforma tecnológica
- Os lojistas configuram estoques, preços e regras no Portal
- Os consumidores compram via App, sempre vinculados a uma farmácia específica

## Usuários do App

| Persona | Descrição | Principal motivação |
|---|---|---|
| **Consumidor Fidelizado** | Cliente habitual de uma farmácia associada | Conveniência e recorrência de compra |
| **Consumidor PBM** | Tem benefício de convênio farmacêutico | Usar o desconto do plano na farmácia de confiança |
| **Consumidor Eventual** | Descobriu a farmácia via indicação ou busca | Confiança na farmácia local |

## North Star Metric (NSM)

> **Volume de Transações por Loja Ativa**

Todas as decisões de produto devem ser avaliadas pelo impacto nesta métrica.

## KPIs 2026

| Indicador | Atual | Meta |
|---|---|---|
| Avaliação do app (Apple + Google) | [verificar] | Acima de 4 estrelas nos 12 apps |
| Taxa de conversão da busca | 5,8 | 15% (anual) / 8,7 (Q1) |
| Faturamento mensal | [verificar] | R$ 1M até dez/2026 |
| Taxa de fraude | [verificar] | ≤ 1% do valor transacionado |
| Conversão de vendas mensal | 22% | 30% (meta Q3) |
| Ticket médio | R$ 66 | [verificar] |
| Média de itens por carrinho | 3 | [verificar] |

## Indicadores Q3 2026 (Squad App)

| Indicador | Atual | Meta Q3 |
|---|---|---|
| Conversão de vendas (mensal) | 22% | 30% |
| PBM (Interplayers) | Não integrado | Integrado |
| NMT (lojas ativas) | 7 | 8 — Pronto para escala nacional |

## Entregas Q3 2026 (Squad App)

- **Novo checkout (Deeplink):** receber todos os dados necessários para formular preço
- **PBM:** integração com PBM concluída e dado disponível para precificação

**Riscos Q3:**
- Apoio ativo do fornecedor PBM para desenvolvimento da solução
- Desenvolvimento do parceiro para atender regra de negócio comercial

**Plano de ação:**
- Desenvolver em paralelo e garantir integração plug-and-play na entrega
- Habilitar a pré-venda nas lojas para trabalharem com PBM

## Entregas Q2 2026 (Squad App) — Histórico

**Principais entregas:**
- Novo onboarding
- Selecionar farmácia na retirada
- Fluxo de gestão de endereços
- Notificações PUSH
- Tratativa limitação de ofertas

**Impacto gerado:**
- 78% dos consumidores encontram uma loja com 2 cliques
- 13,2% dos consumidores não encontram atendimento na região
- App Maxi Popular atualizado

**Pendências identificadas:**
- Lojas sem conteúdo estão constando como onboarding bem-sucedido
- Alinhamento do time de produto com outras áreas da empresa

**Aprendizados e riscos:**
- Vamos considerar lojas sem conteúdo como atendimento indisponível na região
- Conflito de GE dentro do app vs melhor experiência do consumidor

## Funcionalidades Core do App

- Catálogo de produtos por farmácia
- Busca e filtros (categoria, princípio ativo, marca)
- Carrinho e checkout
- Compra de medicamentos controlados (retirada na loja)
- Integração PBM (desconto por convênio farmacêutico)
- Acompanhamento de pedido
- Histórico de compras e recompra
- Notificações push (promoções, status de pedido)
- Gestão de endereços

## Dores Conhecidas dos Consumidores

Identificadas via pesquisa de experiência:
- Dificuldade na busca de produtos (especialmente controlados)
- Fricção na seleção de loja no início da jornada
- Cadastro extenso
- Falta de cupons, descontos e personalização
- Alta predominância de pagamento offline
- Baixo engajamento em ofertas

## Time (Squad App)

| Papel | Nome |
|---|---|
| Gerente de Engenharia | Gabriel Costa |
| Coordenador | Thiago |
| Product Owner | Daniel |
| UX Sênior | Rodrigo Cordeiro Nery |
| Dev Back-end Pleno | Felipe Soares |
| Dev Mobile Sênior | Joceli Antônio Pacheco |
| Dev Mobile Pleno | Leon Vitor Mansoni Viana |
| Analista de QA Júnior | Diego |

## Fora do Escopo do App

- Gestão de estoque e preços → Portal
- Configuração de regras de desconto → Portal
- Relatórios e analytics → Portal
- Edição de pedidos OMS → Portal
- Comissão de vendas → Portal