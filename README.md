
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
