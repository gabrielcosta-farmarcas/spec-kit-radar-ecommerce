# Especificação: Aplicativo de E-commerce — Radar E-commerce

| Campo | Valor |
|---|---|
| **Feature** | `001-app-ecommerce` |
| **Projeto (Jira)** | SQ — ECOMMERCE APP |
| **Status** | Em produção (baseline) + épicos 1–5 (1 item em testes) |
| **Tipo** | Especificação consolidada do produto (baseline + épicos recentes) |
| **Versão** | 2.0 |
| **Atualizado** | 2026-06 |

> Esta spec consolida o comportamento do produto: o **baseline** (regras pré-existentes, documentadas nos handoffs/PDFs) e os **épicos recentes** (jan/2026 →, registrados nos tickets `ECA-###`). Descreve **o quê** e **o porquê** (o **como** vive em `plan.md`).
>
> **Legenda de origem dos requisitos:** *(ECA-###)* = entregue nos épicos recentes · *(baseline: Documento)* = comportamento pré-existente. Onde baseline e épicos conflitam, **prevalece a regra atual (ECA)**, e a anterior é sinalizada como substituída em notas de conflito.

---

## 1. Resumo

O aplicativo permite que o consumidor acesse (login/cadastro), encontre uma farmácia próxima, navegue pelo catálogo e pelas ofertas, monte um carrinho e finalize a compra com retirada na loja ou entrega em casa, pagando na entrega/retirada ou no app (PIX e cartão), acompanhando o histórico e o status dos pedidos. Esta spec cobre o produto completo: o acesso, o onboarding e a homepage, as ofertas, o catálogo (incluindo medicamentos controlados), os endereços, o checkout (carrinho, entrega/retirada, troca de loja e pagamento), o gateway de pagamento e antifraude, as notificações e o histórico de pedidos.

As configurações de loja (catálogo, ofertas, fretes, horários, formas de pagamento, antifraude) são definidas pelo associado no **Portal Radar E-commerce / E-Delivery** e consumidas pelo app em tempo de execução.

## 2. Objetivos e não-objetivos

**Objetivos**
- Reduzir a fricção do primeiro acesso até a homepage de uma loja.
- Habilitar a venda de medicamentos com retenção de receita, em conformidade com a RDC 144 da ANVISA, restritos à retirada na farmácia.
- Oferecer um fluxo de checkout completo (carrinho → entrega/retirada → pagamento) sobre o design system.
- Manter o consumidor informado do status do pedido via push.

**Não-objetivos**
- Cadastro/gestão de produtos e ofertas (responsabilidade do Portal).
- Logística de entrega e separação (sistemas de back office da loja).
- Autenticação/identidade além do necessário para vincular endereço e carteira.

## 3. Conceitos-chave

| Conceito | Definição |
|---|---|
| Módulos da loja | Uma loja pode estar ativa em **vendas**, **ofertas** e/ou **entrega (delivery)**. |
| Medicamento controlado | Produto classificado no Portal (campo **Categoria** = "Medicamentos com Retenção de Receita"). Sujeito à RDC 144. |
| Limitador de oferta | Quantidade máxima de unidades que um cliente pode comprar de um produto no preço de oferta. |
| Frete grátis | Valor mínimo de subtotal, por loja, a partir do qual o frete é gratuito. |
| Tempo de retirada/entrega | Tempo configurado pela loja, somado ao horário atual, para calcular a disponibilidade. |
| Gateway (atual) | Nova integração **Braspag**, apartada da integração E-Commerce **Cielo**, para pagamento online; tokenização/cofre de cartões e PIX. |
| Antifraude | Análise de risco (Cybersource) acionada pela Braspag, na mesma chamada do gateway, para lojas que contratam o serviço. |
| PEC | Base de clientes da rede onde os cadastros feitos pelo app são registrados. |

## 4. Cenários de usuário (principais)

- **Primeiro acesso (com cobertura):** Dado um usuário sem loja definida, quando informa localização ou endereço, então o app o direciona para a melhor loja disponível e exibe a homepage.
- **Primeiro acesso (sem cobertura):** Dado um endereço sem loja com entrega, quando o roteamento não encontra cobertura, então o app exibe "Ainda não atendemos sua região" com as 5 lojas mais próximas para retirada.
- **Compra de controlado:** Dado um controlado na cesta, quando o usuário avança no checkout, então apenas a retirada na loja fica disponível e a tag "Exige receita" é exibida.
- **Checkout com pagamento no app:** Dado um carrinho válido, quando o usuário escolhe "Pagar no app" e cartão de crédito, então confirma com CVV e finaliza a compra.
- **Pagamento PIX:** Dado um pagamento via PIX, quando o código é gerado, então o usuário tem 2h para pagar, com timer regressivo e estado de expiração.

---

## 5. Requisitos funcionais

> Origem de cada requisito sinalizada ao final: *(ECA-###)* = épico recente · *(baseline: Documento)* = comportamento pré-existente.

### 5.1 Medicamentos controlados (Épico 1)

- **FR-001** O app DEVE exibir medicamentos controlados em busca, departamentos e detalhe do produto. *(ECA-96)*
- **FR-002** O app NÃO DEVE rankear medicamentos controlados nas vitrines. *(ECA-96)*
- **FR-003** No detalhe de um controlado, o app DEVE exibir a mensagem "Retirada na loja mediante apresentação da receita." e a tag "Exige receita.". *(ECA-96)*
- **FR-004** Quando houver um controlado na cesta, o app DEVE tornar a entrega em casa indisponível e permitir o pedido apenas por retirada na loja. *(ECA-95)*
- **FR-005** Quando houver um controlado na cesta, a opção de retirada DEVE ser apresentada com o texto "Retirada na loja mediante apresentação da receita". *(ECA-95)*
- **FR-006** A tag "Exige receita" DEVE aparecer no resumo do pedido, na última tela do checkout. *(ECA-95)*
- **FR-007** Medicamentos controlados NÃO DEVEM ser apresentados com preço "De/Por"; somente o preço final, em qualquer tela. *(ECA-200)*
- **FR-008** Medicamentos controlados NÃO DEVEM exibir nenhuma comunicação promocional (percentuais, tags de preço do ERP, termos como "Desconto"/"Oferta"/"Barato"). *(ECA-200)*
- **FR-009** No detalhe de um controlado, o app NÃO DEVE exibir o componente de entrega no endereço. *(ECA-238)*
- **FR-010** O app NÃO DEVE exibir a progress bar de frete quando houver um controlado na cesta. *(ECA-234)*

### 5.2 Onboarding e homepage (Épico 2)

- **FR-011** O app DEVE oferecer entrada por **localização (GPS)** e por **endereço digitado** (autocomplete via Google Maps), listando até 5 sugestões. *(ECA-35, ECA-133)*
- **FR-012** O app DEVE rotear o usuário para a melhor loja, na ordem: (1) loja mais próxima com entrega (≤30 km); (2) loja mais próxima no módulo vendas; (3) loja mais próxima no módulo ofertas. *(ECA-35, ECA-133)*
- **FR-013** Sem loja que atenda a região, o app DEVE exibir a tela de indisponibilidade com as 5 lojas mais próximas do módulo vendas disponíveis para retirada. *(ECA-35)*
- **FR-014** No fluxo por localização, o app DEVE apresentar o mapa com PIN para confirmação e, em seguida, a modal de confirmação/complemento do endereço. *(ECA-274)*
- **FR-015** Na modal de endereço, os campos **Número**, **Rua** e **Bairro** DEVEM ser obrigatórios; **Complemento** opcional; **Número** e **Complemento** são textuais; **Número** vem pré-preenchido quando identificado pelo PIN. *(ECA-274)*
- **FR-016** No fluxo por endereço digitado, se o endereço vier incompleto, o app DEVE direcionar ao complemento dos dados. *(ECA-274)*
- **FR-017** O endereço do onboarding DEVE aparecer no header da homepage, ser salvo no usuário após o login e usado como endereço do carrinho. *(ECA-307)*
- **FR-018** Se já houver endereço salvo e o usuário refizer o onboarding, o endereço anterior DEVE ser substituído pelo novo. *(ECA-307)*
- **FR-019** A tela de indisponibilidade DEVE oferecer o botão "Alterar endereço", reiniciando o fluxo de onboarding. *(ECA-519)*
- **FR-020** A homepage DEVE exibir as vitrines na ordem: Ofertas destaque (se houver) → Ofertas pra você → Mais vendidos. *(ECA-58)*
- **FR-021** A vitrine "Mais vendidos" DEVE apresentar até 50 produtos (10 no carrossel; 50 na listagem), exibindo apenas itens com estoque; produto em oferta NÃO aparece em "Mais vendidos". *(ECA-58)*
- **FR-022** Homepage e cards de produto DEVEM seguir o design system; cards de oferta na home/ofertas iniciam com o CTA "Ativar" e liberam a adição ao carrinho após ativação. *(ECA-34, ECA-59)*
- **FR-023** Componentes DEVEM ser responsivos a fontes aumentadas pelo sistema operacional. *(ECA-35, ECA-300)*

#### Telas de referência

| | | |
|:-:|:-:|:-:|
| <img src="./assets/onboarding-confirmar-localizacao.jpg" width="220"> | <img src="./assets/onboarding-digitar-endereco.jpg" width="220"> | <img src="./assets/onboarding-homepage.jpg" width="220"> |
| *Confirmar localização (PIN no mapa)* | *Digitar endereço (autocomplete)* | *Nova homepage / vitrines* |
| <img src="./assets/onboarding-regiao-nao-atendida.jpg" width="220"> | | |
| *Região não atendida — lojas para retirada* | | |

#### Complementos — Homepage e seleção de loja (baseline)

- **FR-064** O campo de busca DEVE permanecer fixo no scroll da home, abrir a tela de busca ao toque e estar visível apenas em lojas do módulo vendas. *(baseline: Homepage)*
- **FR-065** O endereço do header DEVE seguir a prioridade: (1) localização compartilhada, (2) endereço cadastrado, (3) endereço do dispositivo; não havendo nenhum, fica vazio e DEVE ser preenchido na cesta antes de concluir a compra. *(baseline: Homepage)*
- **FR-066** Ao lado do endereço, o app DEVE exibir o tempo de entrega (quando a loja entrega no raio do cliente) ou a mensagem "Fora do alcance"; o botão "três pontos" abre os detalhes da loja. *(baseline: Homepage)*
- **FR-067** Os banners da home são configurados no Portal E-Delivery (o app apenas exibe a ordem definida pela rede/loja); o banner de CTA de ofertas abre a tela de ofertas; o banner de frete grátis (sem clique) aparece quando a loja tem regra de frete grátis. *(baseline: Homepage)*
- **FR-068** Os departamentos DEVEM seguir a ordem: Cuidados pessoais, Higiene e beleza, Medicamentos, Farmacinha, Infantil, Nutrição e alimentos, Outros. *(baseline: Homepage)*
- **FR-069** Os carrosséis da home DEVEM exibir até 10 produtos; "Ver mais" abre a vitrine completa; produtos sem estoque não são exibidos. *(baseline: Homepage)*
- **FR-070** A permissão de localização nativa define o fluxo: "Permitir uma vez" ou "Durante o uso do app" NÃO passam pela tela de seleção de lojas; "Não permitir" passa pela seleção de lojas. *(baseline: Seleção de lojas)*
- **FR-071** Havendo mais de uma loja que entrega no endereço, o desempate DEVE seguir: P1 mais próxima → P2 frete mais barato → P3 menor tempo de entrega; empate total nas três variáveis → escolha aleatória. *(baseline: Seleção de lojas)*
- **FR-072** As regras de inativação de loja (em construção/indisponível) são mantidas; lojas indisponíveis ainda são rankeadas na listagem; sem resultado da API, exibir placeholder. *(baseline: Seleção de lojas)*

> **Nota de conflito (vitrines):** o baseline previa vitrines adicionais ("Você já comprou" — compras dos últimos 60 dias; "Dermocosméticos"; "Higiene e beleza") e segmentação de público (4 segmentos, atualização manual trimestral pelo time de dados). A estrutura **vigente** segue o **FR-020 (ECA-58)** — Ofertas destaque → Ofertas pra você → Mais vendidos —, que substitui a lista anterior.
> **Nota de conflito (roteamento):** o **FR-012 (ECA-35)** define a prioridade vigente de roteamento; o desempate detalhado do FR-071 aplica-se dentro do conjunto de lojas que entregam no endereço.

### 5.3 Notificações (Épico 3)

- **FR-024** O app DEVE disparar uma push notification a cada transição de status do pedido, com no máximo uma notificação por transição e nenhuma comunicação fora do status atual. *(ECA-194)*
- **FR-025** As mensagens DEVEM seguir a matriz: Fila→Separação ("Tudo certo! / Iniciamos a separação do seu pedido."); Separação→Liberados retirada ("Pedido pronto para retirada! / Venha até a farmácia retirar o seu pedido."); Separação→Liberados entrega ("Seu pedido saiu para entrega! / Em breve o entregador chegará ao seu endereço."); Liberados→Concluídos ("Seu pedido foi finalizado! / Esperamos que tenha tido uma boa experiência."); Cancelado ("Seu pedido foi cancelado! / Se precisar de suporte entre em contato com a farmácia."). *(ECA-194)*

### 5.4 Endereços do consumidor (Épico 4)

- **FR-026** A tela de endereços DEVE ter as seções "Endereço atual" (com tag "Selecionado") e "Outros endereços", sem repetir o atual em Outros. *(ECA-311)*
- **FR-027** Com apenas um endereço salvo, a seção "Outros endereços" NÃO DEVE ser exibida; sem nenhum, exibir placeholder e botão de adicionar; endereços legados passam a "Outros". *(ECA-311)*
- **FR-028** Adicionar endereço DEVE reutilizar o fluxo do onboarding (busca via Maps); o novo endereço passa a ser o selecionado. *(ECA-373)*
- **FR-029** Editar endereço DEVE abrir a tela de adição preenchida; a alteração salva reflete na gestão de endereços. *(ECA-370)*
- **FR-030** Excluir endereço DEVE removê-lo do usuário; o app NÃO DEVE permitir excluir quando há apenas um endereço cadastrado. *(ECA-414)*

#### Telas de referência

| | | |
|:-:|:-:|:-:|
| <img src="./assets/enderecos-estado-vazio.jpg" width="220"> | <img src="./assets/enderecos-gestao.jpg" width="220"> | <img src="./assets/enderecos-selecionar-novo.jpg" width="220"> |
| *Estado vazio (nenhum endereço)* | *Gestão — endereço atual e outros* | *Selecionar novo endereço* |

### 5.5 Checkout — Carrinho (Épico 5)

- **FR-031** O carrinho DEVE permitir adicionar/remover quantidades, exibindo sempre o preço unitário no card (sem alterar ao mudar a quantidade). *(ECA-300)*
- **FR-032** O subtotal DEVE somar os preços finais; o contador considera todas as unidades; o subtexto do card é o "Fabricante" do catálogo; clique na imagem/nome abre o detalhe. *(ECA-300)*
- **FR-033** Condições de parcelamento DEVEM aparecer apenas se a loja as tiver habilitadas no Portal. *(ECA-300)*
- **FR-034** Sem estoque adicional disponível, o app DEVE exibir "Quantidade máxima disponível atingida". *(ECA-300)*
- **FR-035** Alterações de configuração/disponibilidade no Portal DEVEM refletir em carrinhos já montados; o app NÃO DEVE empilhar telas de carrinho. *(ECA-300)*
- **FR-036** Ao exceder o limite de oferta, o produto DEVE ser dividido em dois cards ("Oferta" e "Preço normal"): o "+" na oferta esgotada cria o card "Preço normal" (qtd. 1); a redução baixa prioritariamente o "Preço normal" independentemente do "–" clicado; "Preço normal" em 0 desaparece. *(ECA-515)*
- **FR-037** O banner amarelo de limite DEVE aparecer somente ao clicar no "+" da oferta já no limite, com texto dinâmico vindo do campo "Limitador de oferta" do Portal. *(ECA-515)*
- **FR-038** O carrinho vazio DEVE exibir mensagem informativa e o botão "Explorar produtos" (→ home); o botão voltar retorna à tela anterior. *(ECA-301)*
- **FR-039** A tag "Exige receita" DEVE aparecer no card de todo produto controlado, já na primeira tela do carrinho. *(ECA-520)*
- **FR-040** A tag "Oferta" DEVE aparecer somente para produtos com oferta criada no Portal (não para produtos com preço inicial/final do ERP sem oferta). *(ECA-517)*
- **FR-041** A tag de frete grátis DEVE mostrar quanto falta para o frete grátis (subtotal vs. valor do Portal) e atualizar ao atingir o valor; e-mails @febrafar/@farmarcas sempre veem frete grátis; cestas com controlados não exibem a tag. *(ECA-566)*
- **FR-042** A tag "Preço normal" DEVE ser exibida apenas quando o usuário atinge o limite da oferta. *(ECA-595)*

#### Telas de referência

| | | |
|:-:|:-:|:-:|
| <img src="./assets/checkout-carrinho.jpg" width="220"> | <img src="./assets/checkout-carrinho-limite-oferta.jpg" width="220"> | <img src="./assets/checkout-carrinho-vazio.jpg" width="220"> |
| *Carrinho — lista e subtotal* | *Limite de oferta (card dividido)* | *Carrinho vazio* |

#### Complementos — Carrinho (baseline)

- **FR-073** O componente de quantidade DEVE sempre exibir a quantidade atual: com 1 unidade, o botão à esquerda é o de excluir; com 2+ unidades, é o de subtrair. *(baseline: Cesta)*
- **FR-074** A mensagem "Restam X unidades" DEVE aparecer no card sempre que o produto tiver a partir de 5 unidades em estoque. *(baseline: Cesta)*

> **Nota de conflito (checkout):** o fluxo de cesta do baseline (tela única com "Você economizou", resumo e formas de pagamento antigas) foi **substituído** pelo novo checkout dos FR-031–FR-063 (Épico 5). As regras acima permanecem por descreverem comportamento de quantidade/estoque ainda vigente.

### 5.6 Checkout — Entrega e retirada (Épico 5)

- **FR-043** A seção "Meu Endereço" DEVE usar o endereço atual do usuário (ver FR-017). *(ECA-309)*
- **FR-044** Com controlado na cesta, a entrega em casa DEVE ser ocultada e a tag "Exige receita" exibida. *(ECA-309)*
- **FR-045** Usuário não logado, ao clicar em "Ir para pagamento", DEVE ser redirecionado ao login. *(ECA-309)*
- **FR-046** A retirada DEVE sempre exibir a tag "Grátis" e a farmácia atual; o botão "Trocar" é ocultado quando não há troca disponível. *(ECA-309)*
- **FR-047** A mensagem de retirada ("Disponível hoje a partir das…") DEVE considerar horário atual + tempo de retirada da loja; ao ultrapassar o funcionamento, vira "amanhã a partir das…"; lojas 24h após 00:00 usam "amanhã" e atualizam para "hoje" ao virar o dia. *(ECA-309)*
- **FR-048** A entrega DEVE exibir a tag de frete (valor/grátis) conforme o raio do cliente; a mensagem ("Receba hoje até…") considera o **horário de funcionamento do delivery** (distinto do horário da loja); cenários acima de 2 dias usam data ("Receba até dd/mm/aaaa"). *(ECA-309)*
- **FR-049** Os cálculos de horário NÃO DEVEM considerar o fuso do servidor (UTC+3). *(ECA-309)*
- **FR-050** A troca de farmácia DEVE listar "Loja atual" e "Outras lojas" (mesmo grupo econômico, raio 100 km, ordenadas por distância, até 15 lojas). *(ECA-579)*
- **FR-051** Trocar para loja sem alteração na cesta NÃO exige confirmação; havendo alteração, o app DEVE exibir modal com todas as mudanças para o usuário confirmar e, ao confirmar, retornar à tela de entrega/retirada. *(ECA-579)*

#### Telas de referência

| | | |
|:-:|:-:|:-:|
| <img src="./assets/checkout-entrega-retirada.jpg" width="220"> | <img src="./assets/checkout-entrega-exige-receita.jpg" width="220"> | <img src="./assets/checkout-trocar-loja.jpg" width="220"> |
| *Entrega/retirada* | *Exige receita (controlado)* | *Trocar loja para retirada* |

#### Complementos — Troca de loja e distâncias (baseline)

- **FR-075** Ao prosseguir para a entrega, o app DEVE exibir o modal de confirmação de endereço: manter o endereço → o app verifica se a loja entrega no local (sim: entrega + retirada; não: só retirada); alterar → tela de gestão de endereços. *(baseline: Troca de loja)*
- **FR-076** Ao trocar de endereço na entrega: se a loja atual entrega no novo endereço, mantém a loja; senão, busca outra que entregue e escolhe pelo menor número de alterações na cesta (desempate: disponibilidade → quantidade de unidades); se nenhuma entrega, mantém a loja e marca a entrega como indisponível. *(baseline: Troca de loja)*
- **FR-077** Na retirada, o app mantém a loja atual, exibe o card da loja e o botão "Escolher outra farmácia"; se a loja não faz retirada, exibe "Esta farmácia não efetua retiradas" e desabilita o pagamento. *(baseline: Troca de loja)*
- **FR-078** Busca de farmácias para retirada: header fixo (endereço + busca); busca por endereço (CEP, cidade, bairro, logradouro — nunca por nome da loja); sugestões a partir do 4º caractere (até 5); histórico das últimas 5 buscas, com exclusão; "0 farmácias encontradas" quando nada corresponde. *(baseline: Troca de loja)*
- **FR-079** Listagem de lojas para retirada: raio de 100 km; destaca a loja atual; mostra a contagem de farmácias disponíveis; carrega 6 lojas e +6 a cada scroll; lojas sem estoque de qualquer item da cesta não são listadas; endereço limitado a 35 caracteres. *(baseline: Troca de loja)*
- **FR-080** Lojas com impacto na cesta exibem "Sua cesta irá sofrer alterações"; ao selecioná-las, o app mostra o modal de confirmação; sem impacto, segue direto para a entrega; a troca atualiza todas as configurações e o pagamento para a nova loja. *(baseline: Troca de loja)*
- **FR-081** Ao trocar de loja, o app consulta as alterações (disponibilidade, preço, novas condições) e prioriza a mensagem por produto: estoque indisponível → alteração de quantidade → alteração de preço; ajusta automaticamente a quantidade ao estoque disponível; verifica oferta equivalente na nova loja. *(baseline: Troca de loja)*
- **FR-082** Havendo alterações, o app exibe "Houve alterações na sua cesta"; "Ver alterações" abre o modal; "Manter alterações" segue na entrega; "Trocar endereço" leva à gestão de endereços. *(baseline: Troca de loja)*
- **FR-083** O serviço default na tela de entrega é: entrega selecionada quando disponível; caso contrário, retirada selecionada; a label "Grátis" aparece sempre que o frete (entrega) ou a retirada forem gratuitos. *(baseline: Troca de loja)*
- **FR-084** Formatação de distância: ≥ 1 km em quilômetros (número cheio = 1 casa decimal; quebrado = 2 casas, ex. "2.10km"); < 1 km em metros (até 999 m); < 1 m arredonda para "1m"; mesmo endereço da loja exibe "1m". *(baseline: Troca de loja)*
- **FR-085** Tempo de retirada: exibido em todas as lojas listadas, refletindo o campo "Tempo de retirada" do Portal, sempre em minutos; < 10 min com um algarismo ("9 minutos", não "09"); valor zerado → "Disponível para retirada imediata". *(baseline: Troca de loja)*

> **Relação com ECA-579:** o FR-050/FR-051 (design system) descrevem a versão atual da troca de loja; os FR-075–FR-085 detalham regras operacionais do baseline que seguem válidas (busca, paginação, distâncias, tempo de retirada).

### 5.7 Checkout — Pagamento (Épico 5)

- **FR-052** O botão de pagamento (entrega ou retirada) DEVE ser dinâmico conforme o formato do pedido; as formas só aparecem após o usuário selecionar a modalidade. *(ECA-567)*
- **FR-053** Se a loja oferece apenas uma modalidade, ela DEVE vir pré-selecionada e a outra apresentada como indisponível. *(ECA-567, ECA-568)*
- **FR-054** No pagamento na entrega/retirada, as formas DEVEM seguir a ordem: Pix, Cartão (maquininha), Dinheiro. *(ECA-567)*
- **FR-055** Ao escolher Dinheiro, "Precisa de troco?" é obrigatório; se "Sim", "Troco para quanto?" é obrigatório; se "Não", o campo de troco não aparece. *(ECA-567)*
- **FR-056** O resumo da compra inicia compacto e expande sob demanda; abaixo dele, exibir "Loja para retirada" ou "Endereço de entrega" conforme o pedido. *(ECA-567)*
- **FR-057** No pagamento no app, PIX direciona à tela de PIX; crédito sem cartão salvo exibe "Adicionar novo cartão"; com cartão salvo exibe o cartão (bandeira + 4 últimos dígitos, restante mascarado) e "Alterar ou adicionar cartões". *(ECA-568)*
- **FR-058** A adição de cartão DEVE: refletir os dados no cartão ilustrativo; identificar a bandeira pelo BIN (Visa, Mastercard, Elo, Amex); mascarar o número a cada 4 dígitos; validar mês (01–12) e ano (≥ vigente); forçar caixa alta e apenas letras no titular; mascarar e validar o CPF. *(ECA-569)*
- **FR-059** "Adicionar cartão" DEVE tokenizar o cartão no gateway e salvá-lo na carteira; erros exibem "Verifique os dados do cartão" ou "CPF do titular inválido". *(ECA-569)*
- **FR-060** A gestão de cartões DEVE exibir o ícone de exclusão e ocultar "Adicionar cartão"; excluir remove o cartão da carteira, do checkout e do cofre da Braspag (com mensagem de sucesso); excluir todos retorna à tela de pagamento. *(ECA-577)*
- **FR-061** O PIX DEVE ter validade de 2h (regra Braspag) com timer regressivo (hh:mm:ss) que persiste em segundo plano e reflete o tempo restante real ao retornar; enquanto não pago, o pedido fica "Aguardando pagamento"; ao expirar, "Pagamento expirado". *(ECA-578)*
- **FR-062** O app DEVE permitir copiar o código PIX (feedback "Código Pix copiado") e ver o QR Code. *(ECA-578)*
- **FR-063** Toda venda no app com cartão de crédito DEVE solicitar o CVV em modal após "Finalizar compra"; a modal exibe o cartão selecionado; o CVV enriquece o payload ao gateway; CVV inválido solicita correção; Amex aceita 4 dígitos. *(ECA-665 — em testes)*

#### Telas de referência

| | | |
|:-:|:-:|:-:|
| <img src="./assets/checkout-pagamento-na-entrega.jpg" width="220"> | <img src="./assets/checkout-pagamento-no-app.jpg" width="220"> | <img src="./assets/checkout-adicionar-cartao.jpg" width="220"> |
| *Pagar na entrega (PIX/cartão/dinheiro)* | *Pagar no app (PIX/crédito)* | *Adicionar novo cartão* |
| <img src="./assets/checkout-pagamento-pix.jpg" width="220"> | <img src="./assets/checkout-cvv.jpg" width="220"> | <img src="./assets/checkout-pix-expirado.jpg" width="220"> |
| *Pagamento via PIX (timer + copiar)* | *Confirmação por CVV* | *PIX expirado — gerar novo* |

#### Complementos — PIX e tokenização (baseline)

- **FR-086** Ao concluir a compra, o consumidor DEVE ser direcionado à tela de pagamento do pedido. *(baseline: PIX Online)*
- **FR-087** Ao identificar o pagamento, o app DEVE exibir a mensagem de sucesso e direcionar o consumidor à tela de acompanhamento do pedido. *(baseline: PIX Online)*
- **FR-088** Se o pagamento não ocorrer no prazo, o pedido DEVE ser cancelado e o consumidor direcionado ao acompanhamento com o status "Cancelado". *(baseline: PIX Online — refina o FR-061)*
- **FR-089** O pedido só DEVE aparecer na fila do Portal após a validação do pagamento via PIX. *(baseline: PIX Online)*
- **FR-090** Ao cancelar um pedido pago via PIX, o valor DEVE ser estornado automaticamente ao consumidor. *(baseline: PIX Online)*
- **FR-091** O cartão salvo fica na carteira do consumidor, acessível pela tela de perfil e pela tela de pagamento da cesta. *(baseline: Tokenização)*
- **FR-092** O campo "Apelido" do cartão é opcional (máx. 22 caracteres); se preenchido, passa a ser o nome do cartão na carteira. *(baseline: Tokenização)*
- **FR-093** O dropdown de validade DEVE considerar o ano atual + 12 anos; ao salvar, o app verifica se o cartão é válido. *(baseline: Tokenização)*
- **FR-094** Ao excluir um cartão, o token DEVE ser removido também no gateway. *(baseline: Tokenização)*

> **Nota de migração:** a tokenização vem migrando da integração E-Commerce **Cielo** para o novo **gateway Braspag** (ver 5.10). Os FR-059/FR-060 referenciam o cofre do gateway de pagamento.

### 5.8 Acesso — Login e cadastro (baseline)

- **FR-095** O login DEVE aceitar e-mail ou CPF. *(baseline: Login e cadastro)*
- **FR-096** "Criar minha conta" abre o cadastro; todos os campos são obrigatórios; o botão inicia desabilitado e habilita após o preenchimento completo + aceite dos termos. *(baseline: Login e cadastro)*
- **FR-097** O campo nome aceita apenas letras (sem números). *(baseline: Login e cadastro)*
- **FR-098** "Ler termos" exibe o PDF dos termos de uso. *(baseline: Login e cadastro)*
- **FR-099** "Esqueci minha senha" inicia o reset via CPF; "Continuar" habilita apenas com CPF em formato válido. *(baseline: Login e cadastro)*
- **FR-100** O reset envia um código de confirmação ao e-mail cadastrado; após o código, o usuário define a nova senha. *(baseline: Login e cadastro)*
- **FR-101** A senha DEVE ter no mínimo: 8 caracteres, 1 letra maiúscula e 1 caractere especial. *(baseline: Login e cadastro)*
- **FR-102** Se o CPF não estiver cadastrado ao iniciar o reset, o app DEVE direcionar para a criação de conta. *(baseline: Login e cadastro)*
- **FR-103** Todo cadastro feito pelo app DEVE ser registrado na base de clientes do PEC da rede. *(baseline: Login e cadastro)*

### 5.9 Ofertas (baseline)

- **FR-104** Usuário deslogado: o card de ofertas incentiva o login ("Faça login para ter acesso às ofertas") e o clique leva ao login. *(baseline: Ofertas)*
- **FR-105** Usuário logado: o card do topo informa o número de ofertas ativas; sem ofertas ativas, exibe "Ative ofertas e economize mais!". *(baseline: Ofertas)*
- **FR-106** Ao visualizar as ofertas ativas, o app exibe a mensagem fixa "Ofertas válidas enquanto durarem os estoques. Descontos não cumulativos". *(baseline: Ofertas)*
- **FR-107** A tela apresenta os carrosséis de ofertas e de ofertas destaque (até 10 produtos); "Ver mais" abre a vitrine completa. *(baseline: Ofertas)*
- **FR-108** Ativação: "Ativar" ativa a oferta no CPF do cliente com mensagem de sucesso (erro → tela de erro); após ativar, oculta "Ativar", mostra a compra e exibe a label "Oferta ativada" e a duração da oferta no card. *(baseline: Ofertas; relacionado a ECA-59)*

### 5.10 Gateway de pagamento e antifraude (baseline)

- **FR-109** O app DEVE integrar a um novo gateway de pagamento (Braspag), apartado da integração E-Commerce (Cielo) de pagamento online. *(baseline: Gateway)*
- **FR-110** As chaves do gateway (MerchantID/MerchantKey) são distintas das da API E-Commerce; lojas com pagamento online que contratarem o gateway reconfiguram com as novas credenciais. *(baseline: Gateway)*
- **FR-111** O antifraude (Cybersource) integra pela mesma API do gateway, em chamada única (a Braspag faz a ponte), para lojas que contratarem o serviço. *(baseline: Gateway)*
- **FR-112** O gateway pode ser contratado sem antifraude; o antifraude exige o gateway; o funcionamento depende da configuração dos meios de pagamento no portal da Braspag. *(baseline: Gateway)*

> **⚠️ Segurança:** as credenciais de homologação (MerchantID, MerchantKey, ClientId, ClientSecret), os IDs do Cybersource e as regras de teste (sobrenomes accept/review/reject) constam nos handoffs do parceiro e **não devem ser versionadas neste repositório** — mantê-las em gerenciador de segredos / variáveis de ambiente.

### 5.11 Histórico de pedidos (baseline)

- **FR-113** "Pedidos em andamento" lista pedidos nos status Fila, Em separação e Liberado; o contador de etapas preenche 1, 2 ou 3 marcadores conforme o status. *(baseline: Histórico de pedidos)*
- **FR-114** "Detalhes" leva à tela de acompanhamento do pedido. *(baseline: Histórico de pedidos)*
- **FR-115** "Pedidos concluídos" lista pedidos concluídos e cancelados; o título do card é o nome do primeiro produto inserido na cesta. *(baseline: Histórico de pedidos)*
- **FR-116** As duas seções são ordenadas por data de criação, do mais recente para o mais antigo. *(baseline: Histórico de pedidos)*

---

## 6. Entidades principais

| Entidade | Atributos relevantes |
|---|---|
| Consumidor | id, endereços[], endereço selecionado, carteira (cartões) |
| Conta de usuário | nome, CPF, e-mail, senha (mín. 8 / 1 maiúscula / 1 especial), telefone, data de nascimento, aceite de termos, registro no PEC |
| Endereço | rótulo (Casa/Trabalho/Outro), rua, número, bairro, complemento, CEP, geolocalização, selecionado (bool) |
| Loja | nome, grupo econômico, módulos ativos, horário da loja, horário do delivery, tempo de retirada/entrega, raio, frete grátis, distância |
| Produto | id, sku, nome, fabricante, preço final, preço inicial, oferta, limitador de oferta, `requiresPrescriptionRetention` |
| Oferta | produto, preço de oferta, duração, limitador de oferta, destaque (bool), status (ativada no CPF) |
| Carrinho | itens[] (produto, qtd, origem oferta/normal), subtotal, frete, endereço/loja |
| Cartão | bandeira, 4 últimos dígitos, token (cofre Cielo/Braspag), titular, CPF, apelido (opcional) |
| Pedido | status (Fila, Em separação, Liberado, Concluído, Cancelado), método (entrega/retirada), forma de pagamento, PIX (código, expiração), data de criação |
| Config. de pagamento (loja) | meios habilitados, MerchantID/MerchantKey (gateway), antifraude (on/off) |

## 7. Itens em aberto

- `[EM TESTES]` **FR-063 (ECA-665)** — confirmação por CVV ao finalizar no app.
- `[MIGRAÇÃO]` **Gateway/tokenização** migrando de **Cielo** (E-Commerce) para **Braspag** (FR-094, FR-109–FR-112). Confirmar quais lojas já operam no novo gateway.
- `[CONFLITO RESOLVIDO]` **Vitrines da home** e **roteamento de loja**: prevalece a regra atual (FR-020/ECA-58 e FR-012/ECA-35); baseline anterior preservado nas notas da seção 5.2.
- `[SEGURANÇA]` Credenciais dos handoffs de gateway/tokenização **não** versionadas no repositório (ver nota em 5.10).

## 8. Checklist de revisão

- [x] Requisitos focam em **o quê/porquê**, não em implementação.
- [x] Cada FR tem origem sinalizada (ECA ou baseline) e é verificável.
- [x] Cenários de usuário cobrem os fluxos principais.
- [x] Conformidade regulatória (RDC 144) explicitada para controlados.
- [x] Conflitos baseline × épicos resolvidos a favor da regra atual, com nota.
- [x] Credenciais sensíveis mantidas fora da spec.
- [ ] Validar FR-063 após conclusão dos testes do ECA-665.
- [ ] Confirmar o estado da migração de gateway (Cielo → Braspag).
