# Observações de execução

## Consulta AppleScript de leitura

Em 15/11/2026, o REA executou `/usr/bin/osascript` para consultar versão, quantidade de janelas e quantidade de Spaces e abas da primeira janela do Arc. O processo Arc já estava em execução. A consulta tinha prazo AppleEvent de 15 segundos e prazo do cenário de 20 segundos.

Evidence `ev_1c8bedbb5539c383f0585f38d3c8037ae2c9254d8ffe865fdf1f4262d05842b8`, operação `capture_process_scenario`, registra saída 1 e erro AppleEvent `-1712`. Nenhuma das contagens solicitadas foi retornada. O bundle completo está em `.local/evidence/runtime-automation-probe.json`.

A evidência demonstra a falha desta consulta composta. A causa permanece desconhecida. Não estabelece indisponibilidade geral de automação, ausência de janelas ou qualquer contagem. A consulta não incluiu comandos de criação, fechamento ou navegação de abas. Efeitos de filesystem não foram observados pelo cenário.

O próximo teste de automação deve responder a uma lacuna específica do mapa e usar uma entrada diferente justificada por diagnóstico. O trabalho estático continua independente desta consulta.

## Isolamento da consulta de janelas

A consulta composta foi dividida para delimitar a falha. O cenário que pediu somente `version`, com prazo AppleEvent de cinco segundos, retornou `1.172.2` e saída 0. Evidence `ev_152a5b1fdd85701e3082fdb858a012e1cd266e7e3771ecde4a00cdcccf146d3c`, em `.local/evidence/runtime-version-probe.json`, registra o texto após aproximadamente 120 ms. Esse resultado estabelece o sucesso dessa consulta, preservando em aberto qual caminho interno a respondeu.

O cenário que pediu somente `count windows`, também com prazo AppleEvent de cinco segundos, terminou com saída 1 e erro `-1712` após aproximadamente 5,09 segundos. Evidence `ev_f7e83dc27d50a19ace76208ac4d33384763d2ae47737c503ea5fd669e4015d34`, em `.local/evidence/runtime-window-count-probe.json`, preserva os frames e o comando exato. A consulta de contagem é um caso isolado que reproduz o timeout; nenhuma contagem de janela foi obtida. Seleção de Space, navegação e fechamento de abas continuam sem observação nesta rodada.

## WindowServer e disponibilidade de observação

Uma consulta CoreGraphics por PID enumerou oito entradas associadas ao Arc PID 71319, preservando somente IDs, dimensões, camada e visibilidade. Evidence `ev_58ec571bf5408444295e3544e3c70a8bf2c0b1a86499dc8f643d03501cfb6d8d`, em `.local/evidence/native-ui/window-enumeration.json`, identifica o script e a captura. A entrada 30616 estava marcada como visível, camada 0, com 140 × 189 pontos. As demais entradas incluíam janelas pequenas e ocultas. O inventário não atribui a essas entradas o papel de janela principal, aba ou Space.

A observação REA dessa entrada foi feita pela CLI nativa com o bundle original e sem abrir outro motor Ghidra. A tentativa de acessibilidade com prazo escolhido de 30 segundos encerrou com `provider_timeout`. Após encerramento do processo e verificação dos helpers, uma nova tentativa usou 180 segundos, o padrão documentado, e retornou `capability_unavailable`, orientando conceder Acessibilidade ao host do REA. Os diagnósticos permanecem em `.local/evidence/native-ui/selected-window.json` e `selected-window-attempt02.json`.

O preflight separado `AXIsProcessTrusted()` retornou falso e `CGPreflightScreenCaptureAccess()` retornou verdadeiro no contexto do processo Swift observado. Evidence `ev_6cf056ee90ba77449e2a1b4f1a67adec0b6a9de66d2722b447db7bea9d4d1951`, em `.local/evidence/native-ui/permissions-preflight.json`, preserva a consulta, que não solicitou alteração das permissões. Esses resultados descrevem o contexto consultado. A capacidade de captura pelo helper específico do REA é verificada por uma tentativa visual independente, com acessibilidade desabilitada.

A tentativa somente visual terminou com `capability_unavailable` e motivo `capture-failed`, informando falha ao iniciar transmissão de áudio/vídeo. O diagnóstico está em `.local/evidence/native-ui/selected-window-screen.json`. Uma ativação via `/usr/bin/open -a /Applications/Arc.app` retornou saída 0, mas a enumeração seguinte manteve as mesmas oito entradas e dimensões. Evidences `ev_9cb6721a233a0b1cd58a723f35fdc230f74a10076ed046f51d9483d41c772eec` e `ev_f3dd4eb2c84a98ca6fc970ff1e2000480ad8f38ab53feb8500889e2364b0f7f3`, em `.local/evidence/native-ui/activation-and-enumeration.json`, registram essas duas operações. Não se deduz do retorno de `open` a criação de uma janela.

Uma consulta posterior `CGSessionCopyCurrentDictionary` registrou `screen_locked: true`, `login_done: true` e `on_console: null`. Evidence `ev_956640bbc2816b90e3525d2c7da2be7f206499f0470a950d0f4b93c460a2f03f`, em `.local/evidence/native-ui/session-state.json`, preserva o script e a saída; null mantém a informação ausente como desconhecida. A sessão bloqueada é uma condição ambiental observada, compatível com a limitação visual, sem estabelecer a causa exata da falha de transmissão. A interação por acessibilidade requer a permissão indicada pelo REA. Essas restrições ficam delimitadas às sondas de interface.

## Pilha do processo em execução

O [relatório de loop de eventos](runtime-event-loop.md) analisa uma amostra de um segundo obtida por `/usr/bin/sample` do PID 71319, vinculada à captura REA `ev_e03b2b766a8a7cade596de0f4fbca3d54db8a7a94999a0736b09110032a26cf4`. A pilha contém frames de Arc, ArcCore e AppKit. Os UUIDs relatados foram comparados com as imagens locais e os endereços foram normalizados separadamente. A amostra acrescenta evidência runtime de integração, preservando em aberto chamadas específicas de navegação, transições de aba e as causas dos timeouts de automação.
