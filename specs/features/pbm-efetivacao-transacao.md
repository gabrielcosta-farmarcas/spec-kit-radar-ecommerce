# GPEEDS-XX · [BE] Efetivação da transação PBM no checkout — Efetiva

**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 5 | **Prioridade:** Highest  
**Referência técnica:** `docs/pbm_interplayers/8 - Ficha Técnica Efetivação.pdf`  
**Depende de:** `pbm-autenticacao-token.md`, `pbm-carrinho-hub.md`

---

**O que?**  
Implementar a integração com o serviço Efetiva (`POST /api/transaction/efetiva`) da Interplayers no momento do pagamento no checkout, autorizando a transação PBM para todos os produtos elegíveis em uma única chamada no modelo Varejo 4.0.

**Por que?**  
A Efetivação é o momento em que o HUB reserva o saldo do consumidor e recalcula os descontos considerando toda a lista de produtos (combos, kits, subsídios). A transação fica com status **PENDENTE** no HUB após a Efetivação — sem o Confirma posterior, o Lojista não recebe o ressarcimento da indústria. Sem a Efetivação, toda a integração PBM não gera valor financeiro real para a rede.

**Como?**
- Chamar `POST /api/transaction/efetiva` com o `transactionCode` obtido no Carrinho, CPF do consumidor e lista completa de produtos do pedido (todos os produtos devem ser enviados quando houver NSU)
- Usar `discountAdditionalMultiplied` retornado (já multiplicado pela quantidade autorizada) para calcular o preço final de cada produto
- Armazenar o `transactionCode` retornado — é o NSU Central necessário para o Confirma/Anula
- Verificar `authorizedQuantity` vs `requestedQuantity`: se a quantidade autorizada for inferior à solicitada, sinalizar internamente (não bloquear o checkout)
- Em caso de falha da API de Efetivação: registrar em log estruturado; prosseguir sem desconto PBM e marcar o pedido como sem NSU PBM (para não enviar Confirma/Anula sem NSU válido)
- Não expor mensagem de erro PBM ao consumidor; o checkout deve prosseguir normalmente
- Quando `discountAdditionalMultiplied = 0`, o preço líquido do Lojista é o melhor — a venda ocorre por dentro do programa para fins de ressarcimento da indústria

**Subtarefas sugeridas:**
- `[BE]` Implementar endpoint proxy para `/api/transaction/efetiva` com mapeamento completo de request/response
- `[BE]` Persistir o `transactionCode` (NSU Central) retornado associado ao pedido para uso no Confirma/Anula
- `[BE]` Implementar lógica de fallback: quando a Efetivação falhar, sinalizar que não há NSU PBM para não disparar Confirma/Anula indevido
- `QA` Testar: desconto indústria > desconto rede; desconto rede > desconto indústria; quantidade autorizada < solicitada; falha da API; pedido com múltiplos produtos PBM
