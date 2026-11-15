# Q01/Q05 — amostra da thread principal

Uma captura existente executou `/usr/bin/sample 71319 1 1 -file .local/evidence/native-ui/arc-thread-sample.txt`, registrada no cenário REA `ev_e03b2b766a8a7cade596de0f4fbca3d54db8a7a94999a0736b09110032a26cf4` ([cenário](/Users/juca/Projects/rea-arc/.local/evidence/native-ui/arc-sampling-scenario.json), [amostra](/Users/juca/Projects/rea-arc/.local/evidence/native-ui/arc-thread-sample.txt)). SHA-256 do texto: `68efc0314241b5a4a75d1bda2cc4e32b8effb5b14f942d8022d2ea99dfb814c2`. O relatório identifica Arc 1.172.2 build 90755, ARM64 e thread principal `com.apple.main-thread`. A amostragem dura aproximadamente um segundo, não descreve toda a execução e não determina frequência global ou causalidade.

O [recorte estruturado](/Users/juca/Projects/rea-arc/.local/evidence/native-ui/event-loop-selected.json) contém somente as linhas da thread principal, duas linhas de `Binary Images`, frames Arc/ArcCore e normalizações calculadas. O [extrator](/Users/juca/Projects/rea-arc/.local/evidence/native-ui/extract-event-loop.py) preserva o cálculo. Nenhuma nova amostra foi coletada nesta rodada.

## Imagens e endereços

| Imagem | UUID no sample e na slice local (`dwarfdump --uuid`) | Base carregada | `__TEXT.vmaddr` preferido | Slide derivado |
| --- | --- | --- | --- | --- |
| Arc | `10ED4B9B-6BFE-34D3-AD7F-CCA121AB7592` | `0x1005dc000` | `0x100000000` | `0x5dc000` |
| ArcCore | `4C4C4428-5555-3144-A183-30B39DAD0833` | `0x123cdc000` | `0x0` | `0x123cdc000` |

As bases carregadas e UUIDs vêm de `Binary Images`; os endereços preferidos vêm dos load commands `__TEXT` das slices locais, inspecionados com `llvm-objdump --macho --private-headers`. Para cada imagem, endereço preferido = endereço amostrado − base carregada + `__TEXT.vmaddr`. UUIDs coincidentes sustentam a associação das slices aos registros da amostra; não são medição independente dos bytes que estavam na memória. As imagens têm slides distintos.

## Observação e comparação estática

No ramo comum da pilha da thread principal, o sample apresenta frames de Arc e ArcCore, depois `-[NSApplication run]`, novamente Arc, ArcCore e Arc, e depois `-[NSApplication(NSEventRouting) nextEventMatchingMask:untilDate:inMode:dequeue:]`, seguido de funções AppKit/CoreFoundation e espera no run loop. Isso observa uma pilha amostrada contendo essas imagens e frames; não estabelece que uma interação específica ou navegação tenha acontecido. `ChromeMain + deslocamento` é o rótulo textual do sample para vários frames ArcCore: deslocamentos grandes não identificam a função real.

Comparando frames normalizados do Arc com entradas do metadado Objective-C `ev_59948e08664b7288cfb0326952fb45891a7e70220447a74bc05d8afceb1fb483`:

| Frame observado | Endereço preferido derivado | Entradas de implementação distintas adjacentes |
| --- | --- | --- |
| `0x1006e3118` | `0x100107118` | `BrowserApp nextEventMatchingMask:untilDate:inMode:dequeue:` começa em `0x10010703c`; próxima entrada distinta `0x100107160`. |
| `0x1006e2e90` | `0x100106e90` | `BrowserApp sendAction:to:from:` começa em `0x100106c28`; próxima entrada distinta `0x10010703c`. |
| `0x1006e2fe8` | `0x100106fe8` | Mesmas entradas adjacentes de `0x100106c28` e `0x10010703c`. |

Essas são **coincidências em intervalos entre entradas de despacho**, não provas de que os frames pertencem aos métodos nomeados: a próxima entrada não define limite de função, pode haver funções intermediárias e um frame pode registrar endereço de retorno. O resultado de despacho também é parcial. A atribuição de método exige fronteiras e instruções adicionais; em particular, o nome `sendAction:to:from:` não demonstra qual ação foi enviada. Para ArcCore, o recorte conserva os endereços normalizados individualmente, sem atribuir nomes a partir dos grandes deslocamentos do sample.

A amostra apoia a presença simultânea de Arc, ArcCore e AppKit nessa pilha observada e deixa em aberto a rota de comando interface→motor e o callback→consumidor. Ela complementa, sem substituir, as investigações estáticas das imagens e dos delegates.
