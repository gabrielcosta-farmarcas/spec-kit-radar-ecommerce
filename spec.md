# Especificação: Aplicativo de E-commerce — Radar E-commerce

| Campo | Valor |
|---|---|
| **Feature** | `001-app-ecommerce` |
| **Projeto (Jira)** | SQ — ECOMMERCE APP |
| **Status** | Entregue (33 itens finalizados, 1 em testes) |
| **Tipo** | Especificação consolidada (retrospectiva dos épicos 1–5) |
| **Versão** | 1.0 |
| **Atualizado** | 2026-06 |

> Esta spec consolida o comportamento entregue ao longo de cinco épicos. Descreve **o quê** e **o porquê** do produto (não o **como** de implementação, que vive em `plan.md`). Requisitos funcionais usam IDs `FR-###` com rastreabilidade aos tickets `ECA-###`.

---

## 1. Resumo

O aplicativo permite que o consumidor encontre uma farmácia próxima, navegue pelo catálogo, monte um carrinho e finalize a compra com retirada na loja ou entrega em casa, pagando na entrega/retirada ou no app (PIX e cartão). O escopo desta spec cobre cinco frentes: venda de medicamentos controlados, onboarding e homepage, notificações de pedido, gestão de endereços e o novo checkout.

As configurações de loja (catálogo, ofertas, fretes, horários, formas de pagamento) são definidas pelo associado no **Portal Radar E-commerce** e consumidas pelo app em tempo de execução.

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
| Gateway | Braspag/Cielo — tokenização/cofre de cartões e processamento de PIX e crédito. |

## 4. Cenários de usuário (principais)

- **Primeiro acesso (com cobertura):** Dado um usuário sem loja definida, quando informa localização ou endereço, então o app o direciona para a melhor loja disponível e exibe a homepage.
- **Primeiro acesso (sem cobertura):** Dado um endereço sem loja com entrega, quando o roteamento não encontra cobertura, então o app exibe "Ainda não atendemos sua região" com as 5 lojas mais próximas para retirada.
- **Compra de controlado:** Dado um controlado na cesta, quando o usuário avança no checkout, então apenas a retirada na loja fica disponível e a tag "Exige receita" é exibida.
- **Checkout com pagamento no app:** Dado um carrinho válido, quando o usuário escolhe "Pagar no app" e cartão de crédito, então confirma com CVV e finaliza a compra.
- **Pagamento PIX:** Dado um pagamento via PIX, quando o código é gerado, então o usuário tem 2h para pagar, com timer regressivo e estado de expiração.

---

## 5. Requisitos funcionais

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

---

## 6. Entidades principais

| Entidade | Atributos relevantes |
|---|---|
| Consumidor | id, endereços[], endereço selecionado, carteira (cartões) |
| Endereço | rótulo (Casa/Trabalho/Outro), rua, número, bairro, complemento, CEP, geolocalização, selecionado (bool) |
| Loja | nome, grupo econômico, módulos ativos, horário da loja, horário do delivery, tempo de retirada/entrega, raio, frete grátis, distância |
| Produto | id, sku, nome, fabricante, preço final, preço inicial, oferta, limitador de oferta, `requiresPrescriptionRetention` |
| Carrinho | itens[] (produto, qtd, origem oferta/normal), subtotal, frete, endereço/loja |
| Cartão | bandeira, 4 últimos dígitos, token (cofre), titular, CPF |
| Pedido | status, método (entrega/retirada), forma de pagamento, PIX (código, expiração) |

## 7. Itens em aberto

- `[EM TESTES]` **FR-063 (ECA-665)** — confirmação por CVV ao finalizar no app.

## 8. Checklist de revisão

- [x] Requisitos focam em **o quê/porquê**, não em implementação.
- [x] Cada FR é verificável e rastreável a um ticket ECA.
- [x] Cenários de usuário cobrem os fluxos principais.
- [x] Conformidade regulatória (RDC 144) explicitada para controlados.
- [ ] Validar FR-063 após conclusão dos testes do ECA-665.
