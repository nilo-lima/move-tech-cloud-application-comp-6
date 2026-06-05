# move-tech-cloud-application-comp-6

Ponto de partida da **Competência 6 — Arquitetura de Soluções em Nuvem**.

Este repositório é um template. Use-o como base para criar o seu próprio repositório e trabalhar na competência.

> Parte do curso **Move Tech** — Magalu × Prósper Digital Skills  
> Formação em Cloud Computing para iniciantes

---

## Etapas anteriores

> [move-tech-cloud-application-comp-3](https://github.com/move-tech-cloud-computing/move-tech-cloud-application-comp-3) · [move-tech-cloud-application-comp-4](https://github.com/move-tech-cloud-computing/move-tech-cloud-application-comp-4) · [move-tech-cloud-application-comp-5](https://github.com/move-tech-cloud-computing/move-tech-cloud-application-comp-5)

---

## O que você vai fazer nesta competência

Ao final da Competência 6, você terá **documentado e analisado a arquitetura** da solução que construiu.

- [ ] Desenhar o diagrama de arquitetura da solução na Magalu Cloud
- [ ] Documentar as decisões técnicas tomadas ao longo do curso (ADR)
- [ ] Analisar os trade-offs das escolhas: custo, escalabilidade, disponibilidade
- [ ] Identificar pontos de melhoria e próximos passos

---

## Como rodar localmente

**Pré-requisito:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado (Mac e Windows) ou [Docker Engine](https://docs.docker.com/engine/install/) (Linux).

```bash
docker compose up --build
```

Acesse a documentação interativa em: http://localhost:8000/docs

---

## Configuração do banco de dados no Kubernetes

```bash
kubectl create secret generic db-secret \
  --from-literal=url=postgresql://user:password@<host-mgc>/orders
```

---

## Secrets necessários no GitHub

| Secret | Descrição |
|--------|-----------|
| `MGC_REGISTRY_USER` | Usuário do Container Registry da MGC |
| `MGC_REGISTRY_PASSWORD` | Senha do Container Registry da MGC |
| `MGC_REGISTRY_NAME` | Nome do seu registry na MGC |
| `MGC_KUBECONFIG` | Conteúdo do kubeconfig em base64 (`base64 -w0 kubeconfig.yaml`) |

---

## Solução completa de referência

Ao concluir esta competência, a solução final de referência estará disponível em:  
[move-tech-cloud-application-final](https://github.com/move-tech-cloud-computing/move-tech-cloud-application-final)
