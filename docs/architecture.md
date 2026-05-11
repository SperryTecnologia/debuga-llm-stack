# Arquitetura — debuga.ai LLM Stack

## Visão Geral

A stack de inferência do debuga.ai segue uma arquitetura em camadas que separa roteamento, inferência e avaliação de modelos. O objetivo é permitir transição gradual de providers cloud para inferência local sem impactar o SaaS em produção.

## Componentes

### 1. LLM Gateway (Camada de Roteamento)

O gateway é o ponto de entrada único para todas as requisições de inferência. Ele implementa a API OpenAI-compatible (`/v1/chat/completions`) e decide para qual provider encaminhar cada requisição.

```
┌─────────────────────────────────────────────────┐
│                 LLM Gateway                      │
│                                                  │
│  Request → Auth → Router → Provider → Response   │
│                     │                            │
│                     ├─ Cloud Provider (padrão)    │
│                     └─ vLLM Local (feature flag)  │
│                                                  │
│  Feature Flags:                                  │
│  - ENABLE_LOCAL_INFERENCE=false (padrão)         │
│  - PREFERRED_PROVIDER=cloud|local|auto           │
└─────────────────────────────────────────────────┘
```

**Responsabilidades:**
- Roteamento inteligente entre providers
- Fallback automático (se local falha, usa cloud)
- Health check dos providers
- Logging de latência e throughput
- Rate limiting por API key

### 2. vLLM Engine (Camada de Inferência)

O motor de inferência local utiliza vLLM com PagedAttention para servir modelos Qwen-Coder de forma eficiente em GPU NVIDIA.

```
┌─────────────────────────────────────────────────┐
│               vLLM Engine                        │
│                                                  │
│  API OpenAI-compatible (:8000)                   │
│       │                                          │
│       ▼                                          │
│  ┌──────────────┐    ┌──────────────────┐        │
│  │ Scheduler    │───▶│ PagedAttention   │        │
│  │ (continuous  │    │ (memory-efficient)│        │
│  │  batching)   │    └──────────────────┘        │
│  └──────────────┘                                │
│       │                                          │
│       ▼                                          │
│  ┌──────────────┐                                │
│  │ Qwen-Coder   │  ← modelo carregado em GPU     │
│  │ (7B/14B/32B) │                                │
│  └──────────────┘                                │
│                                                  │
│  Metrics: /metrics (Prometheus-compatible)        │
└─────────────────────────────────────────────────┘
```

**Responsabilidades:**
- Servir modelo via API OpenAI-compatible
- Continuous batching para maximizar throughput
- PagedAttention para uso eficiente de VRAM
- Métricas Prometheus para monitoramento
- Health check endpoint

### 3. Qwen-Coder Lab (Camada de Avaliação)

O laboratório é responsável por avaliar, comparar e otimizar modelos antes de colocá-los em produção no engine.

**Responsabilidades:**
- Benchmark de modelos em tarefas DevOps/segurança
- Comparativo entre tamanhos (7B vs 14B vs 32B)
- Avaliação de impacto de quantização (FP16 vs AWQ vs GPTQ)
- Fine-tuning com LoRA para tarefas específicas
- Validação de qualidade antes de deploy

## Fluxo de Dados

```
┌──────────┐     ┌──────────────┐     ┌─────────────┐
│ debuga.ai│────▶│ LLM Gateway  │────▶│ vLLM Engine │
│  (SaaS)  │     │              │     │             │
│          │     │ PREFERRED=   │     │ Qwen-Coder  │
│          │     │  auto        │     │ 7B/14B/32B  │
└──────────┘     │              │     └─────────────┘
                 │   fallback   │
                 │      │       │
                 │      ▼       │
                 │ Cloud Provider│
                 └──────────────┘
```

1. O SaaS envia requisição ao gateway (formato OpenAI-compatible)
2. O gateway verifica a feature flag `ENABLE_LOCAL_INFERENCE`
3. Se ativo e `PREFERRED_PROVIDER=auto`:
   - Tenta vLLM local primeiro
   - Se falha (timeout, erro, indisponível) → fallback para cloud
4. Se desativado → encaminha direto para cloud provider
5. Resposta retorna ao SaaS no mesmo formato

## Decisões de Arquitetura

| Decisão | Escolha | Justificativa |
|---|---|---|
| Motor de inferência | vLLM | PagedAttention, continuous batching, API OpenAI-compatible nativa |
| Família de modelos | Qwen2.5-Coder | Apache 2.0, otimizado para código, bom desempenho em tarefas técnicas |
| Formato de API | OpenAI-compatible | Padrão de mercado, facilita troca de providers |
| Quantização | AWQ | Melhor trade-off qualidade/velocidade para GPU |
| Feature flag | Variável de ambiente | Zero-deploy para ativar/desativar, sem rebuild |
| Fallback | Automático | Garante disponibilidade mesmo com infra local instável |

## Segurança

- O gateway não expõe endpoints publicamente (rede interna apenas)
- Autenticação via API key entre serviços
- Nenhum dado de cliente é armazenado nos logs de inferência
- Modelos rodam em GPU dedicada sem acesso à internet
- Feature flag permite desligar inferência local instantaneamente

## Monitoramento

```
vLLM Engine ──metrics──▶ Prometheus ──▶ Grafana
     │                                      │
     └──health──▶ Gateway ──logs──▶ Alerting
```

Métricas coletadas:
- Latência por requisição (p50, p95, p99)
- Throughput (tokens/segundo)
- Utilização de GPU (%)
- Queue depth (requisições em espera)
- Taxa de fallback (local → cloud)
