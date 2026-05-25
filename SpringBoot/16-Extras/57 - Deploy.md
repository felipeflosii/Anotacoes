# 57 — Deploy

tags: #springboot #deploy #railway #render #extras
links: [[56 - Docker para Spring Boot]] | [[🗺️ Mapa Principal]]

---

## Opções de deploy para projetos Spring Boot

| Plataforma | Custo | Dificuldade | Ideal para |
|---|---|---|---|
| **Railway** | Gratuito (trial) / ~$5/mês | ⭐ Fácil | Projetos acadêmicos e portfólio |
| **Render** | Gratuito (dorme em 15min) / $7/mês | ⭐ Fácil | Portfólio |
| **Fly.io** | Gratuito (limitado) | ⭐⭐ Médio | APIs pequenas |
| **AWS Elastic Beanstalk** | ~$15/mês | ⭐⭐⭐ Difícil | Produção real |
| **GCP Cloud Run** | Pay-per-use | ⭐⭐ Médio | Produção escalável |
| **VPS (DigitalOcean)** | $6/mês | ⭐⭐⭐ Difícil | Controle total |

---

## Deploy no Railway (recomendado para iniciantes)

### Passo 1 — Preparar o projeto

```yaml
# application-prod.yml — usa variáveis de ambiente do Railway
spring:
  datasource:
    url: ${DATABASE_URL}          # Railway injeta automaticamente
  jpa:
    hibernate:
      ddl-auto: none
  flyway:
    enabled: true

server:
  port: ${PORT:8080}              # Railway define a porta

app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 86400000
```

### Passo 2 — Deploy

```bash
# 1. Criar conta em railway.app
# 2. Instalar CLI:
npm install -g @railway/cli

# 3. Login
railway login

# 4. Na pasta do projeto:
railway init

# 5. Adicionar banco PostgreSQL:
# No dashboard do Railway: Add Service → PostgreSQL
# Railway cria a variável DATABASE_URL automaticamente

# 6. Definir variáveis:
railway variables set JWT_SECRET=suachave
railway variables set SPRING_PROFILES_ACTIVE=prod

# 7. Deploy:
railway up

# 8. Ver logs:
railway logs
```

### Passo 3 — Configurar domínio

No dashboard do Railway: Settings → Domains → Generate Domain
Você recebe uma URL como `https://concessionaria-api.railway.app`

---

## Deploy no Render

```yaml
# render.yaml — arquivo de configuração do Render
services:
  - type: web
    name: concessionaria-api
    env: docker
    dockerfilePath: ./Dockerfile
    envVars:
      - key: SPRING_PROFILES_ACTIVE
        value: prod
      - key: JWT_SECRET
        generateValue: true      # Render gera automaticamente
      - key: DATABASE_URL
        fromDatabase:
          name: concessionaria-db
          property: connectionString

databases:
  - name: concessionaria-db
    databaseName: concessionaria
    user: postgres
    plan: free
```

---

## Deploy em VPS — Ubuntu 22.04

```bash
# === NO SERVIDOR ===

# 1. Instalar Java 21
sudo apt update
sudo apt install -y openjdk-21-jre-headless

# 2. Criar usuário para a aplicação
sudo useradd -m -s /bin/bash appuser

# 3. Copiar o JAR (do CI/CD ou scp)
scp target/concessionaria-api.jar user@servidor:/home/appuser/

# 4. Criar arquivo de serviço systemd
sudo nano /etc/systemd/system/concessionaria.service
```

```ini
# /etc/systemd/system/concessionaria.service
[Unit]
Description=Concessionaria API
After=network.target

[Service]
User=appuser
WorkingDirectory=/home/appuser
ExecStart=/usr/bin/java \
  -Xmx512m \
  -jar /home/appuser/concessionaria-api.jar
Environment=SPRING_PROFILES_ACTIVE=prod
Environment=DB_HOST=localhost
Environment=DB_PORT=5432
Environment=DB_NAME=concessionaria
Environment=DB_USER=app_user
Environment=DB_PASSWORD=senha_segura
Environment=JWT_SECRET=chave_segura_aqui
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# 5. Iniciar e habilitar o serviço
sudo systemctl daemon-reload
sudo systemctl start concessionaria
sudo systemctl enable concessionaria   # inicia automaticamente no boot

# 6. Ver status e logs
sudo systemctl status concessionaria
sudo journalctl -u concessionaria -f

# 7. Configurar Nginx como proxy reverso
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/concessionaria
```

```nginx
# /etc/nginx/sites-available/concessionaria
server {
    listen 80;
    server_name api.seudominio.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
# 8. Habilitar site e SSL (Let's Encrypt)
sudo ln -s /etc/nginx/sites-available/concessionaria /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl restart nginx
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d api.seudominio.com
```

---

## CI/CD básico com GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy para Railway

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Java 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: maven

      - name: Rodar testes
        run: ./mvnw test

      - name: Build JAR
        run: ./mvnw clean package -DskipTests

      - name: Deploy no Railway
        uses: bervProject/railway-deploy@v1.0.0
        with:
          railway_token: ${{ secrets.RAILWAY_TOKEN }}
          service: concessionaria-api
```

---

## Checklist de deploy em produção

```
✅ Variáveis de ambiente definidas (nunca hardcode de senhas)
✅ SPRING_PROFILES_ACTIVE=prod
✅ JWT_SECRET forte (32+ bytes aleatórios)
✅ ddl-auto=none (Flyway gerencia o schema)
✅ show-sql=false
✅ Swagger desabilitado ou protegido
✅ Logs configurados (sem DEBUG em prod)
✅ CORS configurado para o domínio correto
✅ HTTPS habilitado
✅ Health check configurado (/actuator/health)
✅ Backup do banco configurado
✅ Monitoramento básico (UptimeRobot, etc.)
```

---

## Próximas notas
- [[49 - Projeto Concessionária - Visão Geral]] — revisar o projeto completo
- [[🗺️ Mapa Principal]] — índice geral da apostila
