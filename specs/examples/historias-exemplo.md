# Exemplos de Histórias de Usuário — Radar App

Este arquivo contém histórias reais do projeto para servir como referência de formato e qualidade esperada.

---

## ECA-309 · Carrinho - Entrega e retirada
**Épico:** Checkout
**Tipo:** História | **Story Points:** 8 | **Prioridade:** Highest

**User Story**
Como usuário consumidor que tem acesso ao aplicativo, quero definir com clareza e facilidade o método de recebimento do meu pedido no aplicativo.

**O que deve ser feito**
- Tela de definição da entrega do usuário
- Apresentação da loja atual para retirada

**Regras de Negócio**
- A sessão "Meu Endereço" deve apresentar o endereço atual do usuário com base nas regras existentes
- Se a cesta tiver algum medicamento controlado, a opção de entrega em casa deve ser ocultada e a tag "Exige receita" apresentada
- Se o usuário não estiver logado, ao clicar em "Ir para pagamento" deve ser redirecionado para o login
- Retirada na loja sempre deve aparecer com a tag "Grátis"
- A mensagem "Disponível hoje a partir das…" deve considerar o horário do recebimento + tempo de retirada configurado pela loja no Portal
- Se horário atual + tempo de retirada ultrapassar o horário de funcionamento, exibir "Disponível para retirada amanhã a partir das…"
- Em lojas 24h, se retirada for após 00:00, exibir "Disponível para retirada amanhã" e atualizar ao atingir 00:00
- Caso não exista opção de troca de loja, o botão "Trocar" deve ser ocultado
- Entrega em casa deve exibir tag de frete (valor ou grátis) conforme configuração da loja no Portal
- A mensagem "Receba hoje até…" deve considerar horário do recebimento + tempo de entrega configurado pela loja
- Se somatória ultrapassar horário de funcionamento do delivery, exibir "Receba amanhã até…"
- Se somatória ultrapassar 2 dias, exibir data completa: "Receba até xx/xx/xxxx"
- Horário de funcionamento do delivery é configuração separada do horário da loja

**Critérios de Aceite**
- Consigo identificar com clareza o meu endereço atual
- Consigo identificar com clareza o dia e horário para retirar o pedido na loja
- Consigo identificar com clareza a minha loja selecionada para retirada
- Consigo identificar com clareza o valor do frete da entrega
- Consigo identificar com clareza o dia e horário que vou receber o pedido
- Os horários não devem considerar o fuso do servidor UTC+3
- Alterações nas configurações da farmácia no Portal devem refletir nos carrinhos montados antes das alterações

**Link Figma:** https://www.figma.com/design/7Pi4v28TLqe0YhcqqKAEjR/App-%C2%B7-E-commerce?node-id=386-1354

---

## ECA-579 · Carrinho - Troca de farmácia na retirada
**Épico:** Checkout
**Tipo:** História | **Story Points:** 5 | **Prioridade:** Medium

**User Story**
Como usuário consumidor que tem acesso ao aplicativo, quero conseguir selecionar no carrinho uma loja para retirada do meu pedido.

**O que deve ser feito**
- Botão "Trocar" no card da farmácia selecionada na tela de entrega do checkout
- Tela com a farmácia selecionada + outras lojas como opção de troca
- Validação do número de alterações da cesta ao trocar de loja
- Modal com confirmação das alterações na cesta por produto

**Regras de Negócio**
- Ao clicar em "Trocar" o usuário deve ser redirecionado para a tela de lojas disponíveis para retirada
- A tela deve apresentar duas seções: Loja atual e Outras lojas (mesmo grupo econômico, raio de 100km)
- Lojas devem ser ordenadas por distância (mais próxima primeiro)
- Se a farmácia selecionada não gerar alteração na cesta, não exibir modal de confirmação
- Se gerar alteração, exibir modal com todas as mudanças para o usuário confirmar
- Ao confirmar, redirecionar para a tela de entrega/retirada do checkout
- O app pode listar até 15 farmácias como opção de troca

**Critérios de Aceite**
- Consigo identificar com clareza qual a minha loja atual e as opções de troca
- Consigo visualizar com clareza a distância das lojas em relação à minha localização
- Consigo identificar quantas e quais mudanças ocorrerão na minha cesta ao trocar de farmácia
- Consigo identificar o valor total da minha cesta na nova loja
- Alterações nas configurações da farmácia no Portal devem refletir nos carrinhos montados antes das alterações

---

## ECA-578 · Carrinho - Fluxo pagamento PIX
**Épico:** Checkout
**Tipo:** História | **Story Points:** 8 | **Prioridade:** Medium

**User Story**
Como usuário consumidor que tem acesso ao aplicativo, quero realizar um pagamento no aplicativo utilizando PIX.

**O que deve ser feito**
- Tela de pagamento do PIX
- Timer com o tempo de expiração
- Botão para copiar o código PIX
- Tela com QR Code do PIX

**Regras de Negócio**
- A duração do PIX é de 2 horas (Regra Braspag)
- O timer deve exibir contagem regressiva no formato HH:MM:SS (02:00:00)
- Se o consumidor sair da tela, o app deve manter a contagem em segundo plano
- Ao retornar, o timer deve representar exatamente o tempo restante
- Enquanto não pagar, o status do pedido deve ser "Aguardando pagamento"
- Ao copiar o código, exibir mensagem "Código Pix copiado"
- Ao clicar em "Ver QR Code", redirecionar para a tela com QR Code
- Ao expirar o código PIX, o pedido deve ficar com status "Pagamento expirado"

**Critérios de Aceite**
- Ao realizar pagamento via PIX, tenho 2 horas para concluir
- Consigo visualizar exatamente horas, minutos e segundos restantes para expirar
- Consigo copiar o QR Code para usar no aplicativo do banco
- Visualizo o pedido não pago com status "Aguardando pagamento"
- Consigo escanear o QR Code para pagar com outro device

---

## ECA-498 · [FRONT] Ajustar layout dos toasts do app
**Épico:** Melhorias e sustentação
**Tipo:** Tarefa | **Story Points:** 2 | **Prioridade:** Low

**O que?**
Ajustar layout dos toasts do app para o novo formato padronizado.

**Por que?**
A task ECA-374 está em testes e não queremos travar outras entregas. Seguimos com toasts antigos temporariamente e precisamos ajustar.

**Como?**
Ajustar os seguintes toasts para o novo formato:
- Selecionar endereço com sucesso: `success`, mensagem "Endereço selecionado!"
- Falha ao selecionar endereço: `error`, mensagem da failure ou fallback
- Tentar remover quando só há um endereço: `warning`, mensagem "Você precisa ter pelo menos um endereço"
- Falha ao promover próximo endereço após remover o atual: `error`, mensagem da failure ou fallback
- Remover endereço com sucesso: `success`, mensagem "Endereço removido"
- Falha ao remover endereço: `error`, mensagem da failure ou fallback

---

## ECA-643 · [FRONT] Manter o usuário na tela ao selecionar novo endereço
**Épico:** Melhorias e sustentação
**Tipo:** Tarefa | **Story Points:** 1 | **Prioridade:** Low

**O que?**
Manter o usuário na tela ao trocar para um outro endereço.

**Por que?**
Atualmente, após selecionar outro endereço, o consumidor está sendo direcionado para a tela de perfil.

**Como?**
Manter o usuário na tela de gestão de endereços após selecionar um novo endereço.

---

## Padrões observados nestas histórias

Use estes padrões ao gerar novas histórias:

- **User story:** "Como usuário consumidor que tem acesso ao aplicativo, quero [ação]"
- **Seções obrigatórias em histórias:** User Story, Overview, O que deve ser feito, Regras de Negócio, Critérios de Aceite, Link Figma
- **Seções em tarefas técnicas:** O que, Por que, Como
- **Critérios de aceite:** escritos na primeira pessoa ("Consigo...", "Visualizo...", "Tenho...")
- **Regras de negócio:** detalhadas, cobrindo edge cases de horário, estado, fluxo alternativo
- **Subtarefas padrão:** QA (casos de teste), [BE] backend, [APP] frontend mobile
- **Story Points:** 1-2 para tarefas simples, 5-8 para histórias complexas com integrações