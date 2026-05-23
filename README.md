# debuga-llm-stack

**Arquitetura completa da stack de inferência LLM híbrida (GPU local + Cloud) da plataforma [debuga.ai](https://debuga.ai).**

Desenvolvido por [Sperry Tecnologia](https://www.sperrytecnologia.com.br).

---

## Visão Geral

Este repositório documenta a estratégia de inferência multi-camada da debuga.ai: modelos open-source rodando em GPU local (custo zero, dados locais) com fallback automático para providers cloud premium. O resultado é uma plataforma que combina privacidade, performance e qualidade de resposta.

```mermaid
flowchart TB
    subgraph Aplicação["debuga.ai — Aplicação"]
        APP[Backend + Agente Conversacional]
    end

    subgraph Stack["LLM Stack — Inferência Híbrida"]
        direction TB
        ROUTER{{"Router Engine<br/>Decisão Inteligente"}}

        subgraph Local["Camada Local (GPU)"]
            direction LR
            VLLM["vLLM Engine<br/>Continuous Batching"]
            OLLAMA["Ollama<br/>Quick Inference"]
        end

        subgraph Cloud["Camada Cloud (Fallback)"]
            direction LR
            OAI["OpenAI<br/>GPT-4o"]
            ANT["Anthropic<br/>Claude 3.5"]
            GEM["Google<br/>Gemini 2.5"]
            DS["DeepSeek<br/>V3"]
        end
    end

    subgraph Observabilidade["Observabilidade"]
        PROM["Prometheus"]
        GRAF["Grafana"]
    end

    APP --> ROUTER
    ROUTER -->|"Prioridade 1<br/>Custo zero"| Local
    ROUTER -->|"Prioridade 2<br/>Fallback"| Cloud
    VLLM --> PROM
    OLLAMA --> PROM
    PROM --> GRAF
```

---

## Princípios de Design

| Princípio | Implementação |
|-----------|---------------|
| **Privacidade** | Dados processados localmente por padrão — nunca saem do ambiente sem necessidade |
| **Custo zero** | GPU local elimina custo de inferência para 80%+ das requisições |
| **Resiliência** | Fallback automático transparente — usuário nunca percebe indisponibilidade |
| **Qualidade** | Modelos cloud premium para tarefas que exigem raciocínio superior |
| **Observabilidade** | Métricas de latência, tokens, custo e erros em tempo real |

---

## Arquitetura de Camadas

```mermaid
graph TB
    subgraph Camada1["Camada 1 — GPU Local (Prioridade Máxima)"]
        direction LR
        M1["Qwen 2.5 72B<br/>Chat técnico"]
        M2["Qwen Coder 7B<br/>Geração de código"]
        M3["DeepSeek V3<br/>Raciocínio"]
    end

    subgraph Camada2["Camada 2 — Cloud Premium (Fallback Primário)"]
        direction LR
        M4["GPT-4o<br/>Raciocínio complexo"]
        M5["Claude 3.5 Sonnet<br/>Documentação"]
        M6["GPT-Image-1<br/>Geração de imagem"]
    end

    subgraph Camada3["Camada 3 — Cloud Balanced (Custo-Benefício)"]
        direction LR
        M7["Gemini 2.5 Flash<br/>Multimodal rápido"]
        M8["DeepSeek Cloud<br/>Código"]
    end

    Camada1 -->|"GPU indisponível<br/>ou tarefa especializada"| Camada2
    Camada2 -->|"Provider indisponível<br/>ou custo excedido"| Camada3
```

| Camada | Latência | Custo/1M tokens | Privacidade | Uso |
|--------|----------|-----------------|-------------|-----|
| Local (GPU) | < 2s | $0 | Total | 80% das requisições |
| Cloud Premium | 2-5s | $3-15 | Zero-retention | Tarefas complexas |
| Cloud Balanced | 1-3s | $0.10-1 | Zero-retention | Fallback econômico |

---

## Roteamento Inteligente

O Router Engine decide qual modelo utilizar com base em múltiplos critérios ponderados:

```mermaid
flowchart LR
    subgraph Input["Análise da Requisição"]
        A["Tipo de Tarefa"]
        B["Complexidade"]
        C["Contexto (tokens)"]
        D["Histórico"]
    end

    subgraph Infra["Estado da Infraestrutura"]
        E["GPU Health"]
        F["Custo Acumulado"]
        G["Latência Atual"]
    end

    subgraph Engine["Score Engine"]
        H{{"Cálculo<br/>Ponderado"}}
    end

    subgraph Output["Decisão"]
        I["Provider"]
        J["Model ID"]
        K["Fallback Chain"]
    end

    A --> H
    B --> H
    C --> H
    D --> H
    E --> H
    F --> H
    G --> H
    H --> I
    H --> J
    H --> K
```

| Critério | Peso | Descrição |
|----------|------|-----------|
| Disponibilidade GPU | **Alto** | Health check da GPU em tempo real |
| Complexidade da tarefa | **Alto** | Classificação automática por intent |
| Tipo de conteúdo | Médio | Código, texto, imagem, áudio, diagrama |
| Custo acumulado | Médio | Dentro dos limites configurados pelo operador |
| Tamanho do contexto | Médio | Modelos locais têm limite de contexto |
| Latência | Baixo | Tempo de resposta aceitável para o tipo de tarefa |

---

## Mapeamento Tarefa → Modelo

```mermaid
graph LR
    subgraph Tarefas["Classificação de Intenção"]
        T1["Chat Técnico"]
        T2["Geração de Código"]
        T3["Análise de Imagem"]
        T4["Geração de Imagem"]
        T5["Transcrição"]
        T6["Documentação Longa"]
    end

    subgraph Modelos["Modelo Selecionado"]
        Q72["Qwen 72B<br/>(local)"]
        QC["Qwen Coder<br/>(local)"]
        GPT4V["GPT-4o Vision<br/>(cloud)"]
        GIMG["GPT-Image-1<br/>(cloud)"]
        WHIS["Whisper V3<br/>(cloud)"]
        CL35["Claude 3.5<br/>(cloud)"]
    end

    T1 --> Q72
    T2 --> QC
    T3 --> GPT4V
    T4 --> GIMG
    T5 --> WHIS
    T6 --> CL35
```

| Tarefa | Modelo Primário | Fallback | Justificativa |
|--------|----------------|----------|---------------|
| Chat técnico | Qwen 2.5 72B (local) | GPT-4o | Custo zero + qualidade |
| Geração de código | Qwen Coder 7B (local) | Claude 3.5 | Especializado |
| Raciocínio complexo | GPT-4o | Claude 3.5 | Melhor reasoning |
| Análise de imagem | GPT-4o Vision | Gemini 2.5 | Multimodal avançado |
| Geração de imagem | GPT-Image-1 | — | Qualidade superior |
| Transcrição | Whisper V3 | — | Único provider |
| Documentação longa | Claude 3.5 (200K ctx) | GPT-4o | Contexto extenso |

---

## Hardware de Referência

| Componente | Mínimo | Recomendado | Produção |
|-----------|--------|-------------|----------|
| GPU | RTX 3060 12GB | RTX 3090 24GB | A100 40GB+ |
| RAM | 32 GB | 64 GB | 128 GB |
| Storage | 100 GB SSD | 500 GB NVMe | 1+ TB NVMe |
| CPU | 8 cores | 16 cores | 32+ cores |
| Rede | 100 Mbps | 1 Gbps | 10 Gbps |

---

## Stack de Monitoramento

```mermaid
flowchart LR
    subgraph Coleta["Coleta de Métricas"]
        VLLM_M["vLLM<br/>/metrics"]
        APP_M["Aplicação<br/>Custom Metrics"]
    end

    subgraph Armazenamento["Armazenamento"]
        PROM["Prometheus<br/>Time Series DB"]
    end

    subgraph Visualização["Visualização"]
        GRAF["Grafana<br/>Dashboards"]
    end

    subgraph Alertas["Alertas"]
        ALERT["AlertManager<br/>Notificações"]
    end

    VLLM_M --> PROM
    APP_M --> PROM
    PROM --> GRAF
    PROM --> ALERT
```

| Métrica | Descrição | Alerta |
|---------|-----------|--------|
| `vllm_request_latency` | Latência P50/P95/P99 | > 10s P95 |
| `vllm_gpu_utilization` | Uso de VRAM | > 95% |
| `vllm_tokens_per_second` | Throughput de geração | < 20 tok/s |
| `inference_cost_usd` | Custo acumulado cloud | > limite/dia |
| `fallback_rate` | Taxa de fallback para cloud | > 30% |
| `error_rate` | Taxa de erro por provider | > 5% |

---

## Quick Start (Laboratório)

```bash
# 1. Clone o repositório
git clone https://github.com/SperryTecnologia/debuga-llm-stack.git
cd debuga-llm-stack

# 2. Configure variáveis
cp .env.example .env
# Edite .env com seu HF_TOKEN e modelo desejado

# 3. Suba a stack
docker compose up -d

# 4. Verifique saúde
curl http://localhost:8000/health

# 5. Teste inferência
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "Qwen/Qwen2.5-Coder-7B-Instruct", "messages": [{"role": "user", "content": "Hello"}]}'

# 6. (Opcional) Suba monitoramento
docker compose --profile monitoring up -d
```

---

## Estrutura do Repositório

```
debuga-llm-stack/
├── docker-compose.yml       # Stack completa (vLLM + monitoramento)
├── monitoring/
│   └── prometheus.yml       # Configuração de scraping
├── diagrams/
│   ├── stack-overview.mmd   # Diagrama da arquitetura
│   └── routing-flow.mmd     # Fluxo de roteamento
├── docs/
│   ├── architecture.md      # Decisões de arquitetura
│   ├── hardware-guide.md    # Guia de hardware
│   ├── integration-vision.md # Visão de integração
│   └── lab-guide.md         # Guia de laboratório
└── README.md
```

---

## Repositórios Relacionados

| Repositório | Descrição |
|-------------|-----------|
| [debuga-ai](https://github.com/SperryTecnologia/debuga-ai) | Plataforma principal |
| [debuga-qwen-coder-lab](https://github.com/SperryTecnologia/debuga-qwen-coder-lab) | Avaliação de modelos para code generation |
| [debuga-vllm-engine](https://github.com/SperryTecnologia/debuga-vllm-engine) | Serving local com vLLM |
| [debuga-llm-gateway](https://github.com/SperryTecnologia/debuga-llm-gateway) | Gateway OpenAI-compatible |

---

## Licença

Documentação e configurações de laboratório sob licença MIT. O código de produção da plataforma é mantido em repositório privado.

---

*Sperry Tecnologia — Infraestrutura, segurança, DevOps e automação com IA.*
