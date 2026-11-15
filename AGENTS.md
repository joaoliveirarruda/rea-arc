# Investigação do Arc

## Retomada

Leia `docs/investigation/orchestration.md` e `docs/investigation/tasks.md` antes de distribuir trabalho. Consulte `.local/checkpoint.json` para estado operacional e `docs/investigation/findings.md` para conclusões. Verifique a identidade do artefato antes de reutilizar resultados de outra execução.

## Autoridade do REA

Use `/Users/juca/Projects/rea/AGENTS.md`, a skill `/Users/juca/.agents/skills/reverse-engineer-anything/SKILL.md` e suas referências aplicáveis como fontes do fluxo REA. Leia os contratos reais das ferramentas ao preparar chamadas. A aplicação desses documentos aqui orienta a investigação. Regras de contribuição ao REA passam a valer para alterações naquele repositório.

O orquestrador possui a sessão MCP ativa. Trabalhadores usam evidências exportadas ou processos CLI independentes com saídas próprias. Antes de trocar o alvo ou liberar uma sessão, exporte evidências necessárias. Preserve SHA-256, arquitetura, parâmetros, IDs, cobertura e limitações. Classifique conclusões como observadas, derivadas, inferidas ou desconhecidas.

## Execução

O usuário autorizou planejamento, delegação, execução do Arc e uso das contas Arc/Google quando necessário para esta investigação. A publicação externa e o envio de mensagens exigem autorização específica. Guarde capturas brutas e material de conta em `.local/`; os relatórios versionáveis devem conter a evidência técnica necessária e referências locais, preservando credenciais fora deles.

Distribua raciocínio complexo para `gpt-6.1-terra` com esforço `high`, análise intermediária para `gpt-6.1-terra` com esforço `medium` e inventário mecânico para `gpt-6-luna` com esforço `max`. Use `fork_turns="none"` e passe o contexto necessário. Registre indisponibilidade de um modelo antes de escolher substituição.

Cada tarefa deve informar dependências, arquivos de saída exclusivos, critérios de conclusão e estado. O orquestrador revisa evidência e atualiza os registros antes de encerrar a tarefa. Prossiga para a próxima pergunta útil enquanto houver trabalho autorizado. Falhas recuperáveis recebem diagnóstico e correção conforme o REA, sem repetir uma chamada inalterada que tenha falhado.

## Economia de uso

O usuário pediu atenção ao consumo de modelos fortes. Reutilize contexto e evidências, agrupe consultas na mesma sessão e delegue apenas quando houver ganho concreto. Reserve Terra High para decisões complexas, Terra Medium para análise intermediária e Luna Max para tarefas mecânicas. Acompanhe a janela compartilhada e preserve checkpoints antes de sua renovação.
