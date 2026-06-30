# GPEEDS-XX · [BE] Carga Diária de Tabelas PBM — LoadTables

**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 3 | **Prioridade:** Highest  
**Referência técnica:** `docs/pbm_interplayers/`

---

**O que?**  
Implementar job diário que executa a Carga de Tabelas (LoadTables) para cada Lojista habilitado com PBM, sincronizando localmente os produtos e programas PBM disponíveis.

**Por que?**  
A carga local evita consulta online desnecessária ao HUB para cada produto visualizado. Sem ela, não é possível saber quais EANs participam de programas PBM para exibir badge no catálogo, e as consultas de desconto não terão o `tableId` obrigatório.

**Como?**
- Agendar job para executar às **06:00h** (horário em que o HUB já completou atualização com todos os Autorizadores; nunca executar entre 08:00 e 00:00 exceto em emergência)
- Para cada Lojista PBM habilitado, chamar `POST /v1/integration/loadTablesMarketplace` com CNPJ e `ClientId`
- A carga é **sempre completa** — substituir totalmente a tabela anterior (não incremental)
- Manter backup da tabela do dia anterior para uso em caso de falha na execução do dia
- Armazenar localmente para cada EAN: `programName`, `discountMax`, `discountMaxNewPatient`, `qtyForDiscountMax`, `discountMin`, `allowsAdhesion`, `requestHolderId` (CPF obrigatório ou opcional), `requestCoupon`, `imageLink` (logo da indústria), `suggestedPrice`, `suggestedPriceValue`
- Respeitar segmentação geográfica: descontos podem variar por CNPJ, Cidade, UF ou nacional (`State: "ZZBR"`)
- Em caso de falha do job, usar backup do dia anterior e alertar via log/monitoramento

**Subtarefas sugeridas:**
- `[BE]` Implementar job agendado (cron) com lógica de substituição total e backup
- `[BE]` Modelar estrutura de armazenamento local com segmentação por CNPJ/Cidade/UF
- `QA` Testar execução bem-sucedida, falha com fallback para backup, e segmentação geográfica correta
