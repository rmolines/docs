# Robbin Pay Docs — Plano de Execução

> Doc de API para clientes enterprise (dev humano + agente AI).
> Inspiração: Resend, Vercel Agent Resources, Stripe.
> Plataforma alvo: Mintlify (Hobby, free tier).

---

## Fase 0 — Setup e fundação ✅
**Escopo:** Estrutura do projeto, config Mintlify, tema, navegação.
**Entrega:** Repo funcional com Mintlify rodando local, nav completa, placeholder pages.

### Tarefas
- [ ] Inicializar projeto Mintlify (`npx mintlify@latest init`)
- [ ] Configurar `mint.json` (nome, logo, cores Robbin, nav structure)
- [ ] Criar estrutura de diretórios (auth/, payments/, credit/, guides/)
- [ ] Placeholder para todas as páginas (título + "em construção")
- [ ] Validar build local (`npx mintlify dev`)

### Precisa de pesquisa/info
- 🔍 **Logo da Robbin** (SVG dark/light) — pedir ao usuário ou extrair
- 🔍 **Cores da marca** — extrair do briefing ou pedir ao usuário
- 🔍 **Custom domain** — ex: `docs.robbin.com.br` — confirmar com usuário
- 🔍 **Mintlify CLI versão atual** — checar docs na hora do setup

### Tamanho estimado: ~30 min

---

## Fase 1 — Introduction + Quickstart ✅
**Escopo:** As 2 páginas que o cliente vê primeiro. Precisam convencer em 2 min.
**Entrega:** Introduction com proposta de valor + Quickstart funcional (OAuth → Pix Pay → resposta).

### Tarefas
- [ ] `introduction.mdx` — O que é Robbin Pay, para quem, como funciona (diagrama simplificado)
- [ ] `quickstart.mdx` — Passo a passo completo:
  1. Obter credenciais (client_id + client_secret)
  2. Autenticar via OAuth (`POST /oauth/token`)
  3. Executar análise de crédito (`POST /cantu/cards/credit-analysis`)
  4. Fazer Pix Pay parcelado (`POST /cantu/payments/pix-pay`)
  5. Ver resposta (financingId, status)
- [ ] Exemplos em cURL + Python + Node.js
- [ ] Dados de sandbox (CNPJ de teste, QR code de exemplo)

### Precisa de pesquisa/info
- ❓ **Fluxo real de sandbox** — como o cliente obtém credenciais de teste? Dashboard? Email?
- ❓ **Dados de teste** — CNPJs, QR codes Pix que funcionam no sandbox
- ❓ **Sequência obrigatória** — precisa rodar credit-analysis antes de pix-pay? Ou são independentes?
- ❓ **Response real** — exemplos de response body de cada endpoint (happy path + erro)
- 🔍 **Mintlify code tabs syntax** — checar docs para tabs multi-linguagem

### Tamanho estimado: ~1h

---

## Fase 2 — Authentication ✅
**Escopo:** Toda a parte de auth documentada: OAuth, session, refresh, reset, sandbox vs prod.
**Entrega:** Seção completa de auth com exemplos copy-paste.

### Tarefas
- [ ] `authentication/oauth.mdx` — Client credentials flow (server-to-server)
  - Request/response completo
  - Scopes disponíveis (se houver)
  - Token expiration e refresh strategy
- [ ] `authentication/session.mdx` — Signin + refresh token (se for exposto ao cliente)
- [ ] `authentication/environments.mdx` — Sandbox vs Production
  - URLs diferentes? Mesma URL com keys diferentes?
  - Dados de teste (CNPJs, valores)
  - Limitações do sandbox
- [ ] Tabela comparativa: quando usar OAuth vs Session

### Precisa de pesquisa/info
- ❓ **OAuth scopes** — existem scopes diferentes? Ou é tudo-ou-nada?
- ❓ **Session auth é público?** — o signin é para o portal do parceiro ou para o app do lojista?
- ❓ **Sandbox URL** — é a mesma base URL ou diferente?
- ❓ **Token TTL** — quanto tempo dura o access_token? E o refresh?
- ❓ **Rate limits por auth method** — existem?

### Tamanho estimado: ~45 min

---

## Fase 3 — Core: Pix Pay + Credit Analysis
**Escopo:** Os dois endpoints que são o coração do produto.
**Entrega:** Guias task-based + API reference detalhado.

### Tarefas
- [ ] `payments/pix-pay.mdx` — Guia completo do Pix Pay parcelado
  - Conceito: o que acontece por trás (CCB, Celcoin, FIDC)
  - Diagrama do fluxo (Mermaid ou imagem)
  - Request/response com todos os campos explicados
  - Exemplos: 1x, 3x, 6x — como mudam os installmentAmounts
  - Edge cases: valor mínimo? Máximo? Limite de parcelas?
- [ ] `payments/webhooks.mdx` — Status updates via webhook
  - Eventos disponíveis (payment.processing, payment.succeeded, etc.)
  - Payload de exemplo
  - Signature verification
  - Retry policy
- [ ] `credit/analysis.mdx` — Análise de crédito
  - Quando chamar (pré-requisito do Pix Pay?)
  - Request/response
  - Webhook de status (aprovado, negado, pendente)
  - Tempo médio de resposta
- [ ] `credit/card-limits.mdx` — Consulta de limites
  - Quando usar (antes de mostrar opções ao lojista)
  - Response com limite disponível

### Precisa de pesquisa/info
- ❓ **Webhook events** — lista completa de eventos e payloads
- ❓ **Webhook signature** — como é feita a verificação? HMAC? Header name?
- ❓ **installmentAmounts** — o parceiro calcula ou a Robbin retorna? Exemplo real
- ❓ **Valor mínimo/máximo** — limites do Pix Pay
- ❓ **Credit analysis timing** — síncrono ou assíncrono? Quanto tempo leva?
- ❓ **Status possíveis** — do financing (processing → ?)
- ❓ **Fluxo completo real** — diagrama atualizado de quem chama quem

### Tamanho estimado: ~1.5h

---

## Fase 4 — Endpoints secundários + API Reference ✅
**Escopo:** Demais endpoints + referência auto-gerada do OpenAPI.
**Entrega:** Cobertura 100% da API.

### Tarefas
- [ ] `campaigns/import-leads.mdx` — Import de leads (se for público pro parceiro)
- [ ] `dashboards/home.mdx` — Dashboard Metabase embeddable
- [ ] `api-reference/` — Configurar auto-geração via OpenAPI
  - Apontar Mintlify pro `openapi.json`
  - Revisar output gerado (nomes, descrições, exemplos)
  - Adicionar exemplos manuais onde o auto-gerado for fraco
- [ ] `errors.mdx` — Catálogo completo de erros
  - HTTP status codes usados
  - Error codes específicos (CantuErrorSchema)
  - Recovery actions para cada erro

### Precisa de pesquisa/info
- ❓ **Import leads é público?** — parceiro usa isso ou é interno Robbin?
- ❓ **Dashboard** — parceiro acessa? Como? Token Metabase?
- ❓ **Lista completa de error codes** — além do que está no OpenAPI
- 🔍 **Mintlify OpenAPI auto-gen** — checar config e limitações atuais

### Tamanho estimado: ~1h

---

## Fase 5 — Enterprise polish ✅
**Escopo:** Páginas que passam confiança para enterprise: segurança, compliance, SLA.
**Entrega:** Seção enterprise completa.

### Tarefas
- [ ] `security.mdx` — Práticas de segurança
  - Encryption (TLS, at rest)
  - Tokenização
  - IP allowlisting (se houver)
  - Audit logs
- [ ] `compliance.mdx` — Regulatório
  - LGPD (como dados são tratados)
  - Bacen (enquadramento)
  - CCB/FIDC (visão simplificada pro dev, não jurídica)
- [ ] `guides/integration-checklist.mdx` — Checklist de integração
  - Pré-requisitos (contrato, credenciais)
  - Steps técnicos
  - Go-live checklist
  - Contatos de suporte
- [ ] `changelog.mdx` — Template de changelog

### Precisa de pesquisa/info
- ❓ **SLA da API** — uptime garantido? Latência P99?
- ❓ **Suporte** — como o parceiro abre ticket? Email? Slack?
- ❓ **IP allowlisting** — existe? Como configurar?
- ❓ **Audit logs** — parceiro tem acesso?
- ❓ **Go-live process** — o que precisa acontecer pra sair do sandbox?

### Tamanho estimado: ~1h

---

## Fase 6 — AI Agent Layer ✅
**Escopo:** Documentação otimizada para agentes AI (Claude Code, Cursor, Copilot).
**Entrega:** llms.txt + llms-full.txt + AGENTS.md.

### Tarefas
- [ ] `llms.txt` — Índice machine-readable
  - H1: Robbin Pay API
  - Blockquote: resumo do produto
  - Seções: Auth, Payments, Credit, Webhooks
  - Links para cada página .md
- [ ] `llms-full.txt` — Doc completa em markdown flat
  - Compilar todas as páginas em arquivo único
  - Otimizar para token efficiency (sem duplicação, sem boilerplate)
  - Incluir schemas JSON inline
- [ ] `AGENTS.md` — Instruções procedurais
  - "Use OAuth client_credentials para server-to-server, NUNCA session"
  - "Amounts em centavos (R$ 100,00 = 10000)"
  - "Sempre rode credit-analysis antes de pix-pay"
  - Workflow completo: auth → credit → pay → webhook
  - Error recovery: "Se 401 → refresh. Se 422 → checar limite"
  - Boundaries: Always/Ask/Never
- [ ] Testar com Claude Code (apontar llms.txt e ver se agente consegue integrar)

### Precisa de pesquisa/info
- 🔍 **Mintlify llms.txt** — como configurar geração automática
- 🔍 **AGENTS.md spec atual** — verificar formato mais recente
- 🔍 **Exemplos fintech** — como Stripe/Plaid estruturam agent docs

### Tamanho estimado: ~1h

---

## Fase 7 — Review e deploy ✅ (review done, deploy pending)
**Escopo:** QA, ajustes finais, deploy.
**Entrega:** Docs live em domínio customizado.

### Tarefas
- [ ] Review completo de conteúdo (consistência, tom, exemplos)
- [ ] Testar todos os exemplos de código (cURL funciona? Responses batem?)
- [ ] Testar playground com sandbox real
- [ ] SEO basics (titles, descriptions, og:image)
- [ ] Deploy Mintlify (push to GitHub → auto-deploy)
- [ ] Configurar custom domain
- [ ] Compartilhar com os 2 clientes

### Precisa de pesquisa/info
- 🔍 **Mintlify deploy** — verificar processo atual (GitHub integration)
- ❓ **Domínio** — docs.robbin.com.br? Outro?
- ❓ **Acesso** — docs públicas ou com auth?

### Tamanho estimado: ~45 min

---

## Legenda

| Símbolo | Significado |
|---------|-------------|
| ❓ | Info que preciso do usuário (Robbin team) |
| 🔍 | Pesquisa web necessária (docs atualizadas, specs, exemplos) |
| ✅ | Tenho info suficiente pra executar |

---

## Resumo de fases

| Fase | Entrega | Tempo est. | Deps |
|------|---------|-----------|------|
| 0 | Setup Mintlify | 30 min | Logo, cores |
| 1 | Intro + Quickstart | 1h | Fluxo sandbox, dados teste |
| 2 | Authentication | 45 min | Scopes, TTL, sandbox URL |
| 3 | Core (Pix Pay + Credit) | 1.5h | Webhooks, payloads reais |
| 4 | Endpoints + API Ref | 1h | OpenAPI atualizado |
| 5 | Enterprise polish | 1h | SLA, compliance info |
| 6 | AI Agent Layer | 1h | — (construído sobre fases anteriores) |
| 7 | Review + Deploy | 45 min | Domínio, acesso |

**Total estimado: ~7-8 horas de trabalho**
Executável em 2-3 sessões se as ❓ forem respondidas entre fases.

---

## Pendentes consolidados (TBD nos docs)

Itens marcados como TBD nos arquivos. Preencher quando tiver a info.

| Item | Arquivo | Status |
|------|---------|--------|
| Token TTL real (expires_in) | `authentication/oauth.mdx` | ⏳ pendente |
| Sandbox URL vs Prod URL (mesma ou diferente?) | `authentication/environments.mdx` | ⏳ pendente |
| Rate limits da API | `authentication/oauth.mdx` | ⏳ pendente |
| Dados de teste (CNPJ, QR Code Pix sandbox) | `authentication/environments.mdx` | ⏳ pendente |
| Webhook events reais + payloads | `payments/webhooks.mdx` | ⏳ pendente |
| Webhook signature verification | `payments/webhooks.mdx` | ⏳ pendente |
| Card-limits response schema real | `credit/card-limits.mdx` | ⏳ pendente |
| Lista completa de error codes | `api-reference/errors.mdx` | ⏳ pendente |
| SLA / uptime | `enterprise/security.mdx` | ⏳ pendente |
| Suporte (como abrir ticket) | `guides/integration-checklist.mdx` | ⏳ pendente |
| Go-live process | `guides/integration-checklist.mdx` | ⏳ pendente |
| Custom domain | deploy | ⏳ pendente |

Para buscar todos os TBDs no código: `grep -r "TBD\|pendente\|TODO Fase" --include="*.mdx" .`
