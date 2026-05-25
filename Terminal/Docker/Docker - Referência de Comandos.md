# 🐳 Docker — Referência de Comandos

> [!info] O que é Docker?
> Docker é uma plataforma para criar, empacotar e executar aplicações em **containers** isolados. Um container é como uma "caixa" que contém o app + todas as dependências, rodando de forma consistente em qualquer ambiente.

---

## 📦 Comandos Comuns (Common Commands)

| Comando | O que faz |
|--------|-----------|
| `docker run` | Cria e inicia um novo container a partir de uma imagem |
| `docker exec` | Executa um comando dentro de um container **já em execução** |
| `docker ps` | Lista containers em execução (use `-a` para ver todos) |
| `docker build` | Constrói uma imagem a partir de um `Dockerfile` |
| `docker bake` | Constrói imagens a partir de um arquivo de configuração (bake file) |
| `docker pull` | Baixa uma imagem do registry (ex: Docker Hub) |
| `docker push` | Envia uma imagem local para um registry |
| `docker images` | Lista todas as imagens disponíveis localmente |
| `docker login` | Autentica em um registry (ex: Docker Hub, GHCR) |
| `docker logout` | Encerra a sessão do registry |
| `docker search` | Busca imagens disponíveis no Docker Hub |
| `docker version` | Exibe a versão do Docker instalada |
| `docker info` | Exibe informações do sistema Docker (containers, imagens, recursos) |

---

## 🗂️ Comandos de Gerenciamento (Management Commands)

> [!tip] Management Commands vs Common Commands
> Management Commands organizam os recursos por **categoria** e permitem subcomandos mais específicos (ex: `docker container ls`, `docker volume inspect`).

| Comando | O que gerencia |
|--------|----------------|
| `docker builder` | Builds (histórico, cache, pruning) |
| `docker buildx` | Builds avançados com suporte a multi-plataforma |
| `docker compose` | Orquestração de múltiplos containers com `docker-compose.yml` |
| `docker container` | Gerencia containers individualmente |
| `docker context` | Gerencia contextos de conexão com daemons remotos |
| `docker image` | Gerencia imagens (build, tag, inspect, prune) |
| `docker manifest` | Gerencia manifestos de imagens multi-plataforma |
| `docker model` | Docker Model Runner (execução de modelos de IA localmente) |
| `docker network` | Gerencia redes (criação, remoção, inspeção) |
| `docker plugin` | Gerencia plugins instalados no Docker |
| `docker system` | Operações globais do sistema (prune, df, events) |
| `docker volume` | Gerencia volumes para persistência de dados |
| `docker swarm` | Orquestração nativa de containers em cluster (Swarm mode) |

---

## ⚙️ Comandos Avançados

### 🔄 Ciclo de Vida de Containers

| Comando | O que faz |
|--------|-----------|
| `docker create` | Cria um container sem iniciá-lo |
| `docker start` | Inicia um container parado |
| `docker stop` | Para um container de forma graciosa (SIGTERM) |
| `docker restart` | Reinicia um container |
| `docker kill` | Força a parada imediata de um container (SIGKILL) |
| `docker pause` | Pausa todos os processos dentro do container |
| `docker unpause` | Retoma os processos pausados |
| `docker wait` | Bloqueia até o container parar e imprime o exit code |
| `docker rm` | Remove um ou mais containers |

### 🔍 Inspeção e Debug

| Comando | O que faz |
|--------|-----------|
| `docker logs` | Exibe os logs de um container |
| `docker inspect` | Retorna informações detalhadas (JSON) de qualquer objeto Docker |
| `docker stats` | Mostra uso de CPU, memória e rede em tempo real |
| `docker top` | Lista os processos em execução dentro de um container |
| `docker diff` | Mostra alterações no sistema de arquivos do container |
| `docker events` | Exibe eventos em tempo real do daemon Docker |
| `docker port` | Lista os mapeamentos de porta de um container |
| `docker attach` | Conecta o terminal local aos streams de um container em execução |

### 📁 Arquivos e Imagens

| Comando | O que faz |
|--------|-----------|
| `docker cp` | Copia arquivos entre o host e um container |
| `docker commit` | Cria uma nova imagem a partir das alterações de um container |
| `docker tag` | Cria uma tag para uma imagem (ex: `myapp:v1.0`) |
| `docker rmi` | Remove uma ou mais imagens locais |
| `docker history` | Exibe o histórico de camadas de uma imagem |
| `docker save` | Salva uma imagem em um arquivo `.tar` |
| `docker load` | Carrega uma imagem de um arquivo `.tar` |
| `docker export` | Exporta o filesystem de um container como `.tar` |
| `docker import` | Cria uma imagem a partir de um `.tar` exportado |
| `docker update` | Atualiza configurações de recursos de um container em execução |

---

## 🌐 Opções Globais (Global Options)

> [!warning] Estas opções vão **antes** do comando
> Ex: `docker --debug ps` ou `docker -H tcp://host:2376 ps`

| Flag | O que faz |
|------|-----------|
| `--config <path>` | Define o diretório de configuração do cliente (padrão: `~/.docker`) |
| `-c, --context <nome>` | Usa um contexto específico para conectar ao daemon |
| `-D, --debug` | Ativa o modo debug (logs detalhados) |
| `-H, --host <socket>` | Define o socket do daemon Docker a conectar |
| `-l, --log-level` | Define o nível de log: `debug`, `info`, `warn`, `error`, `fatal` |
| `--tls` | Força uso de TLS na conexão |
| `--tlsverify` | Usa TLS e verifica o certificado remoto |
| `--tlscacert <path>` | Certificado CA confiável para TLS |
| `--tlscert <path>` | Caminho para o certificado TLS do cliente |
| `--tlskey <path>` | Caminho para a chave privada TLS |
| `-v, --version` | Imprime a versão e encerra |

---

## 🧩 Conceitos Relacionados

- [[Docker Compose]] — orquestração de múltiplos serviços
- [[Dockerfile]] — arquivo para definir como buildar uma imagem
- [[Docker Volumes]] — persistência de dados fora do container
- [[Docker Networks]] — comunicação entre containers
- [[Docker Hub]] — registry público de imagens

---

## 💡 Exemplos Rápidos

```bash
# Subir um container do PostgreSQL
docker run -d -e POSTGRES_PASSWORD=senha -p 5432:5432 postgres

# Ver logs em tempo real
docker logs -f meu-container

# Entrar no bash de um container em execução
docker exec -it meu-container bash

# Remover todos os containers parados
docker container prune

# Ver uso de espaço em disco pelo Docker
docker system df
```

---

*Fonte: `docker --help` | Referência oficial: https://docs.docker.com*
