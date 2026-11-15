# Inventário estático do Arc para macOS

## Escopo e identidade observada

Alvo lido: `/Applications/Arc.app`, somente o bundle instalado. A coleta não iniciou o Arc e não acessou perfil, conta, preferências ou banco de dados do usuário.

O `Contents/Info.plist` declara bundle ID `company.thebrowser.Browser`, versão `1.172.2`, build `90755`, executável `Arc`, plataforma `MacOSX` e mínimo macOS `13.0.0`. Também declara release `Release`, data de build `10 Nov 2026, 14:37` e commit `a09447ad10300945a3c896dd8d8ef90ac03023b7`. A declaração de versão e build foi confirmada com `/usr/libexec/PlistBuddy`; `plutil -lint` retorna `OK`.

O executável principal em `Contents/MacOS/Arc` é Mach-O universal `x86_64 arm64`, com 167.579.440 bytes e SHA-256:

```text
c2906e92838b7b07451a54a0e3b151a59492347ebd0050b181524e283cbaf7a2
```

## Frameworks

Os nove frameworks possuem binários universais `x86_64 arm64`. Versões e hashes abaixo vêm dos respectivos `Info.plist` e executáveis dentro do bundle.

| Framework | `CFBundleShortVersionString` | `CFBundleVersion` | SHA-256 do binário |
| --- | ---: | ---: | --- |
| ArcCore | 157.0.8103.51 | 8103.51 | `de2626e5739686cd59df1d205e161d9b5020fa0038af209f8a4d9a5f346d3220` |
| FirebaseFirestoreInternal | 10.27.0 | 1 | `532d1cc3930454afb670ff708fa5ff8a420dbe02eb4666847c6fb0d7e496c471` |
| Lottie | 1.0 | 1 | `1c218cd9ca0d8cde0cf3ca041c8086c671e72b9bd27ab9993180a901418c3bac` |
| Sentry | 8.53.1 | 8.53.1 | `e5bc48697f55df6930cc2c242f078116d7d8942c7236bc5e69d215179581043f` |
| Sparkle | 2.9.4 | 2059 | `9bbf46708d85f1e3b68f1d1e31a7c484a2789b87971ca07f382a1755a11ab523` |
| absl | 1.20240116.2 | 1 | `5a02762b0708d0f390278f1b69cacf39ee2c2ad22adae724dd2698ca102ddf1a` |
| grpc | 1.62.1 | 1 | `5fb42b43abb77d0df125921032aadb4bf64f6a160777d55118f39f6affa9586d` |
| grpcpp | 1.62.1 | 1 | `a75800cf5ec3c43027ce9a7f3d8af917fa4dd1885899ba84d7f0199f90c0b482` |
| openssl_grpc | 0.0.32 | 1 | `95237f807722ab95f14301c2d771d065682aa0bafc80cebbba00f02dbbfd80de` |

A evidência JSON contém os hashes dos 22 executáveis declarados nos bundles — Arc, binários dos frameworks, Browser Helpers, componentes Sparkle e DockTile plug-in — além de tamanho, caminho, tipo Mach-O e arquiteturas quando reconhecidos. Ela também contém o SHA-256 de cada `Info.plist` encontrado.

## Recursos e declarações de plist

Em `Contents/Resources`, a varredura contou 723 arquivos regulares, 83 diretórios `.bundle` e 55 diretórios `.lproj`. Os sufixos mais frequentes são arquivos sem extensão (332), `.strings` (94), `.plist` (84), fontes `.ttf`/`.otf` (85 no total), `.car` (45), `.xcprivacy` (24) e `.json` (21). O inventário completo por sufixo e tamanho está na evidência JSON. Foram lidos 111 `Info.plist` sob `Contents`, incluindo bundles, plug-in e helpers.

O plist principal declara:

- Tipos de documento para PDF, HTML e XHTML com papel `Viewer`.
- Esquemas URL `mailto`, `https`, `http` e `arcBrowser`.
- Descrições de uso para Bluetooth, câmera, Desktop, Downloads, localização e microfone.
- `NSAppTransportSecurity.NSAllowsArbitraryLoads = true`.
- Atualização Sparkle automática e verificações automáticas habilitadas; feed `https://releases.arc.net/updates.xml` e intervalo `28800` segundos.
- `NSPrincipalClass = Arc.BrowserApp`, `NSAppleScriptEnabled = true` e categoria `public.app-category.productivity`.

Esses pontos descrevem valores declarados nos plists. Não demonstram que uma permissão foi concedida, que a configuração de rede foi usada, nem que atualização ou qualquer outra funcionalidade ocorreu em runtime.

## Evidência e limites

Dados estruturados e hashes: [arc-inventory.json](../../.local/evidence/inventory/arc-inventory.json), SHA-256 `6d614c2e6c6cccd49b140eaae8ce4a7b30a992f82b062885a3df9d579b97d190`. A evidência registra caminhos, valores extraídos, hashes de plists e binários, inventário por sufixo, comandos auxiliares e limites de escopo. A coleta usou `plistlib`, `find`/`os.walk`, `file`, `lipo`, `PlistBuddy` e `plutil`; não usou REA, Ghidra ou sessão MCP.

Desconhecidos neste inventário: comportamento em execução, acesso de rede, decisões de feature flags, concessões de permissões e uso real de recursos. O relatório é uma leitura de metadados do bundle, não uma observação de runtime.
