# Visão de Integração — debuga.ai LLM Stack

Este documento descreve como a stack LLM se conecta ao debuga.ai em alto nível, sem expor detalhes de implementação do SaaS.

## Contexto

O debuga.ai é um agente autônomo de IA para TI e Segurança da Informação. Atualmente utiliza LLM cloud como motor de inferência. A stack LLM permite transição gradual para inferência local/on-premise.

## Estratégia de Integração

A integração segue o princípio de **zero-disruption**: o SaaS continua funcionando normalmente enquanto a stack local é preparada, testada e ativada de forma incremental.

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  FASE 1 (atual)         FASE 2 (transição)              │
│                                                         │
│  SaaS → Cloud LLM      SaaS → Gateway → Cloud LLM      │
│                                    │                    │
│                                    └─→ vLLM (feature    │
│                                         flag OFF)       │
│                                                         │
│  FASE 3 (híbrido)       FASE 4 (on-premise)             │
│                                                         │
│  SaaS → Gateway         SaaS → Gateway                  │
│           ├─ Cloud         ├─ vLLM (primário)            │
│           └─ vLLM          └─ Cloud (fallback)           │
│              (auto)                                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Fases de Integração

### Fase 1 — Estado Atual

O SaaS utiliza um provider cloud de LLM diretamente. Não há componente local.

### Fase 2 — Gateway como Proxy Transparente

O gateway é inserido entre o SaaS e o provider cloud, mas apenas repassa requisições. Isso permite:
- Validar que o gateway não introduz latência significativa
- Coletar métricas de uso para dimensionar infra local
- Testar fallback sem risco

A feature flag `ENABLE_LOCAL_INFERENCE=false` mantém o vLLM desligado.

### Fase 3 — Roteamento Híbrido

O gateway começa a rotear parte do tráfego para vLLM local:
- Requisições simples (baixa complexidade) → local
- Requisições complexas (contexto longo) → cloud
- Qualquer falha local → fallback automático para cloud

### Fase 4 — Local como Primário

O vLLM local se torna o provider primário, com cloud apenas como fallback para alta demanda ou indisponibilidade.

## Contrato de API

A integração é feita via API OpenAI-compatible. O SaaS não precisa saber se está falando com cloud ou local — o gateway abstrai essa decisão.

```
POST /v1/chat/completions
Content-Type: application/json
Authorization: Bearer <internal-api-key>

{
  "model": "qwen-coder",
  "messages": [...],
  "temperature": 0.7,
  "max_tokens": 2048,
  "stream": true
}
```

## Requisitos para Integração

| Requisito | Descrição |
|---|---|
| API compatível | Gateway deve expor `/v1/chat/completions` idêntico ao formato OpenAI |
| Streaming SSE | Suporte a `stream: true` com Server-Sent Events |
| Latência aceitável | p95 < 2s para primeiro token (TTFT) |
| Disponibilidade | 99.5%+ com fallback automático |
| Feature flag | Ativação/desativação sem deploy |
| Zero-downtime | Transição entre providers sem interrupção de serviço |

## O que NÃO Muda no SaaS

A integração da stack LLM não altera:
- Sistema de autenticação
- Billing e planos
- Contadores de uso
- Frontend/UI
- Ferramentas do agente (DNS, SSL, port scan, etc.)
- Banco de dados

A única alteração no SaaS é a URL do provider de LLM (variável de ambiente).
