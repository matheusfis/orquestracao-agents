---
name: typescript-safe-change
description: Implementa uma mudança TypeScript delimitada respeitando tipos, configuração e padrões existentes. Use quando o pedido autorizar editar arquivos .ts ou .tsx; em projetos Backstage, consulte apenas a referência de frontend ou backend que corresponder ao pacote afetado.
---

# Mudança TypeScript delimitada

Atue apenas no escopo autorizado.
Se o pedido for explicar, diagnosticar ou revisar, permaneça em leitura e entregue conclusões sem editar.

## Preparação proporcional

1. Localize o pacote afetado, o `tsconfig` aplicável, scripts de checagem e testes próximos. Observe versões e convenções reais antes de escolher APIs.
2. Se for Backstage frontend, leia [a referência de frontend](references/backstage-frontend.md). Se for Backstage backend, leia [a referência de backend](references/backstage-backend.md). Não carregue a outra por hábito.
3. Se ambos forem afetados, leia ambas e identifique o contrato entre eles. Registre dependências de implantação quando relevantes.
4. Defina o menor conjunto de arquivos necessário para satisfazer o pedido. Preserve alterações preexistentes de outras pessoas.

Se o pedido mencionar apenas uma função ou componente, examine primeiro suas entradas, saídas e consumidores imediatos.
Em monorepos, não rode uma checagem global para descobrir o pacote: o manifesto e os imports normalmente indicam o limite.
Quando faltar um contrato necessário, busque sua declaração antes de criar um tipo paralelo.

## Implementação

- Preserve ou melhore a checagem de tipos do pacote. Não desative `strict` ou adicione `any`/asserções apenas para calar erros.
- Prefira inferência para valores locais e tipos explícitos nas fronteiras públicas quando isso esclarecer o contrato.
- Para dados externos de forma incerta, modele como `unknown`, valide ou estreite o tipo antes do uso.
- Respeite imports, aliases, módulos, versões e padrões de tratamento de erro do repositório.
- Evite mudanças de dependências, APIs públicas, esquemas ou arquitetura sem relação necessária com a tarefa.
- Se a solução exigir migrar um plugin legado ou alterar contrato entre pacotes, exponha essa dependência antes de expandir o escopo.

Ao mudar uma função pública, procure chamadas e testes próximos para manter o contrato.
Ao lidar com união discriminada, faça estreitamento explícito para os casos relevantes; não use cast para atravessar estados impossíveis.
Para payloads externos, a validação em runtime continua necessária mesmo quando há tipo estático.

## Verificação e entrega

Use primeiro o script existente do pacote afetado para typecheck, lint e testes relevantes.
`tsc --noEmit` só é substituto adequado quando a configuração local o suporta; não invente um comando de monorepo.
Execute a verificação necessária ao pedido, leia a falha e corrija apenas defeitos da mudança.

Ao entregar, diga quais arquivos mudaram, qual comportamento foi alterado, quais checagens passaram e quais limites permanecem. Não afirme que todo o projeto compila com base em uma checagem parcial.

Se uma checagem falhar antes da sua alteração ou fora do pacote, separe esse resultado da falha introduzida pelo trabalho atual. Não faça refatoração ampla para silenciar diagnósticos antigos.

## Fontes oficiais

- [TypeScript `strict`](https://www.typescriptlang.org/tsconfig/strict.html)
- [TypeScript: `any` e `noImplicitAny`](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript: estreitamento de tipos](https://www.typescriptlang.org/docs/handbook/2/narrowing)
- [TypeScript `noEmit`](https://www.typescriptlang.org/tsconfig/noEmit.html)
