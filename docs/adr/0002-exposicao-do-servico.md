# ADR 0002: Exposição do Serviço via Service tipo LoadBalancer (sem Ingress Controller)

- **Status:** Aprovado
- **Data:** 2026-08-05

## Contexto

O repositório expõe a API de Pedidos através de um único `Service` Kubernetes (`k8s/app.yaml`) do tipo `LoadBalancer`, mapeando a porta 80 externa para a porta 8000 do container. Não há nenhum recurso `Ingress` nem manifesto de Ingress Controller (ex.: nginx-ingress) no diretório `k8s/`. O provisionamento do IP público e balanceamento fica inteiramente a cargo do provedor de nuvem (Magalu Cloud), que provisiona um Load Balancer dedicado para esse Service. Atualmente a solução expõe apenas uma rota lógica (a própria API de Pedidos), sem necessidade de roteamento por host/path entre múltiplos serviços.

## Decisão

Expor a aplicação diretamente via `Service` do tipo `LoadBalancer`, sem introduzir um Ingress Controller/recurso `Ingress` nesta etapa do projeto.

## Consequências

**Positivas:**
- Menos componentes para instalar, atualizar e monitorar no cluster (não é preciso operar um Ingress Controller).
- Configuração mais simples e direta para expor um único serviço, como manifestado hoje.
- Um salto de rede a menos entre o cliente e o Pod, quando comparado a um Ingress Controller intermediário.
- Provisionamento do IP público é gerenciado automaticamente pelo provedor cloud a partir do próprio manifesto do Service.

**Negativas:**
- Sem roteamento por host/path, terminação de TLS centralizada, rate limiting ou WAF que um Ingress Controller normalmente oferece.
- Cada novo serviço exposto no futuro exigirá seu próprio Load Balancer - e cada Load Balancer gerenciado tem custo recorrente próprio na nuvem, o que não escala bem se a arquitetura evoluir para múltiplos serviços expostos.
- Se a solução crescer para múltiplos microsserviços ou precisar de HTTPS gerenciado centralizadamente, será necessário migrar para uma estratégia com Ingress Controller, o que implica retrabalho de infraestrutura.
