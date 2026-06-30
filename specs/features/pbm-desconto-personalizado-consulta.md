# GPEEDS-XX · Desconto PBM personalizado com identificação do Consumidor PBM

**Épico:** Integração PBM — Interplayers  
**Tipo:** História | **Story Points:** 8 | **Prioridade:** High  
**Referência técnica:** `docs/pbm_interplayers/`  
**Depende de:** `pbm-autenticacao-token.md`, `pbm-carga-diaria-loadtables.md`, `pbm-badge-catalogo.md`

---

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
- Se o Consumidor não estiver logado, não acionar `ConsultaDesconto`; exibir apenas o `discountMaxNewPatient` da tabela local (ver `pbm-badge-catalogo.md`)

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

**Impacto NSM:** Desconto personalizado aumenta a conversão do Consumidor PBM, segmento de alto valor, elevando diretamente o volume de transações por loja ativa.
