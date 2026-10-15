# Bug Reports — PORTFÓLIO QA | Testes de API REST

**Autoria:** Bianca | QA Júnior em formação
**Sistema testado:** JSONPlaceholder — `https://jsonplaceholder.typicode.com` (API pública de demonstração)
**Execução de referência:** Collection Runner do Postman, 15/10/2026, 11:03:44 (horário de Brasília), ambiente `JSONPlaceholder | QA`

> **Sobre o critério usado.** O JSONPlaceholder não publica um contrato de erros (quais códigos devolver em cada falha). Por isso, os defeitos abaixo foram avaliados contra um **requisito de teste explícito**, baseado na semântica HTTP da RFC 9110: erros causados pelo cliente devem retornar **4xx**, e respostas de erro não devem expor detalhes internos do servidor. Cada defeito foi reproduzido duas vezes: no Collection Runner e em execução individual.

## Resumo da classificação

| ID | Título | Classificação | Severidade | Prioridade | Caso |
|---|---|---|---|---|---|
| BUG-001 | POST com JSON malformado retorna 500 e expõe stack trace | Defeito confirmado | Média | Média | CT-011 |
| BUG-002 | PUT em post inexistente retorna 500 e expõe stack trace | Defeito confirmado | Média | Média | CT-017 |
| OBS-001 | DELETE em post inexistente retorna 200 | Possível defeito não confirmado | Baixa | Baixa | CT-019 |
| LIM-001 | Criação/alteração/exclusão não persistem | Limitação conhecida da API | — | — | CT-009, CT-014, CT-018 |
| LIM-002 | API não valida o corpo das requisições | Limitação conhecida da API | — | — | CT-010, CT-012 |

Nenhum erro de script foi registrado na execução final (Errors = 0 no Runner).

---

## BUG-001 — POST com JSON malformado retorna 500 e expõe stack trace

| Campo | Informação |
|---|---|
| **Classificação** | Defeito confirmado (reproduzível) em relação ao requisito de teste |
| **Descrição** | Quando o corpo da requisição não é um JSON válido, a API responde **500 Internal Server Error** em vez de **400 Bad Request**, e o corpo da resposta traz o stack trace do servidor, com caminhos internos (`/app/node_modules/body-parser/...`). |
| **Ambiente** | Postman web (Cloud Agent) · ambiente `JSONPlaceholder | QA` · `baseUrl = https://jsonplaceholder.typicode.com` |
| **Pré-condições** | Nenhuma |
| **Passos para reproduzir** | 1. Enviar `POST {{baseUrl}}/posts` com o header `Content-Type: application/json; charset=UTF-8`.<br>2. Usar o corpo (sem a chave de fechamento):<br>`{ "title": "JSON sem fechamento", "userId": 1`<br>3. Observar o status e o corpo da resposta. |
| **Resultado esperado** | `400 Bad Request`, com uma mensagem de erro genérica e sem detalhes internos. |
| **Resultado obtido** | `500 Internal Server Error` · corpo iniciando com `SyntaxError: Expected ',' or '}' after property value in JSON at position 49`, seguido de linhas `at JSON.parse`, `at parse (/app/node_modules/body-parser/lib/types/json.js:89:19)`, etc. |
| **Severidade** | Média: o cliente recebe a indicação errada (a falha parece ser do servidor) e a resposta expõe detalhes da implementação (dependências e caminhos de arquivo). |
| **Prioridade** | Média: em uma API de produção, a exposição de stack trace costuma ser priorizada por questões de segurança. Aqui o sistema é de demonstração, então o impacto real é baixo. |
| **Evidências** | [`06-BUG001-CT011-500-stacktrace.jpg`](../evidence/screenshots/06-BUG001-CT011-500-stacktrace.jpg) · [`04-runner-filtro-falhas.jpg`](../evidence/screenshots/04-runner-filtro-falhas.jpg) · Runner: 2 verificações reprovadas no CT-011 |
| **Status da investigação** | Confirmado: reproduzido no Runner e em execução individual. Como é um serviço de terceiros, o defeito foi apenas documentado e não reportado. |

## BUG-002 — PUT em post inexistente retorna 500 e expõe stack trace

| Campo | Informação |
|---|---|
| **Classificação** | Defeito confirmado (reproduzível) em relação ao requisito de teste |
| **Descrição** | Ao tentar substituir (PUT) um post que não existe, a API responde **500** em vez de **404**. O corpo traz um `TypeError` com o stack trace do json-server. |
| **Ambiente** | Igual ao BUG-001 |
| **Pré-condições** | O post de id 9999 não existe (verificado: `GET /posts/9999` → 404 na exploração inicial) |
| **Passos para reproduzir** | 1. Enviar `PUT {{baseUrl}}/posts/9999` com `Content-Type: application/json; charset=UTF-8`.<br>2. Usar o corpo `{ "title": "Tentativa em post inexistente", "body": "x", "userId": 1 }`.<br>3. Observar o status e o corpo da resposta. |
| **Resultado esperado** | `404 Not Found`, sem erro de servidor. |
| **Resultado obtido** | `500 Internal Server Error` · `TypeError: Cannot read properties of undefined (reading 'id') at update (/app/node_modules/json-server/lib/server/router/plural.js:262:24)` … |
| **Severidade** | Média: um cenário previsível (id inexistente) gera uma exceção sem tratamento no servidor e expõe detalhes internos. |
| **Prioridade** | Média (mesmo raciocínio do BUG-001) |
| **Evidências** | [`07-BUG002-CT017-500-stacktrace.jpg`](../evidence/screenshots/07-BUG002-CT017-500-stacktrace.jpg) · [`05c-runner-grid-CT015-CT021.jpg`](../evidence/screenshots/05c-runner-grid-CT015-CT021.jpg) |
| **Status da investigação** | Confirmado: reproduzido no Runner e em execução individual. Observação: o `GET /posts/9999` trata corretamente o mesmo id (404), então a falha está no fluxo de atualização. |

## OBS-001 — DELETE em post inexistente retorna 200

| Campo | Informação |
|---|---|
| **Classificação** | Possível defeito, **ainda não confirmado** |
| **Descrição** | `DELETE /posts/9999` retorna `200 OK` com `{}`, embora o recurso não exista. |
| **Por que não foi confirmado** | A RFC 9110 não define um código obrigatório para esse caso. O `404` é a convenção mais comum, mas algumas APIs devolvem `200`/`204` por causa da idempotência do DELETE. Além disso, a API simula a exclusão sem apagar nada. Seria preciso um requisito do produto para decidir. |
| **Evidência** | CT-019: Runner, status 200, 3/3 verificações aprovadas (o teste verifica apenas a ausência de 5xx e status 200 ou 404) |
| **Próximo passo sugerido** | Em um projeto real: perguntar ao PO/desenvolvimento qual é o comportamento esperado e, só então, ajustar a asserção para o código definido. |

## Limitações conhecidas (não são defeitos)

- **LIM-001: sem persistência.** A documentação do JSONPlaceholder informa que os recursos não são realmente criados, alterados ou excluídos. O **CT-014** comprova isso: o post "criado" no CT-009 (id 101) retorna 404 quando consultado logo em seguida.
- **LIM-002: sem validação de entrada.** `POST /posts` aceita `{}`, campos vazios e `userId` de tipo incorreto, sempre com 201 (CT-010 e CT-012; na exploração inicial, `{"userId":"abc"}` também retornou 201). Por isso, esses casos testam o **comportamento documentado** e não foram tratados como defeitos.

---

## Exemplo didático — SIMULAÇÃO (não é um defeito real)

> ⚠️ **Este relatório é fictício.** Ele foi escrito apenas para mostrar o formato de um bug report de severidade alta. O comportamento descrito **não foi observado** durante os testes.

| Campo | Informação |
|---|---|
| **ID** | SIM-001 |
| **Título** | [SIMULAÇÃO] GET /users/{id} retorna dados de outro usuário |
| **Descrição** | Ao consultar o usuário 2, a API retornaria o registro do usuário 1. |
| **Ambiente** | Hipotético |
| **Passos para reproduzir** | 1. `GET /users/2` 2. Comparar o `id` da resposta com o id solicitado |
| **Resultado esperado** | `id = 2` |
| **Resultado obtido (hipotético)** | `id = 1` |
| **Severidade** | Alta: dados incorretos e possível exposição de dados de terceiros |
| **Prioridade** | Alta |
| **Evidência** | Não se aplica (simulação) |
| **Como o portfólio detectaria** | A asserção "O id retornado é o mesmo que foi solicitado" (CT-001) reprovaria nesse cenário. |
