# Histórias de Usuário — Integração PBM (Interplayers)

**Épico:** Integração PBM — Interplayers  
**Referência técnica:** `docs/pbm_interplayers/`  
**Meta Q3 2026:** PBM integrado e dado disponível para precificação  
**Impacto NSM:** Habilitar transações PBM eleva o Volume de Transações por Loja Ativa para o segmento Consumidor PBM

---

## GPEEDS-XX · [BE] Autenticação com HUB Interplayers — Generate Token
**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 2 | **Prioridade:** Highest

**O que?**  
Implementar serviço de autenticação OAuth com o HUB Interplayers, obtendo e renovando automaticamente o Access Token necessário para todas as chamadas PBM.

**Por que?**  
O Manual de Projeto Interplayers afirma explicitamente: *"Para todas as funcionalidades deve ser enviado a Autenticação – GenerateToken."* Sem token válido, nenhuma operação PBM (LoadTables, Consulta Produto, Consulta Desconto, Carrinho, Efetivação) pode ser executada.

**Como?**
- Implementar cliente HTTP que realiza POST para o endpoint OAuth (`/B2C_1_Varejo/oauth2/v2.0/token`) com `Client_Id`, `Client_Secret`, `Grant_Type` e `Scope`
- Armazenar credenciais exclusivamente via variável de ambiente (nunca em código-fonte ou logs)
- Cachear o `Access_Token` retornado e monitorar o `Expires_In` (em segundos, ex: 1799s = ~30min)
- Implementar renovação automática antes da expiração, sem interromper requisições em andamento
- Em caso de erro (400, 401, 403), registrar em log estruturado sem expor credenciais; não exibir mensagem de erro PBM ao consumidor
- Cada canal (App) possui `Client_Id` e `Client_Secret` exclusivos — não reutilizar credenciais de outros canais

**Subtarefas sugeridas:**
- `[BE]` Implementar módulo de autenticação com cache e renovação automática
- `[BE]` Configurar variáveis de ambiente para credenciais PBM
- `QA` Testar renovação de token próxima da expiração; testar falha de autenticação (mock 401/403)

---

## GPEEDS-XX · [BE] Carga Diária de Tabelas PBM — LoadTables
**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 3 | **Prioridade:** Highest

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

---

## GPEEDS-XX · Badge PBM no catálogo de produtos
**Épico:** Integração PBM — Interplayers  
**Tipo:** História | **Story Points:** 5 | **Prioridade:** High

**User Story**  
Como usuário consumidor que tem acesso ao aplicativo, quero visualizar quando um produto possui benefício PBM disponível para que eu me sinta motivado a me identificar e aproveitar o desconto.

**Visão geral**  
Ao navegar pelo catálogo ou acessar a página de um produto, o app consulta a tabela PBM local (carregada pelo LoadTables) para verificar se o EAN possui programa ativo. Se sim, exibe o badge com logo da indústria e mensagem de desconto disponível — mesmo sem o Consumidor estar identificado, como fator de convencimento para login/adesão.

**O que deve ser feito**
- Badge PBM na listagem de produtos e na página de detalhes do produto
- Logo da indústria (`imageLink`) junto ao badge
- Texto de desconto máximo disponível para novo paciente (`discountMaxNewPatient`)
- Mensagem informativa da indústria (`informativeMessage`) quando presente
- Estado vazio tratado: produto sem PBM não exibe badge

**Regras de Negócio**
- A verificação de elegibilidade PBM deve ser feita **localmente** contra a tabela carregada pelo LoadTables — não acionar o HUB online neste momento
- Exibir badge apenas para produtos em que o EAN do Lojista (CNPJ) consta na tabela PBM do dia
- Se `allowsAdhesion = "N"`, o produto não aceita novas adesões; exibir somente o badge sem CTA de adesão
- Se `requestHolderId = "M"`, indicar que CPF é obrigatório para obter o desconto personalizado
- Se a tabela PBM não tiver sido carregada no dia (falha do job), não exibir badge para evitar dado desatualizado
- O desconto exibido (`discountMaxNewPatient`) é o desconto máximo para primeiro acesso, sem identificação — não é o desconto real do consumidor identificado
- Badge não deve aparecer em produtos controlados que não participam de PBM

**Critérios de Aceite**
- Consigo visualizar o badge PBM com o logo da indústria no card do produto na listagem
- Consigo visualizar o badge PBM com o logo da indústria na página de detalhes do produto
- Visualizo o percentual de desconto máximo disponível para novos pacientes junto ao badge
- Visualizo a mensagem informativa da indústria quando ela estiver disponível
- Não visualizo badge PBM em produtos que não participam de nenhum programa
- Não visualizo badge PBM quando o programa não permite novas adesões (`allowsAdhesion = N`) sem indicação de CTA

**Subtarefas sugeridas:**
- `[BE]` Expor endpoint que retorna, dado um EAN e CNPJ da loja, os dados PBM da tabela local
- `[APP]` Implementar componente de badge PBM (logo + desconto máximo) na listagem e no detalhe do produto
- `QA` Testar produto com PBM (com e sem allowsAdhesion), produto sem PBM, e cenário de tabela não carregada

---

## GPEEDS-XX · Desconto PBM personalizado com identificação do Consumidor PBM
**Épico:** Integração PBM — Interplayers  
**Tipo:** História | **Story Points:** 8 | **Prioridade:** High

**User Story**  
Como usuário consumidor que tem acesso ao aplicativo, quero ver o desconto PBM real aplicado ao meu CPF ao visualizar um produto elegível para que eu tenha certeza de qual é o melhor preço disponível para mim.

**Visão geral**  
Quando o Consumidor PBM está logado e acessa um produto elegível, o app aciona a API `ConsultaDesconto V3` da Interplayers com o CPF do consumidor. O HUB retorna o desconto personalizado, considerando programas em que o consumidor já tem adesão. O app deve sempre exibir o **melhor desconto** (entre o desconto da loja e o desconto PBM). O fluxo pode incluir desvios para aceite de termos LGPD ou adesão ao programa da indústria.

**O que deve ser feito**
- Chamada à API `POST /api/transaction/v3/consultadesconto` com CPF e EAN ao acessar produto elegível
- Exibição do preço com desconto PBM personalizado na página do produto
- Tratamento de desvio de fluxo (`flowDeviation = DESVIOURL`) com exibição de URL de adesão/LGPD
- Envio de dados do consumidor para pré-preenchimento do formulário de adesão (`consumer[]`)
- Cálculo e exibição do melhor desconto (PBM vs loja)

**Regras de Negócio**
- O app deve **sempre enviar preço bruto (`listPrice`) e preço líquido (`netPrice`) do Lojista** em todas as chamadas ao HUB para que ele calcule o desconto adicional corretamente
- O sistema deve sempre oferecer o **MELHOR DESCONTO**: se o desconto da loja for maior que o da indústria, exibir o desconto da loja (`discountValue = 0`); se o da indústria for maior, subtrair o `discountValue` do `netPrice`
- Se `flowDeviation = DESVIOURL`, exibir o link de `informativeLink` em WebView ou navegador externo para adesão ou aceite de termos LGPD; **reenviar a requisição ConsultaDesconto após o consumidor completar o desvio**
- Se houver múltiplos produtos no mesmo pedido e cada um precisar de adesão individual, tratar os desvios de fluxo **sequencialmente por produto**
- Se `discountValue = 0`, exibir o preço líquido do lojista normalmente — o produto ainda deve ser enviado ao serviço Carrinho posteriormente
- Se `coPaymentPercentual` tiver valor, subtrair esse percentual adicional do valor final (subsídio/co-pagamento por terceiros)
- Se `suggestedPrice = "S"`, o desconto é flutuante — calculado para atingir o `suggestedPriceValue` sugerido pela indústria
- Produto com `requestCoupon = "M"` exige número de cupom/cartão junto ao CPF; o app deve apresentar campo para inserção do cupom
- Se o Consumidor não estiver logado, não acionar `ConsultaDesconto`; exibir apenas o `discountMaxNewPatient` da tabela local (ver história de badge PBM)

**Critérios de Aceite**
- Consigo visualizar o preço com desconto PBM específico para o meu CPF na página do produto
- Visualizo sempre o melhor preço entre o desconto da loja e o desconto PBM
- Sou direcionado para um formulário de adesão ao programa da indústria quando ainda não tenho adesão ao produto
- Consigo retornar ao app após completar a adesão e visualizo o desconto PBM aplicado
- Consigo inserir meu número de cupom/cartão quando o programa exige (`requestCoupon = M`)
- Visualizo o preço final correto quando há co-pagamento por terceiros (`coPaymentPercentual`)
- Não sou redirecionado para adesão quando já possuo adesão ao programa para o produto consultado

**Subtarefas sugeridas:**
- `[BE]` Implementar endpoint proxy para `ConsultaDesconto V3` repassando CPF, EAN, preços bruto/líquido e dados do consumidor para pré-preenchimento
- `[BE]` Implementar lógica de "melhor desconto": comparar desconto da loja vs `discountValue` do HUB e retornar o maior
- `[APP]` Implementar exibição de preço PBM personalizado na página de produto com loading state
- `[APP]` Implementar tratamento de `flowDeviation = DESVIOURL`: abrir WebView/browser com `informativeLink` e reenviar requisição ao retornar
- `[APP]` Implementar campo de cupom/cartão para produtos com `requestCoupon = M`
- `QA` Testar desconto PBM > desconto loja; desconto loja > desconto PBM; desvio LGPD; desvio adesão; produto com cupom obrigatório; co-pagamento; consumidor sem adesão; consumidor com adesão
