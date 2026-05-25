# 🌐 Guia Completo: Especialista em Redes, Infraestrutura & Azure AD
### Plano de 5 Anos — Do Iniciante ao Expert

> **Atualizado:** Março 2026 | **Foco:** Redes on-premises, Azure Cloud, Active Directory/Entra ID, Segurança e Infraestrutura Híbrida

---

## 📋 Índice

1. [Visão Geral da Jornada](#visão-geral)
2. [ANO 1 — Fundamentos Sólidos](#ano-1)
3. [ANO 2 — Administração & Azure Core](#ano-2)
4. [ANO 3 — Especialização em Redes Azure & Identidade](#ano-3)
5. [ANO 4 — Arquitetura, Segurança e Hybrid](#ano-4)
6. [ANO 5 — Expert & Liderança Técnica](#ano-5)
7. [Mapa de Certificações Microsoft](#certificações)
8. [Ferramentas e Labs Práticos](#labs)
9. [Recursos Gratuitos e Pagos](#recursos)
10. [Links Essenciais](#links)
11. [Dicas de Carreira](#carreira)

---

## 🗺️ Visão Geral da Jornada <a name="visão-geral"></a>

```
ANO 1          ANO 2          ANO 3          ANO 4          ANO 5
─────────      ─────────      ─────────      ─────────      ─────────
Fundamentos → Administração → Especialista → Arquiteto   → Expert
Redes TCP/IP   Azure IaaS     Azure Ntwrk    Soluções      Liderança
OSI Model      AD/Entra ID    AZ-700         Híbridas      SC-100
Cisco Basics   AZ-104         SC-300         AZ-305        Consultoria
CompTIA N+     Windows Srv    VPN/ExpRoute   Zero Trust    Speaking
```

### Perfil de Destino (Ano 5)
- **Cargo:** Senior Cloud Architect / Azure Network Specialist / Cloud Infrastructure Lead
- **Salário médio (Brasil):** R$ 18.000 – R$ 35.000/mês
- **Salário médio (EUA/Europa):** USD 130.000 – USD 200.000/ano
- **Certificações acumuladas:** AZ-900, AZ-104, AZ-700, SC-300, AZ-305, SC-100

---

## 📅 ANO 1 — Fundamentos Sólidos <a name="ano-1"></a>

> **Meta:** Entender o mundo de redes e computação. Base para tudo que vem depois.

### 🔵 Trimestre 1 (Meses 1–3): Redes Fundamentais

#### Tópicos Obrigatórios
- **Modelo OSI** — 7 camadas e o que cada uma faz
- **Modelo TCP/IP** — Camadas, protocolos, handshake
- **Endereçamento IP** — IPv4, IPv6, sub-redes (CIDR), VLSM
- **Protocolo ARP, DHCP, DNS, NAT/PAT**
- **Switching** — VLANs, Trunking (802.1Q), STP
- **Roteamento** — Estático, RIP, OSPF, BGP (noções)
- **Cabeamento** — Categorias, fibra óptica, patch panels

#### Certificação Alvo
- ✅ **CompTIA Network+** (N10-009) — Certificação vendor-neutral obrigatória

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Professor Messer Network+ | Vídeo | 🆓 Gratuito |
| Cisco NetAcad - Introduction to Networks | Curso | 🆓 Gratuito |
| CBT Nuggets Network+ | Vídeo | 💰 Pago |
| "Computer Networks" - Tanenbaum | Livro | 💰 Pago |

### 🔵 Trimestre 2 (Meses 4–6): Sistemas Operacionais

#### Tópicos Obrigatórios
- **Windows Server** — Instalação, gerenciamento básico, Hyper-V
- **Linux fundamentos** — Comandos, filesystem, usuários, SSH
- **Active Directory (on-premises)** — AD DS, OU, GPO básico, DNS integrado
- **Virtualização** — Conceitos de VM, Hyper-V, VMware Basics

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn — Windows Server | Curso | 🆓 Gratuito |
| Linux Journey (linuxjourney.com) | Interativo | 🆓 Gratuito |
| TechWorld with Nana (YouTube) | Vídeo | 🆓 Gratuito |

### 🔵 Trimestre 3–4 (Meses 7–12): Cloud Fundamentos + AZ-900

#### Tópicos Obrigatórios
- **Cloud Computing** — IaaS, PaaS, SaaS, nuvem pública/privada/híbrida
- **Azure Overview** — Regiões, Zonas de Disponibilidade, Resource Groups
- **Serviços Core Azure** — VMs, Storage, VNet, Azure DNS
- **Modelo de custo Azure** — Pay-as-you-go, Reservas, Calculadora

#### Certificação Alvo
- ✅ **AZ-900** — Microsoft Azure Fundamentals

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-900 Path | Curso oficial | 🆓 Gratuito |
| John Savill AZ-900 (YouTube) | Vídeo | 🆓 Gratuito |
| ExamTopics AZ-900 | Simulados | 🆓/💰 |
| Udemy — AZ-900 (Scott Duffy) | Vídeo | 💰 Pago |

---

## 📅 ANO 2 — Administração & Azure Core <a name="ano-2"></a>

> **Meta:** Operar ambientes Azure no dia a dia. Administrar identidades e infraestrutura.

### 🟡 Trimestre 1–2 (Meses 13–18): Azure Administrator — AZ-104

#### Tópicos Obrigatórios
- **Azure Active Directory / Entra ID** — Usuários, grupos, RBAC
- **Gerenciamento de Identidades** — SSO, MFA, Conditional Access básico
- **Virtual Networks (VNet)** — Sub-redes, NSG, Route Tables
- **Azure Storage** — Blob, File, Queue, Table, redundância
- **Azure Compute** — VMs (Windows/Linux), Scale Sets, Availability Sets
- **Azure Monitor** — Log Analytics, Alertas, Métricas
- **Azure Backup e Recovery Services**
- **Azure Policy e Governance** — Management Groups, Tags, Budgets
- **PowerShell e Azure CLI** — Automação básica

#### Certificação Alvo
- ✅ **AZ-104** — Microsoft Azure Administrator Associate

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-104 Path | Curso oficial | 🆓 Gratuito |
| John Savill AZ-104 Study Cram (YouTube) | Vídeo | 🆓 Gratuito |
| TFTEC Cloud (tftec.com.br) | Plataforma BR | 💰 Pago |
| Udemy — AZ-104 (Alan Rodrigues) | Vídeo | 💰 Pago |
| A Cloud Guru AZ-104 | Hands-on Labs | 💰 Pago |

### 🟡 Trimestre 3–4 (Meses 19–24): Windows Server & Active Directory Avançado

#### Tópicos Obrigatórios
- **Active Directory Avançado** — Trusts, Sites & Services, Replication
- **Group Policy (GPO)** — Criação, herança, aplicação, troubleshooting
- **DNS Avançado** — Zonas, registros, integração com AD
- **DHCP, IPAM** — Gerenciamento de endereços
- **PKI (Public Key Infrastructure)** — Certificate Authority, certificados
- **Windows Server Hybrid** — Azure AD Connect, Hybrid Join
- **Azure AD Connect / Entra Connect** — Sincronização de identidades

#### Certificação Alvo (Opcional, mas recomendado)
- ⭐ **AZ-800** — Administering Windows Server Hybrid Core Infrastructure

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-800 Path | Curso oficial | 🆓 Gratuito |
| TechDirectArchives (YouTube) | Vídeo | 🆓 Gratuito |
| Documentação oficial AD DS | Docs | 🆓 Gratuito |

---

## 📅 ANO 3 — Especialização em Redes Azure & Identidade <a name="ano-3"></a>

> **Meta:** Tornar-se referência em redes cloud e identidade/acesso.

### 🟠 Trimestre 1–2 (Meses 25–30): Azure Network Engineer — AZ-700

#### Tópicos Obrigatórios

**Redes Privadas Azure**
- Virtual Networks (VNet) avançado — Subnetting, peering
- VNet Peering (regional e global)
- User Defined Routes (UDR) e Route Tables
- Network Security Groups (NSG) — Regras, logs, diagnóstico
- Azure Firewall e Azure Firewall Premium
- Azure DDoS Protection

**Conectividade Híbrida**
- VPN Gateway — Tipos (Policy-based, Route-based), SKUs
- Site-to-Site VPN — Configuração e troubleshooting
- Point-to-Site VPN — Protocolos (OpenVPN, IKEv2, SSTP)
- **ExpressRoute** — Circuitos, peering, FastPath, Global Reach
- Azure Virtual WAN — Hub-and-spoke, topologias

**Entrega de Aplicações**
- Azure Load Balancer — Standard vs Basic, Frontend/Backend pools
- Application Gateway — WAF, URL routing, SSL termination
- Azure Front Door — CDN, WAF global, routing policies
- Azure Traffic Manager — Perfis, métodos de roteamento

**Serviços de Acesso Privado**
- Private Endpoint e Private Link
- Service Endpoints
- Azure Bastion — Secure RDP/SSH sem IP público

**DNS Azure**
- Azure DNS Público e Privado
- Private DNS Zones e Auto-registration
- DNS Resolver (Private DNS Resolver)

**Monitoramento de Redes**
- Network Watcher — IP flow verify, Connection troubleshoot
- Azure Monitor para redes
- Flow Logs (NSG Flow Logs)
- Connection Monitor

#### Certificação Alvo
- ✅ **AZ-700** — Designing and Implementing Microsoft Azure Networking Solutions

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-700 Path | Curso oficial | 🆓 Gratuito |
| John Savill AZ-700 (YouTube) | Vídeo | 🆓 Gratuito |
| Udemy AZ-700 (Robin Jackson) | Vídeo | 💰 Pago |
| Azure Network Labs (GitHub MS) | Labs | 🆓 Gratuito |
| Documentação Azure Networking | Docs | 🆓 Gratuito |

### 🟠 Trimestre 3–4 (Meses 31–36): Identidade & Acesso — SC-300

#### Tópicos Obrigatórios

**Microsoft Entra ID (Azure AD)**
- Tenants, Users, Groups, Service Principals
- App Registrations e Enterprise Applications
- Managed Identities — System-assigned vs User-assigned
- B2B (Guest Access) e B2C

**Autenticação**
- MFA — Métodos, políticas, SSPR
- Passwordless Authentication — FIDO2, Windows Hello, Authenticator App
- Legacy Authentication e bloqueio

**Autorização e Controle de Acesso**
- RBAC (Role-Based Access Control) — Built-in vs Custom roles
- Privileged Identity Management (PIM) — Just-in-time access
- Conditional Access — Políticas, Named Locations, Sign-in risk
- Identity Protection — Risk policies, risky users

**Governança de Identidades**
- Access Reviews
- Entitlement Management
- Terms of Use
- Lifecycle Workflows

**Identidade Híbrida**
- Azure AD Connect — PHS, PTA, Federation (ADFS)
- Azure AD Connect Cloud Sync
- Seamless SSO
- Device Join — Azure AD Join vs Hybrid Azure AD Join

**Acesso Externo**
- External Identities (B2B)
- Cross-tenant Access Settings
- Azure AD B2C

#### Certificação Alvo
- ✅ **SC-300** — Microsoft Identity and Access Administrator

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn SC-300 Path | Curso oficial | 🆓 Gratuito |
| John Savill SC-300 Study Cram | Vídeo | 🆓 Gratuito |
| Pluralsight SC-300 | Vídeo | 💰 Pago |
| Microsoft Entra Docs | Docs | 🆓 Gratuito |

---

## 📅 ANO 4 — Arquitetura, Segurança e Hybrid <a name="ano-4"></a>

> **Meta:** Projetar soluções complexas e seguras. Pensar em arquitetura, não só em operação.

### 🔴 Trimestre 1–2 (Meses 37–42): Azure Solutions Architect — AZ-305

#### Tópicos Obrigatórios

**Design de Infraestrutura**
- Well-Architected Framework — Pilares (Custo, Confiabilidade, Segurança, Excelência Operacional, Performance)
- Cloud Adoption Framework (CAF)
- Azure Landing Zones
- Hub-and-Spoke Network Topology
- Disaster Recovery — RTO/RPO, Active-Active vs Active-Passive

**Design de Identidade**
- Estratégias de identidade híbrida
- Multi-tenant design
- RBAC e governance em escala

**Design de Redes**
- Conectividade híbrida em escala (ExpressRoute Global Reach)
- Azure Virtual WAN enterprise
- Network segmentation e micro-segmentação
- Zero Trust Network Access

**Design de Dados**
- Azure SQL, Cosmos DB, Azure Storage
- Data tiers e escolha de banco de dados
- Backup e replication strategies

**Design de Aplicações**
- App Service, AKS, Azure Functions
- API Management
- Event-driven architecture

#### Certificação Alvo
- ✅ **AZ-305** — Designing Microsoft Azure Infrastructure Solutions (Expert)
- *Pré-requisito obrigatório: AZ-104*

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-305 Path | Curso oficial | 🆓 Gratuito |
| John Savill AZ-305 (YouTube) | Vídeo | 🆓 Gratuito |
| Udemy AZ-305 (Scott Duffy) | Vídeo | 💰 Pago |
| Azure Architecture Center | Docs | 🆓 Gratuito |
| Well-Architected Framework | Docs | 🆓 Gratuito |

### 🔴 Trimestre 3–4 (Meses 43–48): Segurança Cloud — AZ-500 + Zero Trust

#### Tópicos Obrigatórios

**Azure Security**
- Azure Security Center / Microsoft Defender for Cloud
- Microsoft Sentinel (SIEM/SOAR)
- Azure Key Vault — Secrets, Keys, Certificates
- Disk Encryption, Storage Encryption, TDE
- Network security em profundidade

**Zero Trust**
- Princípios — Verify explicitly, Least privilege, Assume breach
- Zero Trust Network Access (ZTNA)
- Microsoft Entra Private Access
- Microsegmentation

**Compliance e Governança**
- Azure Policy avançado — Custom policies, Initiatives
- Microsoft Purview (Compliance)
- Regulatory compliance (ISO 27001, NIST, SOC 2)

#### Certificação Alvo
- ✅ **AZ-500** — Microsoft Azure Security Technologies

#### Recursos
| Recurso | Tipo | Custo |
|---------|------|-------|
| Microsoft Learn AZ-500 Path | Curso oficial | 🆓 Gratuito |
| Microsoft Cybersecurity Reference Architectures | Docs | 🆓 Gratuito |
| Microsoft Defender for Cloud Docs | Docs | 🆓 Gratuito |

---

## 📅 ANO 5 — Expert & Liderança Técnica <a name="ano-5"></a>

> **Meta:** Ser a referência. Arquitetar soluções enterprise. Liderar equipes e projetos.

### 🟣 Trimestre 1–2 (Meses 49–54): Cybersecurity Architect — SC-100

#### Tópicos Obrigatórios
- **Zero Trust Architecture** completa
- **Microsoft 365 Defender** (XDR)
- **Microsoft Sentinel** avançado — KQL, Playbooks, UEBA
- **SASE** (Secure Access Service Edge)
- **Compliance architecture** — Data Governance, Privacy
- **Multi-cloud security** — Azure + AWS + GCP
- Threat modeling e arquitetura de segurança

#### Certificação Alvo
- ✅ **SC-100** — Microsoft Cybersecurity Architect Expert
- *Pré-requisito: uma cert. Associate de segurança (SC-300 ou AZ-500)*

### 🟣 Trimestre 3–4 (Meses 55–60): Especialização Final + Soft Skills

#### Áreas de Aprofundamento (escolha 1-2)
- **DevOps & IaC** — Terraform, Bicep, GitHub Actions, AZ-400
- **Kubernetes & Containers** — AKS, Helm, GitOps
- **SD-WAN e SASE** — Cisco Meraki, Azure Virtual WAN avançado
- **AI + Infraestrutura** — Azure AI Services, Copilot for Security
- **Multi-cloud** — AWS Solutions Architect ou GCP Professional Cloud Architect

#### Soft Skills Essenciais para o Ano 5
- Comunicação técnica para executivos (apresentações, reports)
- Condução de workshops e Design Sessions
- Escrita técnica (documentação de arquitetura, ADRs)
- Mentoria de equipes
- Gestão de projetos (básico de ITIL, Agile)

---

## 🏆 Mapa de Certificações Microsoft <a name="certificações"></a>

```
NÍVEL FUNDAMENTAL
═══════════════════════════════════════════════════════
  AZ-900          SC-900          MS-900
  Azure Fund.     Security Fund.  M365 Fund.
      │               │
      ▼               ▼
NÍVEL ASSOCIATE (escolha seu foco)
═══════════════════════════════════════════════════════
  AZ-104          AZ-700          SC-300          AZ-500
  Azure Admin     Network Eng.    Identity Admin  Security Eng.
      │               │               │               │
      └───────┬───────┘               └───────┬───────┘
              ▼                               ▼
NÍVEL EXPERT
═══════════════════════════════════════════════════════
       AZ-305                         SC-100
  Solutions Architect           Cybersecurity Architect
  (req: AZ-104)                 (req: SC-300 ou AZ-500)

ESPECIALISTAS
═══════════════════════════════════════════════════════
  AZ-800 + AZ-801       AZ-720            AZ-400
  Windows Server Hybrid  Connectivity TS   DevOps Expert
```

### Tabela de Certificações por Ano

| Ano | Certificação | Dificuldade | Custo (aprox.) |
|-----|-------------|-------------|----------------|
| Ano 1 | CompTIA Network+ | ⭐⭐ | USD 360 |
| Ano 1 | **AZ-900** | ⭐ | USD 165 |
| Ano 2 | **AZ-104** | ⭐⭐⭐ | USD 165 |
| Ano 2 | AZ-800 (opcional) | ⭐⭐⭐ | USD 165 |
| Ano 3 | **AZ-700** | ⭐⭐⭐⭐ | USD 165 |
| Ano 3 | **SC-300** | ⭐⭐⭐ | USD 165 |
| Ano 4 | **AZ-305** | ⭐⭐⭐⭐⭐ | USD 165 |
| Ano 4 | **AZ-500** | ⭐⭐⭐⭐ | USD 165 |
| Ano 5 | **SC-100** | ⭐⭐⭐⭐⭐ | USD 165 |

> 💡 **Dica:** Todas as certs Microsoft (exceto CompTIA) custam USD 165 no Brasil via Pearson VUE. A renovação é **gratuita** pelo Microsoft Learn a cada ano.

---

## 🧪 Ferramentas e Labs Práticos <a name="labs"></a>

### Ambientes de Lab Gratuitos

| Ferramenta | Uso | Link |
|-----------|-----|------|
| Azure Free Account | 200 créditos por 30 dias + 12 meses free tier | portal.azure.com |
| Microsoft Learn Sandbox | Labs gratuitos sem conta Azure | learn.microsoft.com |
| GitHub Student Pack | Créditos Azure para estudantes | education.github.com |
| EVE-NG / GNS3 | Simulação de redes Cisco/Linux | eve-ng.net |
| Packet Tracer (Cisco) | Simulador Cisco gratuito | netacad.com |

### Labs Recomendados por Área

**Redes (Ano 1)**
- Configurar VLANs e inter-VLAN routing no Packet Tracer
- Implementar OSPF e BGP no GNS3
- Sub-rede uma classe B em 30 sub-redes (exercício manual)

**Azure Networking (Ano 3)**
- Criar Hub-and-Spoke VNet topology com Azure Firewall
- Configurar VPN Gateway Site-to-Site com dispositivo simulado
- Implementar Application Gateway com WAF e SSL termination
- Configurar Private Endpoint para Azure SQL Database
- Criar Azure Virtual WAN com múltiplos branches
- Testar Network Watcher — Flow logs, Connection Monitor

**Active Directory / Entra ID (Ano 2–3)**
- Instalar AD DS em Windows Server + criar OUs e GPOs
- Configurar Azure AD Connect (PHS) em lab híbrido
- Implementar Conditional Access com MFA step-up
- Ativar PIM e criar role assignment just-in-time

**Segurança (Ano 4)**
- Habilitar Microsoft Defender for Cloud e revisar recomendações
- Criar regras de Sentinel e investigar um incidente simulado
- Configurar Azure Key Vault com RBAC e auditoria
- Implementar Azure Policy para compliance

### Laboratório Home Lab Mínimo

```
Hardware recomendado:
- PC ou Notebook com 16GB RAM (ideal 32GB)
- SSD com 256GB livre
- Conexão estável à internet

Software gratuito:
- VMware Workstation Player OU VirtualBox (free)
- Windows Server 2022 (ISO gratuita 180 dias)
- Ubuntu Server 22.04 (gratuito)
- EVE-NG Community (gratuito)
```

---

## 📚 Recursos Gratuitos e Pagos <a name="recursos"></a>

### 🆓 Gratuitos — Os Melhores

| Recurso | O que oferece |
|---------|---------------|
| **Microsoft Learn** (learn.microsoft.com) | Trilhas completas, Sandboxes, todos os paths de certificação |
| **John Savill's Technical Training** (YouTube) | Melhor canal gratuito sobre Azure — AZ-104, AZ-700, AZ-305, SC-100 |
| **Professor Messer** (professormesser.com) | CompTIA Network+, Security+ completos e gratuitos |
| **NetworkChuck** (YouTube) | Redes, Linux, Cisco de forma didática |
| **TechWorld with Nana** (YouTube) | DevOps, Kubernetes, Docker |
| **Azure Architecture Center** (docs.microsoft.com) | Guias de referência, padrões de arquitetura |
| **Azure Well-Architected Review** | Ferramenta online para avaliar arquiteturas |
| **GitHub Microsoft Learning** | Repositórios com labs oficiais |
| **KodeKloud** (plano free) | Labs de Linux e Kubernetes |

### 💰 Pagos — Vale o Investimento

| Recurso | Destaque | Custo Estimado |
|---------|---------|----------------|
| **TFTEC Cloud** (tftec.com.br) | Melhor plataforma BR para Azure | R$ 100–200/mês |
| **A Cloud Guru / Pluralsight** | Hands-on labs excelentes | USD 49/mês |
| **Udemy** (cursos específicos) | Ótimo custo-benefício em promoção | R$ 30–80/curso |
| **CBT Nuggets** | Qualidade alta para Network+ e Azure | USD 59/mês |
| **MeasureUp** | Simulados oficiais Microsoft | USD 99/exame |
| **Whizlabs** | Simulados + labs | USD 15–30/cert |

### 📖 Livros Recomendados

| Livro | Área | Nível |
|-------|------|-------|
| "Computer Networks" — Tanenbaum | Redes | Fundamentos |
| "TCP/IP Illustrated Vol. 1" — Stevens | TCP/IP Deep | Intermediário |
| "Azure Infrastructure as a Service" — O'Brien | Azure IaaS | Intermediário |
| "Microsoft Azure Architect Technologies" | AZ-305 | Avançado |
| "Zero Trust Networks" — Gilman & Barth | Security | Avançado |
| "Designing Distributed Systems" — Burns | Arquitetura | Avançado |

---

## 🔗 Links Essenciais <a name="links"></a>

### Documentação Oficial

```
Microsoft Learn (trilhas e sandboxes):
https://learn.microsoft.com/pt-br/

Azure Documentation:
https://docs.microsoft.com/pt-br/azure/

Microsoft Entra (Azure AD) Docs:
https://docs.microsoft.com/pt-br/azure/active-directory/

Azure Architecture Center:
https://docs.microsoft.com/pt-br/azure/architecture/

Azure Networking Docs:
https://learn.microsoft.com/pt-br/azure/networking/fundamentals/networking-overview

Azure Well-Architected Framework:
https://learn.microsoft.com/pt-br/azure/well-architected/

Cloud Adoption Framework:
https://learn.microsoft.com/pt-br/azure/cloud-adoption-framework/
```

### Trilhas de Aprendizado (Microsoft Learn — PT-BR)

```
AZ-900 Fundamentals:
https://learn.microsoft.com/pt-br/credentials/certifications/azure-fundamentals/

AZ-104 Administrator:
https://learn.microsoft.com/pt-br/credentials/certifications/azure-administrator/

AZ-700 Network Engineer:
https://learn.microsoft.com/pt-br/credentials/certifications/azure-network-engineer-associate/

SC-300 Identity Admin:
https://learn.microsoft.com/pt-br/credentials/certifications/identity-and-access-administrator/

AZ-305 Solutions Architect:
https://learn.microsoft.com/pt-br/credentials/certifications/azure-solutions-architect/

AZ-500 Security Engineer:
https://learn.microsoft.com/pt-br/credentials/certifications/azure-security-engineer/

SC-100 Cybersecurity Architect:
https://learn.microsoft.com/pt-br/credentials/certifications/cybersecurity-architect/
```

### Comunidades e Fóruns

```
Reddit r/azure (inglês):
https://www.reddit.com/r/AZURE/

Reddit r/networking (inglês):
https://www.reddit.com/r/networking/

Microsoft Tech Community (inglês/PT):
https://techcommunity.microsoft.com/

TFTEC Discord (PT-BR):
https://discord.gg/tftec

Stack Overflow PT:
https://pt.stackoverflow.com/questions/tagged/azure

Azure Brasil (LinkedIn Group):
buscar "Azure Brasil" no LinkedIn
```

### Canais YouTube Essenciais

```
John Savill's Technical Training (EN):
https://www.youtube.com/@NTFAQGuy

NetworkChuck (EN):
https://www.youtube.com/@NetworkChuck

TFTEC Cloud (PT-BR):
https://www.youtube.com/@TftecCloud

Raphael Andrade - Azure (PT-BR):
buscar "Raphael Andrade Azure" no YouTube

Professor Messer (EN - CompTIA):
https://www.youtube.com/@professormesser
```

### Simulados Gratuitos

```
ExamTopics (AZ-900, AZ-104, AZ-700, SC-300, AZ-305):
https://www.examtopics.com/

MSCertQuiz (simulados práticos):
https://mscertquiz.com/

Microsoft Learn Assessments (oficial):
https://learn.microsoft.com/pt-br/credentials/certifications/
```

---

## 💼 Dicas de Carreira <a name="carreira"></a>

### Como Construir Portfólio Técnico

1. **GitHub** — Poste scripts de automação (PowerShell, Terraform, Bicep)
2. **Blog técnico** — Medium, Dev.to, Hashnode — escreva sobre o que aprende
3. **LinkedIn** — Documente cada certificação e projeto
4. **Projetos open-source** — Contribua para repositórios de Azure tools
5. **Home lab documentado** — Fotos, diagramas, arquitetura no GitHub

### Habilidades Complementares Importantes

| Skill | Por quê importa | Nível necessário |
|-------|----------------|------------------|
| **Terraform** | IaC é obrigatório em 2026 | Intermediário |
| **PowerShell** | Automação Windows/Azure | Intermediário |
| **Azure CLI / Bash** | DevOps pipelines | Básico-Intermediário |
| **KQL (Kusto)** | Sentinel, Log Analytics | Básico |
| **Git/GitHub** | Versionamento de configs | Básico |
| **ITIL v4** | Processos de TI | Fundamentos |
| **Inglês técnico** | Docs, certs, mercado global | Leitura fluente |

### Mercado de Trabalho Brasil (2026)

- **Demanda:** Alta — Azure é dominante em empresas enterprise no Brasil
- **Salários por nível:**
  - Júnior/Trainee (Ano 1-2): R$ 4.000 – R$ 8.000
  - Pleno/Mid (Ano 2-3): R$ 8.000 – R$ 15.000
  - Sênior (Ano 4-5): R$ 15.000 – R$ 30.000
  - Especialista/Arquiteto: R$ 25.000 – R$ 45.000+

### Progressão de Cargo Típica

```
Estágio/Trainee → Analista de TI Jr → Analista de Infra Pleno
→ Especialista Azure / Analista Sênior → Arquiteto de Nuvem
→ Lead Architect / Principal Engineer
```

### Dicas de Ouro

1. **Faça labs TODOS OS DIAS** — Mesmo 30 minutos. Prática > teoria.
2. **Não colecione certs sem experiência** — Uma cert com projeto real > três certs sem lab.
3. **Participe de comunidades** — TFTEC, Microsoft Tech Community, meetups Azure.
4. **Contribua em projetos reais** — Voluntarie para projetos de ONG, startups, hackathons.
5. **Aprenda inglês técnico** — A documentação mais atual sempre está em inglês.
6. **Mantenha certs renovadas** — Renovação gratuita pelo Microsoft Learn, não deixe vencer.
7. **Construa em cima do anterior** — Cada certificação reforça a anterior. Não pule etapas.
8. **Acompanhe as novidades** — O Azure lança dezenas de features por mês. Siga o blog oficial.

---

## 📊 Resumo do Plano de Estudos

| Período | Foco Principal | Certificação | Horas/semana |
|---------|---------------|-------------|--------------|
| Ano 1, Sem 1 | Fundamentos de Redes | CompTIA N+ | 8–10h |
| Ano 1, Sem 2 | Cloud Basics + Azure | AZ-900 | 6–8h |
| Ano 2, Sem 1 | Azure Administration | AZ-104 | 10–12h |
| Ano 2, Sem 2 | Active Directory Híbrido | AZ-800 (opt) | 8–10h |
| Ano 3, Sem 1 | Azure Networking | AZ-700 | 12–15h |
| Ano 3, Sem 2 | Identity & Access | SC-300 | 10–12h |
| Ano 4, Sem 1 | Arquitetura de Soluções | AZ-305 | 12–15h |
| Ano 4, Sem 2 | Segurança Azure | AZ-500 | 10–12h |
| Ano 5, Sem 1 | Cybersecurity Architect | SC-100 | 12–15h |
| Ano 5, Sem 2 | Especialização + Liderança | AZ-400/Terraform | 8–10h |

---

*Guia criado em Março 2026. O mercado de cloud evolui rapidamente — revise e atualize este plano anualmente. Boa sorte na jornada! 🚀*

*Links verificados em: Março 2026*
