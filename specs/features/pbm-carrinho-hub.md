# GPEEDS-XX · [BE] Carrinho PBM — Envio de produtos ao HUB Interplayers

**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 3 | **Prioridade:** Highest  
**Referência técnica:** `docs/pbm_interplayers/7 - Ficha Técnica Carrinho.pdf`  
**Depende de:** `pbm-autenticacao-token.md`, `pbm-desconto-personalizado-consulta.md`

---

**O que?**  
Implementar integração com o serviço Carrinho (`POST /api/transaction/carrinho`) da Interplayers, enviando os produtos do pedido com preços brutos e líquidos do Lojista e recebendo os melhores descontos calculados pelo HUB para cada item.

**Por que?**  
O serviço Carrinho consolida em uma única chamada o desconto final de todos os produtos elegíveis a PBM, considerando combos, kits e regras de programa. É a etapa que precede a Efetivação e que fornece o `transactionCode` necessário para continuidade do fluxo. Sem ela, não é possível apresentar o resumo de pedido com desconto PBM correto nem prosseguir para a autorização.

**Como?**
- Chamar `POST /api/transaction/carrinho` com: `transactionCode` (zeros no primeiro envio; reutilizar o valor retornado pelo HUB em chamadas subsequentes para o mesmo atendimento), `consumer[].holderId` (CPF), e para cada produto: `EAN`, `requestedQuantity`, `listPrice` (preço bruto do Lojista), `netPrice` (preço líquido do Lojista), `discountType` (`B` — padrão bruto)
- Armazenar o `transactionCode` retornado pelo HUB para uso obrigatório na Efetivação
- Calcular preço final por produto: subtrair `discountValue` do `netPrice` quando `discountValue > 0`; quando `discountValue = 0`, manter o preço líquido do Lojista (desconto da rede é superior — a venda segue por dentro do programa para ressarcimento)
- Sempre enviar `listPrice` e `netPrice` do Lojista para que o HUB calcule corretamente o desconto adicional
- Em caso de erro da API (4xx/5xx), registrar em log estruturado sem expor detalhes ao consumidor; não bloquear o checkout — prosseguir sem desconto PBM

**Subtarefas sugeridas:**
- `[BE]` Implementar endpoint proxy para `/api/transaction/carrinho` com mapeamento de request/response
- `[BE]` Persistir `transactionCode` retornado associado ao pedido em andamento
- `QA` Testar: desconto indústria > desconto rede; desconto rede > desconto indústria (`discountValue = 0`); múltiplos produtos; erro da API; primeiro envio (transactionCode zero) vs continuidade do atendimento
