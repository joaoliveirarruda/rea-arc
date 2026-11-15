# Perguntas e critérios de investigação do Arc

Atualizado em 15/11/2026. Objetivo: produzir primeiro um mapa técnico verificável do Arc instalado; uma eventual implementação deve partir desse mapa e declarar suas decisões próprias. Este arquivo é o roteiro e os critérios de cobertura, não um relatório de funcionalidades confirmadas.

## Identidade e fatos já observados

Inspeção local, somente leitura, sem execução do alvo, de `/Applications/Arc.app`:

| Referência local | Observação verificada | Limite |
| --- | --- | --- |
| Q-E01: `Contents/Info.plist`, leitura por `plistlib` | `CFBundleIdentifier=company.thebrowser.Browser`; versão `1.172.2`; build `90755`; executável `Arc`; `NSPrincipalClass=Arc.BrowserApp` | Não identifica por si só o hash do executável, arquitetura ou comportamento. A identidade criptográfica fica com o inventário. |
| Q-E02: filhos imediatos de `Contents/Frameworks` | Há `ArcCore.framework`, `FirebaseFirestoreInternal.framework`, `grpc.framework`, `grpcpp.framework`, `openssl_grpc.framework`, `absl.framework`, `Sparkle.framework`, `Sentry.framework`, `Lottie.framework` e `libswiftCompatibilitySpan.dylib` | Presença de bibliotecas não comprova chamadas, sincronização, telemetria ativa nem implementação de interface. |
| Q-E03: JSON analisado em `Contents/Resources/ARC_WebContentBehaviorConfigurationClient.bundle/Contents/Resources/web_content_behavior.json` | Objeto raiz contém `keepaliveAllowList`, `pipDisallowList`, `pipSiteConfigs`, `tidyTitles` e `tabSuspensionBlocklist` | Chaves são sementes de investigação; consumidores, aplicação de regras e precedência ainda desconhecidos. |

Essas referências são observações locais deste documento, não IDs de Evidence do REA. Associar resultados posteriores aos IDs reais retornados pelo REA; não inventar IDs. O inventário central deve conservar versão, caminhos, hashes e horário de aquisição. Se o app mudar, revalidar a identidade antes de combinar resultados.

## Ordem e registro

Executar P0 antes de aprofundar P1. P2 é opcional depois do mapa principal. As perguntas abaixo estão **propostas/em aberto** até uma resposta com evidência ser registrada no relatório da investigação.

Para cada resposta, guardar: ID da pergunta; estado (`respondida`, `parcial`, `não resolvida`); afirmação delimitada; tipo (`observação estática`, `observação runtime`, `derivação`, `hipótese`); fonte e localização/ID Evidence; escopo examinado; confiança; limitações; próxima pergunta concreta. Um símbolo ou string encontrado identifica uma pista; não confirma o caminho executado. Uma busca sem resultados só delimita o corpus e o padrão consultados.

Cada rodada examina uma pergunta, um produtor de estado e seus consumidores diretos. Começar por inventário e resumo; expandir para um fluxo representativo quando existir uma semente concreta. Como heurística ajustável, após duas rodadas sem nova evidência útil, reavaliar a hipótese, registrar a lacuna e decidir se uma nova fonte concreta justifica continuar ou se convém mudar de pergunta. Essa referência não é um teto de tentativas; ajustar a profundidade ao valor da pergunta, à cobertura disponível e ao custo observado. Trabalhos mais caros devem justificar qual lacuna específica resolverão.

## P0 — mapa técnico mínimo

### Q01. Quais componentes realmente compõem o navegador?

- **Pergunta delimitada:** quais executáveis, frameworks e recursos pertencem à interface, ao motor de navegação e aos processos auxiliares nesta build? Quais dependências são importadas e quais só estão empacotadas?
- **Evidência necessária:** identidade dos Mach-O; arquiteturas; dependências/carregamentos declarados; inventário de helpers/frameworks; plists de bundles. Para atribuir responsabilidade, ao menos um import, chamada, tipo ou recurso consumido pelo componente correspondente.
- **Critério:** diagrama com componentes, processos conhecidos e arestas tipadas como importação, chamada ou hipótese. Localizar explicitamente o motor e o dono da janela; nomes de arquivos sozinhos deixam responsabilidade em aberto.

### Q02. Qual é o modelo persistente da sidebar?

- **Pergunta delimitada:** quais entidades representam item, pasta, favorito, aba fixada, aba temporária e seleção? Ordem e hierarquia pertencem ao Space, à janela ou a uma coleção global?
- **Evidência necessária:** tipos, campos, enums e serializadores; uma sequência produtor → mutação → leitor/renderização. Usar apenas estrutura do armazenamento ou dados sintéticos para validar formato.
- **Critério:** tabela de entidades com identidade, proprietário, relações, ordem e campos persistentes/transitórios; um fluxo completo de criação ou movimentação. Deixar diferenças entre rótulos visuais e tipos internos explícitas.

### Q03. Como Spaces, Profiles e janelas se relacionam?

- **Pergunta delimitada:** qual é a cardinalidade Space→Profile; o que a troca de Space muda; onde residem seleção ativa, favoritos e isolamento de sessão; como a janela referencia esses objetos?
- **Evidência necessária:** identificadores e referências nas estruturas; handlers de troca de Space/Profile; parâmetros de criação/seleção de contexto de navegação. Isolamento de cookies precisa de implementação ou teste sintético, não de nome de pasta.
- **Critério:** modelo relacional e fluxo de troca cobrindo destino, seleção restaurada e contexto de navegação. Distinguir Profile como entidade do Arc de perfil/contexto interno do motor, se ambos existirem.

### Q04. Qual é a máquina de estados de uma aba?

- **Pergunta delimitada:** quais transições distinguem abrir, selecionar, fixar, desafixar, arquivar, fechar, restaurar, suspender e descartar conteúdo? Que estados preservam identidade e histórico?
- **Evidência necessária:** modelo de tab, comandos/handlers e métodos de ciclo de vida do conteúdo; regras de arquivamento; leitores de `keepaliveAllowList` e `tabSuspensionBlocklist` (Q-E03). Um exemplo representativo de transição com precondição e efeito.
- **Critério:** tabela estado/evento/precondição/mutação/efeito no motor; separar estado lógico persistido de processo/renderizador carregado. Suspensão, descarte e arquivamento só são equivalentes se a evidência estabelecer isso.

### Q05. Onde fica a fronteira entre interface nativa e motor de navegação?

- **Pergunta delimitada:** qual adaptador recebe um comando nativo de navegar/selecionar/fechar e qual callback devolve URL, título, estado de carregamento ou foco?
- **Evidência necessária:** tipos importados/exportados, interfaces de bridge, protocolos/delegates/callbacks e chamadas nos dois sentidos. Examinar um comando de navegação e um evento de retorno; incluir thread/fila apenas quando demonstrável.
- **Critério:** dois caminhos com origem, adaptador, API do motor e consumidor de retorno, citando funções ou endereços. Se apenas uma direção estiver demonstrada, marcar a outra como lacuna.

### Q06. Quem grava o estado e quem o restaura?

- **Pergunta delimitada:** quais stores guardam sidebar, Spaces, Profiles e sessão de abas; qual operação grava; como o startup escolhe, migra e restaura esses dados?
- **Evidência necessária:** caminhos construídos pelo código; tipos de serialização; schema/versionamento; chamadores de leitura/gravação e ponto de restauração. Inventário de nomes/formatos em dados do usuário só se necessário; conteúdo pessoal não é necessário para o mapa.
- **Critério:** mapa entidade→store→dono→gatilho de gravação→leitor no startup; distinguir preferências, sessão, cache e histórico. Atomicidade, recuperação e frequência ficam desconhecidas sem evidência específica.

## P1 — comportamento e integração

### Q07. Como comandos, atalhos e Command Bar convergem?

- **Pergunta delimitada:** quais registros/identificadores ligam menu, teclado e Command Bar aos mesmos handlers para abrir aba, trocar Space, fechar e restaurar?
- **Evidência necessária:** registros de comando, selectors/handlers, atalhos e validação por contexto. Recursos localizados podem fornecer rótulos; código ou observação precisa demonstrar despacho.
- **Critério:** quatro comandos representativos com ID, entradas, disponibilidade e destino. Explicar comandos dependentes de foco/contexto; evitar inventariar todos os textos antes de resolver o dispatcher.

### Q08. Qual é o mecanismo de arquivamento automático?

- **Pergunta delimitada:** como o limite temporal é configurado, qual relógio/atividade conta e como favoritos/fixadas/ativas interagem com elegibilidade?
- **Evidência necessária:** configuração e defaults, consumidor, cálculo de elegibilidade e transição de arquivamento. Recursos e texto de preferências são evidência de interface, não do algoritmo.
- **Critério:** regra em pseudocódigo com unidades, exclusões e momento de aplicação, ou indicação explícita das partes não encontradas. Não alterar o relógio do sistema nem forçar arquivamento de abas pessoais.

### Q09. O que é sincronizado e como a identidade atravessa dispositivos?

- **Pergunta delimitada:** quais entidades do Arc entram no sync; qual camada produz operações; onde estão IDs, versões, tombstones e tratamento de conflito; quais partes são apenas locais?
- **Evidência necessária:** modelos/schemas e adaptador ativo, chamadores de leitura/gravação remota, lógica de merge e exclusão. Q-E02 apenas sugere uma biblioteca candidata; comprovar uso no fluxo antes de atribuir backend.
- **Critério:** tabela por entidade com direção, identidade, exclusão, conflito e dependência de login; citar fluxo completo ao menos para uma entidade. Endpoints embarcados não demonstram serviço ativo. Sem teste entre dispositivos, convergência real permanece não verificada.

### Q10. Como múltiplas janelas compartilham ou isolam estado?

- **Pergunta delimitada:** seleção, conteúdo web, sidebar e Space ativo são por janela ou globais? Como uma alteração em uma janela chega às demais?
- **Evidência necessária:** proprietários de modelos/controllers, observadores e construção de janelas; caminho de notificação de uma mudança. Confirmar apenas com janela sintética se necessário.
- **Critério:** matriz entidade×escopo e um fluxo de propagação. Diferenciar mesmo identificador lógico de mesma instância de conteúdo.

### Q11. Como restauração, falhas e migração afetam o modelo?

- **Pergunta delimitada:** quais versões de schema existem e o que acontece quando uma sessão anterior está incompleta ou um item referenciado foi removido?
- **Evidência necessária:** migrações e leitores, fallback e normalização de referências. Inspecionar tratamento implementado; qualquer ensaio de corrupção usa cópia isolada e sintética.
- **Critério:** tabela das migrações localizadas e um caminho de fallback demonstrado; não extrapolar tolerância a toda corrupção ou durabilidade contra crash.

## P2 — expansão opcional

### Q12. Split View, Little Arc, Peek e PiP reutilizam qual modelo?

- **Pergunta delimitada:** para cada recurso encontrado, ele cria aba, view auxiliar, janela ou contexto temporário? Como retorna ao fluxo principal e qual estado persiste?
- **Evidência necessária:** entry point, construção e teardown, relações com a aba principal e serialização. Q-E03 oferece sementes específicas de PiP; recursos de onboarding não comprovam semântica interna.
- **Critério:** um ciclo criar→interagir→fechar/restaurar por recurso que tiver evidência; recurso sem cadeia fica como pista.

### Q13. O que seria necessário reproduzir primeiro?

- **Pergunta delimitada:** qual subconjunto demonstrável de sidebar, Spaces e ciclo de abas permite um protótipo coerente; quais dependências podem ser substituídas por escolhas de implementação?
- **Evidência necessária:** respostas anteriores com proveniência e lacunas, contratos de entrada/saída e casos representativos. Esta etapa é design proposto, não engenharia reversa confirmada.
- **Critério:** escopo mínimo com critérios observáveis, decisões próprias separadas dos fatos do Arc e teste com dados sintéticos. Implementação permanece opcional; não adiar o mapa técnico para construí-la.

## Quando usar runtime

Usar primeiro evidência estática para estrutura, identidade, imports, schemas e pontos de entrada. Reservar runtime para perguntas de ordem temporal, efeito visível, contexto/foco e isolamento que permaneçam ambíguas. Cada ensaio declara alvo, ação, estado inicial sintético, observável esperado e limpeza. Ações locais reversíveis estão dentro do escopo autorizado; publicação ou envio a terceiros não fazem parte desta investigação.

O roteiro não exige abrir conteúdo pessoal do Arc ou do Google, ler cookies/tokens, ativar sync, trocar conta, reiniciar o navegador pessoal ou mudar preferências globais. Quando a observação exigir estado isolado não disponível, registrar a lacuna e continuar perguntas independentes. Capturas devem incluir só a interface e objetos sintéticos necessários à afirmação.

## Entrega e encerramento

O mapa mínimo é utilizável quando Q01–Q06 têm respostas com fontes e limites, mesmo que parte delas seja parcial. Declarar a cobertura real de cada pergunta; não chamar o comportamento completo do Arc de reconstruído. Entregar diagrama de componentes, modelo de entidades, ciclo de abas, caminhos de comando/bridge, mapa de persistência e lacunas priorizadas. O coordenador preserva o bundle de Evidence antes de encerrar a investigação e registra sessões/processos ainda sob sua responsabilidade.
