# Backstage frontend: referência sob demanda

Leia esta referência somente quando o pacote afetado for frontend Backstage.

1. Identifique no próprio pacote se ele usa o sistema novo ou o legado. Não misture APIs das duas gerações por conveniência.
2. No sistema novo, plugins são criados com `createFrontendPlugin` de `@backstage/frontend-plugin-api`; extensões fornecem páginas e outras funções. Confirme o padrão de export do pacote, inclusive `/alpha` quando utilizado.
3. Navegação entre plugins usa route refs e bindings quando esse for o padrão do projeto. Evite fixar caminhos de outro plugin sem verificar a integração local.
4. Para componente/teste, use as utilidades presentes no pacote. A documentação Backstage mostra `renderInTestApp`, React Testing Library, Jest e MSW, mas o manifesto local determina o comando executável.
5. Verifique a coerência de versões `@backstage/*` antes de introduzir imports recentes. Uma migração de arquitetura exige escopo próprio.

Fontes: [status do frontend](https://backstage.io/docs/frontend-system/), [plugins](https://backstage.io/docs/frontend-system/architecture/plugins/), [rotas](https://backstage.io/docs/frontend-system/architecture/routes/), [testes](https://backstage.io/docs/golden-path/plugins/frontend/testing/), [política de versões](https://backstage.io/docs/overview/versioning-policy/).
