
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

--------------------------------------------------


# Estratégia Alternativa de QA com Script Automatizado (sem Cloudflare Pages)

## 🔹 Contexto
Esta abordagem apresenta uma alternativa ao fluxo baseado em ambientes automatizados via Cloudflare Pages. Aqui, o foco é facilitar o trabalho do QA usando **um script inteligente** que posiciona automaticamente o repositório local do desenvolvedor ou QA na branch correta, apontando para o commit certo no submódulo, **sem a necessidade de conhecer Git, submodules ou criar branches adicionais**.

## 🚀 Objetivo do Script
Permitir que o QA rode um simples comando com o número da task e seja automaticamente posicionado na branch correta do `app`, com o `teacher-journey` apontando para a branch específica da task.

## 🔁 Cenário
- O PR da task já foi criado:
  - `fix-#222` (teacher-journey) → `fix-#205`
  - `fix-#222` (app) → `deploy-#205`
- A branch `fix-#222` no app **referencia a branch `fix-#222` do submodule**
- O QA não precisa mais criar uma branch `qa-#222`

## 🛠 Script: `scripts/qa-checkout.sh`

```bash
#!/bin/bash

TASK_ID=$1

if [ -z "$TASK_ID" ]; then
  echo "Uso: ./scripts/qa-checkout.sh 222"
  exit 1
fi

FEATURE_BRANCH="fix-#$TASK_ID"

echo "\n🔍 Alternando para a branch $FEATURE_BRANCH no app..."
git fetch origin
git checkout $FEATURE_BRANCH || {
  echo "❌ Branch $FEATURE_BRANCH não encontrada. Verifique se o PR foi criado."
  exit 1
}
git pull origin $FEATURE_BRANCH

echo "\n🔁 Acessando o submodule teacher-journey..."
cd src/modules/teacher-journey
git fetch origin
git checkout $FEATURE_BRANCH || {
  echo "❌ Branch $FEATURE_BRANCH não encontrada no submodule."
  exit 1
}
git pull origin $FEATURE_BRANCH
cd ../../..

echo "\n✅ Ambiente configurado com sucesso para task #$TASK_ID"
echo "Você está agora na branch $FEATURE_BRANCH do app e do submodule."
```

## ▶️ Como o QA usa

1. Clonar o repositório normalmente (se ainda não tiver)
2. Inicializar os submodules:
```bash
git submodule update --init --recursive
```
3. Rodar o script:
```bash
./scripts/qa-checkout.sh 222
```

## ✅ Benefícios

- Sem necessidade de criar branch `qa-#xxx`
- QA sempre testa diretamente o que está no PR da task
- Evita divergência entre app e submodule
- Simples, direto e seguro para qualquer membro da equipe

## ⚠️ Observações

- O script não cria branches nem faz commits
- Apenas sincroniza localmente o estado correto para testes

