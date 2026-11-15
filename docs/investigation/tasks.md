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
