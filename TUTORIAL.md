# Tutorial: OpenCode dentro do VS Code publicando no GitHub

Este tutorial mostra como usar o OpenCode no **terminal integrado do VS Code** e
como fazer as alterações chegarem sozinhas no **GitHub** — do mesmo jeito que foi
feito neste repositório.

> **Antes de tudo, a verdade sobre "automático":**
> o OpenCode **não** faz `commit` e `push` por conta própria. Ele executa `git`
> porque alguém (você ou o agente, seguindo uma instrução) pediu. O que existe é
> **automação configurada**: credencial salva + um comando/atalho que dispara o
> `git add`, `commit` e `push`. É isso que este tutorial monta.

---

## 1. Pré-requisitos

| Item | Como verificar |
|---|---|
| VS Code | `code --version` |
| Git | `git --version` |
| OpenCode | `opencode --version` |
| Conta no GitHub com um repositório | — |

Se o OpenCode ainda não estiver instalado:

```bash
curl -fsSL https://opencode.ai/v2/install | bash
# ou
npm install -g @opencode/cli
```

---

## 2. Abrir o OpenCode dentro do VS Code (passo a passo)

1. Abra a pasta do projeto:
   ```bash
   code .
   ```
   (ou **File → Open Folder**)

2. Abra o terminal integrado: **`Ctrl+`** (crase) ou **Terminal → New Terminal**.

3. Rode o OpenCode ali dentro:
   ```bash
   opencode
   ```

Pronto: a interface do OpenCode ocupa o terminal integrado, com o VS Code ao lado
mostrando os arquivos sendo alterados em tempo real. Não precisa de extensão.

### Atalho para abrir o OpenCode com uma tecla

Em **File → Preferences → Keyboard Shortcuts → Open Keyboard Shortcuts (JSON)**
(`Ctrl+Shift+P` → *Preferences: Open Keyboard Shortcuts (JSON)*), adicione:

```json
[
  {
    "key": "ctrl+alt+o",
    "command": "workbench.action.terminal.sendSequence",
    "args": { "text": "opencode\n" }
  }
]
```

Agora `Ctrl+Alt+O` digita `opencode` e dá Enter no terminal ativo.

### Abrir junto com o projeto (opcional)

Crie `.vscode/tasks.json` na raiz do projeto:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "OpenCode",
      "type": "shell",
      "command": "opencode",
      "presentation": { "reveal": "always", "panel": "dedicated" },
      "runOptions": { "runOn": "folderOpen" }
    }
  ]
}
```

Ao abrir a pasta, o VS Code abre um terminal dedicado já rodando o OpenCode.
(Pode ser necessário autorizar tarefas automáticas na primeira vez.)

---

## 3. Autorizar o Git a enviar sem pedir senha toda hora

O `push` só é "automático" se a credencial estiver disponível. Duas formas:

### Opção A — GitHub CLI (recomendado)

```bash
gh auth login
gh auth setup-git
```

O `gh` guarda o token no cofre do sistema e o Git usa esse token nas operações.

### Opção B — credential helper do Git

```bash
git config --global credential.helper store
```

Na primeira vez que você fizer `push`, o Git pergunta usuário e senha (use um
**Personal Access Token**, não a senha da conta). Depois disso ele guarda e não
pergunta mais.

> ⚠️ `credential.helper store` grava o token em `~/.git-credentials` **em texto
> puro**. Em máquina compartilhada prefira `gh auth login` ou o cofre do sistema
> (`credential.helper manager` no Windows, `osxkeychain` no macOS).

### Conferir o remote

```bash
git remote -v
```

Deve apontar para algo como:

```
origin  https://github.com/SEU-USUARIO/SEU-REPO.git (fetch)
origin  https://github.com/SEU-USUARIO/SEU-REPO.git (push)
```

**Nunca** coloque o token dentro da URL do remote
(`https://usuario:token@github.com/...`): ele fica no `.git/config` e vaza fácil.

---

## 4. Fazer o envio acontecer

### Nível 1 — Pedindo ao agente (mais simples)

No OpenCode, escreva algo como:

> "Faça commit de tudo e dê push para o GitHub com a mensagem 'docs: atualiza README'."

O agente roda `git status`, `git add`, `git commit` e `git push` por você.

### Nível 2 — Um comando `/publicar` (recomendado)

Crie o arquivo `.opencode/commands/publicar.md`:

```md
---
description: Commit e push de todas as alterações para o GitHub
agent: build
---

Revise as alterações abaixo, escreva uma mensagem de commit descritiva em
português, faça o commit de tudo (exceto arquivos ignorados) e envie com
`git push` para o remote `origin`, na branch atual.

!`git status --short && git diff --stat`
```

Na TUI, digite:

```text
/publicar
```

E o agente executa o fluxo completo. Esse comando já está criado neste
repositório — é só usar.

### Nível 3 — Push automático depois de cada commit (VS Code)

Se você commita pela aba **Source Control** do VS Code, dá para ele empurrar
sozinho logo em seguida. Em **Settings (JSON)**:

```json
{
  "git.enableSmartCommit": true,
  "git.postCommitCommand": "push",
  "git.autofetch": true,
  "git.confirmSync": false
}
```

- `enableSmartCommit`: se não houver nada em *staged*, o VS Code adiciona tudo
  antes de commitar.
- `postCommitCommand: "push"`: após cada commit, faz push.
- `autofetch`: mantém a branch sincronizada com o remote.

Ainda é preciso **disparar o commit** (botão ✓ ou `Ctrl+Enter` na aba Source
Control) — o resto é automático.

### Nível 4 — Hook do Git (push em todo commit, sem clicar em nada)

Crie `.git/hooks/post-commit` com:

```bash
#!/usr/bin/env bash
git push origin HEAD
```

E dê permissão:

```bash
chmod +x .git/hooks/post-commit
```

Agora **todo** commit empurra sozinho. Cuidado: hooks não são versionados e
qualquer commit (inclusive experimental) vai para o GitHub.

### Nível 5 — Plugin do OpenCode (avançado)

Para automatizar de dentro do OpenCode (por exemplo, ao final de cada sessão),
é possível escrever um plugin em `.opencode/plugins/` que escuta eventos e roda
comandos. É o caminho mais poderoso e o mais fácil de errar: consulte a
documentação antes — <https://opencode.ai/v2/docs/build/plugins/>.

---

## 5. O fluxo usado neste repositório

Foi exatamente este:

```bash
# 1. ver o que mudou
git status --short

# 2. adicionar só o que deve ir
git add README.md fracasso.md .gitignore

# 3. commitar com mensagem clara
git commit -m "docs: explica objetivo do RPG e adiciona fracasso.md"

# 4. enviar
git push origin main
```

Resultado no GitHub:

```
145182b..1b5baf8  main -> main
```

---

## 6. Segurança (leia antes de publicar)

- **Nunca** versione arquivos com tokens, senhas ou `.env`.
- Use `.gitignore` desde o primeiro commit:
  ```
  .env
  .env.*
  *.pem
  Documento não-salvo 1
  ```
- Se um token vazar em um commit, **revogue o token no GitHub** imediatamente.
  Reescrever o histórico não desfaz o vazamento.
- Dê preferência a tokens **fine-grained**, com escopo mínimo (só o repositório
  necessário) e validade curta.

---

## 7. Problemas comuns

| Sintoma | Causa provável | Solução |
|---|---|---|
| `Authentication failed` no push | credencial ausente/expirada | `gh auth login` ou regerar o token |
| `rejected (fetch first)` | remote tem commits novos | `git pull --rebase` e depois `git push` |
| `src refspec main does not match` | branch local com outro nome | `git branch -M main` |
| `nothing to commit` | nada foi adicionado | `git add -A` antes |
| Push pede senha toda vez | helper não configurado | `gh auth setup-git` |
| `Ctrl+J` fecha a aba | terminal/navegador captura a tecla | use `Ctrl+Enter` ou `Shift+Enter` para quebrar linha |

---

## 8. Resumo em 5 linhas

1. `code .` → `Ctrl+` → `opencode`
2. `gh auth login` para o Git ter credencial
3. Confira `git remote -v`
4. Use `/publicar` (ou peça ao agente)
5. Confira no GitHub se o commit apareceu

É isso. O "automático" é sempre uma combinação de credencial salva + um gatilho
que você configura.
