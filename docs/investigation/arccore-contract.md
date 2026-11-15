# Q05 — contratos de despacho do ArcCore

O registro `inspect_native_dispatch_metadata` REA `ev_a48a2a9f13465f010f64013f317f23399d622c8b9ab230e74d008d596ca7d1c2` examina a cópia arm64 `/Users/juca/Projects/rea-arc/.local/artifacts/ArcCore.arm64`, SHA-256 `614a79f5752d2fb5df340195b93c5448a80731665c8fac573b9786bc17417764`, derivada do universal SHA-256 `de2626e5739686cd59df1d205e161d9b5020fa0038af209f8a4d9a5f346d3220` conforme identidade registrada pelo coordenador. Foi usado `max_records: 20000`, `max_input_bytes: 536870912`. Esta rodada leu somente o [bundle já exportado](/Users/juca/Projects/rea-arc/.local/evidence/arccore-dispatch-metadata.json), sem repetir o produtor ou abrir motor. O [recorte](/Users/juca/Projects/rea-arc/.local/evidence/arccore-selected.json) preserva registros, localizações, cobertura e parâmetros; o [extrator](/Users/juca/Projects/rea-arc/.local/evidence/select-arccore.py) documenta a seleção.

## Entradas candidatas

Os endereços são `implementation_address` relativos à imagem ArcCore arm64, não endereços do executável Arc nem prova de chamada. Os nomes são seletores encontrados em metadados Objective-C; assinar um seletor não revela o corpo ou seu comportamento.

| Classe | Seletor | Endereço | Pergunta para a próxima análise |
| --- | --- | --- | --- |
| `ArcWebContentsNavigationController` | `load:initiatorOrigin:` | `0x454fdc4` | Onde é entregue o pedido de navegação e quem chama esta entrada? |
| `ArcWebContentsNavigationController` | `load:initiatorOrigin:transition:fromAddressBar:typedWithHttpScheme:` | `0x454fddc` | Qual é a relação entre as duas entradas de carregamento? |
| `ArcWebContentsNavigationController` | `goBack`, `goForward`, `reload` | `0x455031c`, `0x4550698`, `0x4550e7c` | Qual chamada efetiva alcança a navegação do motor? |
| `ArcWebContents` | `setNavigationDelegate:` | `0x45796b4` | Onde o delegate é atribuído e a que objeto aponta? |
| `ArcWebContents` | `navigationController` | `0x454a290` | Quem obtém o controlador? |
| `ArcWebContents` | `title`, `url`, `isLoading` | `0x419f940`, `0x41a3ee0`, `0x4544eec` | Quem atualiza e observa essas propriedades? |
| `ArcBrowser` | `createWebContents:`, `restoreWebContentsWithSessionId:` | `0x451978c`, `0x451add4` | Que caminho cria ou restaura conteúdo? |
| `ArcBrowserContext` | `loadUrl:withDelegate:`, `createWebContents:` | `0x45261cc`, `0x452b8f0` | Qual o escopo e os chamadores de cada operação? |

O registro de ivars associa a `ArcWebContents` os nomes `_web_contents_navigation_delegate_bridge` e `_webContentsObserver`; associa `_browserDelegate` a `ArcBrowser` e `_contextDelegate` a `ArcBrowserContext`. Os registros originais, incluindo offsets reportados e estado de decodificação, estão no recorte. Sua presença não demonstra a identidade do objeto em execução, nem que o callback passa por ele.

## Comparação delimitada com ADK

O [registro do executável Arc](/Users/juca/Projects/rea-arc/docs/investigation/dispatch-metadata.md) associa à classe `_TtC3ADK18WebContentChromium` os seletores `webContents:didStartProvisionalNavigation:` (`0x100540f30`), `webContents:didCommitNavigation:` (`0x100540f98`) e `webContents:didFinishNavigation:` (`0x1005410ac`). São endereços em **outra imagem**, identificada por SHA-256 `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. Os nomes de delegate e os seletores são compatíveis com uma hipótese de fronteira, mas não comprovam atribuição, conformidade, invocação ou direção de dados.

Foi comparado literalmente cada seletor Objective-C de `WebContentChromium` no recorte ADK com os métodos dos 69 registros de protocolo decodificados em ArcCore: não houve interseção exata nessa seleção. Os protocolos cujos nomes ou seletores contêm `navigation`, `webContents` ou `browserContext` foram `NSAccessibility` e `HistorySwiperDelegate`; nenhum estabelece o protocolo de callback do ADK. A comparação não prova que tal protocolo ou ligação inexista fora da cobertura decodificada. O [desassemblado limitado do callback ADK](/Users/juca/Projects/rea-arc/docs/investigation/bridge-call-sites.md) mostra uma chamada indireta ainda não resolvida; a próxima etapa é seguir atribuição de delegate e destino dessa indireção nos corpos correspondentes, sem supor que se referem um ao outro.

## Cobertura

O resultado contém 646 linhas de classe, 4.316 implementações, 908 ivars, 69 protocolos e 32 categorias. Classes/métodos/ivars são **parciais**: 648 entradas examinadas, 646 decodificadas, com duas strings de metadados inválidas ou sem término e propriedades de classe de `NSSecureCoding` não decodificadas. Ponteiros relativos e categorias aparecem completos segundo o relatório. As coleções Swift estão vazias e as facetas de declarações/conformidades/vtables são relatadas completas para o escopo do leitor; formas genéricas/resilientes e outras famílias Swift são explicitamente não suportadas. Nenhuma dessas contagens estabelece ausência de código ou de contratos fora do escopo do leitor. Os motivos completos permanecem no recorte.

Q05 permanece parcial: há candidatos nos dois lados, mas ainda faltam uma cadeia demonstrada interface→motor e outra callback→consumidor, com proveniência dos endereços por imagem.
