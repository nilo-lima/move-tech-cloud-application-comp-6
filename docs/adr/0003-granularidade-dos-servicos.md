# ADR 0003: Granularidade da Aplicação - Monolito Modular

- **Status:** Aprovado
- **Data:** 2026-08-05

## Contexto

O código-fonte (`app/main.py`, `app/models.py`, `app/database.py`) implementa os domínios de **Pedidos** (`Order`) e **Itens** (`Item`) dentro do mesmo processo FastAPI, no mesmo container, publicado como uma única imagem Docker e um único `Deployment` Kubernetes (`cloud-application`, 2 réplicas). Os dois domínios compartilham o mesmo banco de dados PostgreSQL e a mesma transação de sessão SQLAlemy (ex.: criação de item associada diretamente ao pedido via relacionamento ORM, com `cascade="all, delete-orphan"`). O pipeline de CI/CD (`.github/workflows/deploy.yml`) testa, builda e faz deploy de um único artefato. O HPA escala réplicas idênticas do processo inteiro - não há escalonamento independente por domínio.

## Decisão

Adotar um **Monolito Modular** para a API de Pedidos: os domínios de Pedidos e Itens residem no mesmo serviço implantável, com separação apenas em nível de código (módulos/rotas), e não como microsserviços independentes.

## Consequências

**Positivas:**
- Pipeline de CI/CD único e simples (um `pytest`, um build, um deploy), como já implementado em `deploy.yml`.
- Nenhuma chamada de rede entre os domínios Pedidos e Itens - operações são transações locais no mesmo banco/sessão, reduzindo latência e eliminando necessidade de padrões de consistência distribuída.
- Menor custo operacional e de infraestrutura: um único Deployment, um único Service, um único conjunto de métricas/probes para operar.
- Adequado ao escopo atual (um bounded context - "pedidos e itens") e ao tamanho da equipe/projeto.

**Negativas:**
- Pedidos e Itens não podem escalar de forma independente: o HPA escala o processo inteiro mesmo que a carga esteja concentrada em apenas um dos domínios.
- Uma falha ou lentidão introduzida no módulo de Itens pode degradar também as rotas de Pedidos, pois compartilham o mesmo processo/réplica.
- Se a complexidade de negócio crescer (novos domínios, times distintos por domínio, necessidades de deploy independente), será necessário revisitar esta decisão e avaliar a extração para microsserviços.
