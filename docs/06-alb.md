## Troubleshooting real: target unhealthy

Durante o lab, o target ficou temporariamente **não íntegro** (unhealthy).
O health check do ALB estava correto (HTTP, `/`, 200, 30s), mas a instância
não respondia como "saudável".

**Hipóteses testadas:**
1. ❌ Apache não iniciado → validado via SSM que estava `active (running)`;
2. ❌ Path do health check errado → `/` é o correto para o lab;
3. ✅ **Causa raiz:** confusão entre `sg-` (Security Group) e `sgr-`
   (regra do SG) ao verificar qual SG estava anexado ao ALB.

**Lição de arquitetura:**
O SG chaining funciona como *allowlist de confiança*: se o SG do EC2
confia no `sg-ALB` e o ALB está usando `sg-OUTRO`, o tráfego é negado
silenciosamente (sem log de firewall no EC2, porque o tráfego nem chega).

**Debugging em produção:**
- Verificar qual SG está realmente anexado ao ALB (console ou CLI:
  `aws elbv2 describe-load-balancers --load-balancer-arns ...`);
- Confirmar que o SG do EC2 tem regra inbound permitindo tráfego
  **desse SG específico**;
- Usar VPC Flow Logs para ver tráfego rejeitado (custo adicional,
  mas essencial para debug de conectividade).

**Prevenção:**
- Naming consistente (`LoadBalancerSG`, `WebServerSG`) para evitar
  confusão visual na console;
- Tags em todos os recursos para busca rápida;
- IaC (Terraform/CloudFormation) elimina esse tipo de erro humano.
