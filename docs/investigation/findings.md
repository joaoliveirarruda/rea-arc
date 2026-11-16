# Achados verificados

| ID | Conclusão | Tipo | Fonte | Cobertura e lacunas |
| --- | --- | --- | --- | --- |
| F001 | O bundle declara Arc 1.172.2, build 90755, identificador company.thebrowser.Browser | observada | REA inspect_plist ev_6973e351e9e96a245e57b3fa100a02a253ebe3825377c32a21bd4f2233215451 | Declaração Info.plist, sessão de recursos com identidade do executável nula |
| F002 | O bundle declara a classe principal Arc.BrowserApp e o dicionário de automação Arc.sdef | observada | Mesma evidência de F001 | Declarações estáticas, comportamento ainda por verificar |
| F003 | O REA ativo anuncia Ghidra disponível e nenhuma sessão nativa aberta no início | observada | binary_session consultado em 2026-11-15 | Disponibilidade anunciada, suporte ao Arc depende da abertura e análise |
| F004 | A fatia ARM64 possui UUID 10ED4B9B-6BFE-34D3-AD7F-CCA121AB7592, 86 dependências, 8.039 imports e 863 exports retornados | observada | REA inspect_macho ev_905a7257d7bedaa35fa74303666fb812c0c6a017f73b6c06daf38422f93611f5 | Dependências exaustivas segundo a ferramenta; imports e exports não exaustivos. Hash e extração em .local/artifacts/provenance.json |
| F005 | Metadados da fatia ARM64 associam seletores de navegação e automação a implementações; o desassemblado confirma chamadas estáticas delimitadas | observada | REA ev_59948e08664b7288cfb0326952fb45891a7e70220447a74bc05d8afceb1fb483 e [chamadas estáticas](bridge-call-sites.md) | Destinos indiretos, semântica e execução ainda desconhecidos |
| F006 | Uma construção de `ADKClient.discardWebContent` recebe identificador de frame e encaminha para uma operação sobre `WebContentFrame` | derivada de metadados e código estático | [Suspensão de abas](tab-suspension.md) | Identidade do requisito chamado e relação com o callback de finalização ainda não comprovadas; execução não observada |
| F007 | Um fluxo candidato de arquivamento converte limiares nomeados em segundos e compara a data injetada com `SidebarTab.timeLastActiveAt` | derivada de metadados e código estático | [Comandos e arquivamento](commands-and-archiving.md) | Faltam origem completa dos itens, assinante e efeito final; execução não observada |

Bundles e saídas brutas ficam em `.local/evidence/`. Relatórios por assunto devem manter as distinções entre declaração estática, derivação, inferência e observação de execução.
