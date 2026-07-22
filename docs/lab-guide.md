# Guia de laboratório

## Objetivo

Validar um endpoint local de inferência sem conectá-lo automaticamente à produção.

## Sequência

1. Confirme GPU, driver e runtime.
2. Copie `.env.example` para `.env`.
3. Selecione um modelo permitido para o hardware.
4. Revise `docker compose config`.
5. Inicie somente o serviço `vllm`.
6. Valide `/health`, `/v1/models` e uma inferência curta.
7. Registre versões e resultados.
8. Ative monitoramento opcional.
9. Execute benchmarks sintéticos sem dados reais.
10. Desligue o laboratório quando terminar.

## Comandos

```bash
cp .env.example .env
docker compose config
docker compose up -d vllm
docker compose logs -f vllm
curl -fsS http://localhost:8000/health
```

## Checklist de evidência

```text
[ ] modelo e revisão
[ ] licença conferida
[ ] GPU e driver
[ ] imagem/engine e versão
[ ] quantização
[ ] contexto e concorrência
[ ] dataset sintético
[ ] saída bruta preservada
[ ] resultado e limitações documentados
```
