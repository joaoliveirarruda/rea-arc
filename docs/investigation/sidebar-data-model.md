# Modelo interno da sidebar e dos Spaces

Observações estáticas da slice arm64 do Arc 1.172.2, SHA-256 `95e6895ed8773f2bae861ad8230c90d4679ebe7001eac20f22d232e17c8c0d1a`. Os nomes abaixo vêm de descritores e registros de reflexão Swift. Eles não são o código-fonte original e não estabelecem sozinhos comportamento, persistência ou invariantes. Os registros REA completos estão nos bundles `.local/evidence/native-queries-round118/` a `round122/`.

| Tipo | Campos ou casos declarados |
| --- | --- |
| `SidebarContainer` | `items`, `spaces`, `topAppsContainerIDs` |
| `SidebarItem` | `id`, `title`, `childrenIds`, `parentID`, `data`, `createdAt`, `isUnread`, `originatingDevice` |
| `SidebarItemData` | `tab`, `list`, `arcDocument`, `easel`, `itemContainer`, `welcomeToArc`, `splitView`, `sharedObject`, `instantLinks`, `tabGroup` |
| `SidebarTab` | `savedURL`, `savedFavicon`, `savedTitle`, `savedMuteStatus`, `customInfo`, `referrerID`, `activeTabBeforeCreationID`, `timeLastActiveAt`, `liveItemID` |
| `SpaceModel` | `id`, `title`, `containerIDs`, `customInfo`, `profile` |
| `ProfileID` | `directoryBasename`, `machineID` |

Os registros de campos estão nas Evidences `ev_f89cf1ec499e056d7820d110652aa486284d1eb9d278d52e6f78cfe84d4796b3`, `ev_46dad1ca40f5106691f7698ee54e0ccfbe3abf616f15da2e5a15bd599930020b`, `ev_eb83a9db810707b3167e53ee1ba5de0005067c690f22e742ab5901b959778e61`, `ev_c056fd47dcc850602edd95b00e943437786ac565aefc7d2095aa5c32e5575d18`, `ev_fff905ca8b36be51f3a1f5e19de647f741ab674aec59956cbac34fe536d06e21` e `ev_ed737bf082d867c840e4ead31549efdd24087874010ee8c010b2d1cd69543e43`. Os nomes recuperados dos deslocamentos relativos estão nos registros `*-names` das rodadas indicadas.

## Relações de tipo

O tipo simbólico de `SidebarContainer.items` contém referências a `Identifier` e `SidebarItem` em uma codificação compatível com dicionário. O tipo de `spaces` contém referências a `OrderedDictionary`, `Identifier` e `SpaceModel`; a gramática é compatível com um dicionário ordenado de identificadores para Spaces. Essas leituras dos tipos são inferências de alta confiança a partir dos metadados, não observação da ordem efetivamente apresentada na interface. Evidences `ev_33b7dfcaa33fdc846d1c8201efaa401911476250fd8b7181333772839fa6225a`, `ev_b8e58cea11477e378171c6e1e209af6d1cd2dba41fc7688cb7b6babb6ea3d8a3`, `ev_31dd52d5fb9b233e516a9030cd689ec59fb525a24bddbf09dc526faa93982dea`, `ev_a5661d4238de69b926093bdc3eb6e6e9f82e19c81c14e703cefc4c06c997634d`, `ev_5a77ab81f7e3c19077540f5d09d56c78668204a62b4931a23f4325a5986acbd5` e `ev_4f6e425ef8e27412af06b2b4fa3b3c5032b85ced251741e40a0077d5f7e31ec7`.

O campo `SidebarItem.data` aponta diretamente para o descritor de `SidebarItemData`, Evidence `ev_2231a01fe1378cb4b499ea933b540af81c0e6fc13c673bf12a79b6d1b52a5459`. O tipo simbólico de `SpaceModel.profile` aponta indiretamente ao enum `Profile`, cujos casos registrados são `custom` e `initial`; o descritor informa um caso com payload e um sem payload. Evidences `ev_e32f69e61852e8c71e6e956452675593866310376996d42391386048b104f8d5`, `ev_49a43c98c0618ff1eaf2251bca8f79edad1b437833b3c217621e3b68d6a65a41`, `ev_65b86bed5892fdaca4d66cbb7546b57e8258682a49ae5b864ab89e821c484cff`, `ev_0bd50ea01716bf664e8de3fb32bde39eb5b3c81fe104cf6aceb5f0c0a885c696` e `ev_86c22c9cfe7e83b0d0815156bec6319f4a3a272e259f09ef0c1d3ff1dc6ff0be`. Isso confirma um campo tipado por Profile no modelo, mas não cardinalidade global, troca de Space ou isolamento de cookies. A relação entre esse enum e a variante `profile.default` observada no JSON continua aberta.

Ainda faltam os tipos de `containerIDs` e `customInfo`, os consumidores de seleção e o comportamento de atualização. Ponteiros indiretos refletem a memória do Ghidra e podem conter fixups do loader.


## Identificadores dos containers e Profile personalizado

A referência simbólica do payload do caso `Profile.custom` aponta diretamente ao descritor de `ProfileID`, que declara `directoryBasename` e `machineID`. O caso `initial` não tem tipo de payload no registro examinado. Evidence `ev_7cea6b734669494ef5846049ad097a4c38d8d6a192fbd69fc4c2e38ab7a1c859`; descritores e campos estão nas referências acima. Isso identifica a representação associada ao Profile personalizado, sem provar como o navegador cria diretórios ou separa sessões.

A referência de tipo de `SpaceModel.containerIDs` contém os símbolos `ContainerType`, `Identifier` e `SidebarItem`, com gramática compatível com `Dictionary<ContainerType, Identifier<SidebarItem>>`. O enum `ContainerType` registra os casos `unpinned` e `pinned`, um com payload e outro sem. Evidences `ev_9ea93e6d37a877511ed09f19c9f650ec2851ec95cbe782773e57af0dbacd9a62`, `ev_a41572e51c2a6ae248dda5ece941735891d580d6164a6ea42597aef6c191963e`, `ev_e9713e49c9fb1fa7525581aafac3a54cb235a86037c0ce53acca10deaa3532a7`, `ev_687b3b7469eeccd434550b8a7142795ba7996f68feeb1472ce404d00b2dba195` e `ev_4cb44d892e8a7bd2f285b0d87899c2c2ff03a2efe0a1f4e082cfbbab1847d31d`. A tipagem do dicionário é inferida da gramática; a posição visual, as regras de cada container e o payload de `unpinned` ainda precisam de investigação.


O payload de `ContainerType.unpinned` aponta diretamente para `UnpinnedContainerType`, cujos casos registrados são `desktop` e `shared`. O caso `desktop` tem payload e `shared` não tem no descritor examinado. O tipo simbólico do payload de `desktop` inclui uma string com rótulo `machineID` e uma referência indireta ao descritor `WindowID`; a gramática sugere um segundo elemento rotulado `windowID`. Evidences `ev_b041abb3b3fb3401c17a225130b3d1665c91130ce67d7976cdbbf15e2654afed`, `ev_44e78d4cdd5b049cb302e8bb79250f89d8f9e7039da8f1dc778f329a02728702`, `ev_e8097b00ea00947cd3e24b4bdae978b908b8b06d480a39a972f99f9d145f5058`, `ev_f384cc7e3b204f3269b19fc75ae00a05e30cf8c5b716372d3f5c731a09e6f59d`, `ev_15a7f49dca219b0453489616746b952d2896fa8793f56ce8ee1e34b489d37d11` e `ev_da29fc7bd425cfa89623ec19c0edba12f83ac43ea5ed2ad694cfe97bca0d05f1`. Essa estrutura associa nominalmente um container não fixado de desktop a máquina e janela; não estabelece política de sincronização ou compartilhamento entre janelas.


## Identidade de contexto de navegação

`BrowserContextID` declara `contextType` e `profile`. A referência de tipo do primeiro campo aponta diretamente para `BrowserContextType`, com casos `normal` e `incognito`; a do segundo aponta pelo mesmo endereço indireto `0x104234388` ao descritor `Profile` usado em `SpaceModel.profile`. Evidences `ev_1be9ebd0c8d3b5063a8ec34ce49312767de9b7a9bcf7fe87d8c5f625a1758a6a`, `ev_7fd690ac0a29fdbbe22911939673083c491fa456f0f5b8acbed645e73ca456dc`, `ev_2def450b5dd72b9367748891216331b889f01ed440279239c5d17ee6aada37c5`, `ev_cd4dc373df1636f4c7abe906ae1eb7a8b6801b49deca7b861a41a739b3794bf1`, `ev_f3f3c4166dd626411cef83f704c8547684ecdef0c4e4a1bcdebc77ad318262dd` e `ev_24007294e3752112eaa1af558bd5fc78e006fd7dfe03e8803fa286100ce43185`. Essa é uma ligação de tipos entre modelo de Space e identidade de contexto, ainda sem demonstrar o mapeamento entre instâncias ou o isolamento efetivo.

O descritor de `WindowID` contém um campo `rawValue`, Evidences `ev_0c6e06827e48af1219336b7ac8547c57d7cdbf23f83ba722d7843c8681e1f1e3` e `ev_0a95753f0bffc6d52122329f9398be99f4ca64e66759ed6c4e98cc32bded350b`. Já `BrowserContextAttributes` declara `directoryBasename`, `userVisibleName` e `userColor`, Evidences `ev_c80b05a864adeb2fe026ebab196c3ebaa4c5e4a8b3af834141839e7de9bbcfd2`, `ev_7d457d8364060ac4dac213740c823984417072f5dc7fc952d743e9e4ee81c7fc` e `ev_8e3cde349ee1282fb2177b5cd6c37bb7689aedb0d30588145cac69af6125f8eb`. A igualdade do nome `directoryBasename` com um campo de `ProfileID` não prova que os valores coincidam nem que o campo identifique o mesmo diretório em runtime.
