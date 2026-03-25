# Robbin Pay API — Agent Instructions

> Copie este arquivo para a raiz do seu projeto. Agentes AI (Claude Code, Cursor, Copilot, Codex) carregam automaticamente.

## Overview

Robbin Pay é uma API REST de financiamento parcelado B2B. Você envia um QR Code Pix e condições de parcelamento, a Robbin paga o fornecedor à vista e parcela a cobrança pro comprador em até 6x sem juros.

- **Auth:** OAuth 2.0 Client Credentials (server-to-server)
- **Base URL:** via env var `ROBBIN_BASE_URL`
- **Namespace:** via env var `ROBBIN_PARTNER`
- **Docs:** https://docs.robbin.com.br

## Environment Variables

```
ROBBIN_BASE_URL=https://bff-partner.io.robbin.com.br
ROBBIN_CLIENT_ID=<from Robbin team>
ROBBIN_CLIENT_SECRET=<from Robbin team>
ROBBIN_PARTNER=<your partner slug>
```

## Workflow

```
1. POST /oauth/token           → access_token (cache, renew before expiry)
2. GET  /api/v1/{partner}/card-limits/{taxId}  → availableLimit
3. POST /api/v1/{partner}/payments/pix-pay     → financingId, status
4. Receive webhook              → payment.succeeded | payment.failed
```

Always execute steps 1→2→3 in order. Never skip step 2.

## Auth

```
POST /oauth/token
Content-Type: application/json

{
  "client_id": "${ROBBIN_CLIENT_ID}",
  "client_secret": "${ROBBIN_CLIENT_SECRET}",
  "grant_type": "client_credentials"
}

→ { "access_token": "eyJ...", "expires_in": 3600 }
```

Cache the token. Renew 60 seconds before `expires_in`. No refresh token exists — request a new one.

All subsequent requests: `Authorization: Bearer {access_token}`

## Card Limits

```
GET /api/v1/{partner}/card-limits/{taxId}
Authorization: Bearer {token}

→ { "taxId": "12345678000190", "totalLimit": 100000.00, "usedLimit": 30000.00, "availableLimit": 70000.00 }
```

- `taxId`: CNPJ, digits only, 14 chars
- Values in BRL (reais, not centavos)
- 404 = buyer not registered or not approved

## Pix Pay

```
POST /api/v1/{partner}/payments/pix-pay
Authorization: Bearer {token}
Content-Type: application/json

{
  "pixPayload": "<EMV/BRCode QR string>",
  "taxId": "12345678000190",
  "totalAmount": 10000.00,
  "installments": 3,
  "installmentAmounts": [3333.34, 3333.33, 3333.33],
  "metadata": { "orderId": "ORD-001" }
}

→ 201 { "financingId": "fin_abc123", "status": "processing" }
```

### Field rules

| Field | Type | Rules |
|-------|------|-------|
| pixPayload | string | EMV/BRCode from QR. Must be valid and not expired. |
| taxId | string | CNPJ, 14 digits, no punctuation |
| totalAmount | number | BRL, min 0.01. NOT centavos. |
| installments | integer | 1 to 6 |
| installmentAmounts | number[] | Length must equal `installments`. Sum must equal `totalAmount`. |
| metadata | object? | Free-form. Include `orderId` for reconciliation. |

### Centavo adjustment for uneven splits

```python
# 3x R$ 10.000,00
amount = 10000.00
n = 3
base = round(amount / n, 2)          # 3333.33
first = round(amount - base * (n-1), 2)  # 3333.34
installments = [first] + [base] * (n-1)
# [3333.34, 3333.33, 3333.33] — sum = 10000.00 ✓
```

## Error Handling

All errors: `{ "title": "snake_case_code", "message": "Human-readable" }`

Use `title` for code logic. Never parse `message`.

| title | HTTP | Action |
|-------|------|--------|
| invalid_credentials | 401 | Check client_id/secret. Sandbox ≠ production. |
| unauthorized | 401 | Token expired → renew and retry once. |
| customer_not_found | 404 | CNPJ not registered → confirm with user. |
| insufficient_limit | 422 | Check /card-limits, reduce amount. |
| invalid_pix_payload | 422 | QR expired → request new QR from supplier. |
| installments_mismatch | 422 | Array length ≠ installments field. |
| amount_mismatch | 422 | Sum of installmentAmounts ≠ totalAmount. |
| internal_error | 500 | Retry 3x with backoff: 1s, 2s, 4s. |

## Boundaries

### Always

- Use env vars for credentials and URLs — never hardcode
- Call `/card-limits` before `/pix-pay`
- Validate `sum(installmentAmounts) == totalAmount` before sending
- Validate `len(installmentAmounts) == installments` before sending
- Cache OAuth token, renew before expiry
- Retry 5xx with exponential backoff (max 3 attempts)
- Include `orderId` in metadata for reconciliation
- Log full error response body for debugging

### Ask the user first

- Which `{partner}` slug to use
- Whether to target sandbox or production
- Number of installments (1-6)
- The QR Code Pix payload (comes from the supplier)

### Never

- Never expose `client_secret` in frontend, mobile, or SPA code
- Never send `installments` > 6
- Never send `totalAmount` ≤ 0
- Never retry 4xx errors (except 401 → renew token once)
- Never parse error `message` for logic — use `title`
- Never call `/pix-pay` without checking `/card-limits` first
- Never hardcode the base URL — always use env var
