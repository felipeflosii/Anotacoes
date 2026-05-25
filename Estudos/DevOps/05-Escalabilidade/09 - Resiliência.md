---
tags: [devops, resiliencia, circuit-breaker, arquitetura, falhas]
aliases: [Resiliência, Resiliency, Circuit Breaker, Retry]
---

# 🛡️ Resiliência

> [!abstract] Definição
> **Resiliência** é a capacidade do sistema de **continuar funcionando (mesmo que de forma degradada) quando algum componente falha**, sem que a falha se propague e cause colapso total.

---

## A diferença para Alta Disponibilidade

> [!important] Lembre-se
> - [[08 - Alta Disponibilidade]] → evitar que o sistema fique indisponível (prevenção)
> - **Resiliência** → lidar bem quando algo inevitavelmente falha (resposta)

---

## Padrões de Resiliência

### 1. Retry (Tentativa Automática)

Quando uma operação falha, tenta novamente de forma automática.

```
Request → Serviço B → ❌ timeout
               ↓
         aguarda 1s
               ↓
         tenta novamente → ✅ sucesso
```

**Cuidado com retry storm:** muitos retries simultâneos podem sobrecarregar o serviço falho.

**Solução:** usar **exponential backoff** com jitter (aleatoriedade):
```
1a tentativa: aguarda 1s
2a tentativa: aguarda 2s
3a tentativa: aguarda 4s + jitter aleatório
```

---

### 2. Circuit Breaker (Disjuntor)

Inspirado no disjuntor elétrico: se houver muitas falhas seguidas, **"abre o circuito"** e para de tentar por um tempo.

```
Estado FECHADO (normal):
  requests passam → alguns erros → ...muitos erros...

Estado ABERTO (proteção):
  circuito abriu → requests retornam erro imediato
  → sem chamar o serviço falho → evita cascata

Estado HALF-OPEN (teste):
  após timeout → deixa um request passar
  → ok? fecha o circuito
  → erro? mantém aberto
```

**Por que é importante?**
```
❌ Sem Circuit Breaker:
Serviço A chama B → B está lento/caído
→ A fica esperando
→ threads de A se esgotam
→ A também fica indisponível
→ C que chama A também cai
→ CASCATA DE FALHAS 🔥

✅ Com Circuit Breaker:
B está caído → circuito abre
→ A retorna erro rápido (fail fast)
→ threads de A livres
→ sistema continua funcionando (degradado)
```

---

### 3. Fallback (Plano B)

Quando o serviço falha, retorna um resultado alternativo aceitável.

```
Serviço de recomendações → ❌ offline
        ↓
Fallback: retorna lista genérica de produtos populares
→ usuário vê algo, experiência não quebra completamente
```

Exemplos práticos:
- Serviço de pagamento lento → mostrar "processando, aguarde"
- CDN offline → servir imagem estática local padrão
- Cache expirado, banco lento → retornar cache antigo (stale cache)

---

### 4. Timeout

Sempre defina um **tempo máximo** para esperar por uma resposta.

```
❌ Sem timeout:
Request → Serviço externo
→ fica esperando... 30s... 1min... 5min...
→ thread bloqueada por tempo indefinido

✅ Com timeout configurado:
Request → Serviço externo
→ aguarda 3 segundos
→ timeout → erro rápido → fallback
```

---

### 5. Bulkhead (Antepara)

Isolar falhas em compartimentos para evitar que uma falha em um serviço consuma todos os recursos do sistema.

```
Thread pool isolada por serviço:
[Serviço A] → pool de 10 threads
[Serviço B] → pool de 10 threads (isolada)

→ Serviço B com problemas? Usa só suas 10 threads
→ Não afeta as threads do Serviço A
```

---

## Chaos Engineering

> [!note] Prática avançada
> **Chaos Engineering** é a prática de **intencionalmente injetar falhas** em produção para descobrir fraquezas antes que aconteçam de verdade.

```
"Se você não testa falhas, suas falhas vão te testar em produção."
```

Ferramenta famosa: **Chaos Monkey** (Netflix) — desliga instâncias aleatoriamente em produção para garantir que o sistema é realmente resiliente.

---

## Resumo Visual

```
Falha detectada
      ↓
Retry automático → resolveu? ✅ fim
      ↓ não
Circuit Breaker abre → fail fast
      ↓
Fallback executado → resposta degradada
      ↓
Observabilidade detecta → [[10 - Observabilidade]]
      ↓
Alerta para o time → investigação e correção
```

---

## Links relacionados

- [[08 - Alta Disponibilidade]] — prevenção de downtime
- [[10 - Observabilidade]] — detectar falhas e acionar respostas
- [[07 - Escalabilidade]] — escalar para lidar com falhas
- [[🗺️ MOC — DevOps]] — voltar ao mapa

