# 06 — Load Balancing (ALB)

**Status:** ✅ Concluído · **Módulo:** Load Balancing (ALB)

## O que é
ALB (Application Load Balancer) é o balanceador de camada 7: distribui
requisições HTTP(S) entre targets saudáveis, com health checks automáticos
e failover transparente.

## O que fiz

| Configuração | Valor |
|---|---|
| Tipo | Application Load Balancer |
| Esquema | Internet-facing |
| Subnets | public1 (1a) + public2 (1b) |
| Security Group | LoadBalancerSecurityGroup |
| Listener | HTTP:80 → WebServerTargetGroup |
| Target | Instância i-0e45538b47fde6cb1 (privada) |
| Health check | HTTP `/` → 200, intervalo 30s, threshold 5 |

## O que aprendi
- O ALB é o **único componente com IP público** da aplicação; o backend
  permanece invisível (subnet privada + sem IP público);
- Health check remove target doente automaticamente → resiliência;
- DNS do ALB é estável; os IPs por trás mudam (nunca apontar IP fixo);
- SG chaining: o SG do EC2 confia no SG do ALB, não em CIDRs fixos.

## Trade-offs e decisões

### 1. HTTP-only vs HTTPS em produção
- **Lab:** HTTP-only (simplicidade, sem certificado);
- **Produção:** listener 443 + certificado ACM (gratuito) + redirect
  80→443 + TLS termination no ALB;
- **Hardening avançado:** TLS ponta a ponta (ALB→EC2 em 443) se
  compliance exigir criptografia em toda a rede.

### 2. Adesão (stickiness)
- **Desativada (escolha correta):** app stateless;
- **Ativar apenas se:** app guardar sessão local (e preferir mover
  sessão para Redis/ElastiCache em vez de stickiness, que reduz
  escalabilidade).

### 3. 1 AZ vs multi-AZ
- **Mínimo do ALB:** 2 AZs (requisito da AWS);
- **Benefício:** failover automático se uma AZ falhar;
- **Custo:** ~$0.0225/h por AZ + $0.008/GB processado.

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

## Se eu refizesse
- Adicionar listener 443 com certificado ACM (produção);
- Habilitar access logs do ALB para S3 (auditoria + análise de tráfego);
- Configurar WAF (Web Application Firewall) na frente do ALB;
- Usar VPC Flow Logs desde o início para debug.

![alb-active](../screenshots/06-alb-active.png)
![target-healthy](../screenshots/06-target-healthy.png)
![pagina-no-browser](../screenshots/06-pagina-browser.png)
