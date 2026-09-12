# Plano de Correções — AfroBarber v1 (2026-08 / 2026-09)

Registro histórico das duas rodadas de correções feitas a partir de análises profundas do projeto. Mantido para referência futura — prefira sempre o estado atual do código e do `CLAUDE.md` a este documento.

---

## Rodada 1 — 2026-08-08

### Contexto
Análise encontrou 3 bugs reais, um sistema de precificação duplicado (código morto), documentação desatualizada em vários pontos, 29 arquivos `.cs` em encoding ISO-8859-1 em vez de UTF-8, e zero testes.

### Bugs corrigidos
- **`MobilePerformanceBootstrap.cs`**: campo `iosQualityName` adicionado para corrigir compilação em build iOS.
- **`LoanPanelUI.cs`**: subscrição por lambda trocada por padrão `eventosSuscritos` + métodos nomeados.

### Arquitetura
- Removido `DynamicPricingManager` (código morto — `AplicarMargem()` nunca era chamado).
- 29 arquivos convertidos de ISO-8859-1 para UTF-8.

### Documentação
- Seção "Biblioteca de Cortes" reescrita (dados vivem em `ClientRequestData`, não em `AfroCutDatabase`).
- `Narrative Mission System`: IDs corrigidos — `leo` → `leandro_leo`, `miguel` → `gabriel`.
- `Finance Monthly Bills Manager`: gatilho de `OnCriseFinanceira` corrigido.
- `Daily Challenge System`: adicionado `GanharMaestria` à tabela de tipos.
- Nova seção "Estado do Repositório / Débito Técnico Conhecido".

### Testes
- Scaffold mínimo criado em `Assets/Tests/EditMode/` (2 testes de exemplo).

---

## Rodada 2 — 2026-09-12

### Contexto
Segunda análise profunda. Encontrou 10 bugs/débitos técnicos críticos e importantes, além de funcionalidades inteiras desconectadas dos sistemas que deveriam consumi-las.

### Bugs corrigidos

| # | Arquivo | Bug |
|---|---|---|
| 1 | `BarbershopServiceManager.cs` | `ClientId` volátil — `GetInstanceID()` trocado por `ClientNPC.ClientId` como chave de fidelidade |
| 2 | `Economy/LoanSystem.cs` | `SpendMoney(valorPagamento)` antes do `Mathf.Min` — destruía dinheiro além do saldo devedor |
| 3 | `BarbershopServiceManager.cs` | `MissionSystem.RegisterServiceCompleted` movido para `NotificarSistemasExternos` (cobre ambos os fluxos, não só o fallback) |
| 4 | `Gameplay/VIPClientSystem.cs` | Gorjeta VIP calculada mas nunca registrada na fidelidade; `ClientLoyaltySystem.RegistrarVisita` adicionado com a gorjeta real |
| 5 | `BarbershopServiceManager.cs` | Duplicação de registro de fidelidade para VIP — `NotificarSistemasExternos` agora pula `ClientLoyaltySystem` para VIPs (VIP registra via `NotificarAtendimentoVip`) |
| 6 | `Appointment/ClientAppointmentScheduler.cs` | Leak de event handler — `Dictionary<ClientNPC, Action>` rastreia e remove o handler anterior antes de adicionar novo |
| 7 | `Progression/PlayerXPManager.cs` | `PrestigeSystem.BonusXP10` sem efeito — `AddXP` agora aplica 3 multiplicadores em sequência: upgrade → prestígio → evento cultural |
| 8 | `BarbershopServiceManager.cs` | `CulturalEventSystem.AplicarBonusDinheiro` nunca chamado — `DistributeAdvancedServiceRewards` aplica o bônus e retorna `moneyFinal`; `NotificarSistemasExternos` recebe o valor pós-bônus |

### Funcionalidades desconectadas — agora conectadas

- **BarbershopUpgradeSystem → sistemas externos**: `AplicarEfeitosNosSistemas()` propaga `MultiplicadorAtracao` → `ClientSpawner`, `ReducaoEnergia` → `PlayerEnergySystem`, `MultiplicadorXP` → `PlayerXPManager`.
- **PrestigeSystem.GorjetaExtra20**: `ClientLoyaltySystem.GetMultiplicadorGorjeta` já multiplica por `PrestigeSystem.GetBonusGorjeta()` (era o único perk já conectado).

### Correções de infraestrutura

- `SettingsManager.cs`: `Screen.SetResolution()` e `Screen.fullScreen` não são chamados em `Application.isMobilePlatform` (corrige distorção de tela no Android).
- `PlayAssetDeliveryLoader.cs`: APIs `.states`, `.GetAssetPackState()`, `.GetDownloadStatus()` removidas (CS1061) — usa apenas `isDone` com progresso animado.
- `MobilePerformanceBootstrap.cs`: suporte a iOS removido (`#elif UNITY_IOS` e campo `iosQualityName` eliminados — projeto não tem alvo iOS ativo).
- `AssetPackLoadingUI.cs`: `BloquearMenu(true)` movido de `Start()` para `Awake()` — botão Jogar não fica interativo antes do PAD terminar.

### UI de Conquistas criada

Scripts em `Assets/Afrobarber/Scripts/UI/Achievements/`:
- `AchievementPopupUI.cs` — popup com fila e fade animado, assina `OnConquistaDesbloqueada`
- `AchievementsPanelUI.cs` — painel de lista com progresso (Abrir/Fechar/RefreshUI)
- `AchievementsItemUI.cs` — item individual com estado bloqueado/desbloqueado

API adicionada: `AchievementSystem.GetTodasConquistas()→IReadOnlyList<ConquistaData>`.

**Wiring pendente no Unity Editor**: criar `Popup_Conquista` (com `CanvasGroup`) e `Panel_Conquistas` (com ScrollView) no Canvas da HUD e referenciar os campos SerializeField.

### Documentação (`CLAUDE.md`)
- Seção "Service Flow" reescrita: fluxo de minigame documentado, ordem real de `NotificarSistemasExternos` corrigida (9 itens), `MissionSystem` adicionado à lista.
- Prestige System: "Wiring pendente" corrigido para "Wiring implementado".
- Quick Reference atualizada: `PlayerEnergySystem`, `PlayerXPManager`, `InventoryManager`, `BarbershopRatingManager`, `BarbershopUpgradeSystem` com métodos novos.
- Nova seção "Minigame de Serviço".
- Helpers não-singleton documentados: `ClientSpawnerLocator`, `BootstrapUIBehaviour`, `PersistentGameObject`.
- Débito técnico atualizado para refletir o estado real.

---

## Débito técnico remanescente (2026-09-12)

| Item | Status | Ação necessária |
|---|---|---|
| **BarbershopUpgradeSystem wiring na cena** | Pendente — requer Unity Editor | Criar filho de GameBootstrap, atribuir 7 assets de `ScriptableObjects/Upgrades/` |
| **AchievementSystem UI wiring na cena** | Pendente — requer Unity Editor | Criar `Popup_Conquista` + `Panel_Conquistas` no Canvas da HUD |
| **HDRP/URP em Android** | Não verificável sem device | Validar build real; se materiais ficarem rosas, criar set URP/Lit ou remover troca de pipeline |
| **Cobertura de testes** | Scaffold existe, cobertura zero | Adicionar testes para sistemas principais (fidelidade, finanças, maestria) |
| **Eventos sem assinante** | Documentado, não é bug | `OnAtendimentoConcluido`, eventos NarrativeMissionSystem, `OnRelatorioGerado` — pontos de extensão para UI futura |
| **Sistema Transito** (`Transito/*.cs`) | Existe no código, não documentado | Confirmar se está ativo na cena; se sim, documentar; se não, avaliar remoção |
| **Sistema de Diálogo contextual** (`Dialogue/DialogueContextOptionsProvider` etc.) | Existe no código, não documentado | Idem — confirmar uso na cena antes de documentar |
| **Assembly Definitions** | Ausente | Criar `.asmdef` por pasta principal para reduzir tempo de recompilação |
