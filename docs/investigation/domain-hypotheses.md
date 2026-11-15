# Hipóteses do domínio e próximos testes estáticos

## Estado da investigação

Q02–Q06 continuam **parcialmente exploradas, sem modelo de domínio confirmado**. O conjunto de strings sustenta a presença de vocabulário nominal para sidebar, stores, mutações, Profiles, restauração e ciclo de conteúdo. Ele permite formular hipóteses dirigidas; não estabelece campos, cardinalidades, ordem de execução, gravação em disco ou semântica de runtime.

Fontes consultadas:

- **S1:** `.local/evidence/swift-seeds/strings-selected.jsonl`, 182 ocorrências de `strings -a -t x`, com valor original e offset no arquivo. O relatório [swift-seeds.md](swift-seeds.md) descreve extração, filtros, corpus e demangling seletivo. São observações locais, não registros Evidence REA.
- **M1:** `.local/evidence/arc-macho.json`, Evidence `ev_905a7257d7bedaa35fa74303666fb812c0c6a017f73b6c06daf38422f93611f5`, analisada em [native-boundaries.md](native-boundaries.md). Imports/exports declarados; cobertura dos símbolos não exaustiva.
- Ambas se referem a `.local/artifacts/Arc.arm64`, SHA-256 `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. Valores e offsets abaixo foram conferidos em S1; não houve execução nativa, MCP, Ghidra nem leitura de conteúdo pessoal nesta rodada.

Os offsets `0x…` de S1 são posições no arquivo thin, **não endereços de função**. Mapear a seção/segmento antes de procurar xrefs; uma referência pode ser metadado Swift, reflection, diagnóstico ou rótulo, não uma chamada. Ao registrar um candidato, conservar offset original, endereço virtual, função efetiva, provedor e Evidence.

## Entidades candidatas e limites

| Hipótese | Observação que a motiva | O que permanece desconhecido | Próxima prova exigida |
| --- | --- | --- | --- |
| **H01 — Há representação lógica de item de sidebar separável de view.** | S1: `SidebarItemData` `0x3732bdb`, `SidebarItemDataType` `0x3738ca0`, `SidebarTableRowView` `0x36b90a0`; mangled `$s10BaseModels11SidebarDataP` `0x3bb45f0`, demangled `BaseModels.SidebarData`. | Se os nomes designam struct/class/protocol em uso; quais campos e variantes existem; como view lê os dados. | Recuperar descritores e campos/enum cases; encontrar criação de item e consumidor da view. O sufixo nominal `Type` não prova enum. |
| **H02 — Container, tab e grupo são conceitos próprios do modelo.** | S1: `SidebarContainer` `0x3737450`, `SidebarContainerType` `0x3737690`, `SidebarTab` `0x373504f`, `SidebarTabGroup` `0x3735497`; `SidebarSpaceFoldersQuery` `0x3735b00`. | Hierarquia, propriedade, ordenação, compartilhamento e relação entre folder e tab group. A presença de ambos não os torna equivalentes. | Verificar campos/referências e queries; seguir uma inserção e seu leitor para identificar pai, ID e índice. |
| **H03 — Space e Profile participam de uma atribuição explícita.** | S1: `ProfileID` `0x36fc6fe`, `ProfileAssignedToSpaceEntrypoint` `0x3773730`, `profileAssignedToSpace` `0x3e13440`, `SidebarSpacesQuery` `0x3735c10`. | Cardinalidade, possibilidade de reassociação, entidade dona do vínculo e isolamento de cookies. Os nomes podem incluir eventos analíticos. | Identificar definição do vínculo e handler de atribuição; rastrear contexto selecionado. Usar `ArcBrowserContext` de M1 como semente adicional, sem presumir correspondência 1:1. |
| **H04 — Estado de conteúdo, aba lógica e janela têm representações persistíveis distintas.** | S1: `PersistedWebContentsLifecycleState` `0x36bd1e0`, `PersistedWindowState` `0x36be120`, `PersistedWindowsState` `0x36c1610`. M1 importa `ArcSessionTab`, `ArcSessionID`, `ArcWebContentsConfiguration`. | Estruturas, codec, persistência efetiva e correspondência entre esses tipos; uma aba lógica pode ou não manter conteúdo carregado. | Recuperar cases/campos e escritor/leitor de cada representação; ligar uma transição de aba ao efeito no conteúdo. |
| **H05 — Store possui uma interface e uma implementação candidata, com vocabulário transacional.** | S1: mangled do protocolo `SidebarDataStore.SidebarDataStore` `0x3bffb0c`; `SidebarDataStoreImpl` `0x37adcc0`; `SidebarDataStoreTransaction` `0x37add10`; `SidebarDataStoreChangeSet` `0x37adbf0`; `SidebarDataStoreUpdate` `0x37adc10`; `SidebarDataStoreLiveQuery` `0x37ae000`. | Se transaction significa atomicidade, batching ou só contexto; se change set é persistido, emitido ou aplicado; se live query reativa é usada. | Encontrar conformance/implementação, entrada de mutação, commit/rollback ou equivalente e leitor de atualização. Nome transacional não garante ACID nem disco. |
| **H06 — Existe tratamento de decodificação/compatibilidade e uma etapa nominal de restauração.** | S1: `DeprecatedSidebarItemData` `0x3732c40`, `SidebarDataDecodingError` `0x3732c60`, `Unabled to decode SidebarItemData.` `0x39edac0`, `RestoreSidebarDataFlow` `0x36c1410`. | Formatos antigos, schema, default/fallback e chamada real no startup. `Deprecated` não demonstra migração executada. | Encontrar decoder e branches de compatibilidade; seguir chamador de restauração até inicialização e fonte de dados. |

Essas hipóteses não estabelecem que Favorites sejam globais, que Spaces possuam Profiles exclusivamente ou que pinned/today sejam cases de um enum. Essas relações permanecem perguntas do roteiro até que o código ou um ensaio apropriado as sustente.

## Fluxos propostos para testar

### F01 — Mutação de sidebar e atualização dos consumidores (Q02/Q06)

Caminho **hipotético**: ação de interface → instrução de mutação → alteração de item/container → store → atualização de query/view. Os nomes em S1 dão pontos de partida, sem confirmar ordem ou conectividade:

- `SidebarMutation` `0x373610b`, `SidebarItemMutator` `0x37361d0`, `SidebarSpaceMutator` `0x37361f0`.
- `SidebarInsertionStrategy` `0x3738250` e `SidebarMoveIndexingStyle` `0x37392f0` sugerem perguntas sobre inserção e índice; estratégias e regras ainda desconhecidas.
- `performSidebarMutations(on:newlyAddedItemIDs:newlyRemovedItemIDs:folderID:showNewItemsAsUnread:latestLiveItems:existingSidebarItems:instructions:newHiddenItems:)` `0x3a74160` oferece rótulos textuais de parâmetros para orientar a busca. Não foi identificado como símbolo funcional nem chamada executada.
- `SidebarDataStoreMutationContext` `0x37ad910`, transaction/change set/update de H05 e `SidebarDataStore update` `0x39b4140` permitem procurar preparação, aplicação e notificação.

**Prova para aceitar:** uma função efetiva que altera um campo/coleção de item, sua origem de dados e ao menos um consumidor da mudança; indicar se a operação também grava em store persistente. **Alternativa a testar:** o texto longo pode rotular helper intermediário que só prepara instruções; o dono da alteração pode ser outro componente.

### F02 — Leitura, restauração e instalação no estado da aplicação (Q03/Q06)

Caminho **hipotético**: leitura/decodificação → restauração/normalização → store ativo → controller de sidebar/janela. S1 contém `RestoreSidebarDataFlow` e caminho embarcado `Browser/DataStore/RestoreSidebarDataFlow.swift` `0x39bbe40`, mais `SharedAppFeature/AppSidebarDataStoreIntegration.swift` `0x3acff90` e `SidebarDataStoreIntegration` `0x38ad250`.

A string `Failed to create SidebarContainer from iCloud state. Aborting restoring content.` `0x39b6080` sustenta apenas que há um diagnóstico textual sobre falha de construção a partir de estado nomeado iCloud. O caminho `SidebarSyncCloudKit/SidebarContainer+CloudKit.swift` `0x3a3a8d0` e imports CloudKit em M1 tornam plausível procurar uma integração CloudKit. Não demonstram backend atual, acionamento desse branch, conteúdo efetivamente sincronizado ou dependência obrigatória de rede para startup.

**Prova para aceitar:** identificar chamador no startup e fonte do estado, caminho de sucesso e destino instalado no store; para a falha, identificar a condição que emite o diagnóstico e efeito após abortar. **Alternativas a testar:** restauração pode ser importação legada ou uma rotina de sincronização distinta da restauração local; o diagnóstico pode estar em código sem uso.

### F03 — Restaurar/arquivar aba e ligar contexto/conteúdo (Q03/Q04/Q05)

Caminho **hipotético**: comando de aba → alteração de estado lógico → criação/reuso/descarte de conteúdo em um contexto → atualização visível. S1 contém `archiveTab` `0x37439b8` e `restoreTab(options:)` `0x3a7fd00`; M1 contém imports de `ArcNavigation`, `ArcBackForwardListItem`, `ArcSessionTab`, `ArcBrowserContext` e `ArcWebContentsConfiguration`.

`AutoArchiveTimeThresholdsPerProfile` `0x37b01a0`, `autoArchiveTimeThresholdsPerProfile` `0x39d3dc0` e `setAutoArchiveTabsTimeThresholdForProfile(state:profile:archiveType:threshold:)` `0x3a5f2a0` sugerem investigar configuração por Profile. Os nomes não revelam unidade, valor default, relógio, elegibilidade ou aplicação do limite.

**Prova para aceitar:** um corpo funcional associado a restaurar aba, precondição e alteração observável em modelo; se houver conteúdo, localizar a chamada e o contexto escolhido. Comparar com o candidato de arquivamento para saber se preservam ID. **Alternativa a testar:** `restoreTab` pode pertencer a recurso particular ou a callback/reducer; não atribuir toda restauração de sessão a esse texto.

## Três sementes para xrefs e decompilação

Prioridade de execução quando Ghidra estiver pronto. Procurar primeiro o endereço virtual correto da ocorrência no arquivo arm64, depois referências. Se a ocorrência tiver apenas referências de metadados, seguir o descritor nominal/conformance até funções relacionadas; não decompilar o offset de string como código.

| Ordem | Semente exata de S1 | Pergunta a responder | Evidência mínima do candidato |
| --- | --- | --- | --- |
| **1** | `performSidebarMutations(on:newlyAddedItemIDs:newlyRemovedItemIDs:folderID:showNewItemsAsUnread:latestLiveItems:existingSidebarItems:instructions:newHiddenItems:)`, arquivo `0x3a74160` | Onde se aplica a mutação e quem é dono do item/container? (Q02/Q06) | Xref qualificado, função efetiva e uma escrita em estado; distinguir parâmetros inferidos de assinatura recuperada. Buscar chamador e consumidor direto após essa confirmação. |
| **2** | `RestoreSidebarDataFlow`, arquivo `0x36c1410` | Que fonte alimenta a restauração e quem recebe o resultado? (Q06) | Descritor/tipo ou função associada, leitor/decoder e chamador; provar participação no startup ou declarar outro contexto. A ocorrência ObjC `_TtC3Arc22RestoreSidebarDataFlow` em `0x3ffdff0` é uma âncora alternativa do mesmo alvo nominal, não uma quarta prioridade. |
| **3** | `restoreTab(options:)`, arquivo `0x3a7fd00` | O que restaurar muda no modelo lógico e no conteúdo? (Q04/Q05) | Função efetiva, precondição e efeito em estado; registrar chamadas ArcCore/seleção de contexto se presentes. Se a string não levar à função esperada, conservar a falha como busca delimitada. |

Cada rastreio devolve nomes/endereços e IDs Evidence reais, trechos mínimos que sustentam a interpretação, cobertura e próxima lacuna. Evitar endereços de exports compartilhados como atalhos: M1 contém 454 exports em `0x1009539c8`, cuja causa não foi estabelecida.

## Critérios para promover hipóteses a conclusões

- **Q02:** representação de entidades e relações identificadas, mais um produtor/mutador e consumidor de estado; nomes e organização de arquivos bastam apenas para hipóteses.
- **Q03:** vínculo Space/Profile e seleção de contexto demonstrados separadamente; isolamento de sessão exige prova própria.
- **Q04:** ao menos uma transição com estado anterior, evento, mutação e efeito no conteúdo; não fundir arquivamento, suspensão e descarte por semelhança nominal.
- **Q05:** caminho nativo→ArcCore e retorno ao consumidor da interface com APIs/callbacks identificados; imports sozinhos são fronteira de ligação.
- **Q06:** entidade→store→gatilho de gravação→leitura/restauração, indicando caminho/formato apenas quando recuperados. Transaction, persistido e restore em nomes não bastam para durabilidade.

O resultado desta rodada é uma ordem concreta de testes estáticos. O modelo proposto pode ser corrigido ou descartado conforme os corpos, metadados e relações forem recuperados.
