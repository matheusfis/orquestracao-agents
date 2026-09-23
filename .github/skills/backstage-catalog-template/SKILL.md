---
name: backstage-catalog-template
description: Analisa ou altera descritores do catálogo e Software Templates do Backstage, preservando formato, relações e segredos. Use somente quando a tarefa mencionar catalog-info.yaml, entidades do catálogo, Scaffolder ou template.yaml; edite apenas se o pedido autorizar.
---

# Catálogo e Software Templates Backstage

Esta skill tem uso específico. Não a aplique a uma tarefa genérica de plugin TypeScript.
Uma consulta ou revisão permanece somente leitura; editar YAML, executar ações de Scaffolder ou publicar repositórios exige que o pedido inclua essas ações.

## Identificar o artefato

1. Localize o arquivo pedido e exemplos próximos no mesmo repositório. Registre o pacote, a versão Backstage e a configuração local relevante.
2. Distinga descritor do catálogo (`catalog-info.yaml`, por exemplo `kind: Component`) de Software Template (`kind: Template`, `apiVersion: scaffolder.backstage.io/v1beta3` quando essa versão for usada).
3. Leia o `apiVersion`, `kind`, `metadata`, `spec` e relações existentes antes de propor mudanças. Preserve nomes e referências de entidades usadas por outros arquivos.
4. Consulte a documentação oficial do formato correspondente; não transplante campos de exemplos de outra versão.

Observe o tipo de entidade antes de exigir campos: `Component`, `API`, `Resource`, `System`, `User` e `Group` não compartilham o mesmo `spec`.
Em templates, identifique quais actions a instalação disponibiliza antes de sugerir uma nova etapa.
Leia só as etapas adjacentes à alteração para entender como outputs são consumidos.

## Mudanças autorizadas

- No catálogo, mantenha `apiVersion`/`kind` válidos e preserve nomes/relações não envolvidos no pedido. Se a alteração solicitada atingir `metadata.name`, `spec.owner` ou `spec.system`, confira referências e entidades dependentes.
- Distinga o `spec.owner` da entidade Template do owner do componente gerado. A tarefa pode exigir passar `parameters.owner` para o skeleton e seu `catalog-info.yaml`; siga o fluxo real de valores do template.
- Em templates, preserve o fluxo de parâmetros, etapas e outputs existente. Para uma nova action customizada, siga as convenções da instalação; prefira IDs de etapa em camelCase quando referenciados em expressões, pois hífens podem ser lidos como subtração.
- Mantenha credenciais fora de parâmetros públicos e do YAML versionado. Use mecanismos de segredo do Scaffolder ou configuração backend já adotados.
- Não execute `publish:*`, criação de repositório ou etapa externa apenas para validar o YAML; isso altera sistemas fora do arquivo.
- Se uma mudança no template requer action/backend não presente, explicite a dependência antes de ampliar o escopo.

Em integrações com repositórios, confira a referência a owner/repo e as permissões exigidas pela action já configurada.
Uma verificação local pode confirmar estrutura e expressões; ela não prova que um token, provider ou destino remoto funcionará.
Não simule sucesso de publicação.

## Verificar e entregar

Use validações locais já existentes do projeto, se forem seguras para o escopo.
Para configuração Backstage, `backstage-cli config:check` é pertinente quando há schema/config alterado; não o trate como validador completo de todos os templates.
Confira referências e expressões das etapas por inspeção.
Relate o que foi validado e o que depende de execução real do Scaffolder.

Se houver exemplos próximos, mantenha consistência de indentação e nomes. Evite reformatar o YAML inteiro para uma edição pequena, pois isso dificulta a revisão e pode ocultar alterações semânticas.

## Fontes oficiais

- [Formato dos descritores do catálogo](https://backstage.io/docs/features/software-catalog/descriptor-format/)
- [Software Templates](https://backstage.io/docs/features/software-templates/)
- [Escrita de templates e segredos](https://backstage.io/docs/features/software-templates/writing-templates/)
- [Configuração Backstage](https://backstage.io/docs/conf/)
