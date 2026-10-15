# Test Execution Report — PORTFÓLIO QA | Testes de API REST

**Autoria:** Bianca | QA Júnior em formação · **Tipo:** projeto de estudos

## 1. Dados da execução

| Item | Valor |
|---|---|
| Data e hora | 15/10/2026, 11:03:44 (horário de Brasília) |
| Ferramenta | Postman (web) · Collection Runner · execução pelo Cloud Agent |
| Collection | PORTFÓLIO QA \| Testes de API REST (21 requisições, 5 pastas) |
| Ambiente | `JSONPlaceholder | QA` (`baseUrl`, `maxResponseTime = 3000`) |
| Iterações | 1 |
| Duração total | 8s 233ms |
| Tempo médio de resposta | 335 ms |
| Configuração | Execução funcional, em ordem; "Keep variable values" ligado, porque CT-006, CT-008 e CT-014 dependem de variáveis definidas antes |

### Reexecução automática (GitHub Actions)

| Item | Valor |
|---|---|
| Execução | [run #1](https://github.com/bbmoraes/qa-api-testing-postman/actions/runs/38781401863) · 15/10/2026 14:55:51 (horário de Brasília) |
| Ferramenta | Postman CLI 1.74.0, runner `ubuntu-24.04` |
| Regressão obrigatória | 19 requisições · **66 verificações · 0 falhas** |
| Defeitos conhecidos (informativa) | CT-011 e CT-017 · 6 verificações · **4 falhas**, as mesmas da execução no Postman |
| Evidências | [`10-ci-regressao-19-casos-66-verificacoes.jpg`](../evidence/screenshots/10-ci-regressao-19-casos-66-verificacoes.jpg) · [`11-ci-defeitos-conhecidos-CT011-CT017.jpg`](../evidence/screenshots/11-ci-defeitos-conhecidos-CT011-CT017.jpg) |

O resultado bate com a execução manual: 66 + 6 = 72 verificações, com as mesmas 4 falhas.

## 2. Resultado consolidado

### Casos de teste

| Indicador | Quantidade |
|---|---|
| Planejados | 21 |
| Executados | 21 |
| Aprovados | 19 |
| Reprovados | 2 (CT-011, CT-017) |
| Bloqueados | 0 |
| Não executados | 0 |

**Taxa de aprovação dos casos executados = aprovados ÷ executados = 19 ÷ 21 = 90,5%**
(denominador: os 21 casos executados)

### Verificações automatizadas (`pm.test`)

| Indicador | Quantidade |
|---|---|
| Executadas | 72 (51 nos scripts das requisições + 21 da verificação de tempo no nível da collection, uma por requisição) |
| Aprovadas | 68 |
| Reprovadas | 4 (2 no CT-011 e 2 no CT-017) |
| Ignoradas (skipped) | 0 |
| Erros de script | 0 |

Um caso só é considerado **Aprovado** quando todas as suas verificações passam.

## 3. Resultado por caso

| ID | Método | Endpoint | Status | Tempo | Verificações | Situação |
|---|---|---|---|---|---|---|
| CT-001 | GET | /users/1 | 200 | 824 ms | 7/7 | Aprovado |
| CT-002 | GET | /users/9999 | 404 | 258 ms | 3/3 | Aprovado |
| CT-003 | GET | /users/abc | 404 | 279 ms | 3/3 | Aprovado |
| CT-004 | GET | /users/0 | 404 | 684 ms | 2/2 | Aprovado |
| CT-005 | GET | /users?parametroInexistente=teste | 200 | 272 ms | 4/4 | Aprovado |
| CT-006 | GET | /posts?userId={{userId}} | 200 | 245 ms | 5/5 | Aprovado |
| CT-007 | GET | /posts?userId=999 | 200 | 431 ms | 3/3 | Aprovado |
| CT-008 | GET | /posts/{{postId}}/comments | 200 | 222 ms | 4/4 | Aprovado |
| CT-009 | POST | /posts | 201 | 258 ms | 4/4 | Aprovado |
| CT-010 | POST | /posts (corpo `{}`) | 201 | 266 ms | 3/3 | Aprovado (limitação) |
| CT-011 | POST | /posts (JSON malformado) | **500** | 235 ms | **1/3** | **Reprovado → BUG-001** |
| CT-012 | POST | /posts (campos vazios) | 201 | 296 ms | 3/3 | Aprovado (limitação) |
| CT-013 | POST | /posts (título com 5.000 caracteres) | 201 | 411 ms | 3/3 | Aprovado |
| CT-014 | GET | /posts/{{createdPostId}} | 404 | 182 ms | 3/3 | Aprovado (comprova a limitação) |
| CT-015 | PUT | /posts/1 | 200 | 573 ms | 3/3 | Aprovado |
| CT-016 | PATCH | /posts/1 | 200 | 226 ms | 4/4 | Aprovado |
| CT-017 | PUT | /posts/9999 | **500** | 276 ms | **1/3** | **Reprovado → BUG-002** |
| CT-018 | DELETE | /posts/1 | 200 | 223 ms | 3/3 | Aprovado |
| CT-019 | DELETE | /posts/9999 | 200 | 206 ms | 3/3 | Aprovado (OBS-001) |
| CT-020 | POST | /posts/1 | 404 | 241 ms | 2/2 | Aprovado |
| CT-021 | POST | /users | 201 | 428 ms | 4/4 | Aprovado |

Os valores de status e tempo foram copiados da tela de resultados do Collection Runner. Os tempos variam a cada execução.

## 4. Investigação das falhas

As 4 verificações reprovadas foram analisadas para descobrir se o problema estava **no script ou na API**:

1. **Reexecução individual.** CT-011 e CT-017 foram enviados de novo, um de cada vez, fora do Runner. Os dois repetiram o `500` com stack trace (evidências 06 e 07).
2. **Revisão do script.** As asserções verificam exatamente o requisito definido (4xx e ausência de detalhes internos). Não havia erro de digitação, variável indefinida nem erro de execução (Errors = 0).
3. **Comparação com um cenário equivalente.** `GET /posts/9999` retorna 404 corretamente, o que mostra que a falha do CT-017 está no fluxo de atualização, e não no id usado.
4. **Conclusão.** As falhas são defeitos da API (BUG-001 e BUG-002), e não erros de script. Os scripts **não foram alterados para "passar"**. Durante a execução, o Postman sugeriu automaticamente "Fix failing tests for 500 error", e a sugestão foi ignorada de propósito, porque mudar a asserção para aceitar 500 esconderia o defeito.

## 5. Antes da execução formal

- **Exploração da API.** Antes de escrever os testes, o comportamento de cada endpoint foi verificado com requisições exploratórias feitas no navegador (fetch, a partir da página do Postman). Essas chamadas serviram para definir os cenários e **não entram nos números deste relatório**.
- **Collection de estudos.** A collection "QA | Projeto de API" tem o teste original "CT-004 | POST | Cadastrar usuário válido", com 2 asserções e um exemplo salvo de resposta `201 Created`. O histórico do Postman não mostrava a execução original desse teste, então ele foi **reexecutado** no portfólio como CT-021 (4/4 aprovadas). A collection de estudos não foi alterada.

## 6. Evidências

| Arquivo | Conteúdo |
|---|---|
| [`00a-collection-estudos-CT004-script.jpg`](../evidence/screenshots/00a-collection-estudos-CT004-script.jpg) | Script original da collection de estudos |
| [`00b-collection-estudos-CT004-exemplo-201.jpg`](../evidence/screenshots/00b-collection-estudos-CT004-exemplo-201.jpg) | Exemplo salvo com 201 Created |
| [`01-collection-portfolio-estrutura.jpg`](../evidence/screenshots/01-collection-portfolio-estrutura.jpg) | Estrutura da collection e das pastas |
| [`02-collection-runner-configuracao.jpg`](../evidence/screenshots/02-collection-runner-configuracao.jpg) | Configuração do Runner (21 de 21 selecionadas) |
| [`03-runner-resumo-72-testes.jpg`](../evidence/screenshots/03-runner-resumo-72-testes.jpg) | Resumo: 72 testes, 68 aprovados, 4 reprovados, 0 erros |
| [`04-runner-filtro-falhas.jpg`](../evidence/screenshots/04-runner-filtro-falhas.jpg) | Filtro com as 4 verificações reprovadas |
| `05a/05b/05c-runner-grid-*.jpg` | Visão em grade com aprovadas/reprovadas por requisição |
| [`06-BUG001-CT011-500-stacktrace.jpg`](../evidence/screenshots/06-BUG001-CT011-500-stacktrace.jpg) | CT-011: resposta 500 com stack trace |
| [`07-BUG002-CT017-500-stacktrace.jpg`](../evidence/screenshots/07-BUG002-CT017-500-stacktrace.jpg) | CT-017: resposta 500 com stack trace |
| [`08-CT001-execucao-individual-test-results.jpg`](../evidence/screenshots/08-CT001-execucao-individual-test-results.jpg) | CT-001 executado individualmente, aba Test Results |
| [`09-runner-CT020-CT021.jpg`](../evidence/screenshots/09-runner-CT020-CT021.jpg) | Fim da lista do Runner (CT-020 e CT-021) |

O histórico completo da execução também fica salvo no Postman: collection → aba **Runs**.

## 7. Conclusão

A collection cobre os 5 métodos HTTP (GET, POST, PUT, PATCH e DELETE) com cenários positivos, negativos, de limite e de integração. Os fluxos principais funcionaram como a documentação descreve. Os testes encontraram **dois defeitos reproduzíveis** de tratamento de erro (500 + stack trace, em vez de 4xx), registraram **uma observação** que depende de definição de requisito e **comprovaram com testes** as duas limitações documentadas da API (sem persistência e sem validação).
