---
tags: [devops, escalabilidade, cloud, arquitetura]
aliases: [Escalabilidade, Scaling, Scale]
---

# 📈 Escalabilidade

> [!abstract] Definição
> **Escalabilidade** é a capacidade do sistema de **lidar com aumento de carga** sem perder performance ou disponibilidade — e de fazer isso de forma eficiente e econômica.

---

## Tipos de Escalabilidade

### Escala Vertical (Scale Up)
Aumentar os **recursos da mesma máquina**.

```
Antes:  [Servidor 2 CPU / 4GB RAM]
Depois: [Servidor 8 CPU / 32GB RAM]
```

**Prós:** simples, sem mudança na arquitetura
**Contras:** tem limite físico, ponto único de falha, caro

---

### Escala Horizontal (Scale Out)
Adicionar **mais instâncias** do mesmo serviço.

```
Antes:  [Servidor A]

Depois: [Servidor A]
        [Servidor B]  ← nova instância
        [Servidor C]  ← nova instância
             ↑
        [Load Balancer] distribui o tráfego
```

**Prós:** sem limite teórico, alta disponibilidade, mais barato
**Contras:** requer arquitetura stateless, mais complexidade

> [!tip] Preferência em DevOps
> DevOps e Cloud favorecem fortemente **escala horizontal**. É mais resiliente, econômica e alinhada com arquiteturas modernas.

---

## Auto-Scaling

O sistema **escala sozinho** de acordo com a demanda.

```
Demanda baixa → 2 instâncias rodando
        ↓
Pico de tráfego detectado
        ↓
Auto-scaler dispara novas instâncias
        ↓
Demanda alta → 10 instâncias rodando
        ↓
Tráfego normaliza → instâncias desligadas
        ↓
Demanda baixa → 2 instâncias rodando
```

### Exemplos de auto-scaling:
- **AWS Auto Scaling Groups** — escala EC2 automaticamente
- **Kubernetes HPA** (Horizontal Pod Autoscaler) — escala pods por CPU/memória
- **Kubernetes KEDA** — escala por eventos (filas, tópicos Kafka, etc.)

---

## Load Balancer

Distribui o tráfego entre múltiplas instâncias.

```
Usuários → [Load Balancer]
                 ├── [Instância 1]
                 ├── [Instância 2]
                 └── [Instância 3]
```

Algoritmos comuns:
- **Round Robin** — alterna entre instâncias na ordem
- **Least Connections** — manda para a instância com menos conexões
- **IP Hash** — mesmo usuário sempre vai para a mesma instância

---

## Estateless vs Stateful

> [!important] Para escalar horizontalmente, a aplicação precisa ser **stateless**

**Stateful (difícil de escalar):**
```
Usuário faz login → Sessão guardada na Instância A
Próxima request → cai na Instância B → usuário não está logado! ❌
```

**Stateless (fácil de escalar):**
```
Sessão guardada em Redis/banco externo
Qualquer instância pode atender qualquer request ✅
```

---

## DevOps + Escalabilidade

No contexto DevOps, escalar bem significa:

- **IaC** para criar/destruir infra automaticamente → [[06 - Infraestrutura como Código (IaC)]]
- **Pipelines** que fazem deploy em múltiplas instâncias → [[04 - CD — Entrega e Deploy Contínuos]]
- **Observabilidade** para detectar quando escalar → [[10 - Observabilidade]]
- **Containers** para facilitar a replicação de instâncias (Docker, Kubernetes)

---

## Links relacionados

- [[08 - Alta Disponibilidade]] — escalabilidade + redundância
- [[06 - Infraestrutura como Código (IaC)]] — automatizar o scaling
- [[10 - Observabilidade]] — detectar necessidade de escalar
- [[🗺️ MOC — DevOps]] — voltar ao mapa

