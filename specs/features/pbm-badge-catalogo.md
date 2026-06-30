# GPEEDS-XX · Badge PBM no catálogo de produtos

**Épico:** Integração PBM — Interplayers  
**Tipo:** História | **Story Points:** 5 | **Prioridade:** High  
**Referência técnica:** `docs/pbm_interplayers/`  
**Depende de:** `pbm-carga-diaria-loadtables.md`

---

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

**Impacto NSM:** Badge aumenta a taxa de identificação PBM no catálogo, habilitando o desconto personalizado e elevando o volume de transações por loja ativa.
