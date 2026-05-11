# debuga.ai LLM Stack

Documentação e arquitetura da stack de inferência LLM do [debuga.ai](https://debuga.ai) — agente autônomo de IA para TI e Segurança da Informação.

## Visão Geral

O **debuga.ai LLM Stack** é a camada de inferência planejada para oferecer inferência local e on-premise com modelos open-source otimizados para tarefas de DevOps, infraestrutura e segurança da informação.

A stack é baseada em [vLLM](https://github.com/vllm-project/vllm) como motor de inferência e compatível com modelos da família [Qwen2.5-Coder](https://huggingface.co/Qwen), projetados para geração de código e tarefas técnicas.

## Arquitetura

```
┌─────────────────────────────────────────────────────┐
│  debuga.ai (SaaS)                                   │
│       │                                             │
│       ▼                                             │
│  ┌───────────────────────┐                          │
│  │  debuga-llm-gateway   │  ← roteamento cloud/local│
│  │   ├─ cloud provider   │  ← fallback padrão       │
│  │   └─ vllm (local)     │  ← quando ativo          │
│  └───────────────────────┘                          │
│       │                                             │
│       ▼                                             │
│  ┌───────────────────────┐                          │
│  │  debuga-vllm-engine   │  ← Qwen-Coder models     │
│  └───────────────────────┘                          │
└─────────────────────────────────────────────────────┘
```

## Repositórios do Ecossistema

| Repositório | Função | Status |
|---|---|---|
| **debuga-llm-stack** (este) | Documentação central, arquitetura e Docker Compose de laboratório | Ativo |
| [debuga-vllm-engine](https://github.com/SperryTecnologia/debuga-vllm-engine) | Motor de inferência vLLM com configs para Qwen-Coder | Planejado |
| [debuga-qwen-coder-lab](https://github.com/SperryTecnologia/debuga-qwen-coder-lab) | Laboratório de avaliação e benchmarks | Planejado |
| [debuga-llm-gateway](https://github.com/SperryTecnologia/debuga-llm-gateway) | Gateway OpenAI-compatible com roteamento | Planejado |

## Quick Start (Laboratório Local)

```bash
# Clone este repositório
git clone https://github.com/SperryTecnologia/debuga-llm-stack.git
cd debuga-llm-stack

# Configure variáveis de ambiente
cp .env.example .env
# Edite .env com seu HuggingFace token

# Suba o ambiente de laboratório
docker compose up -d

# Verifique a saúde do serviço
curl http://localhost:8000/health
```

> **Nota:** O Docker Compose de laboratório é um exemplo genérico para experimentação local. Não representa a configuração de produção do debuga.ai.

## Documentação

| Documento | Descrição |
|---|---|
| [Arquitetura](docs/architecture.md) | Visão técnica da stack e fluxo de dados |
| [Guia de Laboratório](docs/lab-guide.md) | Passo a passo para montar o ambiente local |
| [Visão de Integração](docs/integration-vision.md) | Como a stack se conecta ao debuga.ai (alto nível) |
| [Guia de Hardware](docs/hardware-guide.md) | Requisitos de GPU e recomendações por modelo |

## Diagramas

- [Stack Overview](diagrams/stack-overview.mmd) — Visão geral dos componentes
- [Routing Flow](diagrams/routing-flow.mmd) — Fluxo de roteamento entre providers

## Tecnologias

- **Motor de inferência:** [vLLM](https://github.com/vllm-project/vllm) (Apache 2.0)
- **Modelos:** [Qwen2.5-Coder](https://huggingface.co/Qwen) (Apache 2.0)
- **Gateway:** API OpenAI-compatible com fallback automático
- **Monitoramento:** Prometheus + Grafana (exemplos genéricos)

## Atribuições

Este projeto utiliza e referencia software open-source de terceiros:

- **vLLM** é um projeto da [vLLM Team](https://github.com/vllm-project/vllm), licenciado sob Apache 2.0
- **Qwen2.5-Coder** é uma família de modelos criada pela [equipe Qwen/Alibaba](https://huggingface.co/Qwen), licenciada sob Apache 2.0
- **HuggingFace Transformers** é mantido pela [Hugging Face](https://huggingface.co), licenciado sob Apache 2.0

A Sperry Tecnologia não é autora desses projetos upstream. O debuga.ai LLM Stack é construído **com base em** (based on) essas tecnologias.

Veja [THIRD_PARTY_LICENSES](./THIRD_PARTY_LICENSES) para detalhes completos.

## Licença

Este repositório é licenciado sob [Apache License 2.0](./LICENSE).

## Aviso

Este repositório contém apenas documentação, exemplos genéricos e configurações de laboratório. Não contém secrets, dados de clientes, prompts proprietários ou configurações de produção do debuga.ai.

---

Desenvolvido por [Sperry Tecnologia](https://www.sperrytecnologia.com.br) — Tecnologia e Segurança da Informação
