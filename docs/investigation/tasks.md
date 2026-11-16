# Fila da investigação

| ID | Pergunta ou entrega | Executor | Estado | Dependência | Critério |
| --- | --- | --- | --- | --- | --- |
| O01 | Estruturar execução e retomada | Orquestrador | concluída | — | Instruções, fila e checkpoint utilizáveis |
| M01 | Qual é a composição e identidade do pacote? | Luna max / inventory | concluída | — | Versões, hashes, componentes, fontes e limites em inventory.md |
| M02 | Como executar investigação nativa e recuperar falhas pelo REA? | Terra high / rea_workflow | concluída | — | Procedimento compatível com contratos e referências em rea-workflow.md |
| M03 | Quais perguntas definem o mapa de subsistemas? | Sol medium / investigation_design | concluída | — | Perguntas priorizadas com evidência e aceitação em questions.md |
| M04 | Quais declarações de recursos e automação o Arc fornece? | Orquestrador + Luna max | concluída | — | Plist e recursos examinados com evidências exportadas |
| M05 | Como o código nativo se conecta ao motor de navegação? | Terra high | parcial | M01, M02 | Componentes e fronteiras com fontes, chamadas verificadas e lacunas |
| M06 | Quais entidades e transições controlam abas e Spaces? | Terra high | parcial | M03, M05 | Modelo com referências a código/dados e observação quando necessária |
| M07 | Como o estado é persistido e sincronizado? | Terra high | parcial | M06 | Formatos, responsáveis e limites demonstrados |
| M08 | Como comandos e atalhos acionam navegação e interface? | Terra high | parcial | M04, M06 | Entradas e efeitos com cobertura declarada |
| M09 | Consolidar mapa e contratos para reconstrução | Orquestrador + Terra high | em andamento | M05–M08 | Questões classificadas e decisões rastreáveis |
| I01 | Implementar primeira unidade sustentada pelo mapa | a atribuir | pendente | Contrato selecionado de M09 | Código utilizável e verificação significativa |

Os trabalhos iniciais de M05, M06 e M09 incluíram contribuições de Sol Medium, antes da alteração de modelos solicitada pelo usuário.

M08/Q08 avança com a seleção de limiares, dependência de data, filtro de `SidebarTab` e um ramo que exclui itens não lidos documentados em [comandos e arquivamento](commands-and-archiving.md). O discriminador 1 foi associado estaticamente a `littleBrowser`; permanecem abertos os demais ramos e o destino que aplica o arquivamento. M06/Q04 mantém parcial o vínculo entre `discardWebContent`, o callback de finalização e o motor, detalhado em [suspensão de abas](tab-suspension.md).

Revisão independente local com Luna max confirmou a associação do byte `1` a `littleBrowser` após análise do retorno do decodificador. A ordem do FieldRecord, isoladamente, não seria prova suficiente. A revisão não alterou arquivos nem abriu outra sessão Ghidra.

Revisão local com Luna max identificou que o getter de características ainda precisava de prova por key path; o orquestrador seguiu o key path até `_tabTraits` e registrou as novas evidências. A revisão da ação de arquivamento sustentou a correspondência por ordem dos metadados, mas a análise mantém `autoArchiveItems` como candidato até confirmar o discriminador por um consumidor ou construtor explícito.
