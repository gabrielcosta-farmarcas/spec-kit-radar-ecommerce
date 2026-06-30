# GPEEDS-XX · [BE] Autenticação com HUB Interplayers — Generate Token

**Épico:** Integração PBM — Interplayers  
**Tipo:** Tarefa | **Story Points:** 2 | **Prioridade:** Highest  
**Referência técnica:** `docs/pbm_interplayers/`

---

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
