
# Documento de Padronização e Arquitetura de Branches QA

## 🌎 Visão Geral
Este documento descreve a organização e estrutura de branches para controle de qualidade (QA) e deploy no projeto Eduprime utilizando Git + Submódulos e Cloudflare Pages.

A proposta visa padronizar ambientes de teste isolados por tarefa, melhorar o fluxo de revisão de PRs, evitar sobrescrita de código e permitir testes de QA antes de merges definitivos em branches de deploy.

## 🔹 Estrutura de Branches

### App Principal (Repositório `app`):

| Tipo    | Nome         | Finalidade |
|---------|--------------|------------|
| Deploy  | deploy-#XXX  | Branch agregadora de várias tasks, merge na `main` |
| QA      | qa-#XXX      | Cria ambiente de QA para task específica, apontando para `fix-#XXX` do submódulo |

### Submódulo (`teacher-journey`):

| Tipo     | Nome         | Finalidade |
|----------|--------------|------------|
| Feature  | fix-#XXX     | Branch específica da task |
| Agrupadora | fix-#205 / fix-#188 | Branch que agrupa tasks para merge na `main` |

## 🌐 Deploys Automáticos com Cloudflare Pages

- Adicione: `deploy-*`, `qa-*`
- Salvar

## 🚀 Criando Ambiente QA Manualmente

```bash
git checkout -b qa-#222
cd src/modules/teacher-journey
git checkout fix-#222
cd ../../..
git add src/modules/teacher-journey
git commit -m "QA: Ambiente isolado para task #222"
git push origin qa-#222
```

Link gerado: `https://qa--222.app-c6w.pages.dev`

## 📋 Fluxo de Pull Requests

| Origem             | Destino        | Objetivo |
|--------------------|----------------|----------|
| fix-#222 (submódulo) | fix-#205      | Agregar task |
| qa-#222 (app)      | deploy-#205    | Preparar merge geral |
| deploy-#205        | main           | Merge final |

## 📄 Template QA

```
### Teste da Task #222

- Branch QA: qa-#222
- Link: https://qa--222.app-c6w.pages.dev
- PRs:
  - fix-#222 → fix-#205 (teacher-journey)
  - qa-#222 → deploy-#205 (app)
- Objetivo: Validação de campos
```

## ✅ Benefícios

- Testes isolados e rastreáveis
- Evita sobrescrita de código
- QA aprova antes de merge geral
- Escalável para times grandes


////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////



# Estratégia de QA com Script Automatizado no PowerShell (sem Cloudflare Pages)

## 🧩 Visão Geral

Esta estratégia permite que membros do time de QA posicionem automaticamente o ambiente local para testar uma task específica, sem precisar entender de Git, submodules ou criar branches adicionais. Usamos um script `.ps1` que troca para a branch correta do app e ajusta o submodule automaticamente.

---

## 🎯 Objetivo

- Automatizar o checkout de branches específicas (ex: `fix-#222`)
- Alinhar o submodule (`teacher-journey`) com a mesma branch
- Permitir testes locais diretamente a partir do PR da task
- Evitar criação de branches `qa-#XXX`

---

## 🛠 Script PowerShell: `scripts/qa-checkout.ps1`

```powershell
param(
    [string]$TaskId
)

if (-not $TaskId) {
    Write-Host "Uso: ./scripts/qa-checkout.ps1 -TaskId 222"
    exit
}

$branchName = "fix-#$TaskId"

# Salva o caminho atual
$projectRoot = Get-Location

Write-Host "`n Alternando para a branch $branchName no app..."
git fetch origin
git checkout $branchName
git pull origin $branchName

Write-Host "`n Acessando o submodule teacher-journey..."
Set-Location "src/modules/teacher-journey"
git fetch origin
git checkout $branchName
git pull origin $branchName

# Volta para o diretório do app
Set-Location $projectRoot

Write-Host "`n Ambiente configurado com sucesso para task #$TaskId"
```

---

## ▶️ Como usar

1. Certifique-se de que o repositório está clonado e com submodules:
```powershell
git submodule update --init --recursive
```

2. Execute o script com o número da task:
```powershell
.\scripts\qa-checkout.ps1 -TaskId 222
```

---

## ✅ Benefícios

- QA não precisa entender Git ou submodules
- Não é necessário criar branch `qa-#XXX`
- Garantia de ambiente alinhado com os PRs abertos
- Evita conflitos e facilita validação isolada

---

## ⚠️ Observações

- O script apenas troca branches — não faz commits nem push
- Requer que as branches `fix-#XXX` já existam localmente ou no origin

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////


# 🧠 Entendendo o 'Desencabeçamento' de Submodules no Git

## ❓ O que é o 'desencabeçamento'?

Em projetos com **Git Submodules**, como no caso do `app` que usa `teacher-journey` como submódulo, o repositório principal **não acompanha branches** do submodule. Ele **aponta para um commit fixo**.

> ❗ Isso significa que, mesmo que a branch `fix-#222` seja mergeada em `fix-#205` no submódulo, o `app` continuará apontando para o commit anterior se não for atualizado manualmente.

---

## 💥 Exemplo real do problema

### Situação:

- `deploy-#205` (app) aponta para `fix-#205` (submódulo)
- `fix-#222` (submódulo) → PR → `fix-#205` ✅ mergeado
- `fix-#222` (app) → PR → `deploy-#205` ✅ mergeado

### Resultado:

Mesmo que `fix-#222` tenha sido incorporado em `fix-#205` (submódulo), ao mergear `fix-#222` (app) em `deploy-#205`, **o `app` passa a apontar para o commit antigo (`fix-#222`)**, e **não para o topo de `fix-#205`**.

---

## ✅ Como evitar esse comportamento?

### 🔹 1. **Nunca mergear branch de feature no app**
Espere que o merge no submódulo aconteça primeiro.

Depois, no app:

```bash
cd src/modules/teacher-journey
git checkout fix-#205
git pull origin fix-#205
cd ../../..
git add src/modules/teacher-journey
git commit -m "Atualiza submodule para topo de fix-#205"
```

---

### 🔹 2. **Script pós-merge**
Crie um script para garantir que o submodule esteja sempre alinhado com a branch agrupadora (`fix-#205`):

```bash
#!/bin/bash
BRANCH=$1

cd src/modules/teacher-journey
git checkout $BRANCH
git pull origin $BRANCH
cd ../../..
git add src/modules/teacher-journey
git commit -m "Atualiza submodule para topo de $BRANCH"
```

Uso:

```bash
./scripts/update-submodule.sh fix-#205
```

---

### 🔹 3. **Automatização via CI/CD (Avançado)**

Você pode configurar um workflow no GitHub Actions que:

- Roda após merge de PRs na branch `fix-#205` do submódulo
- Cria automaticamente um commit no app atualizando o submodule
- Garante que o `deploy-#205` sempre aponta para o commit mais recente

#### Exemplo de fluxo:

1. PR mergeado em `deploy-#205` (submodule)
2. Seguido por um reapontamento automático
3. O CI do app:
   - Clona o projeto
   - Atualiza o submodule para `fix-#205`
   - Comita essa atualização no `deploy-#205`

> Isso exige um token com permissões de commit no repositório `app`.

---

## ✅ Conclusão

O Git com submodules **não segue branches** — ele salva **commits fixos**. Para garantir que sua branch principal (`deploy-#205`) aponte para o commit correto após merges, é necessário:

- Atualizar o submodule manualmente no app
- Ou automatizar isso via script ou CI

