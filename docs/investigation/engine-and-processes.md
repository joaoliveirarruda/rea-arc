# ArcCore e processos do Arc

## Snapshot de processos

A coleta usou `ps -axo pid=,ppid=,comm=` e reteve apenas PID, PPID e `comm` (que, para estes processos neste host, contém o caminho executável). Argumentos de processo e URLs não foram coletados.

O processo principal observado foi Arc PID `71319`, PPID `1`, em `/Applications/Arc.app/Contents/MacOS/Arc`. Ele tinha 23 filhos no snapshot: 18 processos com executável `Browser Helper (Aperitif Renderer)` e 5 `Browser Helper (Aperitif)`, todos sob `ArcCore.framework/Helpers/`. Os caminhos de processo sem `Versions/A` resolvem para os executáveis inventariados em `ArcCore.framework/Versions/A/Helpers/`; 24 dos 26 processos correlacionam com os executáveis registrados anteriormente (Arc + 23 helpers).

Também apareceram dois processos com PPID `1`, fora da árvore PID 71319: `Autoupdate` sob `Sparkle.framework` instalado e um `Updater` em caminho de cache com bundle ID `company.thebrowser.Browser`. O segundo não foi atribuído à árvore do Arc; o caminho exato está no JSON. Processos e PIDs são observações transitórias do snapshot.

## ArcCore: build e dependências estáticas

O binário inspecionado foi `/Applications/Arc.app/Contents/Frameworks/ArcCore.framework/Versions/A/ArcCore`, Mach-O universal com slices `x86_64` e `arm64`, tipo shared library, tamanho 507.526.464 bytes. SHA-256: `de2626e5739686cd59df1d205e161d9b5020fa0038af209f8a4d9a5f346d3220`; confere com o inventário inicial.

O `Info.plist` do framework declara ID `company.thebrowser.browser.framework`, `CFBundleShortVersionString = 157.0.8103.51` e `CFBundleVersion = 8103.51`. Isso é metadado de versão do framework ArcCore, não uma versão de runtime do Chromium verificada. O plist de origem tem SHA-256 `1fdfb675e5db986422a95147804a4656b7a9b8572e3c8a06e4e80742d1592ca9`.

`otool -L` lista 68 nomes de load por arquitetura, idênticos entre as slices: o próprio ArcCore e dependências Apple em `/System/Library` ou `/usr/lib`. Entre elas estão AppKit, CoreFoundation, CoreGraphics, CFNetwork, AVFoundation, Metal, Security e CoreBluetooth. A lista completa com versões de compatibilidade/current está no arquivo bruto e no JSON. Isso descreve load commands gravados no Mach-O, não todos os módulos carregados durante execução.

## Marcadores e símbolos

A varredura `strings` reteve somente tokens Chromium/Chrome/CEF e suas contagens, sem linhas de contexto. Há marcadores `Chrome`/`Chromium` e oito tokens distintos `chromium/<dígitos>`; estes não identificam, por si, uma versão semântica do navegador. A busca CEF de comprimento mínimo 3 encontrou variantes isoladas de `CEF`, mas nenhum token `libcef`, `cef_version` ou nome de API `Cef*`. Como não se reteve contexto, os tokens isolados não provam presença nem ausência de código CEF. Nenhum desses marcadores determina a versão efetiva do motor em runtime.

`nm -gU` encontrou, em cada arquitetura, 147 símbolos globais definidos `_OBJC_CLASS_$_Arc*` e 105 `_OBJC_METACLASS_$_Arc*`; os conjuntos coincidem entre arm64 e x86_64. Exemplos incluem `ArcBrowser`, `ArcBrowserApplication`, `ArcBrowserContext`, `ArcWebContents` e `ArcWebContentsNavigationController`. Isso atesta nomes na tabela de símbolos exportados do binário; não demonstra que cada classe seja alcançada ou usada.

## Caminhos `txt`/`mem` observados em runtime

Em `2026-11-15 03:18:21 UTC`, `lsof -nP -a -p 71319 -d txt,mem -Fn` confirmou PID 71319 como o executável Arc e enumerou entradas `txt`/`mem`. Depois de filtrar, foram preservados 104 caminhos únicos sob `/Applications/Arc.app/`; 156 campos de nome fora desse bundle foram descartados antes de gravar a evidência. Nenhum conteúdo de arquivo, socket, argumento, URL ou caminho de perfil foi persistido.

Entre os caminhos do bundle estavam ArcCore, Lottie, Sparkle e os recursos `chrome_100_percent.pak`, `chrome_200_percent.pak`, `resources.pak`, `icudtl.dat` e `pt_BR.lproj/locale.pak` sob `ArcCore.framework/Resources`. Esta amostra confirma que esses arquivos do pacote estavam enumerados como entradas de texto/memória do processo naquele momento. Junto aos marcadores no binário, sustenta uma atribuição ao stack de recursos da família Chromium; não verifica uma versão exata do motor. A enumeração é não atômica: mapeamentos podem mudar durante a chamada.

## Evidência

O registro estruturado, com PIDs/PPIDs/caminhos filtrados, versões, hashes, dependências `otool`, tokens retidos, símbolos selecionados e a lista filtrada `lsof`, está em [engine-and-processes.json](../../.local/evidence/engine/engine-and-processes.json), SHA-256 `2a98a0147708ca0f388fc2645d7062fb3c6e5b7c6fbd42006330f023da1cc673`. A saída bruta de `otool -L` está em [otool-L.txt](../../.local/evidence/engine/otool-L.txt), SHA-256 `1666b05226c0a7c86712fa107f6292a57f3877e9969764696a3a3607fdb6e6d0`. A versão ArcCore e o hash do executável foram reutilizados de [arc-inventory.json](../../.local/evidence/inventory/arc-inventory.json).

Não houve interação com a interface, automação, Ghidra, REA/MCP, rede ou conteúdo de browsing/perfil.
