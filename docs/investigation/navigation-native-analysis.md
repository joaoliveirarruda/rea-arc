# Q05 — análise nativa parcial de navegação

O [resultado de `batch_decompile`](/Users/juca/Projects/rea-arc/.local/evidence/native-queries-round01/batch_decompile.json), Evidence `ev_bc8a451544fc46ea632b91d9042d1b06d2d7f68a6add7f64b5ea5a73772981a1`, contém três itens `ok` no executável Arc arm64 de SHA-256 `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. O Ghidra 12.1.4 restaurou um Program previamente salvo, sem repetir autoanálise, conforme as limitações do registro. Os trechos são **pseudocódigo derivado**, não fonte original nem observação de execução. As variáveis `unaff_x20`, `extraout_x8` e tipos/assinaturas inferidos mostram recuperação incompleta da ABI Swift; argumentos, retornos e algumas associações precisam ser confirmados em assembly e dados de tipo. Nenhuma consulta ao motor foi executada nesta rodada.

## Setter AppleScript: entrada até payload

O [desassemblado anterior](/Users/juca/Projects/rea-arc/docs/investigation/bridge-call-sites.md) estabelece que a implementação Objective-C `AppleScriptTabWrapper setURL:` em `0x100458c9c` chama diretamente `FUN_100458cf0`. O pseudocódigo deste último mostra operações rotuladas `Foundation::URL::_init`, um teste sobre o resultado representado como opcional, acesso ao campo `AppleScriptTabWrapper::window` e verificações associadas a `AppleScriptWindowWrapper::browserWindow` e `viewStore`. No caminho exibido em que as verificações permitem prosseguir, ele monta uma estrutura com dados associados a `AppleScriptTabWrapper::tab`, valores temporários e um valor do tipo `URL`; depois chama `_swift_storeEnumTagMultiPayload` e `FUN_10084bd90`.

Essa sequência é evidência estática de um candidato de encaminhamento a partir do setter. O campo `viewStore` é lido no pseudocódigo, porém `unaff_x20` e valores de registradores implícitos limitam a interpretação do receptor e dos argumentos reais. Não se conhece ainda o destino funcional do payload, nem se essa chamada chega ao controlador ArcCore. A hipótese de despacho de uma ação de navegação requer inspecionar `FUN_10084bd90` e os dados que ela consome.

## Callback de navegação: delegate e helper

Para `_TtC3ADK18WebContentChromium::webContents:didCommitNavigation:` em `0x100540f98`, o pseudocódigo acessa `navigationDelegate`, chama `_swift_unknownObjectWeakLoadStrong` e, quando o valor representado é não nulo, chama indiretamente o ponteiro em deslocamento `+0x18` de uma estrutura associada ao delegate. Essa indireção é compatível com despacho por tabela de witness/protocolo Swift, mas o destino concreto e o protocolo ainda não estão identificados. Depois, o pseudocódigo prepara valores locais e chama `FUN_100736cfc`, passando `FUN_10054208c` e uma área local segundo a representação do decompilador. O [assembly anterior](/Users/juca/Projects/rea-arc/docs/investigation/bridge-call-sites.md) confirma a chamada indireta e a referência de endereço, não a assinatura Swift completa.

O terceiro item, `FUN_100736cfc`, apresenta uma chamada a `FUN_100836d68`, `_swift_getWitnessTable` e `Sequence::_forEach`. O próprio decompilador usa `unaff_x20` e lhe atribui assinatura `void`, apesar dos valores preparados no chamador; portanto não se deve tratar essa assinatura ou a coleção percorrida como resolvidas. A associação a uma iteração é uma leitura do pseudocódigo, não prova de qual lista de observadores foi percorrida nem de que o callback executou em runtime.

## Próximos alvos

1. Decompilar `0x10084bd90`, identificar o objeto receptor, a estrutura/tag e seu consumidor; corroborar parâmetros e registradores em assembly.
2. Inspecionar `0x10054208c` e `0x100836d68`, além dos dados de witness usados no callback, para determinar o que o `forEach` aplica e a quem.
3. Localizar atribuições a `navigationDelegate` e resolver o alvo concreto em `+0x18`; só então relacionar esse callback aos seletores do ArcCore. A cadeia interface→motor e a entrega callback→consumidor permanecem parciais.
