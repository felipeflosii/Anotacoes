## Sequência do vídeo

### 1. Máquina local — criar infra

```bash
cd ~/Documentos/FIAP/Challenge/java-base/JornadaPet-java-Sprint1
bash deploy.sh
```

> Aguarda terminar e mostra o IP no final.

---

### 2. Máquina local — SSH na VM

```bash
ssh azureuser@57.156.56.214
```

---

### 3. Dentro da VM — deploy

```bash
git clone https://github.com/Challenge-2TDSPG-2026/JornadaPet-java-Sprint1.git challenge-clyvo
cd challenge-clyvo
mkdir -p sql
docker pull felipeflosii/challenge-clyvo-vet:v1
docker pull gvenzl/oracle-xe:21-slim
docker compose up -d
```

Aguarda ~2min e:

```bash
docker compose ps
docker exec jornadapet-app whoami
docker volume ls
docker exec jornadapet-app printenv | grep SPRING_DATASOURCE
```

---

### 4. Máquina local — CRUD externo

```bash
# Register + Login
curl -X POST http://57.156.56.200:8080/auth/register \
  -H "Content-Type: application/json" \
  -d '{"nome":"Tutor Teste","email":"tutor@fiap.com.br","senha":"123456","telefone":"11999990003","cpf":"33333333333"}'

TOKEN=$(curl -s -X POST http://57.156.56.200:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"tutor@fiap.com.br","senha":"123456"}' | grep -o '"token":"[^"]*"' | cut -d'"' -f4)

echo "TOKEN: $TOKEN"

# GET
curl -s -H "Authorization: Bearer $TOKEN" http://57.156.56.200:8080/tutores

# POST
curl -s -X POST http://57.156.56.200:8080/tutores \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"nome":"Ana Lima","email":"ana@fiap.com.br","telefone":"11988880001","cpf":"44444444444"}'

# PUT
curl -s -X PUT http://57.156.56.200:8080/tutores/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"nome":"Maria Silva Atualizada","email":"maria@email.com","telefone":"11999990001","cpf":"11111111111"}'

# DELETE
curl -s -X DELETE http://57.156.56.214:8080/tutores/3 \
  -H "Authorization: Bearer $TOKEN" -w " HTTP %{http_code}"

# GET final
curl -s -H "Authorization: Bearer $TOKEN" http://57.156.56.214:8080/tutores
```

---

### 5. Dentro da VM — Oracle

```bash
docker exec oracle-db bash -c "echo 'SELECT table_name FROM user_tables;' | sqlplus APP_USER/AppPassword123@XEPDB1"

docker exec oracle-db bash -c "echo 'SELECT * FROM TUTOR;' | sqlplus APP_USER/AppPassword123@XEPDB1"
```

---

### 6. Máquina local — deletar VM

```bash
az group delete --name rg-challenge-clyvo-vet --yes
```

Aguarda e confirma:

```bash
az group list --output table
```

Tira print quando sumir — essa é a evidência de remoção para o PDF.

---

> ⚠️ **Atenção:** A VM atual já tem dados. Se gravar o vídeo agora o `register` vai dar conflito de email e o `git clone` vai dar erro de pasta existente. Antes de gravar, limpa tudo:

```bash
# Na VM
docker compose down -v
cd ~
rm -rf challenge-clyvo
```

E na máquina local, deleta e recria o RG antes de gravar:

```bash
az group delete --name rg-challenge-clyvo-vet --yes --no-wait
# aguarda sumir e roda o deploy.sh de novo já gravando
```