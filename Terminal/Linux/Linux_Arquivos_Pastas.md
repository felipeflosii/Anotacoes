# 🐧 Linux — Manipulação de Arquivos e Pastas

> Referência rápida de comandos para criar, mover, copiar, renomear, deletar e inspecionar arquivos e diretórios.

---

## 📁 Criar

### Arquivo vazio
```bash
touch arquivo.txt
touch a.txt b.txt c.txt        # múltiplos de uma vez
```

### Arquivo com conteúdo
```bash
echo "conteúdo" > arquivo.txt  # cria/sobrescreve
echo "linha" >> arquivo.txt    # acrescenta ao final
cat > arquivo.txt              # digita até Ctrl+D
```

### Pasta
```bash
mkdir pasta
mkdir -p pai/filho/neto        # cria toda a hierarquia de uma vez
mkdir -p src/{components,pages,utils}  # múltiplas subpastas
```

---

## 📋 Copiar

### Arquivo
```bash
cp origem.txt destino.txt
cp origem.txt /caminho/pasta/          # copia para pasta, mantém nome
cp -i origem.txt destino.txt           # pede confirmação se já existir
```

### Pasta (requer -r)
```bash
cp -r pasta_origem pasta_destino
cp -r pasta_origem /caminho/destino/
cp -rp pasta_origem pasta_destino      # preserva permissões e datas
```

---

## ✂️ Mover / Renomear

> `mv` serve para as duas operações: mover e renomear.

### Renomear
```bash
mv nome_antigo.txt nome_novo.txt
mv pasta_antiga pasta_nova
```

### Mover
```bash
mv arquivo.txt /caminho/destino/
mv arquivo.txt /caminho/destino/novo_nome.txt   # move e renomeia
mv *.txt /caminho/destino/                       # move todos os .txt
```

### Com confirmação
```bash
mv -i arquivo.txt destino/     # pergunta antes de sobrescrever
mv -n arquivo.txt destino/     # nunca sobrescreve
```

---

## 🗑️ Deletar

### Arquivo
```bash
rm arquivo.txt
rm a.txt b.txt c.txt           # múltiplos
rm *.log                       # por padrão/glob
rm -i arquivo.txt              # pede confirmação
```

### Pasta vazia
```bash
rmdir pasta
rmdir -p pai/filho/neto        # remove hierarquia vazia
```

### Pasta com conteúdo
```bash
rm -r pasta                    # recursivo
rm -rf pasta                   # recursivo + forçado (sem confirmação)
```

> [!danger] Cuidado com `rm -rf`
> Não há lixeira no terminal. O comando `rm -rf /caminho` é **irreversível**. Sempre confira o caminho antes de executar.

---

## 🔍 Listar e Inspecionar

### Listar conteúdo
```bash
ls                             # listagem simples
ls -l                          # formato longo (permissões, tamanho, data)
ls -a                          # inclui arquivos ocultos (.)
ls -lh                         # tamanhos legíveis (KB, MB)
ls -lt                         # ordena por data de modificação
ls -R                          # lista recursivamente
```

### Ver onde você está
```bash
pwd                            # exibe o diretório atual
```

### Informações do arquivo
```bash
file arquivo.txt               # tipo do arquivo
stat arquivo.txt               # metadados completos (tamanho, datas, permissões)
du -sh pasta/                  # tamanho total da pasta
du -sh *                       # tamanho de cada item no diretório atual
```

---

## 🔗 Links

### Link simbólico (atalho)
```bash
ln -s /caminho/original link_nome     # cria symlink
ln -sf /novo/caminho link_nome        # atualiza symlink existente
```

### Link físico (hard link)
```bash
ln arquivo.txt link_nome
```

---

## 🔎 Localizar Arquivos

```bash
find . -name "arquivo.txt"            # busca pelo nome exato
find . -name "*.log"                  # busca por padrão
find . -type d -name "pasta"          # busca somente diretórios
find . -type f -mtime -7              # modificados nos últimos 7 dias
find . -size +10M                     # maiores que 10 MB
find . -empty                         # arquivos/pastas vazios

locate arquivo.txt                    # busca rápida via índice (requer updatedb)
which python3                         # localiza executável no PATH
```

---

## 📄 Visualizar Conteúdo

```bash
cat arquivo.txt                       # exibe tudo de uma vez
less arquivo.txt                      # paginado (q para sair)
head arquivo.txt                      # primeiras 10 linhas
head -n 20 arquivo.txt                # primeiras N linhas
tail arquivo.txt                      # últimas 10 linhas
tail -n 20 arquivo.txt                # últimas N linhas
tail -f arquivo.log                   # acompanha em tempo real
```

---

## 🔐 Permissões

```bash
chmod 755 arquivo.sh                  # rwxr-xr-x
chmod +x script.sh                    # torna executável
chmod -R 644 pasta/                   # aplica recursivamente
chown usuario:grupo arquivo.txt       # muda dono e grupo
chown -R usuario:grupo pasta/         # recursivo
```

### Referência de permissões numéricas

| Número | Permissão |
|--------|-----------|
| 7 | rwx (leitura + escrita + execução) |
| 6 | rw- (leitura + escrita) |
| 5 | r-x (leitura + execução) |
| 4 | r-- (somente leitura) |
| 0 | --- (nenhuma) |

---

## 📦 Compactar e Descompactar

### tar
```bash
tar -czf arquivo.tar.gz pasta/        # compacta (gzip)
tar -xzf arquivo.tar.gz               # descompacta
tar -tzf arquivo.tar.gz               # lista conteúdo sem extrair
tar -xzf arquivo.tar.gz -C /destino/  # extrai em pasta específica
```

### zip / unzip
```bash
zip -r arquivo.zip pasta/
zip arquivo.zip a.txt b.txt
unzip arquivo.zip
unzip arquivo.zip -d /destino/
```

---

## ⚡ Atalhos Úteis

| Comando | Descrição |
|---------|-----------|
| `cd ~` | vai para o home do usuário |
| `cd -` | volta para o diretório anterior |
| `cd ..` | sobe um nível |
| `cp -r src/ src_bak/` | faz backup rápido de uma pasta |
| `mv arquivo{.txt,.bak}` | renomeia extensão com brace expansion |
| `ls -lh \| sort -k5 -h` | ordena listagem por tamanho |
| `rm -rf pasta && mkdir pasta` | limpa e recria uma pasta |
