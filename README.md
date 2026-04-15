# LFC Opções — WordPress Plugin

> Plugin companheiro do tema [**Fazenda Canoa**](https://github.com/rafaelruch/fazenda-canoa-theme). Centraliza opções de contato (WhatsApp, e-mail, horário, book), implementa CPT para captação de leads, endpoint AJAX seguro e webhook gancho pronto para n8n/RD Station/Zapier. Arquivos no **root** do repo — deploy direto via Deployer for Git.

**Plugin Name:** Fazenda Canoa — Opções & Leads
**Versão:** 1.0.0
**Autor:** [RUCH](https://ruch.digital)

---

## 🚀 Deploy via Deployer for Git

1. Admin WordPress → **Deployer for Git** → Adicionar
2. **URL do repositório:** `https://github.com/rafaelruch/lfc-opcoes-plugin.git`
3. **Pasta destino:** `wp-content/plugins/lfc-opcoes`
4. **Branch:** `main`
5. Salvar e executar

Depois:
- **Plugins** → ativar **"Fazenda Canoa — Opções & Leads"**
- **Configurações → Fazenda Canoa** → preencher contatos + (opcional) webhook/GSC/GA4/Pixel

## 📁 Estrutura

```
lfc-opcoes/              # === ROOT do repo ===
├── lfc-opcoes.php       # Main (bootstrap + helpers + shortcodes)
├── admin/
│   └── options-page.php # Página em Configurações → Fazenda Canoa
├── includes/
│   ├── cpt-leads.php    # CPT lfc_lead (armazena leads no admin)
│   └── lead-endpoint.php # Endpoint AJAX + webhook dispatcher
└── assets/              # (vazio; placeholder para futuros assets do admin)
```

## ⚙️ O que o plugin entrega

### 1. Página de opções em `Configurações → Fazenda Canoa`

Contatos comerciais:
- WhatsApp (formato `55DDDNNNNNNNN`)
- E-mail
- Telefone fixo
- Horário de atendimento
- Mensagem padrão do WhatsApp

Book do empreendimento:
- URL do PDF do book (enviado no WhatsApp após captação)

Integração (webhook):
- URL do webhook (n8n/RD/Zapier/Make)
- Secret token (enviado no header `X-LFC-Secret`)

SEO e Analytics:
- Google Search Console — código de verificação
- Google Analytics 4 — ID de medição (`G-XXXXXXXXXX`)
- Meta Pixel ID (Facebook Ads)

### 2. CPT `lfc_lead` (Leads no admin)

Cada lead capturado vira um post do tipo `lfc_lead`:
- Listagem com colunas: Nome · WhatsApp · E-mail · Interesse · Origem · Data
- WhatsApp clicável (abre conversa direta)
- Metadados: IP, user-agent, referrer, contexto da captação, source (modal/book/consultor-form)

### 3. Endpoint AJAX `admin-ajax.php?action=lfc_submit_lead`

- Valida nonce `lfc_lead`
- Honeypot anti-bot (campo `website`)
- Sanitização server-side de todos os campos
- Cria post `lfc_lead`
- Dispara webhook (se configurado) em modo `blocking => false`

### 4. Helper functions e shortcodes

```php
lfc_get_options()           // Array com todas as opções
lfc_whatsapp_url($msg = '') // URL wa.me pronta

[lfc_whatsapp]              // Formata o número: (62) 99999-9999
[lfc_email]                 // E-mail salvo
[lfc_horario]               // Horário de atendimento
```

### 5. Output automático no `<head>` quando preenchido

- GSC verification meta tag
- GA4 gtag script (com `anonymize_ip`)
- Meta Pixel (com noscript fallback)

## 🔄 Fluxo de captação

```
Front-end preenche modal
       ↓
POST admin-ajax.php (com nonce + honeypot)
       ↓
Plugin valida + sanitiza
       ↓
Cria post lfc_lead (visível em Admin → Leads)
       ↓
Se webhook configurado → POST JSON assíncrono
       ↓
Retorna success ao front-end
       ↓
Front-end abre WhatsApp com mensagem pré-preenchida
```

## 🔐 Segurança

- Nonce em todos os submits (`lfc_lead`)
- Honeypot field `website` (invisible) anti-bot
- `sanitize_text_field`, `sanitize_email`, `esc_url_raw` em tudo
- CPT `lfc_lead` com `create_posts => do_not_allow` (só via endpoint)
- Webhook com `X-LFC-Secret` header para validação

## 📦 Payload do webhook

Quando configurado, cada lead dispara:

```json
{
  "lead_id": 123,
  "nome": "João Silva",
  "telefone": "(62) 9 0000-0000",
  "email": "joao@email.com",
  "interesse": "Lote frente-lago",
  "contexto": "hero",
  "source": "modal",
  "timestamp": "2026-04-15T16:34:00-03:00",
  "site": "https://lago.fazendacanoa.com.br"
}
```

Com header: `X-LFC-Secret: <seu-token>`

## 🔧 Dependências

- WordPress **6.5+**
- PHP **7.4+**
- Funciona standalone, mas é companheiro do tema [fazenda-canoa-theme](https://github.com/rafaelruch/fazenda-canoa-theme)

## 📝 Licença

GPL-2.0-or-later

---

Desenvolvido por [RUCH](https://ruch.digital).
