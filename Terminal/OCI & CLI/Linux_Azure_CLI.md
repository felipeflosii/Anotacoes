# ☁️ Azure CLI — Referência de Comandos

> Comandos `az` para gerenciar recursos na Azure via terminal. Zorin OS / Ubuntu.

---

## 🔧 Instalação e Autenticação

```bash
# Instalar Azure CLI (Zorin OS / Ubuntu)
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

az version                                 # verifica versão instalada
az login                                   # abre browser para login
az login --use-device-code                 # login por código (sem browser)
az logout                                  # encerra sessão
```

---

## 📋 Conta e Subscriptions

```bash
az account list --output table             # lista subscriptions
az account show                            # subscription ativa
az account set --subscription "nome ou id" # muda subscription ativa
```

---

## 🗂️ Resource Groups

```bash
az group list --output table               # lista todos os RGs
az group create --name meu-rg --location brazilsouth
az group show --name meu-rg
az group delete --name meu-rg --yes        # deleta RG e todos os recursos
az group delete --name meu-rg --yes --no-wait  # deleta sem aguardar
```

---

## 🖥️ Máquinas Virtuais

```bash
# Listar
az vm list --output table
az vm list --resource-group meu-rg --output table

# Criar VM Ubuntu
az vm create \
  --resource-group meu-rg \
  --name minha-vm \
  --image Ubuntu2204 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --size Standard_B1s

# Ligar / Desligar / Reiniciar
az vm start  --resource-group meu-rg --name minha-vm
az vm stop   --resource-group meu-rg --name minha-vm
az vm restart --resource-group meu-rg --name minha-vm
az vm deallocate --resource-group meu-rg --name minha-vm  # para e desaloca

# Deletar
az vm delete --resource-group meu-rg --name minha-vm --yes

# IP público
az vm list-ip-addresses --resource-group meu-rg --name minha-vm --output table
```

---

## 🌐 Rede

```bash
# Abrir porta (NSG rule)
az vm open-port --resource-group meu-rg --name minha-vm --port 8080
az vm open-port --resource-group meu-rg --name minha-vm --port 443 --priority 110

# Listar NSGs
az network nsg list --resource-group meu-rg --output table

# Listar regras de um NSG
az network nsg rule list --resource-group meu-rg --nsg-name meu-nsg --output table
```

---

## 🐳 Container Instances (ACI)

```bash
# Criar container
az container create \
  --resource-group meu-rg \
  --name meu-container \
  --image nginx \
  --ports 80 \
  --dns-name-label meu-app-dns \
  --os-type Linux

# Listar / Status
az container list --output table
az container show --resource-group meu-rg --name meu-container

# Logs
az container logs --resource-group meu-rg --name meu-container

# Deletar
az container delete --resource-group meu-rg --name meu-container --yes
```

---

## 🗄️ Azure Container Registry (ACR)

```bash
# Criar registry
az acr create --resource-group meu-rg --name meuregistry --sku Basic

# Login no registry
az acr login --name meuregistry

# Listar imagens
az acr repository list --name meuregistry --output table

# Build e push direto no ACR
az acr build --registry meuregistry --image minha-app:v1 .
```

---

## 🔑 SSH em VM

```bash
# Pegar IP e conectar
IP=$(az vm list-ip-addresses \
  --resource-group meu-rg \
  --name minha-vm \
  --query "[0].virtualMachine.network.publicIpAddresses[0].ipAddress" \
  --output tsv)

ssh azureuser@$IP
```

---

## 📊 Monitoramento e Logs

```bash
az monitor activity-log list --output table  # log de atividade
az monitor metrics list --resource <resource-id> --metric-names "Percentage CPU"
```

---

## ⚡ Atalhos Úteis

| Comando | Descrição |
|---------|-----------|
| `az configure --defaults group=meu-rg` | define RG padrão |
| `az find "az vm"` | busca comandos relacionados |
| `az <comando> --help` | ajuda de qualquer comando |
| `--output table` | saída em tabela legível |
| `--output json` | saída em JSON |
| `--query "[0].name"` | filtra campo com JMESPath |
