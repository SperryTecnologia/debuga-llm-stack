# Visão de integração

## Objetivo

Permitir adoção gradual de inferência local sem alterar o contrato consumido pela aplicação.

## Etapas

| Etapa | Ação | Critério de avanço |
|---|---|---|
| 1. Baseline | medir provider atual | métricas e erros conhecidos |
| 2. Shadow | enviar cópia controlada para o local | sem impacto ao usuário |
| 3. Canário | pequena parcela de tráfego autorizada | qualidade e estabilidade aceitas |
| 4. Preferência local | local como primário para tarefas elegíveis | fallback validado |
| 5. Otimização | ajustar roteamento e capacidade | evidência contínua |

## Guardrails

- classificação de dados antes de fallback cloud;
- timeout por provider;
- circuit breaker quando implementado;
- limite de custo;
- logs sem segredos;
- rollback para configuração conhecida.

Disponibilidade e latência são objetivos medidos, não garantias deste repositório.
