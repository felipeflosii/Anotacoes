---
tags: [devops, iac, terraform, ansible, infraestrutura]
aliases: [IaC, Infraestrutura como Código, Infrastructure as Code]
---

# 🏗️ Infraestrutura como Código (IaC)

> [!abstract] Definição
> **Infrastructure as Code (IaC)** é a prática de definir e gerenciar infraestrutura (servidores, redes, bancos, DNS…) através de **arquivos de código versionados**, ao invés de configurações manuais.

---

## O Problema Antes da IaC

```
❌ Antes — "ClickOps":
→ Sysadmin entra no painel da AWS
→ Clica para criar servidor
→ Configura manualmente na mão
→ Documento? "Tá na minha cabeça"
→ Servidor quebrou? Ninguém sabe recriar
→ Dev e produção com configs diferentes
```

---

## A Solução: IaC

```hcl
# main.tf (Terraform)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name        = "servidor-web"
    Environment = "producao"
  }
}
```

```
✅ Com IaC:
→ Infraestrutura declarada em arquivo
→ Arquivo versionado no Git
→ Qualquer pessoa pode recriar o ambiente
→ Dev == Staging == Produção
→ Rollback é um git revert
```

---

## Declarativo vs Imperativo

| Estilo | Como funciona | Exemplo |
|---|---|---|
| **Declarativo** | Você diz **o que quer**, a ferramenta decide como | Terraform, CloudFormation |
| **Imperativo** | Você diz **como fazer**, passo a passo | Ansible (modo ad-hoc), scripts Bash |

> [!tip] Preferência do mercado
> Ferramentas declarativas são geralmente preferidas para provisionamento, pois a ferramenta gerencia o estado e evita conflitos.

---

## Principais Ferramentas

### Terraform
- Declarativo, multi-cloud
- Gerencia estado da infraestrutura
- HCL (HashiCorp Configuration Language)
- Plano: `terraform plan` → Aplicar: `terraform apply`

```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws" }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "meu_bucket" {
  bucket = "meu-bucket-devops-exemplo"
}
```

### Ansible
- Imperativo/procedural
- Agentless (usa SSH)
- YAML — Playbooks
- Bom para **configuração** e **provisionamento de software**

```yaml
# playbook.yml
- name: Configurar servidor web
  hosts: servidores
  become: yes
  tasks:
    - name: Instalar nginx
      apt:
        name: nginx
        state: present

    - name: Iniciar nginx
      service:
        name: nginx
        state: started
```

### Comparação: Terraform vs Ansible

| | Terraform | Ansible |
|---|---|---|
| **Foco** | Provisionar infra | Configurar software |
| **Estado** | Gerencia estado | Stateless |
| **Linguagem** | HCL | YAML |
| **Uso ideal** | Criar VMs, redes, buckets | Instalar apps, configs |

---

## Vantagens da IaC

- **Versionamento** — cada mudança é um commit rastreável
- **Reprodutibilidade** — ambiente idêntico em qualquer lugar
- **Rollback fácil** — revert no Git, aplica novamente
- **Revisão via PR** — mudanças na infra passam por code review
- **Documentação viva** — o código é a documentação

---

## GitOps

> [!note] Conceito avançado
> **GitOps** leva IaC ao extremo: o Git é a **fonte única de verdade** para tudo — código, infra e configuração. Qualquer mudança passa pelo Git, e ferramentas como ArgoCD sincronizam automaticamente.

```
Dev faz PR com mudança na infra
    ↓
Code review aprovado → merge
    ↓
ArgoCD / Flux detecta mudança no Git
    ↓
Aplica automaticamente no cluster Kubernetes
```

---

## Links relacionados

- [[05 - Automação]] — IaC é automação de infraestrutura
- [[07 - Escalabilidade]] — escalar com IaC
- [[08 - Alta Disponibilidade]] — ambientes redundantes com IaC
- [[🗺️ MOC — DevOps]] — voltar ao mapa

