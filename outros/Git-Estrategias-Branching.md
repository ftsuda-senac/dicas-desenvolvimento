# Git Avançado e Estratégias de Branching — Guia Prático

> **Objetivo:** Cobrir de forma abrangente os conceitos, comandos e estratégias do Git — desde a configuração inicial e operações essenciais até fluxos de branching, resolução de conflitos, automação com hooks e boas práticas para trabalho em equipe — com exemplos práticos voltados para projetos Java/Spring Boot e desenvolvimento web moderno.

---

## Sumário

1. [Fundamentos do Git](#1-fundamentos-do-git)
2. [Configuração Inicial](#2-configuração-inicial)
3. [Comandos Essenciais](#3-comandos-essenciais)
4. [Trabalhando com Branches](#4-trabalhando-com-branches)
5. [Merge e Rebase](#5-merge-e-rebase)
6. [Resolução de Conflitos](#6-resolução-de-conflitos)
7. [Estratégias de Branching](#7-estratégias-de-branching)
8. [Conventional Commits](#8-conventional-commits)
9. [Tags e Versionamento Semântico](#9-tags-e-versionamento-semântico)
10. [Comandos Avançados](#10-comandos-avançados)
11. [Git Stash — Gerenciamento de Trabalho Temporário](#11-git-stash--gerenciamento-de-trabalho-temporário)
12. [Git Worktrees — Múltiplos Diretórios de Trabalho](#12-git-worktrees--múltiplos-diretórios-de-trabalho)
13. [Git Hooks — Automação Local](#13-git-hooks--automação-local)
14. [Pull Requests e Code Review](#14-pull-requests-e-code-review)
15. [.gitignore — Padrões por Tipo de Projeto](#15-gitignore--padrões-por-tipo-de-projeto)
16. [.gitattributes — Controle de Atributos de Arquivos](#16-gitattributes--controle-de-atributos-de-arquivos)
17. [Git LFS — Large File Storage](#17-git-lfs--large-file-storage)
18. [Git Aliases e Produtividade](#18-git-aliases-e-produtividade)
19. [Ferramentas Gráficas e Integração com IDEs](#19-ferramentas-gráficas-e-integração-com-ides)
20. [Submódulos e Monorepos](#20-submódulos-e-monorepos)
21. [Migrações entre Sistemas de Controle de Versão](#21-migrações-entre-sistemas-de-controle-de-versão)
22. [Cenários Comuns e Soluções](#22-cenários-comuns-e-soluções)
23. [Boas Práticas e Checklist](#23-boas-práticas-e-checklist)
24. [Arquivos de Configuração do Repositório — GitHub e GitLab](#24-arquivos-de-configuração-do-repositório--github-e-gitlab)

---

## 1. Fundamentos do Git

### 1.1 O que é Git

Git é um sistema de controle de versão distribuído criado por Linus Torvalds em 2005. Diferente de sistemas centralizados (SVN, CVS), cada desenvolvedor possui uma cópia completa do repositório, incluindo todo o histórico.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  Centralizado vs. Distribuído                       │
│                                                                     │
│  Centralizado (SVN)              Distribuído (Git)                  │
│                                                                     │
│       ┌──────────┐              ┌──────────┐                        │
│       │ Servidor │              │ Remoto   │                        │
│       │ Central  │              │ (origin) │                        │
│       └────┬─────┘              └────┬─────┘                        │
│           │                     ┌────┼─────┐                        │
│     ┌─────┼─────┐          ┌───┴──┐ │  ┌──┴───┐                    │
│  ┌──┴──┐  │  ┌──┴──┐   ┌──┴──┐   │ │  │  ┌───┴──┐                 │
│  │ Dev │  │  │ Dev │   │ Dev │   │ │  │  │ Dev  │                  │
│  │  A  │  │  │  B  │   │  A  │   │ │  │  │  B   │                  │
│  │(WC) │  │  │(WC) │   │Repo │   │ │  │  │Repo  │                  │
│  └─────┘  │  └─────┘   │Compl│   │ │  │  │Compl │                  │
│        ┌──┴──┐          └─────┘   │ │  │  └──────┘                  │
│        │ Dev │                 ┌───┴─┴──┐                            │
│        │  C  │                 │ Dev C  │                            │
│        │(WC) │                 │ Repo   │                            │
│        └─────┘                 │ Compl  │                            │
│                                └────────┘                            │
│  WC = Working Copy                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

| Característica | Centralizado (SVN) | Distribuído (Git) |
|---------------|-------------------|-------------------|
| **Repositório** | Único no servidor | Cada dev tem cópia completa |
| **Trabalho offline** | Não | Sim (commit, branch, log) |
| **Velocidade** | Dependente da rede | Operações locais muito rápidas |
| **Branches** | Custosos (cópia de diretório) | Leves (ponteiro para commit) |
| **Backup** | Centralizado | Natural (cada clone é um backup) |

### 1.2 Os três estados do Git

Todo arquivo no Git pode estar em um de três estados principais:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Working Directory      Staging Area       Repository       │
│  (Diretório de          (Index/Stage)      (.git)           │
│   trabalho)                                                 │
│                                                             │
│  ┌──────────┐          ┌──────────┐       ┌──────────┐     │
│  │          │ git add  │          │commit │          │     │
│  │ Arquivos │────────▶│ Arquivos │──────▶│ Commits  │     │
│  │ modific. │          │ preparad.│       │ salvos   │     │
│  │          │◀────────│          │       │          │     │
│  └──────────┘ checkout └──────────┘       └──────────┘     │
│                                                             │
│    Modified              Staged             Committed        │
└─────────────────────────────────────────────────────────────┘
```

| Estado | Descrição | Comando para avançar |
|--------|-----------|---------------------|
| **Modified** | Arquivo alterado no diretório de trabalho | `git add` |
| **Staged** | Arquivo marcado para entrar no próximo commit | `git commit` |
| **Committed** | Alteração salva permanentemente no repositório | — |

### 1.3 Anatomia de um commit

```
┌──────────────────────────────────────────────────┐
│  Commit: a1b2c3d                                 │
│  ─────────────────────────────────────────────── │
│  Parent:  f4e5d6a                                │
│  Author:  João Silva <joao@email.com>            │
│  Date:    2025-06-15 14:30:00 -0300              │
│  Message: feat(auth): adicionar login com OAuth  │
│                                                  │
│  Tree: (snapshot de todos os arquivos)            │
│    ├── src/main/java/Auth.java  → blob abc123    │
│    ├── src/main/java/User.java  → blob def456    │
│    └── pom.xml                  → blob 789ghi    │
└──────────────────────────────────────────────────┘
```

Cada commit contém:
- **Hash SHA-1** — identificador único de 40 caracteres
- **Parent(s)** — referência ao(s) commit(s) anterior(es) (merge commits têm dois parents)
- **Tree** — snapshot completo de todos os arquivos naquele momento
- **Author/Committer** — quem criou e quem aplicou o commit
- **Message** — descrição da alteração

### 1.4 Git Internals — objetos e estrutura interna

Internamente, o Git é um banco de dados de objetos endereçados por conteúdo. Todo o conteúdo é armazenado em quatro tipos de objetos na pasta `.git/objects/`.

```
.git/
├── objects/             ← banco de dados de objetos
│   ├── ab/c123...       (blob, tree, commit ou tag)
│   ├── pack/            (packfiles comprimidos)
│   └── info/
├── refs/                ← ponteiros (branches e tags)
│   ├── heads/
│   │   ├── main         (arquivo com hash do último commit)
│   │   └── develop
│   └── tags/
│       └── v1.0.0
├── HEAD                 ← ponteiro para a branch atual
├── index                ← staging area (binário)
└── config               ← configuração local do repositório
```

| Objeto | Descrição | Conteúdo |
|--------|-----------|----------|
| **blob** | Conteúdo de um arquivo (sem nome, sem metadata) | Dados brutos do arquivo |
| **tree** | Estrutura de diretório (lista de blobs e sub-trees) | Nomes de arquivos + ponteiros para blobs/trees |
| **commit** | Snapshot + metadata | Ponteiro para tree raiz + parent(s) + autor + mensagem |
| **tag** | Tag anotada | Ponteiro para commit + nome + autor + mensagem |

```
commit a1b2c3d
  │
  ├── tree 4e5f6a7  (raiz do projeto)
  │     ├── blob 1a2b3c  README.md
  │     ├── blob 4d5e6f  pom.xml
  │     └── tree 7g8h9i  src/
  │           └── tree abc123  main/java/
  │                 └── blob def456  App.java
  │
  └── parent f4e5d6a  (commit anterior)
```

```bash
# Explorar objetos internos
git cat-file -t a1b2c3d            # tipo do objeto (commit, tree, blob, tag)
git cat-file -p a1b2c3d            # conteúdo legível do objeto

# Ver o conteúdo de uma tree
git ls-tree HEAD
git ls-tree -r HEAD                 # recursivo

# Ver o hash de um arquivo específico
git hash-object src/main/java/App.java

# Contar objetos no repositório
git count-objects -vH
```

### 1.5 DAG — Directed Acyclic Graph

O histórico do Git forma um **grafo acíclico dirigido** (DAG): cada commit aponta para seu(s) pai(s), nunca formando ciclos.

```
    C1 ← C2 ← C3 ← C5 ← C7   (main)
               ↑         ↑
               C4 ← C6 ──┘     (feature, mergeado em C7)

    Branches são ponteiros para commits:
    main    → C7
    feature → C6
    HEAD    → main → C7
```

Essa estrutura explica por que:
- **Branches são baratos** — são apenas um arquivo com 40 bytes (o hash do commit)
- **Merges criam commits com dois parents** — C7 aponta para C5 e C6
- **Rebase reescreve o grafo** — cria novos commits com novos hashes
- **Garbage collection** remove commits inalcançáveis (sem branch/tag apontando)

### 1.6 Packfiles e garbage collection

Com o tempo, o diretório `.git/objects/` acumula objetos soltos (loose objects). O Git compacta periodicamente esses objetos em **packfiles** para economizar espaço.

```bash
# Executar garbage collection manualmente
git gc                             # compacta objetos e remove inalcançáveis
git gc --aggressive                # compactação mais agressiva (lento)

# Manutenção automática (Git 2.30+)
git maintenance start              # agenda tarefas de manutenção
git maintenance run --task=gc      # executar tarefa específica

# Ver estatísticas de armazenamento
git count-objects -vH
# Exemplo de saída:
# count: 150          ← objetos soltos
# size: 1.20 MiB      ← tamanho dos objetos soltos
# in-pack: 12500      ← objetos em packfiles
# packs: 3            ← número de packfiles
# size-pack: 45.3 MiB ← tamanho total dos packfiles

# Verificar integridade do repositório
git fsck
git fsck --unreachable             # listar objetos inalcançáveis
```

| Operação | Quando ocorre | O que faz |
|----------|---------------|-----------|
| **Auto gc** | Após ~6700 objetos soltos | Compacta em packfiles |
| **Prune** | Durante `git gc` | Remove objetos inalcançáveis com mais de 2 semanas |
| **Repack** | Durante `git gc` | Reorganiza packfiles para melhor compressão |
| **Maintenance** | Agendado (cron/scheduled task) | Conjunto de tarefas de otimização |

---

## 2. Configuração Inicial

### 2.1 Identidade do desenvolvedor

```bash
# Configuração global (vale para todos os repositórios do usuário)
git config --global user.name "João Silva"
git config --global user.email "joao@email.com"

# Configuração local (sobrescreve a global para um projeto específico)
git config --local user.name "João Silva (Empresa)"
git config --local user.email "joao@empresa.com"

# Verificar configurações ativas
git config --list --show-origin
```

### 2.2 Configurações recomendadas

```bash
# Editor padrão
git config --global core.editor "code --wait"   # VS Code
git config --global core.editor "vim"            # Vim

# Branch padrão ao criar repositórios
git config --global init.defaultBranch main

# Tratamento de fim de linha (CRLF)
git config --global core.autocrlf true    # Windows
git config --global core.autocrlf input   # macOS/Linux

# Colorir saída no terminal
git config --global color.ui auto

# Pull com rebase por padrão (evita merge commits desnecessários)
git config --global pull.rebase true

# Ferramenta de merge e diff
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait --merge $REMOTE $LOCAL $BASE $MERGED'
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'
```

### 2.3 Chaves SSH para autenticação

```bash
# Gerar chave SSH (Ed25519 é recomendado)
ssh-keygen -t ed25519 -C "joao@email.com"

# Iniciar o agente SSH e adicionar a chave
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copiar a chave pública para adicionar no GitHub/GitLab
cat ~/.ssh/id_ed25519.pub

# Testar conexão
ssh -T git@github.com
```

### 2.4 Múltiplas contas SSH (pessoal + trabalho)

```
# ~/.ssh/config
Host github.com-pessoal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_pessoal

Host github.com-trabalho
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_trabalho
```

```bash
# Clonar usando o host configurado
git clone git@github.com-trabalho:empresa/projeto.git
```

### 2.5 Assinatura de commits com GPG

Commits assinados recebem o badge **"Verified"** no GitHub/GitLab, garantindo que o commit foi realmente feito por quem diz ser o autor.

```bash
# 1. Gerar chave GPG
gpg --full-generate-key
# Escolher: RSA and RSA, 4096 bits, sem expiração (ou com)

# 2. Listar chaves
gpg --list-secret-keys --keyid-format=long
# Saída:
# sec   rsa4096/ABC1234567890DEF 2025-01-15
#       1234567890ABCDEF1234567890ABCDEF12345678
# uid   João Silva <joao@email.com>

# 3. Configurar Git para usar a chave
git config --global user.signingkey ABC1234567890DEF
git config --global commit.gpgsign true      # assinar todos os commits
git config --global tag.gpgsign true         # assinar todas as tags

# 4. Exportar a chave pública (adicionar no GitHub/GitLab)
gpg --armor --export ABC1234567890DEF
# Copiar a saída e adicionar em: GitHub → Settings → SSH and GPG keys → New GPG key

# 5. Commitar com assinatura (automático se gpgsign=true)
git commit -S -m "feat: commit assinado"

# 6. Verificar assinatura
git log --show-signature -1
git verify-commit HEAD
```

### 2.6 Assinatura de commits com SSH (Git 2.34+)

Alternativa mais simples ao GPG — usa a mesma chave SSH que já existe para autenticação.

```bash
# 1. Configurar Git para usar SSH signing
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true

# 2. Criar arquivo de allowed signers (para verificação local)
echo "joao@email.com $(cat ~/.ssh/id_ed25519.pub)" > ~/.ssh/allowed_signers
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers

# 3. Commits são assinados automaticamente
git commit -m "feat: commit assinado com SSH"

# 4. Verificar
git log --show-signature -1
```

| Aspecto | GPG | SSH (Git 2.34+) |
|---------|-----|-----------------|
| **Setup** | Complexo (gerar chave GPG separada) | Simples (usa chave SSH existente) |
| **Suporte GitHub** | Sim | Sim |
| **Suporte GitLab** | Sim | Sim (16.x+) |
| **Web of trust** | Sim (rede de confiança GPG) | Não |
| **Expiração de chave** | Configurável | Não nativo |
| **Recomendação** | Projetos com requisitos de compliance | Maioria dos projetos |

---

## 3. Comandos Essenciais

### 3.1 Inicialização e clonagem

```bash
# Criar novo repositório
git init meu-projeto
cd meu-projeto

# Clonar repositório existente
git clone https://github.com/usuario/projeto.git
git clone git@github.com:usuario/projeto.git          # via SSH
git clone --depth 1 https://github.com/user/repo.git  # clone raso (só último commit)
git clone --branch develop https://github.com/user/repo.git  # clonar branch específica
```

### 3.2 Ciclo básico de trabalho

```bash
# 1. Verificar status dos arquivos
git status
git status -s          # formato curto

# 2. Adicionar alterações ao staging
git add arquivo.java                  # arquivo específico
git add src/main/java/               # diretório inteiro
git add *.java                        # por padrão glob
git add -p                            # modo interativo: escolhe pedaços (hunks)

# 3. Commitar as alterações
git commit -m "feat: adicionar endpoint de listagem"
git commit -am "fix: corrigir validação de email"  # add + commit (só tracked files)

# 4. Enviar para o remoto
git push origin main
git push -u origin feature/login     # primeira vez: vincula branch local ao remoto

# 5. Atualizar repositório local
git pull                              # fetch + merge (ou rebase, se configurado)
git fetch                             # apenas baixa, sem aplicar
git fetch --prune                     # remove referências de branches remotas deletadas
```

### 3.3 Visualização de histórico

```bash
# Log básico
git log
git log --oneline                     # resumo em uma linha
git log --oneline --graph --all       # visualização gráfica de todas as branches
git log -n 5                          # últimos 5 commits
git log --since="2025-01-01"          # commits desde uma data
git log --author="João"               # commits de um autor

# Log com diff
git log -p                            # mostra diff de cada commit
git log --stat                        # mostra arquivos alterados e estatísticas

# Formatação customizada
git log --pretty=format:"%h %an %ar - %s"
# Saída: a1b2c3d João Silva 2 days ago - feat: adicionar login

# Buscar commits pela mensagem
git log --grep="login"

# Buscar commits que alteraram um trecho de código
git log -S "methodName"               # pickaxe: busca adição/remoção do texto
git log -G "regex.*pattern"            # busca por regex
```

### 3.4 Diff — comparando alterações

```bash
# Diferença entre working directory e staging
git diff

# Diferença entre staging e último commit
git diff --staged
git diff --cached                     # sinônimo

# Diferença entre dois commits
git diff abc1234 def5678

# Diferença entre branches
git diff main...feature/login         # alterações da feature desde que divergiu de main
git diff main feature/login           # diferença pura entre os pontos atuais

# Diferença de um arquivo específico
git diff -- src/main/java/App.java

# Resumo estatístico
git diff --stat main...feature/login

# Apenas listar nomes de arquivos alterados
git diff --name-only main...feature/login
git diff --name-status main...feature/login  # com status (M, A, D, R)
```

---

## 4. Trabalhando com Branches

### 4.1 Conceito de branch

No Git, uma branch é apenas um ponteiro leve para um commit. Criar uma branch não copia arquivos — apenas cria um novo ponteiro.

```
                    HEAD
                     │
                     ▼
               ┌──────────┐
               │   main    │
               └─────┬─────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    ┌───────┐   ┌───────┐   ┌───────┐
    │  C1   │◀──│  C2   │◀──│  C3   │
    └───────┘   └───────┘   └───────┘

Após criar uma branch:

               ┌──────────┐     ┌──────────────┐
               │   main   │     │ feature/login │ ◀── HEAD
               └─────┬─────┘     └──────┬────────┘
                     │                  │
        ┌────────────┼──────────────────┘
        ▼            ▼
    ┌───────┐   ┌───────┐   ┌───────┐   ┌───────┐
    │  C1   │◀──│  C2   │◀──│  C3   │◀──│  C4   │
    └───────┘   └───────┘   └───────┘   └───────┘
```

### 4.2 Comandos de branch

```bash
# Listar branches
git branch                    # locais
git branch -r                 # remotas
git branch -a                 # todas (locais + remotas)
git branch -v                 # com último commit de cada

# Criar branch
git branch feature/login                    # cria sem mudar para ela
git checkout -b feature/login               # cria e muda (forma clássica)
git switch -c feature/login                 # cria e muda (forma moderna, Git 2.23+)

# Criar branch a partir de um commit ou outra branch
git checkout -b hotfix/fix-npe main
git checkout -b feature/api v2.0.0          # a partir de uma tag

# Mudar de branch
git checkout develop
git switch develop                          # forma moderna

# Renomear branch
git branch -m nome-antigo nome-novo
git branch -m nome-novo                     # renomeia a branch atual

# Deletar branch
git branch -d feature/login                 # seguro: só deleta se já foi mergeada
git branch -D feature/login                 # forçado: deleta mesmo sem merge

# Deletar branch remota
git push origin --delete feature/login
git push origin :feature/login              # sintaxe alternativa

# Limpar referências locais de branches remotas deletadas
git fetch --prune
git remote prune origin
```

### 4.3 Tracking de branches remotas

```bash
# Vincular branch local a uma remota
git branch --set-upstream-to=origin/develop develop
git push -u origin feature/login            # -u faz o tracking automaticamente

# Ver relação entre branches locais e remotas
git branch -vv

# Exemplo de saída:
#   develop    a1b2c3d [origin/develop] feat: adicionar cache
#   main       f4e5d6a [origin/main] release: v2.1.0
# * feature/x  789abcd fix: ajustar query
```

---

## 5. Merge e Rebase

### 5.1 Merge — unindo branches

O merge combina o trabalho de duas branches criando um **merge commit** (quando não é fast-forward).

```
Antes do merge:
    main:     C1 ── C2 ── C3
                     \
    feature:          C4 ── C5

Após: git checkout main && git merge feature

    Fast-forward (se main não teve commits novos):
    main:     C1 ── C2 ── C4 ── C5
                                 ▲
                               main, feature

    Three-way merge (se main teve commits novos):
    main:     C1 ── C2 ── C3 ────── M (merge commit)
                     \              /
    feature:          C4 ── C5 ───┘
```

```bash
# Merge padrão
git checkout main
git merge feature/login

# Merge sem fast-forward (sempre cria merge commit — recomendado para manter histórico)
git merge --no-ff feature/login

# Merge com mensagem customizada
git merge --no-ff feature/login -m "Merge feature/login: autenticação OAuth2"

# Abortar merge em caso de conflito
git merge --abort

# Merge squash — combina todos os commits da feature em um único commit
git merge --squash feature/login
git commit -m "feat(auth): adicionar autenticação OAuth2"
```

### 5.2 Rebase — reescrevendo o histórico

O rebase move (reaplica) os commits de uma branch para cima de outra, criando um histórico linear.

```
Antes do rebase:
    main:     C1 ── C2 ── C3
                     \
    feature:          C4 ── C5

Após: git checkout feature && git rebase main

    main:     C1 ── C2 ── C3
                            \
    feature:                 C4' ── C5'   (commits reescritos)
```

```bash
# Rebase básico
git checkout feature/login
git rebase main

# Rebase interativo — reorganizar, editar, juntar commits
git rebase -i HEAD~3

# Opções do rebase interativo:
# pick   — manter o commit como está
# reword — manter mas alterar a mensagem
# edit   — pausar para editar o commit
# squash — juntar com o commit anterior (mantém mensagem)
# fixup  — juntar com o anterior (descarta mensagem)
# drop   — remover o commit

# Continuar após resolver conflito no rebase
git rebase --continue

# Abortar rebase
git rebase --abort

# Pular commit com conflito (cuidado: perde as alterações desse commit)
git rebase --skip
```

### 5.3 Merge vs. Rebase — quando usar cada um

| Aspecto | Merge | Rebase |
|---------|-------|--------|
| **Histórico** | Preserva a ramificação real | Cria histórico linear |
| **Merge commit** | Sim (com `--no-ff`) | Não |
| **Segurança** | Não reescreve histórico | Reescreve commits (novos hashes) |
| **Conflitos** | Resolve uma vez | Pode precisar resolver em cada commit |
| **Rastreabilidade** | Fácil ver quando e onde houve merge | Commits parecem ter sido feitos sequencialmente |
| **Ideal para** | Branches compartilhadas, PRs | Branches locais antes do push |

**Regra de ouro:** nunca faça rebase de commits que já foram compartilhados (pushed). Rebase é seguro apenas para commits locais.

```bash
# Fluxo recomendado: rebase + merge no-ff
# 1. Na feature branch, rebasa sobre main
git checkout feature/login
git rebase main

# 2. Volta para main e faz merge com no-ff
git checkout main
git merge --no-ff feature/login
```

---

## 6. Resolução de Conflitos

### 6.1 Quando conflitos acontecem

Conflitos ocorrem quando duas branches alteram a **mesma região** de um arquivo de formas diferentes.

```
Branch main alterou:     Branch feature alterou:
───────────────────     ───────────────────────
public String getName() public String getName()
{                       {
    return name;            return this.name
}                              .trim();
                        }
```

### 6.2 Anatomia de um conflito

Quando o Git encontra um conflito, ele marca o arquivo assim:

```java
public class UserService {

<<<<<<< HEAD
    // Versão da branch atual (main)
    public String getName() {
        return name;
    }
=======
    // Versão da branch que está sendo mergeada (feature)
    public String getName() {
        return this.name.trim();
    }
>>>>>>> feature/login

}
```

| Marcador | Significado |
|----------|------------|
| `<<<<<<< HEAD` | Início da versão da branch atual |
| `=======` | Separador entre as versões |
| `>>>>>>> feature/login` | Fim da versão da branch recebida |

### 6.3 Resolvendo conflitos

```bash
# 1. Identificar arquivos com conflito
git status
# Saída: both modified: src/main/java/UserService.java

# 2. Abrir o arquivo e resolver manualmente
#    (remover marcadores e manter o código desejado)

# 3. Marcar como resolvido
git add src/main/java/UserService.java

# 4. Completar o merge
git commit
# ou se estiver num rebase:
git rebase --continue
```

### 6.4 Ferramentas visuais para conflitos

```bash
# Usar ferramenta de merge configurada
git mergetool

# VS Code: abre automaticamente com opções Accept Current/Incoming/Both

# IntelliJ IDEA: Git > Resolve Conflicts (interface visual de 3 painéis)
```

### 6.5 Estratégias para reduzir conflitos

| Prática | Descrição |
|---------|-----------|
| Branches curtas | Quanto menos tempo a branch viver, menos diverge |
| Merge/rebase frequente de `main` | `git pull --rebase origin main` na feature branch |
| Responsabilidade por arquivo | Evitar que múltiplos devs editem o mesmo arquivo |
| Commits pequenos e focados | Facilita identificar a origem do conflito |
| Comunicação na equipe | Avisar quando for mexer em áreas compartilhadas |

---

## 7. Estratégias de Branching

### 7.1 Git Flow

Modelo estruturado criado por Vincent Driessen, ideal para projetos com ciclos de release definidos.

```
Tag v1.0           Tag v1.1           Tag v2.0
  │                  │                  │
main      ●──────────●──────────────────●──────────
           \        / \                / \
release     \  ●───●   \          ●───●   \
             \│         \        /         \
develop       ●──●──●──●─●──●──●──●──●──●──●──●──●
               \     /    \       /         \    /
feature/A       ●──●──●    \     /           ●──●
                            \   /
feature/B                    ●─●
                              \
hotfix                         ●─── (merge em main E develop)
```

| Branch | Propósito | Criada a partir de | Merge para |
|--------|-----------|-------------------|------------|
| `main` | Código em produção (sempre estável) | — | — |
| `develop` | Integração de features em desenvolvimento | `main` | `release` |
| `feature/*` | Desenvolvimento de funcionalidades | `develop` | `develop` |
| `release/*` | Preparação e estabilização para produção | `develop` | `main` + `develop` |
| `hotfix/*` | Correções urgentes em produção | `main` | `main` + `develop` |

```bash
# Exemplo de fluxo Git Flow
# 1. Criar feature
git checkout develop
git checkout -b feature/carrinho-compras

# 2. Desenvolver e commitar
git add .
git commit -m "feat(cart): adicionar modelo do carrinho"

# 3. Finalizar feature
git checkout develop
git merge --no-ff feature/carrinho-compras
git branch -d feature/carrinho-compras

# 4. Criar release
git checkout develop
git checkout -b release/1.2.0

# 5. Ajustes finais na release (apenas bug fixes)
git commit -m "fix: corrigir formatação de preço na release"

# 6. Finalizar release
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"
git checkout develop
git merge --no-ff release/1.2.0
git branch -d release/1.2.0

# 7. Hotfix (se necessário)
git checkout main
git checkout -b hotfix/fix-payment
git commit -m "fix(payment): corrigir cálculo de desconto"
git checkout main
git merge --no-ff hotfix/fix-payment
git tag -a v1.2.1 -m "Hotfix 1.2.1"
git checkout develop
git merge --no-ff hotfix/fix-payment
git branch -d hotfix/fix-payment
```

**Quando usar Git Flow:**
- Projetos com releases versionadas e agendadas
- Produtos que precisam manter múltiplas versões em produção
- Equipes grandes com processos de QA formais

### 7.2 GitHub Flow

Modelo simplificado e leve, ideal para deploy contínuo.

```
main      ●─────●─────●─────●─────●─────●
           \   /       \   /       \   /
feature     ●─●         ●─●         ●─●
              │           │           │
            PR+Review   PR+Review   PR+Review
              │           │           │
            Deploy      Deploy      Deploy
```

**Regras do GitHub Flow:**
1. `main` é **sempre** deployável
2. Criar branch descritiva a partir de `main`
3. Fazer commits regulares e push para o remoto
4. Abrir Pull Request para discussão e code review
5. Após aprovação e testes verdes, fazer merge para `main`
6. Deploy imediato após merge

```bash
# 1. Criar branch a partir de main
git checkout main
git pull
git checkout -b feature/notificacoes-email

# 2. Desenvolver com commits frequentes
git add .
git commit -m "feat(notification): adicionar serviço de email"
git push -u origin feature/notificacoes-email

# 3. Abrir PR no GitHub e aguardar review

# 4. Após aprovação, merge via interface do GitHub (ou CLI)
gh pr merge --squash        # GitHub CLI

# 5. Limpar branch local
git checkout main
git pull
git branch -d feature/notificacoes-email
```

**Quando usar GitHub Flow:**
- Deploy contínuo (CD)
- Projetos SaaS com uma versão em produção
- Equipes pequenas a médias
- Quando simplicidade é prioridade

### 7.3 Trunk-Based Development

O modelo mais enxuto: todos commitam diretamente na branch principal (trunk), com branches de vida muito curta (< 1 dia).

```
main/trunk  ●──●──●──●──●──●──●──●──●──●──●──●
              \  /     \  /              │
feature        ●        ●          commit direto
         (< 1 dia)  (< 1 dia)      no trunk
```

| Prática | Descrição |
|---------|-----------|
| **Branches curtas** | Menos de 1 dia de vida, idealmente horas |
| **Feature flags** | Código incompleto entra no trunk desabilitado |
| **CI rigoroso** | Build + testes em cada push |
| **Pair programming** | Substitui code review formal em alguns times |
| **Release branches** | Opcionais, criadas a partir do trunk para estabilização |

```bash
# Fluxo típico trunk-based
git checkout main
git pull

# Branch curta (opcional — alguns times commitam direto)
git checkout -b feat/add-cache
# ... trabalhar por algumas horas ...
git commit -m "feat: adicionar cache Redis no serviço de produtos"
git checkout main
git pull --rebase
git merge feat/add-cache
git push
git branch -d feat/add-cache
```

**Quando usar Trunk-Based:**
- Equipes com CI/CD maduro e alta confiança nos testes
- Feature flags implementados
- Cultura de pair programming ou mob programming
- Necessidade de integração contínua real

### 7.4 GitLab Flow

Combina simplicidade do GitHub Flow com branches de ambiente para controle de deploy.

```
main          ●──●──●──●──●──●──●
               \   /       \   /
feature         ●─●         ●─●

Branches de ambiente:

main         ●──●──●──●──●──●──●   (desenvolvimento)
              │     │           │
staging       ●─────●───────────●   (homologação)
                    │           │
production          ●───────────●   (produção)
```

```bash
# Fluxo com branches de ambiente
# Desenvolvimento → merge para main
git checkout main
git merge --no-ff feature/relatorio

# Promover para staging (homologação)
git checkout staging
git merge main
git push origin staging        # trigger de deploy para staging

# Após aprovação do QA, promover para produção
git checkout production
git merge staging
git push origin production     # trigger de deploy para produção
git tag -a v1.5.0 -m "Release 1.5.0"
```

### 7.5 Comparativo de estratégias

| Aspecto | Git Flow | GitHub Flow | Trunk-Based | GitLab Flow |
|---------|----------|-------------|-------------|-------------|
| **Complexidade** | Alta | Baixa | Muito baixa | Média |
| **Branches simultâneas** | Muitas | Poucas | Mínimas | Moderadas |
| **Frequência de deploy** | Por release | Contínuo | Contínuo | Controlado por ambiente |
| **Ideal para** | Produtos versionados | SaaS, startups | Times maduros com CI/CD | Fluxos com ambientes |
| **Risco de conflitos** | Alto (branches longas) | Baixo | Muito baixo | Moderado |
| **Overhead de processo** | Alto | Baixo | Mínimo | Médio |
| **Feature flags** | Opcional | Opcional | Essencial | Opcional |

### 7.6 Guia de decisão

```
Precisa manter múltiplas versões em produção?
├── Sim → Git Flow
└── Não
    ├── Tem branches de ambiente (staging, production)?
    │   ├── Sim → GitLab Flow
    │   └── Não
    │       ├── CI/CD maduro com feature flags?
    │       │   ├── Sim → Trunk-Based Development
    │       │   └── Não → GitHub Flow
    └──────────────────────────────────
```

### 7.7 Forking Workflow — contribuição em projetos open source

O Forking Workflow é o modelo padrão para contribuir em projetos de terceiros. Cada contribuidor trabalha em uma **cópia pessoal** (fork) do repositório e propõe mudanças via Pull Request.

```
Repositório original (upstream):
  github.com/spring-projects/spring-boot

           ┌───────── fork ──────────┐
           ▼                         │
Fork pessoal:                   Repositório upstream:
  github.com/joao/spring-boot   github.com/spring-projects/spring-boot
           │                         ▲
           │         Pull Request    │
           └─────────────────────────┘
```

```bash
# 1. Fork pelo GitHub/GitLab (via interface web ou CLI)
gh repo fork spring-projects/spring-boot --clone

# 2. Configurar o upstream (repositório original)
git remote add upstream https://github.com/spring-projects/spring-boot.git
git remote -v
# origin    https://github.com/joao/spring-boot.git (fetch/push)
# upstream  https://github.com/spring-projects/spring-boot.git (fetch)

# 3. Manter o fork atualizado com o upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Alternativa mais limpa: rebase
git pull --rebase upstream main
git push origin main

# 4. Criar branch para a contribuição
git checkout -b fix/corrigir-doc-readme

# 5. Desenvolver e commitar
git add .
git commit -m "docs: corrigir exemplo de configuração no README"

# 6. Push para o fork pessoal
git push origin fix/corrigir-doc-readme

# 7. Abrir Pull Request do fork para o upstream
gh pr create --repo spring-projects/spring-boot \
    --title "docs: corrigir exemplo de configuração no README" \
    --body "Corrige o exemplo de application.yml que estava com indentação errada"
```

**Fork vs. Branch — quando usar cada um:**

| Aspecto | Fork | Branch no mesmo repo |
|---------|------|---------------------|
| **Permissão** | Não precisa de acesso ao repo original | Precisa de permissão de escrita |
| **Isolamento** | Completo (repositório separado) | Compartilha o mesmo repositório |
| **Uso típico** | Open source, contribuição externa | Equipes internas |
| **CI/CD** | Roda no fork (pode ter limitações) | Roda no repo principal |
| **Sincronização** | Manual (`git fetch upstream`) | Automática (`git pull`) |

---

## 8. Conventional Commits

### 8.1 Formato

Formato padronizado de mensagens de commit que facilita geração de changelogs e versionamento semântico automático.

```
<tipo>(<escopo>): <descrição>

[corpo opcional — explica o "porquê"]

[rodapé opcional — breaking changes, referências]
```

### 8.2 Tipos padrão

| Tipo | Descrição | Impacto no semver |
|------|-----------|-------------------|
| `feat` | Nova funcionalidade | Minor (1.**X**.0) |
| `fix` | Correção de bug | Patch (1.0.**X**) |
| `docs` | Apenas documentação | — |
| `style` | Formatação, semicolons, espaços | — |
| `refactor` | Refatoração sem mudança de comportamento | — |
| `perf` | Melhoria de performance | — |
| `test` | Adição/correção de testes | — |
| `build` | Build system, dependências (Maven, npm) | — |
| `ci` | Configuração de CI/CD | — |
| `chore` | Tarefas de manutenção | — |
| `revert` | Reverter commit anterior | — |

### 8.3 Exemplos práticos

```bash
# Feature simples
git commit -m "feat(auth): adicionar login com Google OAuth2"

# Bug fix com corpo explicativo
git commit -m "fix(api): corrigir paginação retornando registros duplicados

A query JPQL usava DISTINCT incorretamente quando havia JOIN FETCH
com coleções, causando duplicação na paginação offset.

Closes #142"

# Breaking change
git commit -m "feat(api)!: alterar formato de resposta de /users

BREAKING CHANGE: o campo 'name' foi dividido em 'firstName' e 'lastName'.
Clientes da API precisam atualizar o parsing da resposta.

Migration guide: docs/migration-v3.md"

# Refatoração
git commit -m "refactor(service): extrair cálculo de desconto para DiscountCalculator"

# Chore
git commit -m "chore(deps): atualizar Spring Boot 3.3 → 3.4"
```

### 8.4 Ferramentas para Conventional Commits

| Ferramenta | Linguagem | Propósito |
|-----------|-----------|-----------|
| **commitlint** | Node.js | Validar mensagens de commit em hooks |
| **commitizen** | Node.js/Python | CLI interativo para criar commits formatados |
| **standard-version** | Node.js | Geração automática de changelog e bump de versão |
| **semantic-release** | Node.js | Release automatizado baseado nos commits |
| **cocogitto** | Rust | Validação e geração de changelog |

```json
// .commitlintrc.json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "scope-enum": [2, "always", ["auth", "api", "ui", "db", "ci", "deps"]],
    "subject-max-length": [2, "always", 72]
  }
}
```

---

## 9. Tags e Versionamento Semântico

### 9.1 Tipos de tags

```bash
# Tag leve (apenas um ponteiro, sem metadados)
git tag v1.0.0

# Tag anotada (recomendada: armazena autor, data, mensagem)
git tag -a v1.0.0 -m "Release 1.0.0 — MVP com auth e CRUD de produtos"

# Tag em commit específico
git tag -a v1.0.0 abc1234 -m "Release 1.0.0"

# Listar tags
git tag
git tag -l "v1.*"             # filtrar com glob

# Ver detalhes de uma tag
git show v1.0.0

# Enviar tags para o remoto
git push origin v1.0.0        # tag específica
git push origin --tags         # todas as tags

# Deletar tag
git tag -d v1.0.0              # local
git push origin --delete v1.0.0  # remota
```

### 9.2 Versionamento Semântico (SemVer)

```
MAJOR.MINOR.PATCH
  │     │     │
  │     │     └── Correções de bugs compatíveis
  │     └──────── Funcionalidades novas compatíveis
  └────────────── Mudanças incompatíveis (breaking changes)

Exemplos:
  1.0.0 → 1.0.1  (patch: bug fix)
  1.0.1 → 1.1.0  (minor: nova feature)
  1.1.0 → 2.0.0  (major: breaking change)

Pré-release:
  2.0.0-alpha.1
  2.0.0-beta.1
  2.0.0-rc.1     (release candidate)
```

### 9.3 Fluxo de release com tags

```bash
# 1. Criar branch de release
git checkout -b release/1.2.0 develop

# 2. Ajustes finais (bump de versão no pom.xml, changelog)
# Atualizar <version>1.2.0</version> no pom.xml
git commit -am "chore(release): preparar versão 1.2.0"

# 3. Merge e tag
git checkout main
git merge --no-ff release/1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin main --tags

# 4. Merge back para develop
git checkout develop
git merge --no-ff release/1.2.0
git branch -d release/1.2.0
```

### 9.4 Releases no GitHub e GitLab

Releases são publicações associadas a tags que incluem notas descritivas e assets (binários, JARs, arquivos zip). São o mecanismo oficial para distribuir versões estáveis.

#### Criando releases via GitHub CLI

```bash
# Criar release a partir de uma tag existente
gh release create v1.2.0 \
    --title "Release 1.2.0" \
    --notes "### Novidades
- Endpoint de busca por categoria (#89)
- Cache Redis no serviço de produtos (#92)

### Correções
- Paginação duplicada com JOIN FETCH (#88)"

# Criar release com upload de assets
gh release create v1.2.0 \
    --title "Release 1.2.0" \
    --generate-notes \
    target/app-1.2.0.jar \
    docs/migration-guide.pdf

# Gerar release notes automaticamente a partir dos PRs mergeados
gh release create v1.2.0 --generate-notes

# Criar release como draft (para revisão antes de publicar)
gh release create v1.2.0 --draft --generate-notes

# Criar pre-release (alpha, beta, RC)
gh release create v2.0.0-beta.1 --prerelease --title "v2.0.0 Beta 1"

# Listar releases
gh release list

# Baixar assets de uma release
gh release download v1.2.0 --pattern "*.jar"
```

#### Release notes automáticas (GitHub)

O GitHub pode gerar release notes automaticamente com base nos PRs mergeados. Configuração via `.github/release.yml`:

```yaml
# .github/release.yml
changelog:
  exclude:
    labels:
      - ignore-for-release
    authors:
      - dependabot
  categories:
    - title: "🚀 Novas funcionalidades"
      labels:
        - enhancement
        - feature
    - title: "🐛 Correções"
      labels:
        - bug
        - fix
    - title: "📝 Documentação"
      labels:
        - documentation
    - title: "🔧 Manutenção"
      labels:
        - chore
        - dependencies
    - title: "💥 Breaking Changes"
      labels:
        - breaking-change
    - title: "Outros"
      labels:
        - "*"
```

#### Releases no GitLab

No GitLab, releases são criadas via interface web, API ou CI/CD:

```bash
# Via API do GitLab
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
     --data "name=Release 1.2.0" \
     --data "tag_name=v1.2.0" \
     --data "description=### Novidades..." \
     "https://gitlab.com/api/v4/projects/$PROJECT_ID/releases"
```

| Aspecto | GitHub Releases | GitLab Releases |
|---------|----------------|-----------------|
| **CLI** | `gh release create` | API REST ou `release-cli` |
| **Auto-notes** | Sim (`.github/release.yml`) | Sim (via CI/CD) |
| **Assets** | Upload direto na release | Via Package Registry ou links |
| **Draft** | Sim | Não nativo |
| **Pre-release** | Badge visual | Via milestones |

---

## 10. Comandos Avançados

### 10.1 Cherry-pick — trazer commits específicos

```bash
# Aplicar um commit de outra branch na branch atual
git cherry-pick abc1234

# Cherry-pick de múltiplos commits
git cherry-pick abc1234 def5678

# Cherry-pick de um range
git cherry-pick abc1234..def5678     # exclui abc1234
git cherry-pick abc1234^..def5678    # inclui abc1234

# Cherry-pick sem commitar automaticamente (para combinar com outras mudanças)
git cherry-pick --no-commit abc1234

# Abortar cherry-pick com conflito
git cherry-pick --abort
```

**Casos de uso:** trazer um bug fix de `develop` para `main` sem merge completo, portar uma correção para uma release branch antiga.

### 10.2 Bisect — encontrar o commit que introduziu um bug

```bash
# Iniciar busca binária
git bisect start

# Marcar o estado atual como ruim (tem o bug)
git bisect bad

# Marcar um commit antigo como bom (sem o bug)
git bisect good v1.0.0

# O Git faz checkout de um commit intermediário — testar e marcar:
git bisect good    # se funciona
git bisect bad     # se tem o bug
# Repetir até encontrar o commit exato

# Finalizar o bisect
git bisect reset

# Bisect automatizado com script de teste
git bisect start HEAD v1.0.0
git bisect run mvn test -pl modulo-afetado
# O Git executa o teste em cada passo e identifica o commit automaticamente
```

### 10.3 Blame — rastreando autoria

```bash
# Ver quem alterou cada linha
git blame src/main/java/UserService.java

# Blame de um trecho específico (linhas 10 a 30)
git blame -L 10,30 src/main/java/UserService.java

# Ignorar alterações de formatação/whitespace
git blame -w src/main/java/UserService.java

# Mostrar o commit que moveu ou copiou o código (detecta renomeações)
git blame -C src/main/java/UserService.java

# Formato compacto
git blame --date=short src/main/java/UserService.java
```

### 10.4 Reflog — histórico de movimentações do HEAD

O reflog registra todas as movimentações do HEAD, incluindo operações destrutivas. É a rede de segurança do Git.

```bash
# Ver histórico do reflog
git reflog
# Saída típica:
# a1b2c3d HEAD@{0}: commit: feat: adicionar cache
# f4e5d6a HEAD@{1}: checkout: moving from develop to feature/cache
# 789abcd HEAD@{2}: reset: moving to HEAD~2
# bcd1234 HEAD@{3}: commit: fix: corrigir query

# Recuperar commit "perdido" após reset acidental
git reflog
# Encontrar o hash do commit desejado
git checkout -b recuperar-trabalho a1b2c3d

# Reflog com data
git reflog --date=iso

# Desfazer o último rebase usando reflog
git reflog
# Encontrar o estado anterior ao rebase
git reset --hard HEAD@{5}
```

### 10.5 Reset — desfazendo alterações

```bash
# Reset soft — desfaz commit, mantém mudanças no staging
git reset --soft HEAD~1

# Reset mixed (padrão) — desfaz commit, mantém mudanças no working directory
git reset --mixed HEAD~1
git reset HEAD~1              # equivalente

# Reset hard — desfaz commit E descarta todas as mudanças (CUIDADO!)
git reset --hard HEAD~1

# Remover arquivo do staging (sem perder alterações)
git reset HEAD arquivo.java
git restore --staged arquivo.java    # forma moderna (Git 2.23+)
```

### 10.6 Revert — desfazer commit preservando histórico

Diferente do `reset`, o `revert` cria um **novo commit** que desfaz as alterações, preservando o histórico.

```bash
# Reverter um commit
git revert abc1234

# Reverter sem commitar automaticamente
git revert --no-commit abc1234

# Reverter um merge commit (precisa especificar o parent)
git revert -m 1 abc1234
# -m 1 = manter a linha do primeiro parent (geralmente main)
# -m 2 = manter a linha do segundo parent (a feature)
```

### 10.7 Clean — remover arquivos não rastreados

```bash
# Ver o que seria removido (dry-run)
git clean -n
git clean -nd              # incluindo diretórios

# Remover arquivos não rastreados
git clean -f               # apenas arquivos
git clean -fd              # arquivos e diretórios
git clean -fdx             # incluindo arquivos do .gitignore (ex: node_modules, target)
```

---

## 11. Git Stash — Gerenciamento de Trabalho Temporário

### 11.1 Comandos básicos do stash

```bash
# Salvar alterações no stash
git stash
git stash push -m "WIP: implementação do checkout"

# Incluir arquivos não rastreados
git stash push -u -m "WIP: incluindo novos arquivos"
git stash push --include-untracked -m "WIP"

# Incluir tudo, inclusive arquivos do .gitignore
git stash push -a -m "WIP: incluindo tudo"

# Stash de arquivos específicos
git stash push -m "WIP: apenas service" -- src/main/java/UserService.java

# Stash parcial (hunks interativos)
git stash push -p -m "WIP: apenas parte das alterações"
```

### 11.2 Recuperando do stash

```bash
# Aplicar o stash mais recente (mantém no stash)
git stash apply

# Aplicar e remover do stash
git stash pop

# Aplicar stash específico
git stash apply stash@{2}
git stash pop stash@{2}

# Listar stashes salvos
git stash list
# Saída:
# stash@{0}: On feature/login: WIP: implementação do checkout
# stash@{1}: On develop: WIP: refatoração do serviço

# Ver o conteúdo de um stash
git stash show                    # resumo
git stash show -p                 # diff completo
git stash show stash@{1} -p      # diff de stash específico
```

### 11.3 Gerenciamento do stash

```bash
# Criar branch a partir de um stash
git stash branch feature/recuperar stash@{0}

# Remover stash específico
git stash drop stash@{0}

# Limpar todos os stashes
git stash clear
```

---

## 12. Git Worktrees — Múltiplos Diretórios de Trabalho

### 12.1 O que são worktrees

Worktrees permitem ter **múltiplos diretórios de trabalho** vinculados ao mesmo repositório Git. Cada worktree faz checkout de uma branch diferente simultaneamente, sem precisar de stash ou commit de trabalho em progresso.

```
Sem worktrees (uma branch por vez):
┌───────────────────────┐
│ /projeto              │
│ Branch: feature/login │
│ (precisa stash para   │
│  trocar de branch)    │
└───────────────────────┘

Com worktrees (múltiplas branches simultâneas):
┌───────────────────────┐   ┌───────────────────────┐   ┌───────────────────────┐
│ /projeto              │   │ /projeto-hotfix       │   │ /projeto-review       │
│ Branch: feature/login │   │ Branch: hotfix/fix-db │   │ Branch: feature/api   │
│ (trabalho principal)  │   │ (correção urgente)    │   │ (code review)         │
└───────────────────────┘   └───────────────────────┘   └───────────────────────┘
         │                           │                           │
         └───────────── mesmo repositório .git ──────────────────┘
```

### 12.2 Comandos de worktree

```bash
# Criar worktree para uma branch existente
git worktree add ../projeto-hotfix hotfix/fix-db

# Criar worktree com nova branch
git worktree add -b hotfix/urgent ../projeto-hotfix main

# Listar worktrees ativos
git worktree list
# Saída:
# /home/dev/projeto         abc1234 [feature/login]
# /home/dev/projeto-hotfix  def5678 [hotfix/fix-db]

# Remover worktree (após terminar o trabalho)
git worktree remove ../projeto-hotfix

# Forçar remoção (se houver alterações não commitadas)
git worktree remove --force ../projeto-hotfix

# Limpar referências de worktrees removidos manualmente
git worktree prune
```

### 12.3 Casos de uso práticos

| Cenário | Sem worktree | Com worktree |
|---------|-------------|--------------|
| **Hotfix urgente** durante desenvolvimento | Stash → checkout → fix → commit → checkout → stash pop | Cria worktree → fix → commit → remove worktree |
| **Code review local** | Stash → checkout PR → testar → voltar | Cria worktree com a branch do PR |
| **Comparar comportamento** entre branches | Alternar entre branches repetidamente | Duas instâncias da IDE, uma em cada worktree |
| **Build em paralelo** | Impossível (uma branch por vez) | Build em cada worktree simultaneamente |
| **Testes de longa duração** | Bloqueia o diretório principal | Roda em worktree separado |

```bash
# Cenário: receber pedido de hotfix enquanto desenvolve uma feature
# 1. Sem interromper o trabalho na feature, cria worktree para o hotfix
git worktree add -b hotfix/fix-payment ../meu-projeto-hotfix main

# 2. Trabalhar no hotfix no novo diretório
cd ../meu-projeto-hotfix
# ... corrigir o bug ...
git add .
git commit -m "fix(payment): corrigir cálculo de desconto"
git push origin hotfix/fix-payment

# 3. Voltar para o trabalho na feature (nada mudou)
cd ../meu-projeto
# ... continuar desenvolvendo ...

# 4. Após merge do hotfix, remover o worktree
git worktree remove ../meu-projeto-hotfix

# Cenário: code review local de um PR
git worktree add ../review-pr-42 origin/feature/nova-api
cd ../review-pr-42
mvn test
# ... revisar e testar ...
cd ../meu-projeto
git worktree remove ../review-pr-42
```

### 12.4 Limitações e cuidados

| Limitação | Descrição |
|-----------|-----------|
| **Uma branch por worktree** | A mesma branch não pode estar em dois worktrees |
| **Compartilha `.git`** | Hooks, config e refs são compartilhados |
| **Stash é compartilhado** | `git stash list` mostra stashes de todos os worktrees |
| **Submodules** | Cada worktree precisa inicializar submodules separadamente |
| **IDE** | Cada worktree deve ser aberto como projeto separado na IDE |

---

## 13. Git Hooks — Automação Local

### 13.1 O que são hooks

Hooks são scripts que o Git executa automaticamente antes ou depois de eventos como commit, push e merge. Ficam em `.git/hooks/` e precisam ser executáveis.

```
.git/hooks/
├── pre-commit          ← antes do commit (lint, formatação)
├── prepare-commit-msg  ← antes de abrir o editor de mensagem
├── commit-msg          ← valida a mensagem do commit
├── post-commit         ← após o commit (notificações)
├── pre-push            ← antes do push (testes)
├── pre-rebase          ← antes do rebase
├── post-merge          ← após um merge
└── post-checkout       ← após trocar de branch
```

### 13.2 Hooks mais úteis

**pre-commit — verificação antes de commitar:**

```bash
#!/bin/sh
# .git/hooks/pre-commit

# Rodar formatação (Java com google-java-format)
echo "Verificando formatação..."
mvn spotless:check -q
if [ $? -ne 0 ]; then
    echo "ERRO: código não formatado. Execute 'mvn spotless:apply'"
    exit 1
fi

# Verificar se não há System.out.println em código Java
if git diff --cached --name-only | grep '\.java$' | xargs grep -l 'System\.out\.print' 2>/dev/null; then
    echo "ERRO: remova System.out.println antes de commitar"
    exit 1
fi

echo "Pre-commit OK"
```

**commit-msg — validar mensagem de commit:**

```bash
#!/bin/sh
# .git/hooks/commit-msg

COMMIT_MSG_FILE=$1
COMMIT_MSG=$(cat "$COMMIT_MSG_FILE")

# Validar formato de Conventional Commits
PATTERN="^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(.+\))?(!)?: .{1,72}"

if ! echo "$COMMIT_MSG" | head -1 | grep -qE "$PATTERN"; then
    echo "ERRO: mensagem de commit fora do padrão Conventional Commits"
    echo "Formato: <tipo>(<escopo>): <descrição>"
    echo "Tipos: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert"
    exit 1
fi
```

**pre-push — executar testes antes do push:**

```bash
#!/bin/sh
# .git/hooks/pre-push

echo "Executando testes antes do push..."
mvn test -q
if [ $? -ne 0 ]; then
    echo "ERRO: testes falharam. Corrija antes de fazer push."
    exit 1
fi

echo "Testes OK, push autorizado."
```

### 13.3 Compartilhando hooks com a equipe

Hooks ficam em `.git/hooks/`, que não é versionado. Soluções para compartilhar:

**Opção 1 — Diretório versionado + configuração:**

```bash
# Criar diretório de hooks no projeto
mkdir .githooks

# Copiar hooks para o diretório
cp .git/hooks/pre-commit .githooks/

# Configurar o Git para usar o diretório
git config core.hooksPath .githooks

# Versionar os hooks
git add .githooks/
```

**Opção 2 — Husky (projetos Node.js):**

```bash
npm install --save-dev husky
npx husky init
echo "npx commitlint --edit \$1" > .husky/commit-msg
```

**Opção 3 — Maven plugin (projetos Java):**

```xml
<!-- pom.xml — git-build-hook plugin -->
<plugin>
    <groupId>com.rudikershaw.gitbuildhook</groupId>
    <artifactId>git-build-hook-maven-plugin</artifactId>
    <version>3.5.0</version>
    <configuration>
        <installHooks>
            <pre-commit>.githooks/pre-commit</pre-commit>
            <commit-msg>.githooks/commit-msg</commit-msg>
        </installHooks>
    </configuration>
    <executions>
        <execution>
            <goals><goal>install</goal></goals>
        </execution>
    </executions>
</plugin>
```

---

## 14. Pull Requests e Code Review

### 14.1 Anatomia de um bom Pull Request

| Elemento | Descrição |
|----------|-----------|
| **Título** | Claro, curto, no formato Conventional Commits |
| **Descrição** | O que mudou, por quê e como testar |
| **Tamanho** | Idealmente < 400 linhas de diff |
| **Screenshots** | Para mudanças visuais (UI) |
| **Testes** | Cobrir o cenário adicionado/corrigido |
| **Checklist** | Confirmar que padrões foram seguidos |

**Template de PR (`.github/pull_request_template.md`):**

```markdown
## Descrição
<!-- O que foi feito e por quê -->

## Tipo de mudança
- [ ] Bug fix
- [ ] Nova funcionalidade
- [ ] Breaking change
- [ ] Refatoração
- [ ] Documentação

## Como testar
<!-- Passos para verificar a mudança -->

## Checklist
- [ ] Testes adicionados/atualizados
- [ ] Documentação atualizada (se aplicável)
- [ ] Sem warnings novos no build
- [ ] PR com menos de 400 linhas de diff
```

### 14.2 Boas práticas de code review

**Para o autor do PR:**

| Prática | Descrição |
|---------|-----------|
| Auto-review primeiro | Revise seu próprio diff antes de pedir review |
| PRs pequenos e focados | Um PR por funcionalidade/correção |
| Descrição completa | Contexto suficiente para o reviewer |
| Responder com rapidez | Não deixar review pendente por dias |
| Não levar para o pessoal | Feedback é sobre o código, não sobre você |

**Para o reviewer:**

| Prática | Descrição |
|---------|-----------|
| Ser construtivo | Sugerir alternativas, não apenas criticar |
| Priorizar feedback | Diferenciar bloqueante de sugestão (nit) |
| Revisar em tempo hábil | < 24h para primeira revisão |
| Testar localmente | Quando a mudança é crítica |
| Aprovar quando satisfeito | Não travar PRs por detalhes menores |

### 14.3 GitHub CLI para Pull Requests

```bash
# Criar PR
gh pr create --title "feat(auth): adicionar login OAuth2" \
  --body "Implementa autenticação via Google OAuth2" \
  --reviewer joao,maria \
  --label "feature,auth"

# Listar PRs
gh pr list
gh pr list --author @me
gh pr list --state open --label bug

# Ver detalhes de um PR
gh pr view 42
gh pr diff 42

# Fazer checkout de um PR para testar localmente
gh pr checkout 42

# Aprovar/solicitar mudanças
gh pr review 42 --approve
gh pr review 42 --request-changes --body "Falta tratamento de erro no endpoint"

# Merge do PR
gh pr merge 42 --squash --delete-branch
gh pr merge 42 --rebase
gh pr merge 42 --merge              # merge commit
```

### 14.4 Estratégias de merge em PRs

| Estratégia | Resultado | Quando usar |
|-----------|-----------|-------------|
| **Merge commit** | Preserva todos os commits + merge commit | Branches com histórico significativo |
| **Squash and merge** | Combina tudo em um commit | Features com commits intermediários (WIP) |
| **Rebase and merge** | Reaplica commits sobre main (linear) | Commits limpos e bem organizados |

```
Merge commit:
main: ── C1 ── C2 ── C3 ──── M
                \            /
feature:         C4 ── C5 ──┘

Squash and merge:
main: ── C1 ── C2 ── C3 ── S (commit único com todas as mudanças)

Rebase and merge:
main: ── C1 ── C2 ── C3 ── C4' ── C5' (reescritos sobre main)
```

---

## 15. .gitignore — Padrões por Tipo de Projeto

### 15.1 Sintaxe do .gitignore

```gitignore
# Comentário
*.log                   # Ignorar por extensão
/build/                 # Ignorar diretório na raiz (/ inicial)
build/                  # Ignorar diretório em qualquer nível
doc/*.pdf               # Ignorar PDFs apenas no diretório doc/
doc/**/*.pdf            # Ignorar PDFs em doc/ e subdiretórios
!important.log          # Negar uma regra anterior (não ignorar este arquivo)
**/temp                 # Ignorar em qualquer nível de profundidade
```

### 15.2 .gitignore para Java/Spring Boot + Maven

```gitignore
# Build output
target/
build/
*.jar
*.war
*.ear
*.class

# IDE — IntelliJ IDEA
.idea/
*.iml
out/

# IDE — Eclipse
.settings/
.classpath
.project
bin/

# IDE — VS Code
.vscode/
!.vscode/settings.json
!.vscode/launch.json

# IDE — NetBeans
nbproject/
nb-configuration.xml

# Maven
.mvn/timing.properties
.mvn/wrapper/maven-wrapper.jar

# Gradle
.gradle/
build/

# Environment e secrets
.env
.env.local
.env.*.local
application-local.yml
application-local.properties

# OS
.DS_Store
Thumbs.db
Desktop.ini
*.swp
*~

# Logs
*.log
logs/
```

### 15.3 .gitignore para Node.js / Frontend

```gitignore
# Dependências
node_modules/
bower_components/

# Build output
dist/
build/
.next/
.nuxt/
.output/

# Cache
.cache/
.parcel-cache/
.turbo/

# Environment
.env
.env.local
.env.*.local

# Testes
coverage/

# Logs
*.log
npm-debug.log*
yarn-debug.log*

# IDE
.vscode/
!.vscode/settings.json
.idea/

# OS
.DS_Store
Thumbs.db
```

### 15.4 .gitignore global

```bash
# Configurar gitignore global (para coisas pessoais como IDE, OS)
git config --global core.excludesFile ~/.gitignore_global
```

```gitignore
# ~/.gitignore_global
.DS_Store
Thumbs.db
*.swp
*~
.idea/
*.iml
.vscode/
```

### 15.5 Ignorar arquivos já rastreados

```bash
# Parar de rastrear um arquivo sem deletá-lo
git rm --cached arquivo-sensivel.properties
echo "arquivo-sensivel.properties" >> .gitignore
git commit -m "chore: remover arquivo sensível do tracking"

# Parar de rastrear um diretório inteiro
git rm -r --cached node_modules/
```

---

## 16. .gitattributes — Controle de Atributos de Arquivos

O `.gitattributes` permite configurar como o Git trata arquivos específicos — normalização de line endings, drivers de diff/merge, classificação de linguagem no GitHub e integração com Git LFS.

### 16.1 Sintaxe e padrões

```gitattributes
# padrão    atributo=valor
*.java      text diff=java
*.png       binary
*.sh        text eol=lf
```

### 16.2 Normalização de line endings

O atributo mais comum. Garante consistência independente do sistema operacional dos desenvolvedores.

```gitattributes
# .gitattributes
# Normalização automática para todos os arquivos de texto
* text=auto

# Forçar LF para scripts e configurações
*.sh        text eol=lf
*.bash      text eol=lf
*.yml       text eol=lf
*.yaml      text eol=lf
*.json      text eol=lf
*.xml       text eol=lf
*.properties text eol=lf
*.java      text eol=lf
*.sql       text eol=lf
*.md        text eol=lf

# Forçar CRLF para arquivos específicos do Windows
*.bat       text eol=crlf
*.cmd       text eol=crlf
*.ps1       text eol=crlf

# Arquivos binários (não normalizar)
*.png       binary
*.jpg       binary
*.gif       binary
*.ico       binary
*.jar       binary
*.war       binary
*.pdf       binary
*.zip       binary
*.ttf       binary
*.woff      binary
*.woff2     binary
```

```bash
# Renormalizar arquivos existentes após adicionar .gitattributes
git add --renormalize .
git commit -m "chore: normalizar line endings"
```

### 16.3 Drivers de diff customizados

Permite visualizar diffs úteis para arquivos que normalmente seriam tratados como binários.

```gitattributes
# Diff semântico por linguagem (Git já reconhece vários)
*.java      diff=java
*.py        diff=python
*.rb        diff=ruby
*.html      diff=html
*.css       diff=css

# Diff para documentos (requer ferramentas externas)
*.docx      diff=word
*.pdf       diff=pdf
```

```bash
# Configurar driver de diff para .docx
git config diff.word.textconv docx2txt
# Agora "git diff" mostra diferenças textuais em arquivos Word
```

### 16.4 Linguist — estatísticas de linguagem no GitHub

O GitHub usa o [Linguist](https://github.com/github-linguist/linguist) para calcular as estatísticas de linguagem exibidas na página do repositório. O `.gitattributes` permite ajustá-las.

```gitattributes
# Marcar diretórios como gerado (exclui das estatísticas e do diff em PRs)
src/generated/**    linguist-generated
docs/api/**         linguist-generated

# Marcar como vendor (exclui das estatísticas)
vendor/**           linguist-vendored
node_modules/**     linguist-vendored
third-party/**      linguist-vendored

# Marcar como documentação
docs/**             linguist-documentation

# Forçar detecção de linguagem (útil para extensões ambíguas)
*.h                 linguist-language=C
*.jsx               linguist-language=JavaScript

# Marcar como não detectável (exclui das estatísticas)
data/*.json         linguist-detectable=false
```

### 16.5 Estratégias de merge por arquivo

```gitattributes
# Sempre manter a versão local em caso de conflito (merge=ours)
# Útil para arquivos gerados automaticamente
package-lock.json   merge=ours
yarn.lock           merge=ours

# Union merge — combina ambas as versões sem conflito
# Útil para arquivos de changelog onde ambas as entradas são válidas
CHANGELOG.md        merge=union
```

```bash
# Registrar o driver "ours"
git config merge.ours.driver true
```

### 16.6 Export-ignore — excluir do archive

Controla quais arquivos são excluídos ao usar `git archive` (geração de releases/downloads).

```gitattributes
# Excluir do archive/release
.github/            export-ignore
.gitignore          export-ignore
.gitattributes      export-ignore
.editorconfig       export-ignore
tests/              export-ignore
docs/               export-ignore
*.md                export-ignore
docker-compose.yml  export-ignore
Dockerfile          export-ignore
Makefile            export-ignore
```

```bash
# Gerar arquivo zip sem os itens marcados como export-ignore
git archive --format=zip --prefix=projeto-v1.0.0/ HEAD -o release.zip
```

---

## 17. Git LFS — Large File Storage

### 17.1 O que é Git LFS

Git LFS (Large File Storage) substitui arquivos grandes (binários, assets, datasets) por ponteiros leves no repositório Git, armazenando o conteúdo real em um servidor LFS separado. Isso mantém o repositório rápido mesmo com arquivos pesados.

```
Sem LFS:                           Com LFS:
┌───────────────────────┐          ┌───────────────────────┐
│ Repositório Git       │          │ Repositório Git       │
│ ├── src/App.java (2KB)│          │ ├── src/App.java (2KB)│
│ ├── model.bin  (500MB)│          │ ├── model.bin  (130B) │ ← ponteiro
│ └── data.csv   (200MB)│          │ └── data.csv   (130B) │ ← ponteiro
│                       │          │                       │
│ Total: ~700MB         │          │ Total: ~2KB           │
│ (cada clone baixa     │          │                       │
│  TODO o histórico)    │          │ Servidor LFS          │
└───────────────────────┘          │ ├── model.bin  (500MB)│
                                   │ └── data.csv   (200MB)│
                                   │ (baixa sob demanda)   │
                                   └───────────────────────┘
```

### 17.2 Instalação e configuração

```bash
# Instalar Git LFS
# Windows (via Git for Windows — já inclui LFS)
# macOS
brew install git-lfs
# Linux (Debian/Ubuntu)
sudo apt install git-lfs

# Inicializar LFS no repositório
git lfs install

# Verificar versão
git lfs version
```

### 17.3 Rastreamento de arquivos

```bash
# Rastrear arquivos por extensão
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "*.bin"
git lfs track "*.mp4"

# Rastrear por diretório
git lfs track "assets/**"
git lfs track "data/models/**"

# O comando acima adiciona ao .gitattributes:
# *.psd filter=lfs diff=lfs merge=lfs -text
# *.zip filter=lfs diff=lfs merge=lfs -text

# IMPORTANTE: commitar o .gitattributes
git add .gitattributes
git commit -m "chore: configurar Git LFS para binários"

# Depois, adicionar e commitar normalmente
git add assets/logo.psd
git commit -m "feat: adicionar logo do projeto"
git push
```

### 17.4 Comandos LFS essenciais

```bash
# Ver arquivos rastreados pelo LFS
git lfs ls-files

# Ver padrões de rastreamento configurados
git lfs track

# Parar de rastrear um padrão
git lfs untrack "*.mp4"

# Status do LFS
git lfs status

# Ver informações do servidor LFS
git lfs env

# Migrar arquivos existentes para LFS (reescreve histórico)
git lfs migrate import --include="*.psd,*.zip" --everything

# Baixar arquivos LFS de um clone existente
git lfs pull

# Fazer fetch dos objetos LFS sem checkout
git lfs fetch
git lfs fetch --recent    # apenas commits recentes
```

### 17.5 Limites e custos por plataforma

| Plataforma | Armazenamento gratuito | Banda gratuita | Custo adicional |
|-----------|----------------------|----------------|-----------------|
| **GitHub** | 1 GB | 1 GB/mês | $5/mês por 50 GB storage + 50 GB banda |
| **GitLab** | 5 GB (total do projeto) | Incluído | Varia por plano |
| **Bitbucket** | 1 GB | 5 GB/mês | $10/mês por 100 GB |
| **Azure DevOps** | Ilimitado | Ilimitado | Incluído |

### 17.6 Boas práticas com LFS

| Prática | Descrição |
|---------|-----------|
| Configurar **antes** de adicionar arquivos grandes | Migrar depois requer reescrita de histórico |
| Commitar `.gitattributes` primeiro | Garantir que a equipe toda usa LFS para os mesmos arquivos |
| Usar `.lfsconfig` para server URL customizado | Útil para servidores LFS self-hosted |
| Considerar limites de banda | CI/CD pode consumir banda rapidamente |
| Avaliar alternativas | Para datasets muito grandes, considerar DVC (Data Version Control) |

```ini
# .lfsconfig (opcional — URL do servidor LFS customizado)
[lfs]
    url = https://lfs.empresa.com/org/repo
```

---

## 18. Git Aliases e Produtividade

### 18.1 Aliases úteis

```bash
# Log visual compacto
git config --global alias.lg "log --oneline --graph --all --decorate"

# Status curto
git config --global alias.st "status -sb"

# Diff resumido
git config --global alias.df "diff --stat"

# Último commit
git config --global alias.last "log -1 HEAD --stat"

# Listar branches com último commit
git config --global alias.br "branch -vv"

# Desfazer último commit (mantendo alterações)
git config --global alias.undo "reset --soft HEAD~1"

# Commit com mensagem
git config --global alias.cm "commit -m"

# Adicionar tudo e commitar
git config --global alias.ac "!git add -A && git commit -m"

# Branches mergeadas (candidatas a delete)
git config --global alias.merged "branch --merged main"

# Limpar branches já mergeadas
git config --global alias.cleanup "!git branch --merged main | grep -v 'main' | xargs git branch -d"

# Diff entre branch atual e main
git config --global alias.changes "diff main...HEAD --stat"

# Amend sem editar mensagem
git config --global alias.amend "commit --amend --no-edit"

# Stash com nome
git config --global alias.save "stash push -m"

# WIP commit (trabalho em progresso)
git config --global alias.wip "!git add -A && git commit -m 'chore: WIP'"
```

### 18.2 Arquivo `.gitconfig` completo

```ini
# ~/.gitconfig
[user]
    name = João Silva
    email = joao@email.com

[core]
    editor = code --wait
    autocrlf = input
    excludesFile = ~/.gitignore_global

[init]
    defaultBranch = main

[pull]
    rebase = true

[push]
    autoSetupRemote = true

[fetch]
    prune = true

[merge]
    conflictstyle = diff3

[diff]
    colorMoved = default

[alias]
    lg = log --oneline --graph --all --decorate
    st = status -sb
    br = branch -vv
    last = log -1 HEAD --stat
    undo = reset --soft HEAD~1
    cm = commit -m
    save = stash push -m
    cleanup = !git branch --merged main | grep -v 'main' | xargs git branch -d

[rerere]
    enabled = true
```

A opção `rerere` (Reuse Recorded Resolution) faz o Git lembrar como você resolveu um conflito e aplicar automaticamente a mesma resolução se o mesmo conflito aparecer novamente.

---

## 19. Ferramentas Gráficas e Integração com IDEs

### 19.1 Clientes gráficos dedicados

| Ferramenta | Plataformas | Licença | Destaque |
|-----------|-------------|---------|----------|
| **GitKraken** | Windows, macOS, Linux | Freemium | Interface visual rica, integração com GitHub/GitLab/Bitbucket, board de issues |
| **SourceTree** | Windows, macOS | Gratuito (Atlassian) | Integração profunda com Bitbucket/Jira, bom para Git Flow |
| **Fork** | Windows, macOS | Pago (avaliação gratuita) | Rápido, leve, merge conflict resolver embutido |
| **GitHub Desktop** | Windows, macOS | Gratuito (open source) | Simplicidade, integração nativa com GitHub, ideal para iniciantes |
| **Tortoise Git** | Windows | Gratuito (open source) | Integração com Windows Explorer (menu de contexto) |
| **Lazygit** | Windows, macOS, Linux | Gratuito (open source) | Interface TUI no terminal, muito rápido, ideal para quem prefere teclado |
| **GitUI** | Windows, macOS, Linux | Gratuito (open source) | TUI leve escrito em Rust, alternativa ao Lazygit |

### 19.2 Git no VS Code

O VS Code oferece suporte a Git integrado e extensões que ampliam significativamente a produtividade.

**Funcionalidades nativas:**

| Recurso | Descrição |
|---------|-----------|
| **Source Control** | Painel lateral para staging, commits, diff inline |
| **Gutter indicators** | Barras coloridas na margem indicando linhas adicionadas/modificadas/removidas |
| **Merge editor** | Editor visual de 3 painéis para resolução de conflitos |
| **Timeline** | Histórico de alterações por arquivo (commits + saves locais) |
| **Branch switching** | Barra de status inferior para trocar de branch |

**Extensões recomendadas:**

| Extensão | Funcionalidade |
|----------|---------------|
| **GitLens** | Blame inline, histórico de arquivo, comparação entre branches, CodeLens com autoria |
| **Git Graph** | Visualização gráfica do histórico (similar a `git log --graph`) |
| **Git History** | Navegação visual pelo histórico de commits e arquivos |
| **GitHub Pull Requests** | Criar, revisar e fazer merge de PRs sem sair do editor |
| **GitLab Workflow** | Integração com GitLab MRs, pipelines e issues |

```jsonc
// settings.json — configurações Git úteis no VS Code
{
    "git.autofetch": true,
    "git.confirmSync": false,
    "git.enableSmartCommit": true,
    "git.pruneOnFetch": true,
    "diffEditor.ignoreTrimWhitespace": false,
    "gitlens.codeLens.enabled": true,
    "gitlens.currentLine.enabled": true
}
```

### 19.3 Git no IntelliJ IDEA

O IntelliJ (e demais IDEs JetBrains) tem uma das integrações Git mais completas do mercado.

| Recurso | Atalho / Local | Descrição |
|---------|---------------|-----------|
| **Commit** | `Ctrl+K` | Janela de commit com diff, análise de código e checklist |
| **Push** | `Ctrl+Shift+K` | Push com visualização de commits pendentes |
| **Update (Pull)** | `Ctrl+T` | Pull com opção de merge ou rebase |
| **Log** | `Alt+9` → Git | Visualização gráfica completa do histórico |
| **Blame** | Clique direito na margem → Annotate | Blame inline com informações do commit |
| **Diff** | Selecionar arquivo → `Ctrl+D` | Diff visual entre versões |
| **Resolve Conflicts** | Git → Resolve Conflicts | Merge tool de 3 painéis |
| **Branches** | Barra inferior ou `Ctrl+Shift+` ` | Criar, trocar, comparar branches |
| **Cherry-pick** | Log → botão direito no commit | Cherry-pick visual |
| **Interactive Rebase** | Log → botão direito na branch | Rebase interativo visual (drag & drop) |
| **Shelve/Unshelve** | `Ctrl+Shift+H` | Similar ao stash, integrado à IDE |
| **Changelists** | Commit tool window | Agrupar alterações logicamente antes de commitar |

### 19.4 Comparativo de fluxos: terminal vs. GUI

| Cenário | Terminal | GUI |
|---------|---------|-----|
| **Commit + push simples** | Rápido com aliases | Igualmente rápido |
| **Rebase interativo** | Poderoso com flags | Mais visual (drag & drop no IntelliJ) |
| **Resolução de conflitos** | Trabalhoso com editor de texto | Muito mais fácil com merge tool visual |
| **Blame / histórico** | Informativo mas denso | Navegação intuitiva com cliques |
| **Operações em massa** | Scripts e pipes | Limitado |
| **Branches complexos** | `git log --graph` | Gráfico interativo |
| **Aprendizado do Git** | Entende o que acontece por baixo | Pode abstrair demais |

A recomendação é dominar o terminal para entender os conceitos e usar GUIs para produtividade no dia a dia — especialmente para resolução de conflitos e navegação de histórico.

---

## 20. Submódulos e Monorepos

### 20.1 Git Submodules

Submodules permitem incluir um repositório Git dentro de outro, mantendo históricos separados.

```bash
# Adicionar submódulo
git submodule add https://github.com/org/shared-lib.git libs/shared

# Clonar projeto com submódulos
git clone --recurse-submodules https://github.com/org/projeto.git
# ou depois de clonar:
git submodule update --init --recursive

# Atualizar submódulo para última versão
cd libs/shared
git pull origin main
cd ../..
git add libs/shared
git commit -m "chore: atualizar shared-lib para v2.3.0"

# Remover submódulo
git submodule deinit libs/shared
git rm libs/shared
rm -rf .git/modules/libs/shared
```

### 20.2 Monorepo vs. Multi-repo

| Aspecto | Monorepo | Multi-repo |
|---------|----------|------------|
| **Estrutura** | Todos os projetos em um repositório | Um repositório por projeto |
| **Compartilhamento de código** | Direto (imports locais) | Via pacotes publicados |
| **CI/CD** | Precisa de build seletivo | Pipelines independentes |
| **Refatoração cross-project** | Um commit, um PR | Múltiplos PRs coordenados |
| **Tamanho do repositório** | Pode crescer muito | Cada repo é enxuto |
| **Ferramentas** | Nx, Turborepo, Bazel, Lerna | Git padrão |
| **Exemplo** | Google, Meta, Uber | Netflix, Amazon |

```
# Exemplo de estrutura monorepo
monorepo/
├── apps/
│   ├── api/          (Spring Boot)
│   ├── web/          (React)
│   └── mobile/       (Flutter)
├── libs/
│   ├── shared-dto/
│   └── auth-common/
├── tools/
│   └── scripts/
├── .github/
│   └── workflows/
├── pom.xml           (parent POM)
└── package.json      (workspaces)
```

### 20.3 Git Sparse Checkout e Partial Clone

Em monorepos muito grandes, não é prático baixar e fazer checkout de todos os arquivos. O Git oferece mecanismos para trabalhar com apenas parte do repositório.

#### Partial Clone — clone sem baixar todos os blobs

```bash
# Clone sem blobs (baixa apenas metadados — blobs são baixados sob demanda)
git clone --filter=blob:none https://github.com/org/monorepo.git

# Clone sem blobs nem trees (mais agressivo — blobless + treeless)
git clone --filter=tree:0 https://github.com/org/monorepo.git

# Clone raso (shallow) — apenas os últimos N commits
git clone --depth 1 https://github.com/org/monorepo.git
git clone --depth 10 --no-single-branch https://github.com/org/monorepo.git
```

| Tipo | O que baixa | Quando usar |
|------|-------------|-------------|
| `--filter=blob:none` | Commits + trees (blobs sob demanda) | Monorepos onde você navega o histórico |
| `--filter=tree:0` | Apenas commits (trees + blobs sob demanda) | Clones muito rápidos, pouca navegação |
| `--depth N` | Últimos N commits com tudo | CI/CD, builds que não precisam de histórico |

#### Sparse Checkout — checkout de diretórios específicos

```bash
# Ativar sparse checkout
git sparse-checkout init --cone

# Selecionar apenas os diretórios desejados
git sparse-checkout set apps/api libs/shared-dto

# Adicionar mais diretórios
git sparse-checkout add apps/web

# Ver diretórios ativos
git sparse-checkout list

# Desativar (voltar ao checkout completo)
git sparse-checkout disable
```

```bash
# Fluxo completo: partial clone + sparse checkout
git clone --filter=blob:none https://github.com/org/monorepo.git
cd monorepo
git sparse-checkout init --cone
git sparse-checkout set apps/api libs/shared-dto

# Resultado: apenas apps/api/ e libs/shared-dto/ estão no working directory
# Os demais diretórios existem no histórico mas não ocupam disco
```

**Modo cone vs. modo não-cone:**

| Modo | Sintaxe | Descrição |
|------|---------|-----------|
| **Cone** (recomendado) | Diretórios inteiros (`apps/api`) | Mais rápido, usa padrões de diretório |
| **Non-cone** | Padrões glob (`*.java`, `!test/`) | Mais flexível, mais lento |

```bash
# Non-cone (para filtrar por padrão de arquivo)
git sparse-checkout init
git sparse-checkout set '/*.md' '/pom.xml' '/apps/api/**'
```

---

## 21. Migrações entre Sistemas de Controle de Versão

### 21.1 SVN para Git

```bash
# Migração com git svn (preserva histórico)
# 1. Criar mapeamento de autores SVN → Git
# authors.txt:
# joao.silva = João Silva <joao@email.com>
# maria.santos = Maria Santos <maria@email.com>

# 2. Clonar o repositório SVN
git svn clone https://svn.empresa.com/projeto \
    --stdlayout \
    --authors-file=authors.txt \
    --no-metadata \
    projeto-git

# --stdlayout: assume estrutura trunk/branches/tags
# --authors-file: mapeia usuários SVN para Git
# --no-metadata: não adiciona referência SVN nos commits

# 3. Converter tags SVN em tags Git
cd projeto-git
for tag in $(git branch -r | grep 'tags/' | sed 's/ *tags\///'); do
    git tag "$tag" "tags/$tag"
    git branch -r -d "tags/$tag"
done

# 4. Converter branches SVN em branches Git locais
for branch in $(git branch -r | grep -v 'trunk' | grep -v 'tags/'); do
    git branch "${branch#origin/}" "$branch"
done

# 5. Adicionar remote e push
git remote add origin git@github.com:org/projeto.git
git push origin --all
git push origin --tags
```

### 21.2 Mercurial (Hg) para Git

```bash
# Usando hg-fast-export
git init projeto-git
cd projeto-git
hg-fast-export -r /caminho/para/repo-hg -A authors.txt
git checkout HEAD
```

### 21.3 Migração entre provedores (GitHub ↔ GitLab ↔ Bitbucket)

```bash
# Mirror completo (preserva branches, tags, refs)
# 1. Clone mirror do repositório de origem
git clone --mirror https://github.com/org/projeto.git

# 2. Push mirror para o novo destino
cd projeto.git
git remote set-url origin https://gitlab.com/org/projeto.git
git push --mirror

# 3. Clonar normalmente do novo destino
cd ..
git clone https://gitlab.com/org/projeto.git
```

| Elemento | Migra automaticamente | Precisa de ferramenta externa |
|----------|----------------------|------------------------------|
| **Código e histórico** | Sim (`--mirror`) | — |
| **Branches e tags** | Sim (`--mirror`) | — |
| **Issues** | Não | GitHub CLI, GitLab API, ferramentas como `gitport` |
| **Pull/Merge Requests** | Não | APIs das plataformas |
| **Wikis** | Parcial (é um repo Git separado) | Clone + push manual |
| **CI/CD configs** | Código migra, mas sintaxe difere | Reescrita manual |
| **CODEOWNERS** | Código migra, mas sintaxe pode diferir | Ajuste manual |

### 21.4 Mirror e sincronização entre repositórios

```bash
# Manter dois remotos sincronizados (ex: GitHub + GitLab)
git remote add github git@github.com:org/projeto.git
git remote add gitlab git@gitlab.com:org/projeto.git

# Push para ambos
git push github main
git push gitlab main

# Configurar push para múltiplos remotos de uma vez
git remote set-url --add --push origin git@github.com:org/projeto.git
git remote set-url --add --push origin git@gitlab.com:org/projeto.git
# Agora: git push origin main → envia para ambos
```

---

## 22. Cenários Comuns e Soluções

### 22.1 "Commitei na branch errada"

```bash
# Mover o último commit para outra branch
# 1. Anotar o hash do commit
git log --oneline -1
# Saída: a1b2c3d feat: nova funcionalidade

# 2. Desfazer o commit (mantendo mudanças)
git reset --soft HEAD~1

# 3. Stash, trocar de branch, aplicar
git stash
git checkout branch-correta
git stash pop
git add .
git commit -m "feat: nova funcionalidade"
```

### 22.2 "Preciso alterar a mensagem do último commit"

```bash
# Se ainda não fez push
git commit --amend -m "feat(auth): mensagem corrigida"

# Se já fez push (CUIDADO: reescreve histórico remoto)
git commit --amend -m "feat(auth): mensagem corrigida"
git push --force-with-lease
```

### 22.3 "Preciso adicionar arquivo ao último commit"

```bash
# Se ainda não fez push
git add arquivo-esquecido.java
git commit --amend --no-edit
```

### 22.4 "Fiz push de arquivo sensível (senha, chave)"

```bash
# 1. IMEDIATAMENTE: rotacionar/invalidar a credencial exposta

# 2. Remover do histórico com git filter-repo (substituto do filter-branch)
pip install git-filter-repo
git filter-repo --invert-paths --path arquivo-sensivel.env

# 3. Forçar push e avisar a equipe
git push --force-with-lease --all

# 4. Adicionar ao .gitignore
echo "arquivo-sensivel.env" >> .gitignore
```

### 22.5 "Preciso desfazer um merge já pushado"

```bash
# Criar um commit de revert (seguro — não reescreve histórico)
git revert -m 1 <hash-do-merge-commit>
git push
```

### 22.6 "Branch está muito atrás de main e com conflitos"

```bash
# Opção 1: rebase (histórico limpo, pode ter mais conflitos)
git checkout feature/minha-branch
git fetch origin
git rebase origin/main
# Resolver conflitos em cada commit, se necessário
git push --force-with-lease

# Opção 2: merge de main na feature (mais seguro)
git checkout feature/minha-branch
git merge origin/main
# Resolver conflitos uma vez
git push
```

### 22.7 "Quero descartar todas as alterações locais"

```bash
# Descartar mudanças em arquivos tracked
git checkout -- .           # forma clássica
git restore .               # forma moderna (Git 2.23+)

# Descartar staged e unstaged
git reset --hard HEAD

# Descartar tudo + arquivos untracked
git reset --hard HEAD
git clean -fd
```

### 22.8 "Quero dividir um commit grande em vários menores"

```bash
# Rebase interativo marcando o commit como "edit"
git rebase -i HEAD~3
# Marcar o commit desejado como "edit"

# Quando o rebase pausar nesse commit:
git reset HEAD~1             # desfaz o commit, mantém as alterações

# Criar commits menores e focados
git add src/service/
git commit -m "refactor(service): extrair lógica de desconto"

git add src/controller/
git commit -m "feat(api): adicionar endpoint de desconto"

git add src/test/
git commit -m "test(discount): adicionar testes unitários"

# Continuar o rebase
git rebase --continue
```

---

## 23. Boas Práticas e Checklist

### 23.1 Regras gerais

| Categoria | Prática |
|-----------|---------|
| **Commits** | Commits pequenos, atômicos e com mensagens descritivas |
| **Branches** | Nomes descritivos com prefixo (`feature/`, `fix/`, `chore/`) |
| **Pull** | Sempre puxar antes de começar a trabalhar (`git pull --rebase`) |
| **Push** | Nunca force push em branches compartilhadas |
| **Secrets** | Nunca commitar senhas, tokens ou chaves — usar `.env` + `.gitignore` |
| **Review** | Todo código para `main` passa por PR + code review |
| **CI** | Build e testes automatizados em cada PR |
| **Histórico** | Manter histórico limpo — squash commits de WIP antes de merge |

### 23.2 Convenção de nomes de branches

```
feature/descricao-curta       Nova funcionalidade
fix/descricao-do-bug          Correção de bug
hotfix/descricao-urgente      Correção urgente em produção
chore/descricao-tarefa        Manutenção (deps, CI, config)
docs/descricao-doc            Documentação
refactor/descricao            Refatoração
test/descricao                Adição de testes
release/x.y.z                 Preparação de release

Exemplos:
  feature/carrinho-compras
  feature/JIRA-1234-login-oauth
  fix/paginacao-duplicada
  hotfix/corrigir-calculo-desconto
  chore/atualizar-spring-boot-3.4
  release/2.1.0
```

### 23.3 Checklist antes de abrir um PR

```
□ Branch atualizada com main (rebase ou merge)
□ Código compila sem erros
□ Testes passando localmente
□ Sem System.out.println ou console.log de debug
□ Sem arquivos sensíveis (.env, credentials)
□ Sem dependências desnecessárias adicionadas
□ Mensagens de commit seguem Conventional Commits
□ PR com menos de 400 linhas de diff
□ Descrição do PR explica o "porquê" da mudança
□ Screenshots para mudanças visuais (se aplicável)
```

### 23.4 Proteção de branches (GitHub)

Configurações recomendadas para a branch `main`:

| Regra | Descrição |
|-------|-----------|
| **Require pull request reviews** | Pelo menos 1 aprovação antes do merge |
| **Require status checks** | CI precisa passar (build + testes) |
| **Require branches to be up to date** | Branch deve estar atualizada com main |
| **Require signed commits** | Commits devem ser assinados com GPG |
| **Restrict pushes** | Apenas merge via PR, sem push direto |
| **Include administrators** | Regras valem também para admins |
| **Require linear history** | Forçar squash ou rebase (sem merge commits) |

---

## 24. Arquivos de Configuração do Repositório — GitHub e GitLab

GitHub e GitLab reconhecem arquivos e diretórios especiais na raiz ou em pastas convencionais do repositório. Esses arquivos configuram comportamentos da plataforma, padronizam a colaboração e melhoram a apresentação do projeto — sem necessidade de painel administrativo.

### 24.1 Visão geral dos arquivos reconhecidos

```
projeto/
├── .github/                           # ── GitHub ──
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml             # Template de issue (formulário YAML)
│   │   ├── feature_request.yml
│   │   └── config.yml                 # Configuração de criação de issues
│   ├── PULL_REQUEST_TEMPLATE.md       # Template padrão de PR
│   ├── PULL_REQUEST_TEMPLATE/
│   │   ├── feature.md                 # Templates múltiplos de PR
│   │   └── bugfix.md
│   ├── CODEOWNERS                     # Revisores automáticos por caminho
│   ├── FUNDING.yml                    # Links de financiamento/sponsor
│   ├── dependabot.yml                 # Atualização automática de deps
│   ├── SECURITY.md                    # Política de segurança
│   └── workflows/                     # GitHub Actions (CI/CD)
│       └── ci.yml
│
├── .gitlab/                           # ── GitLab ──
│   ├── issue_templates/
│   │   ├── Bug.md
│   │   └── Feature.md
│   └── merge_request_templates/
│       ├── Default.md
│       └── Hotfix.md
│
├── README.md                          # Apresentação do projeto
├── LICENSE                            # Licença open source
├── CONTRIBUTING.md                    # Guia para contribuidores
├── CODE_OF_CONDUCT.md                 # Código de conduta
├── CHANGELOG.md                       # Histórico de mudanças
├── CODEOWNERS                         # (GitLab usa na raiz)
└── .editorconfig                      # Formatação cross-IDE
```

### 24.2 README.md — Apresentação do projeto

O `README.md` é o cartão de visitas do repositório. GitHub e GitLab renderizam automaticamente na página principal.

```markdown
# Nome do Projeto

> Descrição curta e objetiva do que o projeto faz.

![Build Status](https://img.shields.io/github/actions/workflow/status/org/repo/ci.yml)
![License](https://img.shields.io/github/license/org/repo)
![Version](https://img.shields.io/github/v/release/org/repo)

## Pré-requisitos

- Java 21+
- Maven 3.9+
- Docker (para testes de integração)

## Como executar

```bash
git clone https://github.com/org/repo.git
cd repo
mvn spring-boot:run
```

## Estrutura do projeto

```
src/
├── main/java/com/example/
│   ├── controller/
│   ├── service/
│   ├── repository/
│   └── model/
└── test/
```

## Contribuindo

Veja [CONTRIBUTING.md](CONTRIBUTING.md) para o guia de contribuição.

## Licença

Este projeto está licenciado sob a [MIT License](LICENSE).
```

**Badges (shields.io):** imagens dinâmicas que mostram status do build, cobertura, versão, licença, etc. Geradas via [shields.io](https://shields.io).

### 24.3 CONTRIBUTING.md — Guia de contribuição

Arquivo reconhecido por ambas as plataformas. O GitHub exibe um link automático quando alguém abre uma issue ou PR.

```markdown
# Guia de Contribuição

## Como contribuir

1. Fork o repositório
2. Crie uma branch (`git checkout -b feature/minha-feature`)
3. Faça commits seguindo [Conventional Commits](https://www.conventionalcommits.org/pt-br/)
4. Abra um Pull Request

## Padrões de código

- **Java:** seguir o Google Java Style Guide
- **Formatação:** executar `mvn spotless:apply` antes de commitar
- **Testes:** todo código novo deve ter cobertura de testes

## Configuração do ambiente

```bash
# Clonar e configurar
git clone https://github.com/org/repo.git
cd repo
cp .env.example .env
mvn clean install

# Rodar testes
mvn test

# Rodar a aplicação
mvn spring-boot:run -Dspring-boot.run.profiles=local
```

## Mensagens de commit

Seguimos o padrão Conventional Commits:

- `feat(escopo): descrição` — nova funcionalidade
- `fix(escopo): descrição` — correção de bug
- `docs: descrição` — documentação

## Processo de review

- PRs precisam de pelo menos 1 aprovação
- CI precisa estar verde
- Resolva todos os comentários antes do merge
```

### 24.4 CODE_OF_CONDUCT.md — Código de conduta

Estabelece expectativas de comportamento na comunidade do projeto. O GitHub detecta automaticamente e exibe na aba "Community".

```markdown
# Código de Conduta

## Nosso compromisso

Estamos comprometidos em proporcionar um ambiente acolhedor e
inclusivo para todos, independentemente de experiência, gênero,
identidade, orientação, deficiência, aparência, raça, etnia,
idade, religião ou nacionalidade.

## Comportamento esperado

- Linguagem acolhedora e inclusiva
- Respeito por diferentes pontos de vista
- Aceitação de críticas construtivas
- Foco no que é melhor para a comunidade

## Comportamento inaceitável

- Linguagem ou imagens de cunho sexual
- Trolling, comentários depreciativos ou ataques pessoais
- Assédio público ou privado
- Publicação de informações privadas de terceiros

## Aplicação

Casos de comportamento abusivo devem ser reportados para
[email de contato]. Todas as denúncias serão analisadas.
```

### 24.5 SECURITY.md — Política de segurança

Orienta como reportar vulnerabilidades de forma responsável. O GitHub exibe na aba "Security" do repositório.

```markdown
# Política de Segurança

## Versões suportadas

| Versão | Suporte          |
|--------|------------------|
| 2.x    | ✅ Suportada     |
| 1.x    | ⚠️ Apenas crítico |
| < 1.0  | ❌ Sem suporte   |

## Reportando vulnerabilidades

**NÃO abra uma issue pública.**

Envie um email para security@empresa.com com:

1. Descrição da vulnerabilidade
2. Passos para reproduzir
3. Impacto potencial
4. Sugestão de correção (se houver)

Responderemos em até 48 horas com um plano de ação.

## Reconhecimento

Agradecemos publicamente (com autorização) pesquisadores que
reportam vulnerabilidades de forma responsável.
```

### 24.6 LICENSE — Licença do projeto

O GitHub detecta automaticamente o tipo de licença e exibe um badge na página do repositório.

| Licença | Uso | Permite uso comercial | Exige atribuição | Copyleft |
|---------|-----|----------------------|-------------------|----------|
| **MIT** | Permissiva, simples | Sim | Sim | Não |
| **Apache 2.0** | Permissiva, com proteção de patentes | Sim | Sim | Não |
| **GPL 3.0** | Copyleft forte | Sim | Sim | Sim (derivados devem ser GPL) |
| **LGPL 3.0** | Copyleft fraco | Sim | Sim | Parcial (só se modificar a lib) |
| **BSD 2/3** | Permissiva, mínima | Sim | Sim | Não |
| **Unlicense** | Domínio público | Sim | Não | Não |

### 24.7 CHANGELOG.md — Histórico de mudanças

Registro estruturado das alterações por versão. Pode ser gerado automaticamente a partir de Conventional Commits.

```markdown
# Changelog

Todas as mudanças notáveis do projeto são documentadas neste arquivo.

O formato segue [Keep a Changelog](https://keepachangelog.com/pt-BR/),
e o projeto adere ao [Versionamento Semântico](https://semver.org/lang/pt-BR/).

## [2.1.0] - 2025-06-15

### Adicionado
- Endpoint de busca de produtos por categoria (#89)
- Cache Redis no serviço de produtos (#92)

### Corrigido
- Paginação retornando registros duplicados com JOIN FETCH (#88)
- Timeout na consulta de relatórios com mais de 10k registros (#91)

### Alterado
- Atualizado Spring Boot 3.3 → 3.4
- Migrado de RestTemplate para RestClient

## [2.0.0] - 2025-03-01

### Adicionado
- Autenticação OAuth2 com Google e GitHub (#45)

### Removido
- Suporte a Java 17 (mínimo agora é Java 21)

### Breaking Changes
- Campo `name` dividido em `firstName` e `lastName` na API `/users`
```

### 24.8 CODEOWNERS — Revisores automáticos

Define quem é automaticamente adicionado como reviewer quando arquivos específicos são alterados em um PR/MR.

**GitHub** — arquivo em `.github/CODEOWNERS`, `CODEOWNERS` ou `docs/CODEOWNERS`:

```
# Sintaxe: <padrão>  <owners>

# Owners padrão para todo o repositório
*                           @org/tech-leads

# Backend Java — time de backend
src/main/java/              @org/backend-team
pom.xml                     @org/backend-team @org/tech-leads

# Frontend — time de frontend
src/main/resources/static/  @org/frontend-team
*.tsx                       @org/frontend-team
*.ts                        @org/frontend-team
package.json                @org/frontend-team

# Infraestrutura — time de DevOps
Dockerfile                  @org/devops
docker-compose*.yml         @org/devops
.github/workflows/          @org/devops @org/tech-leads
k8s/                        @org/devops

# Banco de dados — DBA
src/main/resources/db/      @org/dba-team
**/migration/               @org/dba-team

# Segurança — qualquer alteração em auth
**/security/                @org/security-team
**/auth/                    @org/security-team

# Documentação — qualquer pessoa pode revisar
docs/                       @org/tech-writers
*.md                        @org/tech-writers
```

**GitLab** — arquivo `CODEOWNERS` na raiz do repositório:

```
# GitLab usa a mesma sintaxe, mas suporta seções opcionais

[Backend]
src/main/java/              @backend-team

[Frontend]
*.tsx                       @frontend-team

[DevOps]
Dockerfile                  @devops
.gitlab-ci.yml              @devops @tech-leads

# Seção opcional (não bloqueia o MR se o reviewer não aprovar)
^[Docs]
*.md                        @tech-writers
```

| Aspecto | GitHub | GitLab |
|---------|--------|--------|
| **Localização** | `.github/CODEOWNERS` | `CODEOWNERS` (raiz) |
| **Seções** | Não suporta | Sim (`[Nome]` e `^[Opcional]`) |
| **Aprovação obrigatória** | Via branch protection rules | Via approval rules |
| **Wildcards** | `*`, `**`, `?` | `*`, `**`, `?` |
| **Owners** | `@user`, `@org/team`, `email` | `@user`, `@group` |

### 24.9 Templates de Issues — GitHub

#### Formato YAML (formulário interativo — recomendado)

```yaml
# .github/ISSUE_TEMPLATE/bug_report.yml
name: "🐛 Bug Report"
description: "Reportar um bug encontrado na aplicação"
title: "[Bug]: "
labels: ["bug", "triage"]
assignees: []
body:
  - type: markdown
    attributes:
      value: |
        Obrigado por reportar! Preencha o formulário abaixo.

  - type: input
    id: version
    attributes:
      label: Versão da aplicação
      description: "Em qual versão o bug ocorre?"
      placeholder: "ex: 2.1.0"
    validations:
      required: true

  - type: dropdown
    id: environment
    attributes:
      label: Ambiente
      options:
        - Produção
        - Homologação
        - Desenvolvimento
        - Local
    validations:
      required: true

  - type: textarea
    id: description
    attributes:
      label: Descrição do bug
      description: "Descreva claramente o que aconteceu"
      placeholder: "Ao clicar no botão X, deveria acontecer Y, mas acontece Z..."
    validations:
      required: true

  - type: textarea
    id: steps
    attributes:
      label: Passos para reproduzir
      description: "Liste os passos para reproduzir o problema"
      value: |
        1. Ir para '...'
        2. Clicar em '...'
        3. Rolar até '...'
        4. Observar o erro

  - type: textarea
    id: expected
    attributes:
      label: Comportamento esperado
      description: "O que deveria ter acontecido?"

  - type: textarea
    id: screenshots
    attributes:
      label: Screenshots
      description: "Se aplicável, adicione screenshots"

  - type: textarea
    id: logs
    attributes:
      label: Logs de erro
      description: "Cole a stack trace ou mensagem de erro"
      render: shell

  - type: checkboxes
    id: checklist
    attributes:
      label: Checklist
      options:
        - label: Verifiquei se não existe issue duplicada
          required: true
        - label: Testei na versão mais recente
```

```yaml
# .github/ISSUE_TEMPLATE/feature_request.yml
name: "✨ Feature Request"
description: "Sugerir uma nova funcionalidade"
title: "[Feature]: "
labels: ["enhancement"]
body:
  - type: textarea
    id: problem
    attributes:
      label: Problema ou necessidade
      description: "Qual problema esta funcionalidade resolve?"
      placeholder: "Eu frequentemente preciso de... / É frustrante quando..."
    validations:
      required: true

  - type: textarea
    id: solution
    attributes:
      label: Solução proposta
      description: "Descreva a solução que você gostaria"
    validations:
      required: true

  - type: textarea
    id: alternatives
    attributes:
      label: Alternativas consideradas
      description: "Quais alternativas você já considerou?"

  - type: dropdown
    id: priority
    attributes:
      label: Prioridade sugerida
      options:
        - Baixa (nice to have)
        - Média (melhoria significativa)
        - Alta (impede fluxo de trabalho)
```

#### Configuração de issues

```yaml
# .github/ISSUE_TEMPLATE/config.yml
blank_issues_enabled: false   # Desabilita issues sem template
contact_links:
  - name: "💬 Dúvidas e discussões"
    url: https://github.com/org/repo/discussions
    about: "Use Discussions para perguntas e dúvidas gerais"
  - name: "📖 Documentação"
    url: https://docs.exemplo.com
    about: "Consulte a documentação antes de abrir uma issue"
```

### 24.10 Templates de Issues — GitLab

No GitLab, templates de issues usam Markdown puro em `.gitlab/issue_templates/`:

```markdown
<!-- .gitlab/issue_templates/Bug.md -->
## Descrição do bug

<!-- Descreva claramente o que aconteceu -->

## Passos para reproduzir

1. Ir para '...'
2. Clicar em '...'
3. Observar o erro

## Comportamento esperado

<!-- O que deveria ter acontecido? -->

## Comportamento atual

<!-- O que realmente acontece? -->

## Ambiente

- **Versão:** 
- **Browser:** 
- **SO:** 

## Logs / Screenshots

<!-- Cole logs de erro ou adicione screenshots -->

/label ~bug ~triage
/assign @
```

```markdown
<!-- .gitlab/issue_templates/Feature.md -->
## Descrição da funcionalidade

<!-- O que você gostaria que fosse implementado? -->

## Problema que resolve

<!-- Qual problema ou necessidade motiva esta feature? -->

## Proposta de solução

<!-- Descreva como você imagina a solução -->

## Critérios de aceite

- [ ] Critério 1
- [ ] Critério 2
- [ ] Testes adicionados

/label ~feature ~backlog
```

Os comandos com `/` no final dos templates GitLab são **Quick Actions** — aplicam labels, assignees e outras ações automaticamente ao criar a issue.

### 24.11 Templates de Pull Request / Merge Request

**GitHub — Template único padrão:**

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE.md -->
## Descrição

<!-- O que foi feito e por quê? Referencie a issue com #numero -->

Closes #

## Tipo de mudança

- [ ] 🐛 Bug fix
- [ ] ✨ Nova funcionalidade
- [ ] 💥 Breaking change
- [ ] ♻️ Refatoração
- [ ] 📝 Documentação
- [ ] 🔧 Configuração / CI

## Como testar

<!-- Passos para o reviewer verificar a mudança -->

1. Checkout desta branch
2. Executar `mvn spring-boot:run`
3. Acessar `http://localhost:8080/...`

## Screenshots (se aplicável)

<!-- Para mudanças de UI, inclua antes/depois -->

| Antes | Depois |
|-------|--------|
|       |        |

## Checklist

- [ ] Código segue os padrões do projeto
- [ ] Testes adicionados/atualizados
- [ ] Documentação atualizada (se aplicável)
- [ ] Sem warnings novos no build
- [ ] Self-review realizado
- [ ] PR com escopo focado (< 400 linhas de diff)
```

**GitHub — Templates múltiplos:**

Quando há múltiplos templates em `.github/PULL_REQUEST_TEMPLATE/`, o autor escolhe ao criar o PR adicionando `?template=nome.md` na URL.

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE/feature.md -->
## Nova funcionalidade

**Issue relacionada:** #

### O que foi implementado

### Impacto em outras funcionalidades

### Testes

- [ ] Testes unitários
- [ ] Testes de integração
- [ ] Teste manual
```

```markdown
<!-- .github/PULL_REQUEST_TEMPLATE/hotfix.md -->
## Hotfix

**Severidade:** 🔴 Crítica / 🟡 Alta / 🟢 Média

**Issue:** #

### Causa raiz

### Correção aplicada

### Como validar em produção

### Rollback plan
```

**GitLab — Templates de Merge Request:**

```markdown
<!-- .gitlab/merge_request_templates/Default.md -->
## Descrição

<!-- O que foi feito e por quê? -->

## Issue relacionada

Closes #

## Checklist

- [ ] Testes passando
- [ ] Code review solicitado
- [ ] Documentação atualizada

/assign me
/label ~review
```

### 24.12 FUNDING.yml — Financiamento e sponsors (GitHub)

Exibe um botão "Sponsor" na página do repositório com links para plataformas de financiamento.

```yaml
# .github/FUNDING.yml
github: [username]                    # GitHub Sponsors
patreon: username
open_collective: project-name
ko_fi: username
buy_me_a_coffee: username
custom: ["https://exemplo.com/donate", "https://pix.exemplo.com"]
```

### 24.13 dependabot.yml — Atualização automática de dependências (GitHub)

O Dependabot abre PRs automaticamente quando há atualizações de dependências disponíveis.

```yaml
# .github/dependabot.yml
version: 2
updates:
  # Dependências Java/Maven
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "America/Sao_Paulo"
    open-pull-requests-limit: 5
    labels:
      - "dependencies"
      - "java"
    reviewers:
      - "org/backend-team"
    commit-message:
      prefix: "chore(deps)"
    # Ignorar atualizações major de libs específicas
    ignore:
      - dependency-name: "org.springframework.boot:*"
        update-types: ["version-update:semver-major"]

  # Dependências Node.js/npm
  - package-ecosystem: "npm"
    directory: "/frontend"
    schedule:
      interval: "weekly"
    labels:
      - "dependencies"
      - "frontend"
    commit-message:
      prefix: "chore(deps)"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "monthly"
    labels:
      - "ci"
    commit-message:
      prefix: "ci(deps)"

  # Docker
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"
    labels:
      - "docker"
```

### 24.14 .editorconfig — Formatação cross-IDE

Não é específico de GitHub/GitLab, mas é essencial para consistência em equipe. A maioria das IDEs (IntelliJ, VS Code com extensão, Eclipse) respeita esse arquivo automaticamente.

```ini
# .editorconfig
root = true

# Padrão para todos os arquivos
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 4

# Java
[*.java]
indent_size = 4

# XML, YML, JSON
[*.{xml,yml,yaml,json}]
indent_size = 2

# JavaScript, TypeScript, CSS
[*.{js,jsx,ts,tsx,css,scss}]
indent_size = 2

# Markdown (espaços finais são significativos)
[*.md]
trim_trailing_whitespace = false

# Makefiles (precisam de tab)
[Makefile]
indent_style = tab

# Shell scripts
[*.sh]
end_of_line = lf

# Arquivos batch Windows
[*.{bat,cmd}]
end_of_line = crlf
```

### 24.15 Comparativo GitHub vs. GitLab

| Recurso | GitHub | GitLab |
|---------|--------|--------|
| **Issue templates** | `.github/ISSUE_TEMPLATE/` (YAML ou MD) | `.gitlab/issue_templates/` (MD) |
| **PR/MR templates** | `.github/PULL_REQUEST_TEMPLATE.md` | `.gitlab/merge_request_templates/` |
| **Formulários interativos** | Sim (YAML com dropdowns, checkboxes) | Não (apenas Markdown) |
| **CODEOWNERS** | `.github/CODEOWNERS` | `CODEOWNERS` (raiz) |
| **Seções em CODEOWNERS** | Não | Sim (`[Seção]`, `^[Opcional]`) |
| **Quick Actions em templates** | Não | Sim (`/label`, `/assign`, `/milestone`) |
| **Dependabot** | Nativo (`.github/dependabot.yml`) | Via integração externa |
| **Funding/Sponsors** | Nativo (`FUNDING.yml`) | Não |
| **Wiki** | Repositório separado | Integrado ao projeto |
| **Discussions** | Sim (fórum por categorias) | Não (usa issues) |
| **Packages** | GitHub Packages (npm, Maven, Docker) | GitLab Package Registry |
| **Pages** | `gh-pages` branch ou `/docs` | Pipeline CI com artifact |

---

## Referências

- [Pro Git Book (gratuito)](https://git-scm.com/book/pt-br/v2) — livro oficial do Git em português
- [Git Reference](https://git-scm.com/docs) — documentação oficial de comandos
- [Conventional Commits](https://www.conventionalcommits.org/pt-br/) — especificação em português
- [Semantic Versioning (SemVer)](https://semver.org/lang/pt-BR/) — especificação em português
- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow) — documentação oficial
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials/comparing-workflows) — comparativo de workflows
- [gitignore.io](https://www.toptal.com/developers/gitignore) — gerador de .gitignore por tecnologia
