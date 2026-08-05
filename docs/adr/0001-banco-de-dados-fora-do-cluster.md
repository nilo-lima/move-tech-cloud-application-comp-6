# ADR 0001: Utilização de Banco de Dados PostgreSQL Gerenciado Fora do Cluster Kubernetes

- **Status:** Aprovado
- **Data:** 2026-08-05

## Contexto

A API de Pedidos persiste dados relacionais (pedidos e itens) via SQLAlchemy/psycopg2, conectando-se ao PostgreSQL através da variável `DATABASE_URL`. Em produção, essa string é injetada no `Deployment` (`k8s/app.yaml`) a partir de um `Secret` (`db-secret`), preenchido em tempo de deploy com o valor do secret `DATABASE_URL` do GitHub. Não existe nenhum manifesto de `StatefulSet`, `PersistentVolumeClaim` ou Pod de PostgreSQL no diretório `k8s/` - o banco roda fora do cluster, em um serviço gerenciado (DBaaS). O uso de PostgreSQL em container só aparece no `docker-compose.yml`, exclusivamente para desenvolvimento local.

Além disso, o próprio manifesto do HPA documenta uma preocupação relevante: cada réplica da aplicação abre seu próprio pool de conexões contra o mesmo PostgreSQL, o que limita o `maxReplicas` a 6 para não esgotar as conexões do banco.

## Decisão

Manter o PostgreSQL como um serviço gerenciado (DBaaS) externo ao cluster Kubernetes, nunca como Pod/StatefulSet interno. A aplicação se conecta a ele via TCP/TLS na porta 5432, usando uma credencial injetada por Secret do Kubernetes.

## Consequências

**Positivas:**
- Elimina a complexidade de gerenciar volumes persistentes, backups e failover de um banco stateful dentro do K8s.
- Backups automatizados e alta disponibilidade ficam a cargo do provedor.
- O ciclo de vida do banco (upgrades, escala vertical) é independente dos deploys da aplicação - um rollout do Deployment não arrisca o estado dos dados.
- Simplifica o HPA: escalar a aplicação horizontalmente não implica escalar armazenamento junto.

**Negativas:**
- Custo direto adicional do serviço gerenciado, em comparação a rodar Postgres em Pods com storage local.
- Latência de rede adicional (salto TCP/TLS fora do cluster) em cada consulta.
- Cada réplica nova da aplicação abre seu próprio pool de conexões contra o mesmo banco - motivo pelo qual o `maxReplicas` do HPA foi limitado a 6, exigindo atenção ao limite de conexões concorrentes do plano de DBaaS contratado.
- Dependência de disponibilidade do provedor externo como ponto único de falha para persistência.
