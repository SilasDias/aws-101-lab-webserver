# 02 — Security Groups (SGs)

**Status:** ✅ Concluído · **Módulo:** Resource Security (SGs)

## O que é
Firewall de nível de recurso, **stateful**: permitida a ida,
a volta entra automaticamente (diferente de NACLs, que são stateless).

## O que fiz
Microssegmentação em duas camadas:

| SG | Inbound | Outbound |
|---|---|---|
| LoadBalancerSG | TCP 80 de 0.0.0.0/0 | todo (default) |
| WebServerSG | TCP 80 **do LoadBalancerSG** | todo (default) |

## O que aprendi
- Chaining de SGs: a origem da regra é *outro SG*, não um CIDR —
  expressa intenção ("só o ALB fala com o web server") e não quebra
  se a topologia mudar;
- Stateful vs stateless (SG vs NACL);
- Zero porta 22 → administração via SSM (módulo 05).

## Trade-offs e decisões

### 1. TLS termina no ALB (HTTP no backend)
O lab usa apenas HTTP (80). O padrão de produção é **TLS termination
no ALB**: internet → ALB em 443, redirect 80 → 443, e ALB → EC2 em
HTTP dentro da rede privada.
- **Vantagem:** certificado gerenciado em um ponto só (ACM), sem custo
  de criptografia no EC2 e operação simples.
- **Quando mudar:** requisitos de compliance que exijam criptografia
  ponta a ponta → 443 também nos targets (e regra 443 no WebServerSG).

### 2. Outbound aberto vs outbound restrito
Mantive o default da AWS ("todo o tráfego" de saída). Como SGs são
stateful, o outbound só controla conexões **iniciadas pela instância**;
respostas do inbound permitido entram automaticamente.
- **Vantagem do aberto:** simplicidade operacional (updates, pacotes e
  agents funcionam sem regras extras).
- **Custo:** instância comprometida pode iniciar conexões para qualquer
  destino (exfiltração/C2). Hardening possível: restringir saída a 443
  e deixar o S3 no VPC Endpoint. Em ambientes maduros, o equilíbrio
  costuma ser SG aberto + controle de egress via NACLs/Network Firewall.

### 3. Chaining vs CIDR como origem
Referenciar o SG do ALB (e não o CIDR das subnets públicas) foi uma
decisão deliberada: a regra sobrevive a mudanças de topologia e lê-se
como política ("somente o load balancer"), não como endereço.

## Se eu refizesse
- Descrições significativas em todas as regras (a do LB ficou genérica);
- Produção: 443 no ALB + redirect 80 → 443;
- Hardening: outbound restrito no WebServerSG.

![LoadBalancerSG: TCP 80 aberto para a internet](../screenshots/02-LoadBalancerSecurityGroup.png)
![WebServerSG: TCP 80 restrito ao LoadBalancerSG](../screenshots/02-WebServerSecurityGroup.png)
