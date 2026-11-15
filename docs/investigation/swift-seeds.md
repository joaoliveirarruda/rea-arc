# Sementes estáticas Swift para Q02, Q03, Q04 e Q06

A slice examinada é `/Users/juca/Projects/rea-arc/.local/artifacts/Arc.arm64`, SHA-256 `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. Foram executados `/usr/bin/nm -n` e `/usr/bin/strings -a -t x`, com `swift-demangle --compact` apenas para nomes Swift selecionados. Não houve execução do Arc, Ghidra ou chamadas MCP nesta rodada. Os resultados são pistas estáticas; nenhum confirma que o código foi executado, nem por si só estabelece relações entre entidades.

O [índice filtrado de strings](/Users/juca/Projects/rea-arc/.local/evidence/swift-seeds/strings-selected.jsonl) guarda 182 ocorrências, valor original, offset hexadecimal no arquivo, fonte, SHA-256 e demangling quando aplicável. O [extrator](/Users/juca/Projects/rea-arc/.local/evidence/swift-seeds/extract.py) contém a expressão e o procedimento exatos. Os arquivos [nm completo](/Users/juca/Projects/rea-arc/.local/evidence/swift-seeds/nm-all.txt) e [strings completo](/Users/juca/Projects/rea-arc/.local/evidence/swift-seeds/strings-all.txt) registram o corpus efetivamente consultado; são dados locais, não IDs de Evidence REA. A seleção `nm` com a mesma expressão não encontrou símbolos definidos ([arquivo de resultado vazio](/Users/juca/Projects/rea-arc/.local/evidence/swift-seeds/nm-selected.jsonl)). Essa busca literal é delimitada e não prova ausência de implementações ou de símbolos de outra forma codificados. Offsets de `strings` são posições no arquivo thin, **não endereços de código**. A tabela `nm` contém também símbolos diferentes compartilhando endereço, portanto o nome sozinho não identifica um corpo distinto.

| Pergunta | Pistas selecionadas e offsets de arquivo | Próximo teste estático |
| --- | --- | --- |
| Q02 | `SidebarItemData` `0x3732bdb`; `SidebarTab` `0x373504f`; `SidebarTabGroup` `0x3735497`; `SidebarSpaceFoldersQuery` `0x3735b00`; `SidebarItemMutator` `0x37361d0`; `SidebarSpaceMutator` `0x37361f0` | Localizar metadados/tipos e referências a cada string, então buscar uma implementação de mutação e seus consumidores. Verificar campos e proprietário antes de estabelecer hierarquia ou ordem. |
| Q03 | `ProfileID` `0x36fc6fe`; `ProfileAssignedToSpaceEntrypoint` `0x3773730`; `SidebarSpacesQuery` `0x3735c10`; `SidebarSpaceViewController` `0x36c87d0` | Investigar referências ao identificador e ao ponto nomeado; encontrar estruturas e handler de seleção. O nome do evento não estabelece cardinalidade ou isolamento. |
| Q04 | `PersistedWebContentsLifecycleState` `0x36bd1e0`; `archiveTab` `0x37439b8`; `restoreTab(options:)` `0x3a7fd00`; `TablistViewController` `0x36be940` | Localizar referências aos textos e candidatos de função que os usam; inspecionar precondições, efeitos e chamadas antes de montar uma transição. Os textos podem não ser símbolos de função. |
| Q06 | `RestoreSidebarDataFlow` `0x36c1410`; `PersistedWindowState` `0x36be120`; `PersistedWindowsState` `0x36c1610`; `SidebarDataStoreImpl` `0x37adcc0`; `SidebarDataStoreTransaction` `0x37add10` | Procurar escritores e leitores associados, serialização e invocação no startup. A presença de nomes de store ou restauração não identifica caminho em disco nem prova gravação. |

Algumas strings completas são nomes mangled decodificados seletivamente; ambos os valores e seus offsets constam no índice:

| Offset no arquivo | Nome original | `swift-demangle` |
| --- | --- | --- |
| `0x3bb45f0` | `$s10BaseModels11SidebarDataP` | `BaseModels.SidebarData` |
| `0x3bffb0c` | `$s16SidebarDataStoreAAP` | `SidebarDataStore.SidebarDataStore` |
| `0x3bffc56` | `$s16SidebarDataStore0abC5QueryP` | `SidebarDataStore.SidebarDataStoreQuery` |
| `0x3b78668` | `$s3Arc29TablistViewControllerDelegateP` | `Arc.TablistViewControllerDelegate` |
| `0x3b7f786` | `$s3Arc34SidebarSpaceViewControllerDelegateP` | `Arc.SidebarSpaceViewControllerDelegate` |

Para priorizar decompilação, comece pelas referências a `SidebarItemMutator`/`SidebarSpaceMutator` e à string `performSidebarMutations(on:newlyAddedItemIDs:newlyRemovedItemIDs:folderID:showNewItemsAsUnread:latestLiveItems:existingSidebarItems:instructions:newHiddenItems:)` (`0x3a74160`), depois pelas referências a `RestoreSidebarDataFlow` e `SidebarDataStoreImpl`. Em seguida examine um candidato associado a `restoreTab(options:)` e outro ao vínculo `ProfileID`/Space. Esses são alvos de busca e possíveis candidatos, não endereços de função já identificados. Ao localizar uma função, registre o endereço efetivo, arquitetura, provedor e Evidence; investigue chamadores e consumidores apenas até responder à pergunta. Q02/Q03/Q04/Q06 permanecem em aberto nesta rodada.
