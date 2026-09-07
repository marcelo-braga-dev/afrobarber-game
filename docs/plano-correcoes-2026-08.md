# Plano de Correções — AfroBarber v1 (2026-08)

Registro histórico da rodada de correções feita a partir de uma análise profunda do projeto em 2026-08-08. Mantido para referência futura — se algo aqui já foi superado por mudanças posteriores, prefira o estado atual do código/`CLAUDE.md` a este documento.

## Contexto

A análise (código + `CLAUDE.md` + config de projeto + VCS) encontrou 3 bugs reais reproduzíveis, um sistema de precificação duplicado (um dos dois era código morto), documentação (`CLAUDE.md`) desatualizada em vários pontos, 29 arquivos `.cs` salvos em encoding errado (ISO-8859-1 em vez de UTF-8), e zero testes automatizados apesar de infraestrutura documentada para isso.

## O que foi corrigido nesta rodada

### Bugs
- **`MobilePerformanceBootstrap.cs`**: adicionado o campo `iosQualityName`, que faltava e quebrava a compilação em build iOS (`#elif UNITY_IOS` referenciava um campo inexistente).
- **`LoanPanelUI.cs`**: trocado o `AddListener`/`RemoveListener` por lambda (que nunca desinscrevia de fato) pelo padrão `eventosSuscritos` + métodos nomeados, igual ao já usado em `BarberBookPanelUI.cs`.

### Higiene de código
- 29 arquivos `.cs` convertidos de ISO-8859-1 para UTF-8 (conteúdo e quebras de linha CRLF preservados, só o encoding mudou). Lista completa no commit/diff desta mudança.

### Arquitetura
- Removido `DynamicPricingManager` (`Economy/DynamicPricingManager.cs` + GameObject correspondente em `GameScene.unity`): era um segundo sistema de precificação cujo método `AplicarMargem()` nunca era chamado por nenhum outro script. Quem calcula preço de verdade é `GlobalGameplayManagement` (`Gameplay/GlobalGameplayManagement.cs`), que não estava documentado no `CLAUDE.md` e agora está.

### Documentação (`CLAUDE.md`)
- Seção "Biblioteca de Cortes" reescrita: dados de corte vivem em `ClientRequestData` (não mais em `AfroCutDatabase`, que está em `_Deprecated/` só para não quebrar o asset legado órfão `MainAfroCutDatabase.asset`).
- `Post-service calls`: `UnlockCut(request.afroCutId)` → `UnlockCut(request.RequestId)` (incondicional).
- `Narrative Mission System`: personagens corrigidos — `leo` → `leandro_leo`, `miguel` → `gabriel`.
- Nota "Dual finance systems" corrigida: `PlayerWallet` não existe mais; nota de precificação duplicada substituída pela explicação do `GlobalGameplayManagement`.
- `TryStartService`: removida a alegação falsa de checagem de loadout incompleto.
- `Appointment System`: `CascadeDelayAppointments()` → `ApplyCascadeDelay(...)`; adicionada nota de que agendamentos não são persistidos e não há cancelamento público funcional.
- `Finance Monthly Bills Manager`: `OnCriseFinanceira` corrigido para refletir o gatilho real (`FinanceManager.CurrentCash < 0`).
- `Daily Challenge System`: adicionado `GanharMaestria` à tabela de tipos.
- Nova seção "Estado do Repositório / Débito Técnico Conhecido" com o resumo dos itens abaixo.

### Testes
- Criado scaffold mínimo em `Assets/Tests/EditMode/` (assembly definition + 2 testes de exemplo sobre lógica estática pura), como ponto de partida.

## O que ficou como débito técnico (decisão consciente, não esquecimento)

- **Git aninhado em `Assets/Afrobarber/Scripts/.git`**: não foi tocado. Está desatualizado em relação ao disco (remoto `github.com/marcelo-braga-dev/afrobarber`) e deve ser tratado como espelho manual, não como histórico confiável, até alguém decidir resincronizá-lo ou descontinuá-lo.
- **Padrão `eventosSuscritos` em UI**: só o bug confirmado (`LoanPanelUI`) foi corrigido. Outros arquivos de UI que se inscrevem em eventos sem esse padrão continuam como estavam — candidatos a uma passada dedicada futura.
- **Risco HDRP/URP em mobile**: `MobilePerformanceBootstrap` continua trocando de quality level/pipeline em Android/iOS. A maioria dos materiais do projeto só tem shader `HDRP/Lit`. Não foi alterado nada aqui — precisa de validação em build/device real e de uma decisão de produção (duplicar materiais para URP ou parar de trocar de pipeline em mobile).
- **Cobertura de testes**: o scaffold criado é só um ponto de partida, não cobertura real dos sistemas principais (maestria, fidelidade, narrativa, financeiro, etc.).

## Verificação feita

- Greps de sanidade confirmando que não sobrou nenhuma referência a `DynamicPricingManager` (código ou cena) nem fileID órfão na `GameScene.unity`.
- Todos os 29 arquivos confirmados como `UTF-8 text` via `file`, com amostra de texto acentuado revisada visualmente.
- Cada correção do `CLAUDE.md` foi conferida linha a linha contra o código-fonte citado.
- Este ambiente não tem Unity Editor instalado, então a compilação e o teste em Play Mode **não foram verificados automaticamente**. Recomendação: abrir o projeto no Unity, deixar recompilar, abrir a `GameScene` e testar abrir/fechar o painel de empréstimos e checar o preço de um serviço antes de dar commit/push nesse estado.
