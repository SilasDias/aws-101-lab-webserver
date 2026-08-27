# ☁️ AWS 101 Lab — Web Server Altamente Disponível

> Projeto construído na **Imersão AWS — Dia 1: Fundamentos Cloud**,
> baseado no workshop oficial *AWS 101 Workshop*.

## 🎯 Objetivo
Provisionar uma aplicação web segura e disponível: servidor Linux em
**subnet privada**, exposto por **ALB**, saída de internet via **NAT Gateway**,
acesso ao **S3** via **VPC Endpoint** e administração via **SSM** (sem SSH).

## 🏗️ Arquitetura
![arquitetura](diagrams/architecture.png)
Internet → IGW → ALB (pública) → EC2 (privada) → NAT GW / VPC Endpoint (S3)

## 📖 Documentação passo a passo
1. [Networking (VPC)](docs/01-networking-vpc.md)
2. [Security Secret (SGs)](docs/02-security-groups.md)

## 💰 Custo
~US$ 0,44/hora. Cleanup completo em [09-cleanup-checklist.md](docs/09-cleanup-checklist.md).

## 🚀 Próximos passos
- [ ] Challenge: Auto Scaling Group
- [ ] Reescrever o lab em Terraform (IaC)
- [ ] Workshop AWS 101 — Containers

## 🔐 Segurança deste repo
Sem credenciais reais; screenshots com account ID ocultado.
