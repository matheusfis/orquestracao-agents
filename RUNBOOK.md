# Runbook — Orquestração econômica com VS Code, Copilot e OpenSpec

**Revisão:** 23/09/2026 · **Base:** VS Code 1.139.0 estável  
**Foco:** TypeScript e Backstage · **Orçamento informado:** 7.500 AI credits

> **Recomendação:** VS Code como interface principal, harness **Copilot**, modelos explícitos e skills sob demanda. Use **Local** quando precisar dos prompts OpenSpec `/opsx-*` ou de ferramentas específicas do editor. Deixe o **Copilot CLI conectado ao VS Code** como alternativa para tarefas de terminal.

Este pacote configura agentes e skills. **A instalação e a configuração do OpenSpec ficam no seu outro runbook.** Aqui ele já deve estar funcional: explicamos somente seu uso no planejamento, na execução por tarefas e no fechamento das mudanças. Nenhum software foi instalado ou conta Copilot configurada durante a criação desta entrega.

## Sumário

1. [Novidades da versão 1.139.0](#1-novidades-da-versão-11390)
2. [VS Code ou Copilot CLI?](#2-vs-code-ou-copilot-cli)
3. [Arquitetura e arquivos](#3-arquitetura-e-arquivos)
4. [Configurar o VS Code](#4-configurar-o-vs-code)
5. [Configurar e usar o Copilot CLI](#5-configurar-e-usar-o-copilot-cli)
6. [Modelos, orçamento e contexto](#6-modelos-orçamento-e-contexto)
7. [Agentes e skills](#7-agentes-e-skills)
8. [Operar com OpenSpec já configurado](#8-operar-com-openspec-já-configurado)
9. [Prompts de uso](#9-prompts-de-uso)
10. [Validação e diagnóstico](#10-validação-e-diagnóstico)
11. [Referências](#11-referências)

## 1. Novidades da versão 1.139.0

A página oficial registra **1.139.0 Stable, lançada em 23/09/2026**. A release melhora a operação de sessões; não anuncia um novo formato de agentes ou skills. [R1]

| Novidade | Aplicação neste fluxo |
|---|---|
| Dev Containers remotos em SSH, Tunnel e WSL | Útil se o projeto já tiver ambiente containerizado e Docker no host. |
| Catálogo de sessões mais rápido | Facilita manter uma mudança por conversa; não reduz tokens por si. |
| Compact View, filtros e renomeação | Facilita identificar mudança, responsável e estado. |
| `sessions.showChatTabs` — Preview | Altera a apresentação dos chats, preservando seus contextos. |

O suporte remoto usa `chat.agentHost.devContainer.enabled`, somente na **Agents Window**, com liberação gradual. Não é requisito deste pacote. A arquitetura Agent Host permite acompanhar uma sessão em mais de uma janela; isso não funde contextos de chats distintos. [R1, R2]

## 2. VS Code ou Copilot CLI?

Separe **interface** (editor/terminal), **harness** (motor da sessão/ferramentas) e **modelo** (Sol/Luna/Opus). Escolher Opus dentro do Copilot não significa executar Claude Code. [R2]

| Caminho | Quando usar | Particularidade |
|---|---|---|
| **VS Code + Copilot harness — padrão sugerido** | Implementação e revisão com acompanhamento visual | Agent Host não carrega `.prompt.md`; use skills OpenSpec. |
| **VS Code + Local + agente personalizado** | Prompts `/opsx-*` e ferramentas do editor/extensões | É outro harness, embora também use modelos do Copilot. |
| **Copilot CLI + conexão `/ide`** | Testes, scripts e fluxo centrado no terminal | Perfis e roteamento têm semântica própria; limite de créditos é suave. |
| Cloud ou outro provedor | Trabalho remoto/PR ou recurso específico | Fora deste pacote inicial; políticas/cobrança podem mudar. |

Esta é uma recomendação operacional, **não uma afirmação de que o editor custa menos que o CLI**. Compare custo por resultado. A conexão CLI–VS Code compartilha contexto do editor, diagnósticos e visualização de diffs. [R3, R4]

**Comece assim:** uma mudança pequena, uma sessão, modelos fixos e um escritor por vez. Não é necessário abrir um terminal por worker. Só introduza paralelismo após provar independência entre os lotes.

## 3. Arquitetura e arquivos

~~~text
Você aprova os artefatos OpenSpec
             │
      Orchestrator · Sol
             ├─ Researcher · Luna     somente se houver dúvida
             ├─ Implementer · Sol     produção
             ├─ Test Writer · Luna    testes delimitados
             └─ Reviewer · Sol       leitura crítica
             │
      diff + testes + tasks.md + validate
             │
      aceite humano → archive

Opus 5.5: decisão difícil ou revisão de alto risco em sessão curta,
não coordenador obrigatório de todas as tarefas.
~~~

Agentes definem **papéis e ferramentas**; skills trazem **procedimentos especializados**; OpenSpec mantém **intenção, requisitos e tarefas**. Nenhum deles, sozinho, é um sandbox do sistema operacional.

~~~text
RUNBOOK.md
.github/
  agents/                         ← cinco perfis para VS Code
    orchestrator.agent.md
    researcher.agent.md
    implementer.agent.md
    test-writer.agent.md
    reviewer.agent.md
  skills/                         ← compartilhadas por VS Code e CLI
    backstage-repo-scout/SKILL.md
    typescript-safe-change/SKILL.md
      references/                 ← frontend/backend, sob demanda
    focused-tests/SKILL.md
    backstage-catalog-template/SKILL.md
profiles/copilot-cli/agents/       ← cinco perfis alternativos para CLI
config/vscode.settings.example.jsonc
templates/WORK-ORDER.md
templates/COST-LOG.md
~~~

Não copie `profiles/` para dentro de `.github/agents`: são alternativas, não agentes extras. Não substitua skills `openspec-*`, prompts `opsx-*` ou instruções existentes.

Não adicionamos um `copilot-instructions.md` global: repetiria regras em muitas chamadas. Preserve o que seu repositório já tiver; mantenha ali somente convenções realmente globais. Os procedimentos especializados ficam nas skills. [R3, R5]

## 4. Configurar o VS Code

### 4.1 Preparar o repositório

1. Atualize VS Code para **1.139.0+** e a extensão oficial GitHub Copilot para a versão estável compatível. Entre na conta dos créditos. Confira modelos disponíveis e políticas da organização: a documentação não garante acesso à sua conta. [R1, R8]
2. Abra a **raiz do repositório TypeScript/Backstage**. Confira alterações pendentes e use sua branch de trabalho.
3. Copie os cinco arquivos de `.github/agents/` e as quatro pastas de `.github/skills/` para os mesmos caminhos no projeto. Se houver colisão de nomes, compare antes; não sobrescreva personalizações.
4. Em Settings, abra o JSON e **mescle** as chaves desejadas de [config/vscode.settings.example.jsonc](config/vscode.settings.example.jsonc). Não substitua todo o arquivo.
5. Mantenha delegação recursiva desabilitada. Se abrir apenas um pacote do monorepo, `chat.useCustomizationsInParentRepositories` permite buscar customizações no repositório pai; inicialmente prefira abrir a raiz. [R3, R5, R9]

Antes do piloto, confirme a toolchain do próprio projeto: README, `packageManager`, `engines`, arquivo de versão do Node e scripts do pacote. Prepare as dependências pelo procedimento já adotado pelo repositório e execute uma validação focal de baseline. Registre falhas anteriores. O pacote de agentes não escolhe gerenciador de pacotes nem autoriza instalações/atualizações automáticas.

### 4.2 Selecionar sessão, modelos e ferramentas

1. No Chat, escolha **Session Target → Copilot** para trabalhar com skills. No piloto, use **Folder** e permissões manuais; não habilite Autopilot ou aprovação irrestrita.
2. Abra **Configure Chat → Agents**. Deve aparecer `Orchestrator`. Os workers estão com `user-invocable: false`: ficam disponíveis para delegação, não no seletor normal.
3. Abra **Configure Chat → Skills** e confira as quatro skills, além das já geradas pelo OpenSpec. Use Diagnostics/Agent Debug Logs para investigar falhas de descoberta. [R3, R5]
4. Selecione Orchestrator e confira **GPT-6 Sol**. Os arquivos VS Code usam nomes qualificados, como `GPT-6 Sol (copilot)`; ajuste pelo seletor/editor de customizações se sua instalação exigir outro identificador.
5. Confira a delegação e as ferramentas. O fluxo documentado do VS Code usa `agent/runSubagent`, habilitado pelo alias `agent`. Ferramentas indisponíveis podem ser ignoradas; **nome no YAML não comprova ativação**. [R3, R9]

Teste inicial, sem alteração de código:

~~~text
Use Researcher para localizar o package.json do pacote que vou alterar
e o comando de teste existente. Só leitura, até 150 palavras.
Use o modelo configurado do worker, sem override e sem outros agentes.
~~~

Abra a chamada do worker e confira modelo **efetivo**, ferramentas e custo. Para testar a skill, peça leitura de `backstage-repo-scout` em um pacote Backstage. Se o harness não expuser as ferramentas/customizações esperadas, use uma sessão **Local** para o fluxo VS Code ou o perfil CLI; não presuma compatibilidade sem esse teste.

### 4.3 Local, Agent Host e permissões

- **Local:** selecione no Session Target quando quiser usar os prompts OpenSpec `/opsx-*` de `.github/prompts/`.
- **Copilot/Agent Host:** use as skills OpenSpec descobertas no menu `/`; não espere os mesmos prompt files.
- **Agents Window:** opcional para acompanhar sessões e ambientes remotos. Worktree isola arquivos de outro checkout, não rede/credenciais. O Copilot documenta Allow all nas sessões de worktree; por isso este piloto começa em Folder com permissões manuais. [R2, R3]

Restrições como “editar apenas testes” são instruções de trabalho. Os agentes com execução de terminal podem escrever por comandos; mantenha aprovações e isolamento apropriados.

## 5. Configurar e usar o Copilot CLI

### 5.1 Instalar e autenticar o CLI

Escolha **um** método. No macOS, por exemplo: [R11]

~~~bash
brew install --cask copilot-cli
copilot --version
~~~

Alternativa npm, com Node.js **22 ou superior**:

~~~bash
npm install -g @github/copilot
copilot --version
~~~

No terminal integrado, na raiz do projeto, execute `copilot`. Faça `/login` se solicitado e confira a conta. Confie somente no repositório reconhecido e revise autorizações. Não use `--allow-all`, hooks de terceiros ou modo sem confirmações no piloto. Esta seção **não instala OpenSpec**. [R11, R12]

### 5.2 Ativar os perfis de CLI

Copie os cinco arquivos de [profiles/copilot-cli/agents](profiles/copilot-cli/agents/) para **`.github/agents/` no checkout escolhido para CLI**, substituindo somente os perfis deste pacote após comparação/backup. As skills usam o mesmo formato, mas **também precisam existir fisicamente nesse checkout**: copie as quatro pastas se necessário e confirme a integração/artefatos OpenSpec já configurados ali. Não mantenha duas versões com o mesmo ID no diretório ativo. [R12, R13]

Confira também perfis homônimos em diretórios ancestrais do repositório e em `~/.copilot/agents`. Não apague personalizações para resolver colisões: identifique a origem/perfil carregado no smoke test e escolha conscientemente qual manter.

Para comparar IDE e CLI, use dois checkouts de teste ou troque os perfis entre sessões. Não rode dois escritores na mesma mudança. Os perfis não são configuração do Copilot cloud agent.

Após confirmar o ID do modelo no CLI, inicie:

~~~bash
copilot --agent=orchestrator --model=gpt-6-sol
~~~

Execute separadamente dentro da sessão:

~~~text
/model
/skills list
/skills info typescript-safe-change
/subagents
/ide
/limits set max-ai-credits 100
~~~

Confira modelos, skills e overrides. O CLI pode conectar-se automaticamente ao workspace confiável do VS Code; `/ide` permite conferir ou selecionar a conexão. [R4, R12, R14]

**Não use Auto neste piloto de roteamento fixo.** No CLI, subagentes em sessão Auto herdam o modelo resolvido do pai, independentemente do `model` do perfil. Confirme `gpt-6-sol` e `gpt-6-luna` no cliente instalado; se resolver outro modelo, pare e ajuste. [R12]

CLI delega pela ferramenta **`task`**; `runSubagent` não é comando de shell. O ID do agente vem do nome do arquivo sem `.agent.md`. Os workers não recebem ferramenta de delegação. Os perfis usam `include-custom-instructions: true`: custa contexto, mas preserva instruções do repositório. Não anule isso com `--no-custom-instructions`. [R12, R13]

### 5.3 Limites e divergências atuais

O limite de créditos está em **Preview**, é **suave** e pode ser excedido pela resposta em andamento. A página de limites exige mínimo de 30 créditos e descreve consumo cumulativo por sessão; a referência de comandos diz **por resposta, reiniciado por mensagem**. Diante dessa divergência oficial, confira `/limits` e o comportamento da versão instalada. Não o trate como garantia do orçamento mensal. [R12, R15]

Exemplo pontual, depois de validar a configuração:

~~~bash
copilot --agent=researcher --model=gpt-6-luna --max-ai-credits 30 \
  -p "Localize o comando de testes do pacote informado. Só leitura; até 150 palavras."
~~~

Não usamos `modelPolicy`/`model-policy`: referência e changelog divergem na grafia. Sem teste do parser local, não prometemos bloqueio rígido de fallback. Também não transportamos a allowlist `agents:` do VS Code como se fosse garantia equivalente no CLI. [R12, R16]

## 6. Modelos, orçamento e contexto

### 6.1 Preços atuais do Copilot

**1 AI credit = US$ 0,01** em uso medido. Os 7.500 créditos equivalem a US$ 75 nessa unidade, não a 7.500 tokens. Em 23/09, a tabela Copilot já lista Sol/Luna — atualização importante em relação à edição anterior. Valores em **US$ por milhão de tokens**. [R17]

| Modelo / entrada por chamada | Entrada | Cache lido | Cache gravado | Saída |
|---|---:|---:|---:|---:|
| GPT-6 Luna, até 272k | 0,10 | 0,01 | 0,125 | 0,50 |
| GPT-6 Luna, acima de 272k | 0,20 | 0,02 | 0,25 | 0,75 |
| GPT-6 Sol, até 272k | 2,00 | 0,20 | 2,50 | 10,00 |
| GPT-6 Sol, acima de 272k | 4,00 | 0,40 | 5,00 | 15,00 |
| Claude Opus 5.5 | 4,00 | 0,20 | 5,00 | 20,00 |

Sol custa metade de Opus em entrada não cacheada e saída; leitura de cache custa igual. Isso **não prova metade do custo por tarefa**. Chamadas, raciocínio, cache e retrabalho mudam o total. Preços das APIs dos fabricantes são referências separadas; não confunda contas OpenAI/Anthropic com saldo Copilot. [R17–R20]

### 6.2 Roteamento recomendado

| Trabalho | Começar com | Escalar quando |
|---|---|---|
| Correção trivial / pergunta focal | Luna direto | Incapacidade demonstrada → Sol |
| Coordenação de poucos lotes | **Sol** | Decisão arquitetural ou ambiguidade exige análise extra |
| Implementação TS/Backstage | Sol | Uma correção sem avanço ou dificuldade real → Opus |
| Testes delimitados | Luna | Fixtures/concorrência complexas ou falha persistente → Sol |
| Revisão focal | Sol | Autorização, migração ou alto risco → Opus/humano |

Opus recebe um **pacote de decisão**: problema, evidências, alternativas e restrições. Depois retorne a execução a Sol/Luna. Se preferir Opus como master, altere somente o modelo do Orchestrator pelo seletor aceito e meça; não aumente o contexto por consequência.

No VS Code, o modelo do worker não pode exceder o tier de custo do pai. Parâmetro explícito da chamada tem precedência sobre o perfil: peça ao coordenador para não sobrescrever modelos. Escalonar pode exigir uma nova sessão adequada. [R9]

**Alternativa a medir:** Auto → Efficiency. A documentação informa disponibilidade em VS Code/CLI e desconto de 10% nos planos pagos. É útil testar em tarefas diretas, mas não equivale ao roteamento fixo, especialmente no CLI. [R21]

### 6.3 Política para os 7.500 créditos

Sugestão de gestão, não configuração automática:

- **1.500** de reserva; **6.000** para trabalho planejado.
- Piloto pequeno: ponto de revisão em **100 créditos**, não estimativa de gasto esperado.
- Mudança maior: revisão em **300 créditos**. Vinte mudanças nesse teto somam 6.000; a quantidade real depende do consumo.
- Após uma correção sem avanço, reavalie abordagem/modelo em vez de repetir indefinidamente.
- Registre custo observado em [templates/COST-LOG.md](templates/COST-LOG.md). Não some o total da sessão aos workers já incluídos.
- Se o agente não enxergar o medidor, informe o valor do painel. Não aceite estimativa apresentada como medição.

Para tarefas pequenas, dispense o master. Não mantenha Opus recebendo logs dos testes nem repita pesquisa com evidências já disponíveis. Preserve instruções estáveis e poucas ferramentas ativas para favorecer reaproveitamento, sem assumir cache gratuito/garantido. [R10]

### 6.4 Metas de contexto

| Papel | Meta de entrada por chamada | Retorno |
|---|---:|---|
| Orchestrator | até 32k | até 250 palavras + estado |
| Researcher | até 8k | até 200 palavras + evidências |
| Implementer | até 24k | até 200 palavras + arquivos/testes |
| Test Writer | até 16k | até 200 palavras + cenários/resultados |
| Reviewer | até 16k | até 200 palavras + achados |

Estas são **metas de escopo**, incluindo instruções, histórico, ferramentas e arquivos. Não existe no schema usado um campo como `contextWindow: 16000` ou `maxInputTokens` que imponha esse teto por agente. Capacidade máxima não significa envio automático de toda a janela. [R3, R12]

O Copilot confirma tier mais caro acima de **272k tokens de entrada** para Sol/Luna. Na API OpenAI, a tarifa ampliada vale para a solicitação inteira, não apenas o excedente. Opus tem capacidade de 1M na API Anthropic, sem obrigar envio de 1M nem confirmar a janela disponível em seu cliente. [R17–R20]

**Margem operacional:** ao observar aproximadamente **100k de entrada no contexto ativo**, pare de acrescentar material. Faça checkpoint com decisões, tasks, caminhos, testes e bloqueios; abra chat novo. Tokens acumulados em muitas chamadas não são o tamanho de um único prompt. `/compact` pode ajudar, mas não é trava financeira. [R10]

Evite árvore inteira, logs completos, snapshots enormes, bundles, lockfiles e todas as skills. Prefira símbolo/caminho específico e trechos mínimos. `search.exclude` não deve ser tratado como bloqueio de acesso ou garantia de economia.

## 7. Agentes e skills

O vínculo é feito pela **descrição de ativação e instrução no corpo do agente**, não por um campo fictício `skills:`. Metadados são descobertos antes; corpo e referências entram quando pertinentes. [R5–R7]

| Skill | Papel principal | Ganho esperado |
|---|---|---|
| `backstage-repo-scout` | Researcher | Descobrir pacote, scripts, versões e arquitetura existente |
| `typescript-safe-change` | Implementer; Reviewer como checklist | Preservar contratos/tipagem; carregar regras Backstage pertinentes |
| `focused-tests` | Test Writer; Reviewer como checklist | Testar comportamento alterado com utilitários locais |
| `backstage-catalog-template` | Implementer; Reviewer | Regras específicas de catálogo e Software Templates |

Use `/typescript-safe-change` ou `/focused-tests` com o escopo. Na delegação, passe o caminho `.github/skills/<nome>/SKILL.md`; se necessário, peça sua leitura explícita. **Skill não amplia autoridade:** Reviewer não edita por encontrar um procedimento de implementação.

As regras técnicas seguem fontes oficiais TS/Backstage: preservar configurações e arquitetura existentes; evitar `any` desnecessário; distinguir frontend legado/novo; proteger segredos e autorização; usar utilitários de teste compatíveis. Detalhes ficam nas skills/referências, não repetidos no master. [R22–R28]

**Inspiração comunitária:** Gem Researcher (investigação graduada), acquire-codebase-knowledge (evidência) e test-gap-audit (auditoria focal), do Awesome Copilot. São exemplos do catálogo mantido pela comunidade no GitHub, **não ranking de popularidade**. O pacote unifica procedimentos relevantes com redação original, sem importar prompts inteiros. Ao adicionar terceiros, revise scripts, permissões, licença e custo; algumas pastas têm licença própria. [R29–R32]

Não usamos `context: fork`, experimental no VS Code, nem `allowed-tools` para pré-aprovar comandos. O isolamento já ocorre nos workers. Não é necessário instalar MCP, marketplace ou framework extra. [R5, R6]

## 8. Operar com OpenSpec já configurado

### 8.1 Conferir disponibilidade e escolher o fluxo

~~~bash
openspec --version
openspec list
~~~

Se não funcionar ou faltarem integrações, use **seu outro runbook de configuração do OpenSpec**. Não inicialize ou regenere arquivos por tentativa aqui.

| Interface/harness | Planejar | Aplicar mudança pequena inteira | Orquestração deste pacote |
|---|---|---|---|
| VS Code Local | `/opsx-propose` | `/opsx-apply` | Orchestrator após aprovar plano |
| VS Code Copilot/Agent Host | Skill OpenSpec de proposta no menu `/` | Skill de aplicação descoberta | Orchestrator com artefatos/tasks selecionadas |
| Copilot CLI | `/openspec-propose`, se listado | `/openspec-apply-change`, se listado | Perfil `orchestrator`, modelo explícito |

Nomes dependem da versão/perfil já configurados. Confirme no menu ou `/skills list`. CLI e Agent Host não consomem os prompt files `opsx-*.prompt.md`. [R3, R6, R33]

**Escolha:** aplicação OpenSpec direta para mudança pequena **ou** Orchestrator para lotes. Nunca ambos sobre as mesmas tasks.

### 8.2 Planejar e obter aceite

Exemplo Backstage: `template-owner-validation`. Em **Local**:

~~~text
/opsx-propose template-owner-validation
Planeje exigir owner do tipo Group no template de criação de serviço
e propagar esse owner ao catalog-info.yaml gerado.
Identifique template, skeleton e testes existentes.
Aceite: owner obrigatório, seleção compatível com o padrão local,
owner preservado no catálogo gerado e teste de entrada inválida.
Separe produção/template e testes. Só planeje; não implemente.
~~~

No Agent Host/CLI, invoque a skill de proposta descoberta com o mesmo pedido. `OwnerPicker`/schemas devem seguir a versão do repositório. [R28, R33]

Revise proposal, design, specs e tasks. O caminho convencional é `openspec/changes/<id>/`; se houver store/schema diferente, siga os caminhos devolvidos pelo OpenSpec. A proposta termina no planejamento: **faça um novo pedido de execução após revisar**. [R34, R35]

~~~bash
openspec status --change template-owner-validation --json
openspec instructions apply --change template-owner-validation
openspec validate template-owner-validation
~~~

`instructions apply` fornece orientação, **não implementa**. `status` acompanha artefatos, não conclusão da implementação. `validate` verifica especificações/estrutura, não substitui testes. [R34]

### 8.3 Distribuir lotes e evitar concorrência indevida

Abra sessão nova com Orchestrator e envie change ID, task IDs, aceite e teto operacional. Use [templates/WORK-ORDER.md](templates/WORK-ORDER.md) para ordens compactas.

| Etapa | Responsável | Pré-condição |
|---|---|---|
| Descobrir template/skeleton/runner | Researcher + scout, se necessário | Dúvida concreta ainda aberta |
| Alterar template e skeleton | Implementer + catalog-template | Paths e contrato definidos |
| Cobrir casos | Test Writer + focused-tests | Contrato estável; arquivos exclusivos |
| Conferir diff e critérios | Reviewer, leitura | Master envia diff focal, arquivos novos e resultados de testes |
| Consolidar checklist e validar specs | Orchestrator | Diff e testes confirmados |

Comece sequencialmente. Depois, no máximo dois workers independentes simultâneos. Arquivos distintos não garantem independência: podem compartilhar interface, fixture ou configuração. Lockfiles e `tasks.md` têm dono único.

**Risco de autoativação:** a skill oficial `openspec-apply-change` implementa tarefas e marca checkboxes. Não a carregue nos workers nem peça “aplique a mudança inteira”. Envie requisito técnico, paths e task ID; o master mantém a visão OpenSpec. Se o trace mostrar aplicação fora do lote, interrompa e refaça a ordem. Não edite a skill gerada como solução. [R36]

No CLI, se o conflito persistir, existe contingência reversível:

~~~bash
copilot skill disable openspec-apply-change
~~~

Ao terminar, restaure com `copilot skill enable openspec-apply-change` e confira `/skills list`. Confirme o nome real antes: isso altera a disponibilidade da skill, não só uma frase do prompt. [R12]

### 8.4 Fechar com evidências

1. Confira diff e preservação das alterações anteriores.
2. Execute testes focais, typecheck/lint pelos scripts existentes. Amplie validação se houver contratos compartilhados.
3. Somente Orchestrator marca `[x]` nas tasks comprovadas. Pendências continuam abertas.
4. Execute `openspec validate template-owner-validation`.
5. Após seu aceite, use comando/skill de archive correspondente ou terminal:

   ~~~bash
   openspec archive template-owner-validation
   ~~~

Archive sincroniza deltas de spec e move a mudança; revise o diff. Não presuma bloqueio automático de tasks incompletas: exija checklist, testes e review antes. Commit/push não fazem parte automática deste fluxo. [R34, R37]

## 9. Prompts de uso

### Piloto no VS Code

~~~text
Orchestrator: execute somente tasks 1.1, 1.2 e 2.1 da mudança
template-owner-validation já aprovada. Localize artefatos via OpenSpec.
Ponto de revisão: 100 AI credits observados. Sol no master/Implementer;
Luna no Researcher/Test Writer, sem substituição silenciosa.
Mostre paths e donos; execute sequencialmente neste piloto.
Use runSubagent quando o isolamento justificar, sem abrir todos os
workers por padrão. Passe requisitos técnicos e skill pertinente,
não a skill OpenSpec de aplicação. Só você atualiza tasks.md após
diff e testes. Não arquive, instale ou faça commit/push.
Se não enxergar créditos, peça o valor do painel sem estimar.
~~~

### Mesmo piloto no CLI

~~~text
Orchestrator: execute o lote aprovado de template-owner-validation.
Delegue pela ferramenta task aos perfis implementer/test-writer se
necessário. Sem Auto, fleet ou autopilot neste piloto.
Um escritor por vez, skills sob demanda, sem openspec-apply-change.
Revisão em 100 créditos observados; o limite suave não substitui o
saldo que informarei. Entregue arquivos, testes e pendências.
~~~

### Alteração pequena, sem master

Selecione Luna diretamente no chat:

~~~text
Use typescript-safe-change para corrigir somente o tratamento de
undefined em [caminho], preservando a API pública. Use o teste existente
[caminho], sem subagentes e sem nova dependência.
Execute validação focal e informe comando/resultado em 150 palavras.
~~~

Se não resolver, reavalie com Sol. Não use Orchestrator para edição direta: ele foi limitado à coordenação.

### Decisão difícil com Opus

Selecione Opus explicitamente numa sessão curta:

~~~text
Analise esta decisão [resumo], sem alterar arquivos.
Evidências: [paths, trechos mínimos, teste e erro].
Alternativas tentadas: [A/B]. Preserve arquitetura e versões locais.
Compare no máximo duas abordagens, recomende uma e devolva ordem
curta para o Implementer Sol. Sem novos agentes ou redesign completo.
~~~

### Reavaliar após dois pilotos

~~~text
Compare estas duas linhas observadas do COST-LOG [cole aqui].
Considere aceite concluído, testes e retrabalho, não apenas tokens.
Diga se pesquisa/review agregaram valor e recomende uma única
alteração de modelo ou tamanho de lote para o próximo piloto.
~~~

## 10. Validação e diagnóstico

### Antes de implementar

- [ ] Versões, conta, saldo e modelos disponíveis conferidos.
- [ ] Harness consciente: Copilot com skills; Local com prompts `opsx-*`.
- [ ] Um conjunto de cinco perfis ativo; YAML sem erros.
- [ ] Quatro skills descobertas e referências locais intactas.
- [ ] Delegação de leitura testada; modelo **efetivo** do worker confirmado.
- [ ] Instruções existentes e permissões preservadas.
- [ ] Proposta aprovada separadamente; paths, donos e aceite definidos.
- [ ] Sem outro apply/escritor no mesmo lote; saldo e reserva registrados.

### Ao concluir

- [ ] Diff dentro do escopo e sem apagar alterações anteriores.
- [ ] Comandos, resultados e validações não executadas registrados.
- [ ] Review proporcional ao risco; dúvidas não tratadas como aprovação.
- [ ] Tasks marcadas com evidências e `openspec validate` aprovado.
- [ ] Créditos medidos sem dupla contagem; retrabalho incluído.
- [ ] Aceite antes de archive e diff de sincronização de specs revisado.

No VS Code, examine custo do turno/subagente e controle de contexto; confira o saldo no painel de uso. Logs ajudam a verificar modelos, ferramentas e skills. Não presuma que o agente consiga ler esses indicadores. [R9, R10]

| Sintoma | Verificação |
|---|---|
| Worker usa modelo do pai | CLI em Auto? Override em `/subagents`? Modelo indisponível? Perfil errado? |
| `/opsx-propose` ausente | Agent Host/CLI usam skill; em Local, confira prompts da integração existente. |
| Skill ausente | Pasta/`name`, YAML, raiz; `/skills reload` no CLI após edição. |
| Ferramenta ignorada | Runtime/nome incompatível; valide painel antes de continuar. |
| Tasks marcadas cedo | Apply autoativado ou dois escritores; pare e confira diff. |
| Mais agentes gastam mais | Corte delegações sem ganho e reduza logs/escopo. |
| Testes não rodam | Informe dependência/runtime ausente; não instale silenciosamente. |
| Contexto cresce | Checkpoint curto e chat novo; janela máxima não é meta. |

**Limite da validação desta entrega:** documentação e checagens estáticas. Não executamos estes agentes na sua conta Copilot nem no seu repositório Backstage. O smoke test é obrigatório antes de confiar em roteamento e custos. Revalide ao atualizar os produtos.

## 11. Referências

Consultadas em **23/09/2026**. Comportamentos são fundamentados nas fontes abaixo; orçamento, lotes e distribuição de papéis são recomendações deste runbook. Fontes comunitárias são inspiração, não documentação normativa.

### VS Code e GitHub Copilot

- **R1.** [VS Code 1.139 — release notes](https://code.visualstudio.com/updates/v1_139)
- **R2.** [VS Code — Agent harnesses](https://code.visualstudio.com/docs/agents/run/agent-harnesses)
- **R3.** [VS Code — Custom agents e diferenças entre harnesses](https://code.visualstudio.com/docs/agent-customization/custom-agents)
- **R4.** [GitHub — Conectar Copilot CLI ao VS Code](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/connecting-vs-code)
- **R5.** [VS Code — Agent Skills](https://code.visualstudio.com/docs/agent-customization/agent-skills)
- **R6.** [GitHub — Skills no Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- **R7.** [Agent Skills — especificação](https://agentskills.io/specification)
- **R8.** [GitHub — Modelos suportados](https://docs.github.com/en/copilot/reference/ai-models/supported-models)
- **R9.** [VS Code — Subagentes](https://code.visualstudio.com/docs/agents/run/subagents)
- **R10.** [VS Code — Otimizar AI credits](https://code.visualstudio.com/docs/agents/guides/optimize-usage)
- **R11.** [GitHub — Instalar Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/set-up-copilot-cli/install-copilot-cli)
- **R12.** [GitHub — Referência de comandos, agentes e skills do CLI](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference)
- **R13.** [GitHub — Criar agentes para CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli)
- **R14.** [GitHub — Invocar agentes](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/invoke-custom-agents)
- **R15.** [GitHub — Limite suave de créditos, Preview](https://docs.github.com/en/copilot/how-tos/copilot-cli/use-copilot-cli/set-session-limit)
- **R16.** [Copilot CLI — Changelog](https://github.com/github/copilot-cli/blob/main/changelog.md)
- **R17.** [GitHub — Preços por modelo e tiers de contexto](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)
- **R21.** [GitHub — Auto: Efficiency, Balance e Intelligence](https://docs.github.com/en/copilot/concepts/models/auto-model-selection)

### OpenAI e Anthropic

- **R18.** [OpenAI — GPT-6 Sol](https://developers.openai.com/api/docs/models/gpt-6-sol)
- **R19.** [OpenAI — GPT-6 Luna](https://developers.openai.com/api/docs/models/gpt-6-luna)
- **R20.** [Anthropic — Claude Opus 5.5](https://platform.claude.com/docs/en/models/opus-5-5/overview)

### TypeScript, Backstage e exemplos comunitários

- **R22.** [TypeScript — strict](https://www.typescriptlang.org/tsconfig/strict.html)
- **R23.** [TypeScript — everyday types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- **R24.** [Backstage — Frontend system](https://backstage.io/docs/frontend-system/)
- **R25.** [Backstage — Backend architecture](https://backstage.io/docs/backend-system/architecture/index/)
- **R26.** [Backstage — httpRouter e autenticação](https://backstage.io/docs/backend-system/core-services/http-router/)
- **R27.** [Backstage — Testes frontend](https://backstage.io/docs/golden-path/plugins/frontend/testing/)
- **R28.** [Backstage — Software Templates](https://backstage.io/docs/features/software-templates/writing-templates/)
- **R29.** [Awesome Copilot — Gem Researcher](https://github.com/github/awesome-copilot/blob/main/agents/gem-researcher.agent.md)
- **R30.** [Awesome Copilot — acquire-codebase-knowledge](https://github.com/github/awesome-copilot/blob/main/skills/acquire-codebase-knowledge/SKILL.md)
- **R31.** [Awesome Copilot — test-gap-audit](https://github.com/github/awesome-copilot/blob/main/skills/test-gap-audit/SKILL.md)
- **R32.** [Awesome Copilot — licença](https://github.com/github/awesome-copilot/blob/main/LICENSE)

### OpenSpec — uso, não configuração

- **R33.** [OpenSpec — Ferramentas suportadas](https://github.com/Fission-AI/OpenSpec/blob/main/docs/supported-tools.md)
- **R34.** [OpenSpec — Referência da CLI](https://github.com/Fission-AI/OpenSpec/blob/main/docs/cli.md)
- **R35.** [OpenSpec — Propose](https://github.com/Fission-AI/OpenSpec/blob/main/skills/openspec-propose/SKILL.md)
- **R36.** [OpenSpec — Apply change](https://github.com/Fission-AI/OpenSpec/blob/main/skills/openspec-apply-change/SKILL.md)
- **R37.** [OpenSpec — Comandos e archive](https://github.com/Fission-AI/OpenSpec/blob/main/docs/commands.md)
