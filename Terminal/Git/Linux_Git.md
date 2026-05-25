# 🌿 Git — Referência de Comandos

> Comandos do dia a dia para controle de versão com Git.

---

## 🚀 Iniciar

```bash
git init                                  # inicia repositório local
git clone https://github.com/user/repo   # clona repositório remoto
git clone https://github.com/user/repo minha-pasta  # clona em pasta específica
```

---

## ⚙️ Configuração

```bash
git config --global user.name "Seu Nome"
git config --global user.email "email@exemplo.com"
git config --global core.editor nano       # editor padrão
git config --list                          # exibe todas as configs
```

---

## 📋 Status e Histórico

```bash
git status                                 # estado atual da árvore de trabalho
git log                                    # histórico completo
git log --oneline                          # histórico resumido (uma linha por commit)
git log --oneline --graph --all            # grafo de branches
git diff                                   # diff não staged
git diff --staged                          # diff staged (pronto para commit)
```

---

## ➕ Stage e Commit

```bash
git add arquivo.txt                        # adiciona arquivo específico
git add .                                  # adiciona tudo
git add -p                                 # adiciona por hunks (interativo)

git commit -m "mensagem"                   # commit com mensagem
git commit -am "mensagem"                  # stage + commit (só arquivos rastreados)
git commit --amend -m "nova mensagem"      # corrige último commit
```

---

## 🌿 Branches

```bash
git branch                                 # lista branches locais
git branch -a                              # lista todas (local + remoto)
git branch minha-branch                    # cria branch
git switch minha-branch                    # muda para branch
git switch -c minha-branch                 # cria e muda
git branch -d minha-branch                 # deleta (seguro)
git branch -D minha-branch                 # deleta forçado
```

---

## 🔀 Merge e Rebase

```bash
git merge minha-branch                     # merge na branch atual
git merge --no-ff minha-branch             # merge com commit de merge explícito
git rebase main                            # reaplica commits em cima de main
git rebase -i HEAD~3                       # rebase interativo (últimos 3 commits)
```

---

## 🌐 Remoto

```bash
git remote -v                              # lista remotos
git remote add origin https://...          # adiciona remoto
git remote remove origin                   # remove remoto
git remote set-url origin https://...      # troca URL do remoto

git fetch                                  # baixa sem aplicar
git pull                                   # fetch + merge
git pull --rebase                          # fetch + rebase
git push origin main                       # envia branch
git push -u origin main                    # envia e define upstream
git push --force-with-lease                # push forçado seguro
```

---

## 🏷️ Tags

```bash
git tag                                    # lista tags
git tag v1.0.0                             # tag leve
git tag -a v1.0.0 -m "Release v1.0.0"     # tag anotada
git push origin v1.0.0                     # envia tag
git push origin --tags                     # envia todas as tags
```

---

## 🧹 Desfazer

```bash
git restore arquivo.txt                    # descarta alterações no working tree
git restore --staged arquivo.txt           # remove do stage
git reset HEAD~1                           # desfaz último commit (mantém arquivos)
git reset --hard HEAD~1                    # desfaz commit e alterações
git revert abc1234                         # cria commit que desfaz outro
```

> [!danger] Cuidado com `--hard`
> `git reset --hard` descarta alterações permanentemente. Prefira `git restore` para casos simples.

---

## 📦 Stash

```bash
git stash                                  # guarda alterações temporariamente
git stash push -m "descricao"              # stash com nome
git stash list                             # lista stashes
git stash pop                              # aplica e remove o último
git stash apply stash@{1}                  # aplica sem remover
git stash drop stash@{0}                   # remove stash específico
git stash clear                            # remove todos
```

---

## 🔎 Buscar

```bash
git grep "termo"                           # busca no código rastreado
git log --all --grep="mensagem"            # busca em mensagens de commit
git log -S "codigo"                        # commits que adicionaram/removeram string
git blame arquivo.txt                      # quem alterou cada linha
```

---

## ⚡ Atalhos Úteis

| Comando | Descrição |
|---------|-----------|
| `git switch -` | volta para a branch anterior |
| `git log --oneline -10` | últimos 10 commits |
| `git diff main..minha-branch` | diff entre branches |
| `git cherry-pick abc1234` | traz commit específico para branch atual |
| `git clean -fd` | remove arquivos não rastreados |
| `git submodule add <url> pasta` | adiciona submódulo |
