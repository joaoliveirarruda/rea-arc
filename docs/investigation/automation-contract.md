# Contrato de automação declarado pelo Arc

## Fonte e limites

O `Contents/Info.plist` aponta `OSAScriptingDefinition` para `Arc.sdef`; o arquivo está em `Contents/Resources/Arc.sdef`. O XML tem 9.794 bytes, foi parseado com `xml.etree.ElementTree` e passou `xmllint --noout`.

SHA-256 do SDEF:

```text
d39068b2a33e7aaf1b4bdf7708eefc331524ba8c59e4b98c8feada0a82a00397
```

O dicionário declara uma suite, quatro classes, 24 propriedades, cinco coleções/elementos e dez comandos. Isso é um contrato declarado no pacote; nenhum Apple Event ou comando foi executado e o funcionamento em runtime não foi verificado.

## Classes e membros

| Classe SDEF | Classe Cocoa | Propriedades declaradas | Elementos e comandos associados |
| --- | --- | --- | --- |
| `application` | `NSApplication` | `name` (somente leitura), `frontmost` (somente leitura), `version` (somente leitura) | `window` (ordem frente-trás); `tab` com acesso declarado `w`, descrito como criação de Little Arc tabs; responde a `quit` via `handleQuitScriptCommand:` |
| `window` | `AppleScriptWindowWrapper` | `id` (r), `name` (r), `index` (r), `closeable` (r), `minimizable` (r), `minimized`, `resizable` (r), `visible`, `zoomable` (r), `zoomed`, `active tab` (r), `active space` (r), `incognito`, `mode` | `tab` (rw), `space`; responde a `close` via `close:` |
| `tab` | `AppleScriptTabWrapper` | `id` (r), `title` (r), `URL` (rw), `loading` (r), `location` | Responde a `select`, `go back`, `go forward`, `reload`, `stop`, `execute` e `close`, por métodos Cocoa mapeados no XML |
| `space` | `AppleScriptSpaceWrapper` | `id` (r), `title` (r) | `tab` (rw); responde a `focus` via `focus:` |

`r` e `rw` reproduzem o atributo `access` do SDEF. Quando o XML omite `access`, a tabela deixa o nome sem marca; não infiro o comportamento padrão. Os elementos e propriedades também mapeiam para chaves Cocoa, incluindo `appleScriptTabs`, `appleScriptWindows`, `appleScriptSpaces`, `activeTab`, `activeSpace`, `URL` e `javascript`; todos os mapeamentos e descrições estão registrados no JSON.

## Comandos

O SDEF define `make` (`NSCreateCommand`, parâmetro `new` e `with properties` opcional, resultado `specifier`), `count` (`NSCountCommand`, parâmetro `each` opcional, resultado `integer`), `close`, `select`, `go back`, `go forward`, `reload`, `stop`, `execute` e `focus`. `close`, `select`, navegação, `reload`, `stop` e `focus` recebem objeto direto do tipo `specifier`. `execute` recebe o objeto direto e parâmetro `javascript` de texto, com resultado de texto.

O dicionário associa `quit` à classe `application` via `responds-to`, mas não contém uma declaração local `<command name="quit">`. Isso descreve apenas a estrutura do arquivo; não conclui se a aplicação aceita a mensagem em runtime.

## Semente proposta para teste posterior

Este AppleScript foi montado a partir das declarações `make`, `tab`, URL gravável, `select`, propriedade `loading` e `close`. Ele cria uma guia de teste, navega para um `data:` autocontido sem dependência de rede, lê URL/título e fecha a mesma guia:

```applescript
tell application "Arc"
    set probeTab to make new tab at end of tabs
    set URL of probeTab to "data:text/html,%3Ctitle%3EREA-Automation-Probe%3C/title%3E%3Cp%3Eok%3C/p%3E"
    select probeTab
    repeat 40 times
        if loading of probeTab is false then exit repeat
        delay 0.25
    end repeat
    set observedURL to URL of probeTab
    set observedTitle to title of probeTab
    close probeTab
    return {observedURL, observedTitle}
end tell
```

Expectativas para uma execução futura: `make` retorna uma referência à guia recém-criada, o URL usa o esquema `data:`, o título após carregamento é `REA-Automation-Probe` e `close` aceita essa referência. O roteiro não foi executado; até essas verificações são hipóteses derivadas das declarações. O teste deve rodar somente em uma sessão Arc apropriada para automação, pois ele altera a interface criando, navegando e fechando uma guia.

## Evidência estruturada

O parse completo de suites, classes, propriedades, elementos, comandos, métodos/chaves Cocoa, atributos brutos, hash e roteiro está em [automation-contract.json](../../.local/evidence/inventory/automation-contract.json), SHA-256 `cb0d0c8dc5a9fbcfd79dd515d90b7ed84bfa4bcccc988aa99790f2aa3f86a35c`. Nenhum perfil, conta ou dado pessoal foi lido.
