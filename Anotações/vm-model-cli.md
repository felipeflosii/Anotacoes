---
title: "Provisionamento de Infraestrutura Azure com Azure CLI"
aliases:
  - "IaC Azure CLI"
  - "Script Azure VM Ubuntu + Docker"
tags:
  - azure
  - azure-cli
  - iac
  - devops
  - redes
  - docker
  - ubuntu
  - obsidian
created: 2026-05-25
updated: 2026-05-25
status: "pronto"
category: "Cloud / Infraestrutura"
---

# Provisionamento de Infraestrutura Azure com Azure CLI

> [!info] Objetivo do Script
> Esse script automatiza a criação de uma infraestrutura básica na Azure usando **Azure CLI**. Ele faz:
> - Criação de um **Resource Group**
> - Criação de uma **Virtual Network (VNet)**
> - Criação de uma **Subnet**
> - Criação de um **Network Security Group (NSG)**
> - Criação de uma **Máquina Virtual Ubuntu**
> - Liberação de portas no firewall
> - Instalação automática do Docker
> - Exibição do IP público da VM

---

## Estrutura Geral

```bash
#!/usr/bin/env bash
```

### O que é isso?
Isso é chamado de **Shebang**.

Ele informa ao Linux qual interpretador deve executar o arquivo.

Neste caso:

```bash
/usr/bin/env bash
```

→ Executa o script usando o **Bash**.

---

## Variáveis do Script

```bash
RG="rg-challenge-clyvo-vet"
LOCATION="chilecentral"
VM="vm-wise-clyvo-dev-01"
```

### O que são essas variáveis?

- **RG**: Nome do Resource Group.
  - Resource Group é um agrupador de recursos da Azure.
  - Tudo ficará dentro dele:
    - VM
    - Rede
    - Firewall
    - Disco
    - IP Público

- **LOCATION**: Região onde os recursos serão criados.
  - `chilecentral` significa datacenter da Azure localizado no Chile.

- **VM**: Nome da máquina virtual.

---

## 1) Criar Resource Group

```bash
az group create --name "$RG" --location "$LOCATION"
```

### O que acontece aqui?
Cria um **Resource Group** na Azure.

### Explicação do comando
- `az`: CLI oficial da Azure.
- `group create`: Cria um grupo de recursos.
- `--name "$RG"`: Define o nome do grupo (`rg-challenge-clyvo-vet`).
- `--location "$LOCATION"`: Define onde os recursos serão armazenados fisicamente.

> [!tip] Na prática
> É como criar uma **“pasta” organizadora** na Azure.

---

## 2) Criar VNet + Subnet

```bash
az network vnet create \
  --resource-group "$RG" --name "vnet_wise_dev" \
  --address-prefixes 10.10.0.0/16 \
  --subnet-name "sub_net_dev" --subnet-prefixes 10.10.1.0/24
```

### O que é uma VNet?
**VNet = Virtual Network**.

É uma rede privada dentro da Azure.

Funciona como a rede do seu roteador de casa, mas dentro da nuvem.

### Explicando os parâmetros
- `vnet create`: Cria uma rede virtual.
- `--resource-group "$RG"`: Coloca a VNet dentro do Resource Group.
- `--name "vnet_wise_dev"`: Nome da rede virtual.
- `--address-prefixes 10.10.0.0/16`: Define o intervalo de IPs da rede.

### O que significa `/16`?
É CIDR de rede:
- Faixa: `10.10.0.0` até `10.10.255.255`
- Total aproximado: **~65 mil IPs**.

### Subnet
- `--subnet-name "sub_net_dev"`: Cria uma subdivisão da rede.
- `--subnet-prefixes 10.10.1.0/24`: Define o bloco da subnet.

Faixa da subnet:
- `10.10.1.0` até `10.10.1.255`
- Total: **256 IPs**.

### Relação VNet e Subnet
```text
VNet
└── Subnet
    └── VM
```

---

## 3) Criar NSG

```bash
az network nsg create --resource-group "$RG" --name "nsg_portalweb_dev"
```

### O que é NSG?
**NSG = Network Security Group**.

É basicamente o firewall da Azure.

Ele controla:
- Quais portas podem entrar
- Quais conexões são permitidas

> [!warning] Na prática
> Ele protege sua VM contra acessos indevidos.

---

## 4) Criar Máquina Virtual

```bash
az vm create \
  --resource-group "$RG" --name "$VM" \
  --image Ubuntu2204 --size Standard_B4ls_v2 \
  --admin-username azureuser --generate-ssh-keys \
  --vnet-name "vnet_wise_dev" --subnet "sub_net_dev" --nsg "nsg_portalweb_dev"
```

### O que acontece aqui?
Cria uma VM Linux Ubuntu dentro da rede criada anteriormente.

### Explicando cada parâmetro
- `vm create`: Cria uma máquina virtual.
- `--image Ubuntu2204`: Sistema operacional da VM (**Ubuntu Server 22.04**).
- `--size Standard_B4ls_v2`: Define o hardware da VM.

### O que muda no size?
Ele define:
- CPU
- Memória RAM
- Performance

- `--admin-username azureuser`: Usuário administrador da VM.
- `--generate-ssh-keys`: Gera automaticamente chave pública e privada para SSH seguro.

### Rede da VM
- `--vnet-name "vnet_wise_dev"`
- `--subnet "sub_net_dev"`

Conecta a VM na rede criada.

### Associação do Firewall
- `--nsg "nsg_portalweb_dev"`

Liga o firewall à VM.

---

## 5) Abrir portas no Firewall

### SSH (porta 22)
```bash
az vm open-port --resource-group "$RG" --name "$VM" --port 22 --priority 1000
```

Porta 22 é usada para:
- SSH
- Acesso remoto Linux

### Porta 8080
```bash
az vm open-port --resource-group "$RG" --name "$VM" --port 8080 --priority 1001
```

Muito usada para:
- APIs Java
- Spring Boot
- Aplicações web

### Porta 1521
```bash
az vm open-port --resource-group "$RG" --name "$VM" --port 1521 --priority 1002
```

Porta padrão do:
- Oracle Database

### O que é prioridade?
`--priority 1000`

O firewall da Azure funciona por ordem.

**Menor número = maior prioridade.**

---

## 6) Instalar Docker automaticamente

```bash
az vm run-command invoke \
  --resource-group "$RG" --name "$VM" \
  --command-id RunShellScript \
  --scripts "sudo apt-get update && sudo apt-get install -y git curl ca-certificates && curl -fsSL https://get.docker.com | sudo sh && sudo usermod -aG docker azureuser"
```

### O que isso faz?
Executa comandos Linux diretamente dentro da VM remotamente.

`run-command invoke` permite executar scripts na VM sem acessar SSH manualmente.

### Sequência dos comandos
1. **Atualizar pacotes**
   ```bash
   sudo apt-get update
   ```
   Atualiza a lista de pacotes do Ubuntu.

2. **Instalar dependências**
   ```bash
   sudo apt-get install -y git curl ca-certificates
   ```
   Instala:
   - Git
   - Curl
   - Certificados HTTPS

3. **Instalar Docker**
   ```bash
   curl -fsSL https://get.docker.com | sudo sh
   ```
   Baixa e instala Docker automaticamente.

4. **Dar permissão de uso do Docker ao usuário**
   ```bash
   sudo usermod -aG docker azureuser
   ```
   Adiciona o usuário ao grupo Docker.

Assim ele consegue usar comandos como `docker ps` sem precisar de `sudo`.

---

## 7) Mostrar IP Público

```bash
az vm show --resource-group "$RG" --name "$VM" \
  --show-details --query publicIps --output tsv
```

### O que isso faz?
Exibe o IP público da VM.

### Explicando os parâmetros
- `vm show`: Mostra informações da VM.
- `--show-details`: Mostra detalhes completos.
- `--query publicIps`: Filtra somente o IP público.
- `--output tsv`: Exibe saída limpa (ex.: `20.xxx.xxx.xxx`) sem JSON.

---

## Fluxo completo da infraestrutura

```text
Resource Group
│
├── VNet
│   └── Subnet
│       └── VM Ubuntu
│
├── NSG (Firewall)
│   ├── Porta 22
│   ├── Porta 8080
│   └── Porta 1521
│
└── IP Público
```

---

## Conceitos importantes envolvidos

### Infraestrutura como Código (IaC)
Esse script é um exemplo de **Infrastructure as Code**.

Ou seja: infraestrutura criada via código.

### Benefícios
- **Automação**: não precisa criar tudo manualmente.
- **Repetibilidade**: recria o ambiente várias vezes.
- **Padronização**: todos os ambientes ficam iguais.
- **Velocidade**: provisionamento em minutos.

### Tecnologias usadas
- Bash
- Azure CLI
- Microsoft Azure
- Linux Ubuntu
- Docker
- Redes Virtuais
- Firewall (NSG)

---

## Resultado final

Ao terminar, você terá:
- ✅ VM Ubuntu funcionando
- ✅ Rede configurada
- ✅ Firewall configurado
- ✅ Docker instalado
- ✅ IP público disponível
- ✅ Ambiente pronto para deploy de aplicações e banco Oracle
