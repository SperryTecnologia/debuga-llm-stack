# Guia de Laboratório — debuga.ai LLM Stack

Este guia descreve como montar um ambiente local de experimentação com vLLM e Qwen-Coder para tarefas de TI e segurança.

## Pré-requisitos

| Requisito | Mínimo | Recomendado |
|---|---|---|
| GPU NVIDIA | 1x com 24GB VRAM | 2x com 48GB VRAM |
| CUDA | 12.1+ | 12.4+ |
| RAM | 32GB | 64GB |
| Disco | 50GB livres | 100GB+ (para múltiplos modelos) |
| Docker | 24.0+ | Última versão estável |
| NVIDIA Container Toolkit | Instalado | Última versão |
| Sistema Operacional | Ubuntu 22.04 LTS | Ubuntu 24.04 LTS |

## Passo 1 — Instalar NVIDIA Container Toolkit

```bash
# Adicionar repositório NVIDIA
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update && sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

## Passo 2 — Configurar Variáveis de Ambiente

```bash
cp .env.example .env
```

Edite o arquivo `.env` com seus valores:

```env
HF_TOKEN=hf_xxxxxxxxxxxxxxxxxxxxxxxxx
VLLM_MODEL_ID=Qwen/Qwen2.5-Coder-7B-Instruct
VLLM_TENSOR_PARALLEL_SIZE=1
VLLM_GPU_MEMORY_UTILIZATION=0.90
```

> **Importante:** Nunca commite o arquivo `.env`. Ele está no `.gitignore`.

## Passo 3 — Subir o Ambiente

```bash
docker compose up -d
```

O Docker Compose irá:
1. Baixar a imagem vLLM com CUDA
2. Baixar o modelo do HuggingFace (primeira execução pode demorar)
3. Iniciar o servidor de inferência na porta 8000

## Passo 4 — Verificar Saúde

```bash
# Health check
curl http://localhost:8000/health

# Listar modelos disponíveis
curl http://localhost:8000/v1/models
```

## Passo 5 — Testar Inferência

```bash
curl http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen2.5-Coder-7B-Instruct",
    "messages": [
      {"role": "system", "content": "Você é um especialista em infraestrutura de TI."},
      {"role": "user", "content": "Escreva um script bash para verificar a saúde de um servidor Linux."}
    ],
    "temperature": 0.7,
    "max_tokens": 1024
  }'
```

## Passo 6 — Monitoramento (Opcional)

Se habilitado no Docker Compose, acesse:

- **Prometheus:** http://localhost:9090
- **Grafana:** http://localhost:3200 (admin/admin)

Métricas disponíveis do vLLM:
- `vllm:num_requests_running` — requisições em processamento
- `vllm:num_requests_waiting` — requisições na fila
- `vllm:gpu_cache_usage_perc` — uso do cache de GPU
- `vllm:avg_generation_throughput_toks_per_s` — throughput médio

## Troubleshooting

| Problema | Causa Provável | Solução |
|---|---|---|
| `CUDA out of memory` | Modelo grande demais para a GPU | Reduza `GPU_MEMORY_UTILIZATION` ou use modelo menor |
| Download lento do modelo | Conexão ou HF rate limit | Verifique `HF_TOKEN` e conexão |
| Container reiniciando | GPU não detectada | Verifique `nvidia-smi` e NVIDIA Container Toolkit |
| Respostas lentas | Modelo sem quantização | Use versão AWQ/GPTQ do modelo |
| Porta 8000 ocupada | Outro serviço na porta | Altere `VLLM_PORT` no `.env` |

## Próximos Passos

Após validar o ambiente local:

1. Execute benchmarks com `debuga-qwen-coder-lab`
2. Compare modelos 7B vs 14B para seu caso de uso
3. Avalie se a latência atende seus requisitos
4. Configure o gateway para roteamento automático
