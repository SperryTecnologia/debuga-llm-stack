# debuga-llm-stack

**Arquitetura pública e documentação técnica da estratégia LLM híbrida da debuga.ai.**

Desenvolvida por [Sperry Tecnologia](https://www.sperrytecnologia.com.br).

---

## O que é

Este repositório documenta a arquitetura de inferência LLM da plataforma [debuga.ai](https://github.com/SperryTecnologia/debuga-ai). Ele registra decisões técnicas, comparações de modelos, estratégias de roteamento e configurações de infraestrutura para inferência híbrida local/cloud.

Este é um repositório de **documentação técnica e pesquisa**, não contém código de produção.

---

## Status

| Aspecto | Classificação |
|---------|--------------|
| Tipo | Documentação pública |
| Código de produção | Não incluso |
| Uso | Referência técnica e pesquisa |
| Atualização | Contínua |

---

## Como se conecta à debuga.ai

A debuga.ai utiliza uma arquitetura de inferência em camadas. Este repositório documenta a estratégia e as decisões por trás dessa arquitetura:

```
┌─────────────────────────────────────────────┐
│           debuga.ai (aplicação)             │
├─────────────────────────────────────────────┤
│         Camada de Roteamento LLM            │
│  ┌─────────────┐    ┌──────────────────┐   │
│  │  GPU Local   │    │  Providers Cloud  │   │
│  │  (Ollama)    │    │  (OpenAI, etc.)   │   │
│  └─────────────┘    └──────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## Arquitetura Conceitual

### Inferência Local (GPU)

A plataforma suporta inferência local via Ollama com GPU NVIDIA dedicada. Benefícios:

- Latência reduzida (sem round-trip para API externa)
- Custo zero por token após investimento em hardware
- Dados sensíveis não saem do ambiente do operador
- Independência de disponibilidade de providers externos

Modelos avaliados para o domínio técnico:

| Modelo | Parâmetros | Caso de uso | VRAM |
|--------|-----------|-------------|------|
| Qwen 2.5 7B Instruct | 7B | Uso geral, raciocínio | 6 GB |
| Qwen 2.5 Coder 7B | 7B | Geração de código, scripts | 6 GB |
| Qwen 2.5 14B Instruct | 14B | Raciocínio complexo | 12 GB |
| DeepSeek Coder V2 Lite | 16B | Código e automação | 12 GB |

### Fallback Cloud

Quando a GPU local está indisponível ou em cold start, o sistema aciona automaticamente um provider cloud:

| Provider | Modelo padrão | Caso de uso |
|----------|--------------|-------------|
| OpenAI | gpt-4o-mini | Fallback geral |
| Anthropic | Claude 3.5 Sonnet | Raciocínio longo |
| Google Gemini | Gemini 2.5 Flash | Custo-benefício |
| OpenRouter | Variável | Acesso a múltiplos modelos |

### Roteamento por Capacidade

O roteamento inteligente direciona consultas com base em:

- **Complexidade** — Tarefas simples para modelos menores, complexas para maiores
- **Domínio** — Código para modelos code-specialized, texto para general-purpose
- **Latência** — Prioridade para GPU local quando disponível
- **Custo** — Respeita limites diários/mensais configurados pelo operador
- **Disponibilidade** — Fallback automático em caso de falha

---

## Logs e Auditoria

A arquitetura prevê logging estruturado de todas as inferências:

| Campo | Descrição |
|-------|-----------|
| Timestamp | Momento da requisição |
| Provider | Local ou cloud (qual) |
| Modelo | Modelo utilizado |
| Tokens (in/out) | Consumo de tokens |
| Latência | Tempo de resposta |
| Motivo do roteamento | Por que aquele provider foi escolhido |
| Custo estimado | Para providers cloud |

---

## Custos e Limites

O sistema implementa controle de custos em múltiplas camadas:

- Limite diário em USD (configurável)
- Limite mensal em USD (configurável)
- Alerta ao atingir threshold (ex: 80%)
- Bloqueio automático ao atingir limite
- Relatório de consumo por usuário/plano

---

## Roadmap Técnico

| Item | Status | Prioridade |
|------|--------|-----------|
| Ollama com GPU local | Produção | — |
| Fallback multi-provider | Produção | — |
| Roteamento por capacidade | Produção | — |
| Controle de custos | Produção | — |
| vLLM para alta concorrência | Pesquisa | Média |
| Fine-tuning para domínio técnico | Planejado | Baixa |
| Gateway dedicado | Pesquisa | Baixa |
| Embedding local para RAG | Planejado | Média |

---

## Uso Previsto

Este repositório é destinado a:

- Equipes técnicas avaliando a arquitetura LLM da debuga.ai
- Operadores planejando infraestrutura de GPU
- Pesquisadores comparando modelos para domínio técnico
- Parceiros avaliando a estratégia de inferência

---

## Limitações

- Este repositório não contém código de produção
- Benchmarks são indicativos e dependem do hardware específico
- Modelos e providers podem mudar conforme evolução do mercado
- Configurações de produção estão no repositório privado

---

## Licença

Documentação pública sob licença MIT. O código de produção da plataforma é privado.

---

## Sperry Tecnologia

Desenvolvido por [Sperry Tecnologia](https://www.sperrytecnologia.com.br) — infraestrutura, segurança, DevOps e automação com IA.
