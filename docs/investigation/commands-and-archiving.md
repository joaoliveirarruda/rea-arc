# Q07 e Q08 — comandos e arquivamento: sementes estáticas

**Estado: ambas em aberto.** A análise filtrou o corpus `strings -a -t x` já existente da slice arm64, sem nova varredura do binário ou leitura do perfil pessoal. O SHA-256 da slice é `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. O [recorte estruturado](/Users/juca/Projects/rea-arc/.local/evidence/commands-archiving/selected.json) conserva offsets de arquivo, textos originais, hashes das fontes, filtro e dados selecionados do dicionário AppleScript; o [extrator](/Users/juca/Projects/rea-arc/.local/evidence/commands-archiving/extract.py) reproduz a seleção. Offsets não são endereços de função. Strings e nomes isolados não demonstram execução, registros ativos ou ligação entre componentes.

## Q07 — comandos, menus e atalhos

O dicionário XML empacotado [Arc.sdef](/Applications/Arc.app/Contents/Resources/Arc.sdef), SHA-256 `d39068b2a33e7aaf1b4bdf7708eefc331524ba8c59e4b98c8feada0a82a00397`, declara a classe `tab` com Cocoa `AppleScriptTabWrapper` e associa `select`, `go back`, `go forward`, `reload`, `stop` e `close` aos seletores `select:`, `goBack:`, `goForward:`, `reload:`, `stopLoading:` e `close:`. Declara também `space` como `AppleScriptSpaceWrapper` e `focus` como `focus:`. Isso é uma interface declarada para AppleScript, não prova de convergência com menu, teclado ou Command Bar. Os seletores têm implementações no [registro de despacho](/Users/juca/Projects/rea-arc/docs/investigation/dispatch-metadata.md); seguir seus corpos é um próximo passo.

No corpus aparecem `KeyBindingMenuManager` (`0x37426f0`), `KeyBindingsClientImplementation` (`0x3742ea0`), `KeyBindingAction` (`0x3815300`), `CommandBarEntryPoint` (`0x3772f00`) e `CommandBarViewReducer` (`0x382ce20`). A string em `0x39f2d30` menciona registro duplicado e unicidade de atributo `arcKeyBindingID`; textos `KeyBindingsClient.registerAction` (`0x39f30c0`) e `KeyBindingsClient.bindingForAction` (`0x39f3030`) são sementes para localizar referências. `archiveTab` (`0x37439b8`) e `newTab` (`0x3743c08`) são outras pistas, ainda sem handler ou atalho comprovado.

O pacote contém `Base.lproj/MainMenu.nib` (SHA-256 `c05560f6665c062b78dd326c4ce56307154010f981a45f4959671fb460ab1963`), mas o formato foi identificado apenas como dados e `ibtool` não está disponível neste host; seu conteúdo não foi decodificado. O `en-GB.lproj/MainMenu.strings` é um plist com apenas dois rótulos, Edit e Copy, sem vínculos suficientes para determinar atalhos. Nenhum mapa de shortcuts ou dispatcher foi estabelecido.

## Q08 — arquivamento automático

As strings incluem `AutoArchiveInactiveThreshold` (`0x37b0160`), `AutoArchiveTimeThresholdsPerProfile` (`0x37b01a0`) e `setAutoArchiveTabsTimeThresholdForProfile(state:profile:archiveType:threshold:)` (`0x3a5f2a0`). Uma string de interface em `0x39baa00` pede um limiar de inatividade em segundos, mas isso não demonstra a unidade de todas as representações internas.

Textos contíguos entre `0x3a307e0` e `0x3a308c0` nomeiam `never`, 30 dias, 7 dias, 24, 12, 6 e 1 hora; `0x3a30900` nomeia 20 segundos para depuração. Eles documentam textos presentes no binário, não estabelecem opções exibidas em determinado contexto, valores internos, padrão ou cálculo de elegibilidade. `restoreTab(options:)` em `0x3a7fd00` é uma pista separada; não demonstra o inverso de arquivar.

O arquivo empacotado `ARC_HomeButton.bundle/Contents/Resources/archive.json` (SHA-256 `b26687a38114a3f72f2de7ab3c8c2febdc59c7c84e9b87a6069f374890942acc`) contém campos de animação como `fr`, `layers` e nome `archive-combined`; não foi tratado como configuração de limites ou padrão. Nesta inspeção delimitada não se encontrou um schema empacotado que determine os defaults. Isso não prova que inexista em outros recursos.

Os próximos passos são: localizar referências ao registro de atalhos e ao redutor da Command Bar, escolher um comando e seguir entrada→handler; depois localizar consumidores de `AutoArchiveInactiveThreshold` e da operação por perfil, investigar relógio, exclusões, comparação e chamada de arquivamento. Até haver essas cadeias, frequência, unidade interna, padrão e exclusões ficam desconhecidos.


## Metadados internos de prazos por Profile

O enum `AutoArchiveInactiveThreshold` declara oito casos sem payload: `twentySeconds`, `oneHour`, `sixHours`, `twelveHours`, `twentyFourHours`, `sevenDays`, `thirtyDays` e `never`. Evidences `ev_f44e63ea2562d4dfdf5ba640e3abbc6c5de2baebc69789220b11a043cb22c874`, `ev_b4b547cffc3230cc84b577efeae0d7fb51ab001201f149366f6f4855f6ff86d2`, `ev_dcfa02b8d1fc7bf6660bdf480b16c6014b2cbaceadb394bc431c657c4c9f36c0` e `ev_7957bb2cc4143b2b6501a04bb4c05fa9561e508a7e53e60d64bea44013c13ced`. Os nomes não estabelecem valores numéricos, disponibilidade na interface ou padrão.

A struct `AutoArchiveTimeThresholdsPerProfile` declara um campo `thresholds`; a referência simbólica de seu tipo é compatível com `Dictionary<Profile, AutoArchiveInactiveThreshold>`, apontando ao mesmo descritor de `Profile` usado por `SpaceModel.profile`. Evidences `ev_b55337944fa1f6e8e49a4db017d07baec5e5f280e7d2b72d075d71c9d49f7606`, `ev_204d47df2bb488fb30e4b36795033871c0d7f6655cf4674f629fe8dfc0b3e414`, `ev_5cf2759e9cf5a7399a0ece1571f1f74ed62d0d6f68bf3c374d709b0d6017635c` e `ev_67fff2ab6ac941807945f0b151edce5d122fd4fcb9ed50f6c037ed460aa5d6dc`. A leitura do tipo genérico é uma inferência dos metadados Swift. Ainda não foram demonstrados o relógio, as exclusões, o algoritmo ou quando a configuração é aplicada.
