
# Estrutura de Versionamento e QA por Task

## 📆 Visão Geral

Este documento define a nova estrutura de versionamento adotada no projeto. Cada task terá um ciclo de vida independente desde a implementação até a entrega final, com ambientes de teste unitários.

---

## 🔹 Convenção de Branches

| Tipo         | Repositório     | Nome da Branch | Finalidade                              |
| ------------ | --------------- | -------------- | --------------------------------------- |
| Feature Task | teacher-journey | `branch-#XXX`  | Implementação de uma tarefa específica  |
| Ambiente QA  | app             | `deploy-#XXX`  | Cria ambiente de teste para a task #XXX |
| Integração   | app             | `homolog`      | Branch de testes integrados e releases  |
| Produção     | ambos           | `main`         | Branch final para produção              |

---

## 💡 Regras de Alinhamento

Cada task seguirá a estrutura:

- `branch-#XXX` em **teacher-journey** ⇒ implementa a task
- `deploy-#XXX` em **app** ⇒ aponta para o `branch-#XXX` do submódulo

> Isso garante que o ambiente de QA disponível para cada tarefa seja isolado e rastreável.

**⚠️ Importante:**
Antes de fazer o merge da `deploy-#XXX` em `homolog`, é necessário garantir que ela **aponte para a branch `main` do submódulo teacher-journey**, e não mais para a branch de feature (`branch-#XXX`).

Esse passo evita o "desencabeçamento" do submódulo, garantindo que a `homolog` esteja sincronizada com a versão aprovada e mergeada na `main` do teacher-journey.

---

## 🔧 Pull Requests

| Origem        | Destino   | Repositório     | Descrição                           |
| ------------- | --------- | --------------- | ----------------------------------- |
| `deploy-#XXX` | `homolog` | app             | Envia a task para testes integrados |
| `branch-#XXX` | `main`    | teacher-journey | Atualiza código final no submódulo  |

---

## 📖 Fluxo de Trabalho por Task

1. Criar branch `branch-#XXX` em `teacher-journey`
2. Criar branch `deploy-#XXX` em `app`, apontando para `branch-#XXX` do submódulo
3. Desenvolver e testar localmente
4. QA testa via ambiente gerado pela `deploy-#XXX`
5. Se aprovado:
   - Abrir PR de `branch-#XXX` ⇒ `main` em teacher-journey
   - Atualizar `deploy-#XXX` para apontar para `main` do submódulo
   - Abrir PR de `deploy-#XXX` ⇒ `homolog` no app
6. Após validação integrada, mergeia `homolog` em `main` no app se desejado

---

## 📘 Hierarquia Visual do Fluxo

```
[branch-#XXX] (teacher-journey)
       |
       |---> PR para ---> [main] (teacher-journey)
       |
[deploy-#XXX] (app) --> aponta para branch-#XXX
       |
       | (após merge da task na main do submódulo)
       |--> reaponta para main do submódulo
       |
       |---> PR para ---> [homolog] (app)
                               |
                               |---> merge final em ---> [main] (app)
```

---

## 📄 Benefícios da Estrutura

- Redução de conflitos entre tarefas
- QA pode testar cada funcionalidade de forma unitária e depois integrada
- Garante consistência de versão na branch `homolog`, sempre apontando para o código oficial do submódulo

---

