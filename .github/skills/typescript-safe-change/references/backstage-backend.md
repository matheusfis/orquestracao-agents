# Backstage backend: referência sob demanda

Leia esta referência somente quando o pacote afetado for backend Backstage.

1. Confirme se o pacote usa o novo sistema. Nele, plugins usam `createBackendPlugin`, módulos usam `createBackendModule` e dependências compartilhadas entram por `coreServices`.
2. Plugins backend são isolados. Uma extensão de plugin existente normalmente é módulo ligado a seus extension points; não crie dependência direta entre implementações de plugins.
3. `httpRouter` exige autenticação por padrão. Não adicione `allow: 'unauthenticated'` sem requisito explícito e justificativa para a rota.
4. Preserve escopo e visibilidade da configuração: `backend` é o padrão; `secret` permanece apenas no backend. Não mova segredos para `frontend`.
5. Para testes, prefira os utilitários já instalados, como `@backstage/backend-test-utils`, e valide endpoints/serviços afetados. Confirme scripts e versões locais antes de sugerir comandos.

Fontes: [arquitetura do backend](https://backstage.io/docs/backend-system/architecture/index/), [HTTP router e autenticação](https://backstage.io/docs/backend-system/core-services/http-router/), [configuração e visibilidade](https://backstage.io/docs/conf/defining/), [política de versões](https://backstage.io/docs/overview/versioning-policy/).
