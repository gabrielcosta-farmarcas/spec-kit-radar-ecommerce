# GPEEDS-XX · [BE] Finalização da transação PBM — Confirma e Anula

**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 3 | **Prioridade:** Highest  
**Referência técnica:** `docs/pbm_interplayers/9 - Ficha Técnica Finalização da Transação (Confirma e Anula).pdf`  
**Depende de:** `pbm-autenticacao-token.md`, `pbm-efetivacao-transacao.md`

---

**O que?**  
Implementar os serviços de finalização da transação PBM: **Confirma** (`POST /api/transaction/confirma`) — enviado após confirmação do pedido no OMS — e **Anula** (`POST /api/transaction/anula`) — enviado em caso de cancelamento ou desistência de compra.

**Por que?**  
Toda transação efetivada fica com status PENDENTE no HUB. O **Confirma** é obrigatório para que o Lojista tenha direito ao ressarcimento da indústria. O **Anula** é obrigatório em cancelamentos: sem ele, o histórico de compras do consumidor acumula quantidade adquirida, podendo reduzir ou eliminar descontos PBM em compras futuras por excesso de limite do programa. Transações pendentes sem finalização são anuladas automaticamente pelo HUB após o período do autorizador, mas essa anulação automática não protege o histórico do consumidor de forma confiável nem garante o ressarcimento.

**Como?**
- **Confirma:** após confirmação do pedido no OMS, chamar `POST /api/transaction/confirma` com `transactionCode` (NSU Central), `consumer[].holderId` (CPF) e `product[].EAN` dos itens da transação; informar `operationId` (número do pedido) e `taxCouponType` (mandatório)
- **Anula:** após cancelamento do pedido, chamar `POST /api/transaction/anula` com `transactionCode` e `holderId`; enviar apenas os EANs dos produtos que faziam parte da transação efetivada
- Implementar fila de retry com backoff exponencial para garantir que Confirma/Anula seja enviado mesmo em falhas temporárias da API da Interplayers
- Registrar em log estruturado o resultado de cada chamada (sucesso, erro, `returnCode`) para auditoria e suporte
- Se não houver `transactionCode` armazenado (Efetivação falhou), não enviar Confirma/Anula — não há transação pendente no HUB

**Subtarefas sugeridas:**
- `[BE]` Implementar integração com `/api/transaction/confirma` com retry, log estruturado e validação de NSU
- `[BE]` Implementar integração com `/api/transaction/anula` com retry, log estruturado e validação de NSU
- `[BE]` Criar listener/hook no OMS: disparar Confirma ao confirmar pedido e Anula ao cancelar
- `QA` Testar: Confirma após pedido concluído (gera ressarcimento); Anula após cancelamento; retry em falha temporária; ausência de NSU (Efetivação falhou); timeout do autorizador (transação auto-anulada pelo HUB)
