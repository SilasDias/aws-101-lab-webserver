# 01 — Networking (VPC)

**O que é:** rede virtual isolada que eu defino (CIDR 10.0.0.0/16).
**Problema que resolve:** isolamento de rede + distribuição em múltiplas AZs.
**O que fiz:** criei subnets públicas/privadas em 2 AZs, IGW, NAT GW, route tables.
**O que aprendi:** subnet pública tem rota para o IGW; privada, para o NAT GW.
**Se eu refizesse:** planejaria o CIDR antes (10.0.0.0/20 vs /24...).
![subnets](../screenshots/01-subnets.png)
