# Guia de Hardware — debuga.ai LLM Stack

Recomendações de hardware para rodar modelos Qwen-Coder com vLLM em diferentes cenários.

## Modelos Suportados

| Modelo | Parâmetros | VRAM (FP16) | VRAM (AWQ) | Contexto Max |
|---|---|---|---|---|
| Qwen2.5-Coder-7B-Instruct | 7B | 16GB | 8GB | 32K tokens |
| Qwen2.5-Coder-14B-Instruct | 14B | 30GB | 16GB | 32K tokens |
| Qwen2.5-Coder-32B-Instruct | 32B | 64GB | 24-32GB | 32K tokens |

## Cenários de Deploy

### Laboratório / Desenvolvimento

Para experimentação e testes iniciais.

| Componente | Especificação |
|---|---|
| GPU | 1x NVIDIA RTX 4090 (24GB) |
| CPU | 8+ cores |
| RAM | 32GB DDR5 |
| Disco | 500GB NVMe SSD |
| Modelo recomendado | Qwen2.5-Coder-7B-Instruct (AWQ) |
| Throughput esperado | ~30-50 tokens/s |
| Custo estimado | R$ 15.000-20.000 (hardware próprio) |

### Produção — Pequena Escala

Para atender 5-20 usuários simultâneos.

| Componente | Especificação |
|---|---|
| GPU | 1x NVIDIA A100 40GB ou 2x RTX 4090 |
| CPU | 16+ cores |
| RAM | 64GB DDR5 ECC |
| Disco | 1TB NVMe SSD |
| Modelo recomendado | Qwen2.5-Coder-14B-Instruct (AWQ) |
| Throughput esperado | ~60-100 tokens/s |
| Custo estimado (cloud) | $800-1.200/mês |

### Produção — Média Escala

Para atender 20-100 usuários simultâneos.

| Componente | Especificação |
|---|---|
| GPU | 2x NVIDIA A100 80GB ou 4x RTX 4090 |
| CPU | 32+ cores |
| RAM | 128GB DDR5 ECC |
| Disco | 2TB NVMe SSD |
| Modelo recomendado | Qwen2.5-Coder-32B-Instruct (AWQ) |
| Tensor parallelism | 2 GPUs |
| Throughput esperado | ~80-150 tokens/s |
| Custo estimado (cloud) | $1.600-2.400/mês |

## GPUs Recomendadas

| GPU | VRAM | Preço (novo) | Melhor para |
|---|---|---|---|
| RTX 4090 | 24GB | ~$1.600 | Lab, modelos 7B-14B |
| RTX A6000 | 48GB | ~$4.500 | Modelos 14B-32B |
| A100 40GB | 40GB | Cloud only | Produção pequena |
| A100 80GB | 80GB | Cloud only | Produção média |
| H100 80GB | 80GB | Cloud only | Alta escala |

## Cloud GPU Providers

Para quem não quer investir em hardware próprio.

| Provider | GPUs disponíveis | Preço/hora (A100) | Observações |
|---|---|---|---|
| RunPod | A100, H100, RTX 4090 | ~$1.10-2.50/h | Spot instances disponíveis |
| Vast.ai | Variado | ~$0.80-2.00/h | Marketplace, preços variáveis |
| Lambda | A100, H100 | ~$1.10-3.00/h | Bom suporte, estável |
| AWS (p4d) | A100 | ~$3.00-4.00/h | Enterprise, SLA |
| GCP (a2) | A100 | ~$3.00-4.00/h | Enterprise, SLA |

## Quantização

A quantização reduz o uso de VRAM com impacto mínimo na qualidade para tarefas de código.

| Método | Redução VRAM | Impacto Qualidade | Velocidade |
|---|---|---|---|
| FP16 (sem quantização) | Baseline | Nenhum | Baseline |
| AWQ (4-bit) | ~50-60% | Mínimo (~1-2% em benchmarks) | +20-30% |
| GPTQ (4-bit) | ~50-60% | Mínimo (~1-3%) | +10-20% |
| GGUF (llama.cpp) | ~50-75% | Variável | CPU-friendly |

**Recomendação:** Use AWQ para deploy com vLLM. É o formato com melhor suporte nativo e melhor trade-off qualidade/velocidade.

## Dimensionamento

Fórmula simplificada para estimar VRAM necessária:

```
VRAM_necessária = (parâmetros × bytes_por_param) + (contexto × overhead_KV_cache)

Exemplos:
- 7B FP16:  7B × 2 bytes = ~14GB + KV cache
- 7B AWQ:   7B × 0.5 bytes = ~3.5GB + KV cache + overhead = ~8GB total
- 14B AWQ:  14B × 0.5 bytes = ~7GB + KV cache + overhead = ~16GB total
- 32B AWQ:  32B × 0.5 bytes = ~16GB + KV cache + overhead = ~24-32GB total
```

## Recomendação para o debuga.ai

Para iniciar o laboratório e validar a stack:

1. **Começar com:** 1x RTX 4090 + Qwen2.5-Coder-7B-Instruct (AWQ)
2. **Validar:** Qualidade das respostas em tarefas DevOps/segurança
3. **Escalar para:** A100 40GB + Qwen2.5-Coder-14B-Instruct quando throughput justificar
4. **Produção futura:** 2x A100 80GB + Qwen2.5-Coder-32B-Instruct para escala

O investimento inicial em hardware próprio (RTX 4090) se paga em 3-6 meses comparado com cloud GPU, assumindo uso contínuo de 8+ horas/dia.
