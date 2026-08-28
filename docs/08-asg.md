# 08 — Scaling (ASG) [Challenge]

**Status:** ✅ Concluído · **Módulo:** Scaling (ASG) - Challenge

## O que é
Auto Scaling Group (ASG) gerencia um conjunto de instâncias EC2 com:
- Auto-recuperação (substitui instâncias doentes automaticamente);
- Escalabilidade (adiciona/remove instâncias baseado em métricas);
- Distribuição multi-AZ (balanceamento geográfico).

## O que fiz

### 1. Launch Template
Criei `Project-AWS-101-LaunchTemplate` baseado na instância original:
- AMI: Amazon Linux 2023;
- Tipo: t2.micro;
- Subnet: não definida (definida no ASG);
- Security Group: WebServerSecurityGroup;
- IAM Profile: WebServerInstanceProfile;
- User data: mesmo script de bootstrap;
- Sem par de chaves (admin via SSM);
- Sem IP público (subnet privada).

### 2. Auto Scaling Group
- Nome: `Project-AWS-101-ASG`;
- Launch Template: `Project-AWS-101-LaunchTemplate`;
- Subnets: ambas as privadas (1a + 1b);
- Target Group: `WebServerTargetGroup` (existente);
- Health Check Grace Period: 300s (5 min para o bootstrap completar);
- Dimensionamento: min=1, max=4, desired=2;
- Política: Target Tracking (CPU 50%).

### 3. Teste de Recuperação
- Terminei manualmente uma instância do ASG;
- ASG detectou a redução (1 < desired=2);
- Lançou nova instância automaticamente;
- Nova instância passou no health check e foi registrada no Target Group;
- App continuou disponível sem downtime perceptível.

## O que aprendi
- Launch Template é o "blueprint" imutável da instância (versão controlada);
- ASG garante desired capacity (se você deleta, ele recria);
- Health Check Grace Period é crítico: muito curto → instância saudável é terminada;
- Target Group aceita instâncias de qualquer origem (manual ou ASG);
- Load balancing distribui tráfego entre instâncias do ASG + instâncias avulsas.

## Trade-offs e decisões

### 1. Launch Template vs Launch Configuration
- **Launch Template (escolha):** versão controlada, suporta Spot/Fleet, T2/T3 unlimited;
- **Launch Configuration (legado):** sem versionamento, sem suporte a Spot;
- **Decisão:** Template é o padrão atual da AWS.

### 2. Subnet no Template vs no ASG
- **Não definir no Template (escolha):** flexibilidade para o ASG escolher subnets;
- **Definir no Template:** trava a subnet (útil se o template é reutilizado em contextos fixos);
- **Decisão:** deixar no ASG para suportar multi-AZ.

### 3. Health Check Grace Period (300s)
- **Muito curto (<180s):** instância termina antes de ficar saudável (falso negativo);
- **Muito longo (>600s):** instância realmente doente demora para ser substituída;
- **Decisão:** 300s (5 min) baseado no tempo de bootstrap (~2-3 min) + margem.

### 4. Política de Scaling
- **Target Tracking (escolha):** ajusta para manter CPU em 50%;
- **Step Scaling:** ações discretas (se CPU > 80% por 5 min, +2 instâncias);
- **Scheduled:** escala em horários fixos (ex.: 8h→3 instâncias, 22h→1);
- **Decisão:** Target Tracking é o mais simples e eficaz para workloads típicos.

## Teste de Recuperação (prova de conceito)

```text
1. ASG com desired=2 → 2 instâncias ativas;
2. Termino manualmente instância i-xxx;
3. ASG detecta: 1 < 2 (desired);
4. ASG lança nova instância i-yyy;
5. Nova instância passa no health check (após grace period);
6. Target Group registra i-yyy como saudável;
7. App continua disponível (load balancing distribui tráfego).
