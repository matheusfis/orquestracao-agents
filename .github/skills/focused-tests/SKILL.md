---
name: focused-tests
description: Planeja ou escreve testes focados em comportamento para uma mudança TypeScript, usando o runner e as convenções do repositório. Use quando o pedido solicitar testes ou autorizar explicitamente sua implementação; adapte a Backstage somente quando o pacote afetado for Backstage.
---

# Testes focados

Responda ao escopo do pedido.
Uma revisão ou diagnóstico de cobertura é somente leitura; não crie ou altere testes sem autorização para isso.

## Selecionar o comportamento

1. Leia a mudança ou funcionalidade, critérios de aceitação, teste vizinho, `package.json` e configuração do runner do pacote.
2. Nomeie o comportamento observável que o teste precisa provar: caminho normal, erro relevante, permissão, limite ou regressão reproduzida. Escolha poucos cenários de alto valor.
3. Prefira o nível de teste mais barato que verifica o contrato real. Evite mockar justamente a lógica que pretende testar.
4. Se não houver mudança de código autorizada, entregue casos propostos com arquivo sugerido e asserções; pare aí.

Para defeitos, comece pelo caso que reproduz o sintoma.
Para mudanças de contrato, teste consumidor e produtor no nível que o repositório já usa.
Um teste de borda vale mais quando cobre a condição que poderia regressar; não enumere casos improváveis sem ligação com o pedido.

## Escrever, quando autorizado

- Siga nome, local, fixtures, factories, mocks e runner já usados no pacote. Não introduza Jest, Vitest, Playwright ou nova dependência apenas por preferência.
- Em TypeScript, mantenha os fixtures tipados e valide dados incertos nas fronteiras. Não use `any` para esconder incompatibilidades.
- Para frontend Backstage, procure `renderInTestApp`, React Testing Library e MSW já disponíveis. Teste texto, estados e chamadas percebidas pelo usuário, em vez de detalhes internos de componentes.
- Para backend Backstage, procure `@backstage/backend-test-utils` e testes de rotas/serviços próximos. Inclua autenticação e respostas de erro quando a alteração as afetar.
- Um teste novo precisa falhar para um defeito plausível e passar para a implementação correta. Não conte testes vazios como cobertura.

Quando testes exigirem rede, banco ou relógio, use as estratégias de isolamento já adotadas.
Prefira asserções sobre resultado, estado ou efeito observável; snapshots amplos podem mascarar mudanças importantes.
Preserve o escopo de arquivos: não ajuste código de produção para forçar o teste a passar sem autorização para corrigir o comportamento.

## Executar e relatar

Execute primeiro o script focado existente para o arquivo ou pacote.
Antes de interpretar sucesso, confirme que o runner descobriu ao menos um teste.
Se fizer sentido para a mudança, rode depois a checagem de tipos ou lint do pacote.
Não rode uma suíte ampla ou dependente de serviços externos sem necessidade para o pedido.

Se o runner não oferecer filtro seguro, explique o limite e escolha o comando de pacote mais próximo. Se a execução falhar por ambiente ou fixture ausente, registre a causa observada; não transforme o erro em alegação de falha funcional.

Relate casos adicionados ou propostos, comando e resultado real, falhas remanescentes e alcance da verificação. Não declare cobertura completa por porcentagem ou por um único teste.

## Fontes oficiais

- [Backstage: testes frontend](https://backstage.io/docs/golden-path/plugins/frontend/testing/)
- [Backstage: sistema de backend](https://backstage.io/docs/backend-system/)
- [TypeScript: `strict`](https://www.typescriptlang.org/tsconfig/strict.html)
