# Guia de hardware

## Princípio

Dimensionamento de LLM não deve ser feito apenas pelo número de parâmetros. Considere:

- formato e quantização dos pesos;
- tamanho de contexto;
- KV-cache;
- concorrência;
- engine e versão;
- overhead do runtime;
- margem para evitar OOM.

## Processo recomendado

1. Escolha um modelo e confirme sua licença.
2. Estime o espaço dos pesos.
3. Reserve margem para contexto e KV-cache.
4. Inicie com concorrência baixa.
5. Meça TTFT, latência total, throughput e falhas.
6. Aumente carga gradualmente.
7. Registre hardware, versões e configuração.

## Classes de laboratório

| Classe | Uso inicial | Observação |
|---|---|---|
| GPU 12–16 GB | modelos menores ou quantizados | contexto e concorrência limitados |
| GPU 24 GB | laboratório com modelos médios quantizados | validar OOM e qualidade |
| GPU 40–48 GB | modelos maiores ou maior concorrência | ainda requer benchmark |
| Multi-GPU | tensor parallel e modelos grandes | aumenta complexidade operacional |

## Métricas mínimas

- time to first token;
- latência p50/p95/p99;
- tokens de prompt e saída;
- throughput agregado;
- uso de GPU e KV-cache;
- fila e requests em execução;
- erros e OOM.

> Não há uma conversão universal entre “usuários simultâneos” e uma GPU. A carga depende do
> tamanho dos prompts, geração, frequência, batching e objetivos de latência.

## Referências

- documentação oficial do engine escolhido;
- model card e licença do modelo;
- documentação oficial do NVIDIA Container Toolkit;
- resultados reproduzíveis do ambiente-alvo.
