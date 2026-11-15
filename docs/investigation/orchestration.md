# Execução da investigação

## Objetivo e entrega

Priorizar o mapa técnico verificável do Arc instalado em `/Applications/Arc.app`. A primeira identidade declarada é 1.172.2, build 90755. A implementação poderá avançar após existirem contratos de comportamento e uma fronteira pequena verificável. O mapa acompanha o estado real de cobertura, inclusive perguntas parciais e desconhecidas.

O trabalho ocorre em `/Users/juca/Projects/rea-arc`. O checkout `/Users/juca/Projects/rea` fornece o REA 6.9.0 e suas instruções. Dados brutos, bundles e checkpoints ficam em `.local/`, ignorado pelo Git.

## Responsabilidades

| Responsabilidade | Modelo e esforço | Entrega |
| --- | --- | --- |
| Definir hipóteses, resolver conflitos e decisões arquiteturais | GPT 6.1 Terra high | Perguntas priorizadas e decisões justificadas |
| Investigar subsistemas, revisar evidências e implementar unidades delimitadas | GPT 6.1 Terra medium | Relatórios e código verificado |
| Inventariar, coletar fatos e organizar dados | GPT 6 Luna max | Inventários com fontes e cobertura |
| Integrar resultados e possuir a sessão REA | Orquestrador deste chat | Fila, mapa técnico, checkpoints e revisão |

Há quatro slots simultâneos, incluindo o orquestrador. As tarefas paralelas recebem perguntas e arquivos exclusivos. Uma única análise nativa pesada é o ponto de partida, com ajuste somente após observação de recursos e necessidade. Os workers recebem os caminhos das evidências existentes para evitar repetir coleta.

## Ciclo de trabalho

1. Ler fila e checkpoint, verificar alvo e consultar a sessão REA quando houver mudança de ciclo de vida ou necessidade de recuperar estado.
2. Escolher a pergunta com maior impacto no mapa e a menor coleta que possa respondê-la.
3. Registrar tarefa, dependências, executor, arquivo de saída e critério de aceitação. Delegar perguntas independentes.
4. Coletar evidências com REA, resumir a resposta e paginar o material retido quando necessário. Ferramentas locais complementam a proveniência.
5. Validar conclusões contra fontes, cobertura e limitações. Divergências voltam como perguntas específicas.
6. Exportar evidências antes de trocar sessão ou depender de estado em memória. Atualizar achados, fila e checkpoint.
7. Continuar com a próxima pergunta. O início de implementação depende de um comportamento definido e de um teste significativo possível.

## Sequência inicial

Inventário e identidade → recursos e interfaces declaradas → vínculo nativo/Chromium → modelos de abas e Spaces → persistência → comandos e navegação → observação controlada das lacunas → especificações de reconstrução → primeira unidade implementável.

A ordem pode mudar conforme evidências e custo. Presença de frameworks, símbolos e configurações gera pistas estáticas. Fluxos de execução precisam da evidência que os estabeleça. Uma busca negativa registra alvo, filtros e páginas examinadas.

## Recuperação e continuidade

Cada checkpoint registra a tarefa atual, saídas concluídas, bundles, sessão possuída e próxima ação concreta. Antes de um novo processo, importar o bundle exportado quando forem necessários IDs retidos. Evidência exportada e checkpoint editável do Ghidra têm finalidades distintas, conforme o guia nativo do REA.

Em timeout, distinguir prazo do cliente, prazo da operação e limpeza pendente. Consultar o guia de recuperação e o estado relevante antes de repetir importação. Alterar somente o controle indicado por erro ou documentação. Uma tarefa bloqueada permite avançar perguntas independentes.

A meta contínua está ativa neste chat. Limites de uso são compartilhados pela conta. O orquestrador pode consultar consumo e registrar horário de liberação, mas uma suspensão da plataforma pode impedir chamadas, espera ativa e retomada. A execução após suspensão depende do agendador de metas e do ciclo de vida do Codex. Os arquivos permitem retomada sem repetir trabalho mesmo quando esse mecanismo não reativa o chat. Não há compra automática de créditos nem consumo automático de benefícios de reset.

## Revisão de entrega

Uma pergunta fica respondida quando a conclusão cita fonte identificável, informa o tipo de evidência, cobre o comportamento perguntado e declara limitações restantes. Perguntas parciais permanecem na fila com a próxima investigação. Antes de implementar, registrar o contrato observável e a escolha de projeto. Antes de declarar a unidade concluída, verificar sua entrada normal e os casos relevantes.
