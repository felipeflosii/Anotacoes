# 🏛️ OCI CLI — Referência de Comandos

> Comandos `oci` para gerenciar recursos na Oracle Cloud Infrastructure via terminal.

---

## 🔧 Instalação e Configuração

```bash
# Instalar OCI CLI (Zorin OS / Ubuntu)
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"

oci --version                              # verifica versão

# Configurar credenciais
oci setup config                           # assistente interativo
# Gera: ~/.oci/config
```

### Estrutura do `~/.oci/config`
```ini
[DEFAULT]
user=ocid1.user.oc1..xxx
fingerprint=xx:xx:xx:...
tenancy=ocid1.tenancy.oc1..xxx
region=sa-saopaulo-1
key_file=~/.oci/oci_api_key.pem
```

---

## 📋 Compartimentos (Compartments)

```bash
oci iam compartment list                   # lista compartimentos
oci iam compartment list --output table

# Pegar OCID do compartimento por nome
oci iam compartment list \
  --query "data[?name=='meu-compartimento'].id" \
  --output json
```

---

## 🖥️ Compute — Instâncias (VMs)

```bash
# Listar instâncias
oci compute instance list \
  --compartment-id <compartment-ocid> \
  --output table

# Detalhes de uma instância
oci compute instance get --instance-id <instance-ocid>

# Ligar / Desligar
oci compute instance action \
  --instance-id <instance-ocid> \
  --action START

oci compute instance action \
  --instance-id <instance-ocid> \
  --action STOP

oci compute instance action \
  --instance-id <instance-ocid> \
  --action SOFTRESET

# Deletar instância
oci compute instance terminate \
  --instance-id <instance-ocid> \
  --preserve-boot-volume false
```

---

## 🌐 Rede (VCN / Subnets)

```bash
# Listar VCNs
oci network vcn list --compartment-id <compartment-ocid> --output table

# Listar subnets
oci network subnet list --compartment-id <compartment-ocid> --output table

# Listar security lists
oci network security-list list \
  --compartment-id <compartment-ocid> \
  --output table
```

---

## 💾 Block Storage

```bash
# Listar volumes
oci bv volume list --compartment-id <compartment-ocid> --output table

# Listar anexos de volume
oci compute volume-attachment list \
  --compartment-id <compartment-ocid> \
  --output table
```

---

## 🗄️ Object Storage

```bash
# Listar buckets
oci os bucket list --compartment-id <compartment-ocid>

# Criar bucket
oci os bucket create \
  --compartment-id <compartment-ocid> \
  --name meu-bucket

# Upload de arquivo
oci os object put \
  --bucket-name meu-bucket \
  --file ./arquivo.txt \
  --name arquivo.txt

# Listar objetos
oci os object list --bucket-name meu-bucket

# Download
oci os object get \
  --bucket-name meu-bucket \
  --name arquivo.txt \
  --file ./download.txt

# Deletar objeto
oci os object delete --bucket-name meu-bucket --name arquivo.txt
```

---

## 🔑 SSH em Instância

```bash
# Pegar IP público
oci compute instance list-vnics \
  --instance-id <instance-ocid> \
  --query "data[0].\"public-ip\"" \
  --output json

ssh -i ~/.ssh/id_rsa opc@<ip-publico>
```

> O usuário padrão das imagens Oracle é `opc`, não `ubuntu`.

---

## ⚡ Atalhos Úteis

| Comando | Descrição |
|---------|-----------|
| `--output table` | saída em tabela legível |
| `--output json` | saída em JSON |
| `--query "data[0].id"` | filtra campo com JMESPath |
| `oci iam region list` | lista regiões disponíveis |
| `oci --help` | ajuda geral |
| `oci compute --help` | ajuda de um serviço |
