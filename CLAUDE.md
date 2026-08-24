# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Engine & Hard Constraints

- **Unity 6000.3.10** — exact version required; do not upgrade
- **Render pipelines**: HDRP 17.3.0 and URP 17.3.0 coexist — use HDRP Lit or URP Lit materials, never Standard
- **Input**: New Input System 1.18.0 only — never `Input.GetKey()` / legacy API
- **Language**: all code, comments, UI strings, and asset names are in **Portuguese**
- **Do not touch**: `Assets/Afrobarber/Scripts/_Deprecated/` (contains `SofaSeats.cs`, `ShopPurchaseHandler.cs`) — isolated legacy code, never reference from active scenes
- **Tests**: Unity Test Framework 1.6.0 — run via Window → General → Test Runner; test files go in `Assets/Tests/`

---

## Bootstrap & Initialization

`Core/GameBootstrap.cs` — singleton (`Instance`), runs `BootstrapRoutine()` coroutine on Awake:
1. Calls `IGameBootstrapInitializable.InitializeFromBootstrap()` on all children
2. Enables `uiObjectsToEnableAfterBootstrap[]` then `delayedObjectsToEnable[]`
3. Exposes `bool IsReady`, `float Progress01`

Core managers survive scene loads via `Core/PersistentGameObject.MakePersistent(gameObject)` → sets parent null + `DontDestroyOnLoad`.

All managers must exist in the scene **before** being called via `.Instance` — `GameBootstrap` initializes them in sequence.

---

## Player Nickname System

`Core/PlayerNicknameManager.cs` — singleton simples que armazena e persiste o nickname (apelido) do jogador, exibido em chat, ranking e outros painéis.

**API:**

- `Nickname` → nome atual (padrão `"Jogador"` se nunca configurado)
- `TemNicknamePersonalizado` → `true` se o jogador já salvou um nome customizado
- `SetNickname(string)` → valida (não vazio, corta em `tamanhoMaximo` chars), persiste e dispara `OnNicknameChanged`

**UI:** `UI/PlayerPanel/PlayerNicknameInputUI.cs` — painel com `TMP_InputField` + botão de confirmação. Se `mostrarNaPrimeiraExecucao = true` e `!TemNicknamePersonalizado`, abre automaticamente no `Start()` (onboarding). Pode ser reaberto a qualquer momento (ex.: menu de configurações) para o jogador trocar o nome.

**Consumido por:**

- `GlobalDialogueManager.AddPlayerMessage` — usa `PlayerNicknameManager.Instance.Nickname` como `senderDisplayName` em vez do `"Você"` fixo
- `DailyChallengeSystem` — entrada do jogador no leaderboard (`GetNomeJogador()`) usa o nickname; assina `OnNicknameChanged` para atualizar a entrada já gerada sem esperar a renovação diária

**Persistência:** `AFROBARBER_PLAYER_NICKNAME` (string).

**Wiring na cena:** adicionar `PlayerNicknameManager` ao `GameBootstrap` como filho (mesmo padrão de `LoanSystem`/`ClientLoyaltySystem`). Adicionar `PlayerNicknameInputUI` a um painel da HUD (ou tela inicial) com `TMP_InputField` e `Button` referenciados.

---

## Client State Machine

`Characters/Clients/ClientNPC.cs` — state field `ClientState currentState`:

```
None → Wandering → GoingToBarbershop → GoingToEntrance → GoingToWaitingPoint → WaitingForService
     → GoingToBarberChair → InService → GoingToCashier → ReturningToCity
```

Key transitions:
- `WaitingForService`: shows `interactionIcon`, player clicks → `OnPlayerClicked()` → `ClientRequestUI.Show(this)`
- `CallForService()` → `BarbershopServiceManager.TryStartService(this)`
- `StartService(walkPoint, sitPoint)` → state = `GoingToBarberChair`
- `MarkServiceCompleted()` → applies final hair, state stays `InService`
- `GoToCashier(cashierPoint)` → state = `GoingToCashier`
- `ForceDespawn()` → destroys, notifies spawner

Scene transform names required by `ClientNPC.Initialize()`:
`entrancePoint`, `barberChairWalkPoint`, `barberChairSitPoint`, `exitPoint`

Waiting area uses `ApproachPoint` (NavMesh-reachable floor) + `SitPoint` (exact body snap position) per seat.

---

## Service Flow

### Service initiation — `Barbershop/BarbershopServiceManager.cs` (singleton)

`TryStartService(ClientNPC client)` returns false when:
- another client is already active (`currentClient != null`)
- client has no `RequestData`
- `PlayerEnergySystem.Instance.CanStartService()` is false
- `barberChairWalkPoint` not set

> Note: the method does NOT validate loadout completeness — if `loadout == null` it just falls back to an empty `PreparedServiceLoadout()` and proceeds.

On success: calls `BarberWorkController.StartService()`, then `WaitClientSitThenStartServiceFlow()` coroutine.

### Two execution paths (controlled by `useAdvancedServiceWorkflow` bool):

**Advanced path** (`Services/Advanced/`):
1. `AdvancedServiceWorkflowManager.TryPreparePlan(client)` — auto-generates steps: Wash → Cut → Finish → Finalize
2. If `openPlanningUIBeforeAdvancedExecution`: opens `ServicePlanningUI` first; player can reorder/swap tools
3. `AdvancedServiceWorkflowManager.TryExecutePlan(client, onFinished)` → `ServiceExecutionSystem.StartExecution()`
4. `AdvancedServiceOutcomeResolver.Resolve()` → produces `AdvancedServiceResult`
5. `HandleAdvancedServiceFinished()` → distributes money/XP/rating

**Fallback auto-service** (time-based, `autoCompleteServiceByTime = true`): simple coroutine timer.

`fallbackToOldAutoServiceIfAdvancedFails = true` should be set during development.

### Tool compatibility — `Services/Advanced/ServiceToolCompatibility.cs` (static):

| Action | Compatible `ProductCategory` |
|---|---|
| Wash | ProdutoCapilar |
| Comb | Pente |
| Cut | MaquinaDeCorte, Tesoura |
| Razor | Navalha, Laminas |
| Finish | ProdutoCapilar, Pente |
| Define | ProdutoCapilar, Pente |
| Beard | Navalha, MaquinaDeCorte, Tesoura |
| Finalize | ProdutoCapilar, Pente |

### Post-service calls (inside `BarbershopServiceManager`):
- `FinanceManager.Instance.RegisterServiceIncome(request, clientName)`
- `PlayerXPManager.Instance.AddXP(xpAmount)`
- `BarbershopRatingManager.Instance.AddReview(rating)`
- `EducationProgressManager.Instance.UnlockCut(request.RequestId)` (unconditional — cut identity IS the request ID, there's no separate `afroCutId` field)

---

## Appointment System

`Appointment/ClientAppointmentScheduler.cs` — singleton que auto-gera agendamentos baseados no tempo de jogo e na reputação da barbearia.

**Estados de um agendamento** (`ClientAppointmentData.AppointmentStatus`):
```
Scheduled → WaitingToSpawn → Spawned → Completed
                           ↘ DelayedByQueue → Spawned
                           ↘ Overbooked / Cancelled / Missed
```

**Componentes:**
| Classe | Responsabilidade |
|---|---|
| `ClientAppointmentScheduler` | Gera e faz spawn de clientes agendados. Integra com `BarbershopRatingManager` para frequência. Não persiste em `PlayerPrefs` — agendamentos existem só em memória (`List<ClientAppointmentData>`) e não sobrevivem a um restart. `Missed` é detectado automaticamente: se `missedAfterMinutes` (padrão 60min de jogo) se passarem do horário marcado sem o cliente ser ativado, o agendamento é marcado `Missed`, removido da lista e o cliente manda uma mensagem de diálogo (`ClientAppointmentProfile.missedMessage`). Não há cancelamento público funcional hoje (`cancelled` existe no modelo mas nada o define como `true` — precisaria de uma ação do jogador ainda não construída). |
| `ClientAppointmentData` | Modelo de dados do agendamento (horário, status, cliente associado). |
| `ClientAppointmentProfile` | ScriptableObject: tolerância de pontualidade, velocidade de caminhada, mensagens de diálogo. |
| `ClientAppointmentDialogueBridge` | Ponte entre eventos do agendamento e `GlobalDialogueManager` (chama `GlobalDialogueManager.Instance.AddSystemMessage(...)` diretamente). |
| `AppointmentPanelUI` | Painel UI que lista todos os agendamentos. Auto-atualiza via evento `OnAppointmentsChanged`. |
| `AppointmentItemUI` | Item individual de agendamento com cor por status e texto formatado. |

**Integração com outros sistemas:**
- Agendamentos respeitam `BarberQueueSystem.AddClientToQueue()` — cliente agendado entra na fila normalmente
- Overbooking configurável: `ClientAppointmentScheduler.overbookingChance`
- Atrasos em cascata quando fila está cheia: `ApplyCascadeDelay(ClientAppointmentData)` (privado)
- Reputação alta → maior frequência de agendamentos automáticos

**Wiring necessário na cena:**
- Adicionar `ClientAppointmentScheduler` ao GameBootstrap como filho
- Conectar `AppointmentPanelUI` à HUD principal (ativar após bootstrap)

---

## Loan System

`Economy/LoanSystem.cs` — singleton que oferece 4 opções de empréstimo com juros semanais compostos.

**Opções padrão** (configuráveis no Inspector):

| Índice | Valor | Taxa/sem | Prazo |
|---|---|---|---|
| 0 | BM$500 | 5% | 4 sem |
| 1 | BM$1.000 | 8% | 6 sem |
| 2 | BM$2.500 | 12% | 10 sem |
| 3 | BM$5.000 | 18% | 16 sem |

**Regras:**
- Máximo 1 empréstimo ativo por vez (`TemEmprestimoAtivo`)
- Juros são processados a cada semana de jogo (`OnDayChanged` → verifica mudança de semana)
- Atraso > prazo: status muda para `EmAtraso`, aplica multa (`multaAtrasoPercent`) e penaliza reputação
- `SpendMoney` do `FinanceManager` é usado para débito — não cria terceiro sistema financeiro
- `TentarPagarTudo(loan.id)` itera sobre todos os empréstimos `Ativo` ou `EmAtraso`

**Wiring na cena:** adicionar `LoanSystem` ao GameBootstrap como filho.

---

## BarberBook — Rede Social

`Social/BarberBookSystem.cs` — singleton que simula posts de clientes após atendimentos.

**Fluxo de publicação:**
1. `BarbershopServiceManager.NotificarSistemasExternos()` chama `PublicarPosAtendimento()` para clientes não-VIP
2. `VIPClientSystem.NotificarAtendimentoVip()` chama `PublicarPosAtendimento(isVip: true)` para VIPs
3. Posts virais (`isVip && rating == Perfeito`) têm 50–200 likes e disparam `OnPostViral`
4. A cada troca de dia de jogo: posts expirados são removidos + `AplicarClientesOrganicos()` converte posts positivos em futuros clientes

**Persistência:** máximo de 60 posts (`maxPostsArmazenados`), expiram em 4 dias de jogo (`diasExpiracaoPost`).

**UI:**
| Classe | Responsabilidade |
|---|---|
| `BarberBookPanelUI` | Painel principal — lista de posts com scroll, popup viral, contador orgânico |
| `BarberBookPostItemUI` | Card individual — fundo dourado (viral), verde (positivo), vermelho (negativo) |
| `BarberBookNotificationBadge` | Badge no botão HUD — mostra `PostsNaoLidos` (max "99+") |

**Cuidado de duplicação:** VIP nunca deve chamar `PublicarPosAtendimento` diretamente de `BarbershopServiceManager` — a rota exclusiva é via `VIPClientSystem.NotificarAtendimentoVip`.

---

## VIP Client System

`Gameplay/VIPClientSystem.cs` — singleton que agenda a aparição de clientes especiais.

**Condições para VIP aparecer hoje:**
- `BarbershopRatingManager.GlobalRating >= ratingMinimoParaVip` (padrão 3.5)
- `PlayerXPManager.CurrentLevel >= nivelMinimoJogador` (padrão 2)
- `diasDesdeUltimoVip >= minDiasEntreVips` (padrão 3 dias)
- `Random.value <= chanceVipPorDia` (padrão 25%)

**Comportamento:**
- Preço 3× (`multiplicadorPrecoVip`)
- Paciência reduzida: 6 min (vs padrão)
- Gorjeta 2.5× (`multiplicadorGorjetaVip`)
- Post VIP no BarberBook ao ser atendido (via `NotificarAtendimentoVip`)

**Wiring:** `ClientSpawner` deve verificar `VipAgendadoHoje` e chamar `MarcarClienteComoVip(cliente)` ao spawnar o cliente VIP.

---

## Weather System

`Core/WeatherSystem.cs` — singleton auto-dirigido de clima; ninguém precisa chamar `.Instance` nele porque ele reage sozinho a `GameTimeSystem.OnWorkDayStarted` e empurra efeitos para outros sistemas.

**Tipos** (`WeatherType`): `Ensolarado`, `Nublado`, `Chuvoso`, `Tempestade`. Cada um tem `multiplicadorDemanda`, `multiplicadorGorjeta`, `chanceHumorPositivo` e uma `descricao` (configuráveis no Inspector via `WeatherEffect`).

**Fluxo:**
1. A cada novo dia de trabalho (`OnWorkDayStarted`), sorteia um novo clima com pesos por estação do ano (hemisfério sul: verão/outono/inverno/primavera, calculados a partir de `GameTimeSystem.CurrentMonth`).
2. `SetClima(tipo)` aplica o efeito: chama `ClientSpawner.SetDemandMultiplierFromWeather(multiplicadorDemanda)` (via `FindFirstObjectByType`) e posta a `descricao` do clima como mensagem de sistema via `GlobalDialogueManager.Instance.AddSystemMessage(...)`.
3. Dispara `OnClimaAlterado` (`UnityEvent<WeatherType>`).

**API:** `ClimaAtual`, `EfeitoAtual`, `SetClima(tipo)`, `SortearNovoClima()`, `GetEfeito(tipo)`, `OnClimaAlterado`.

---

## Prestige System

`Gameplay/PrestigeSystem.cs` — singleton de reset com progressão permanente (New Game+).

**Pré-requisito:** jogador nível 5 ("Lenda AfroBarber") + `nivelPrestigio < maxNivelPrestigio` (3).

**Perks disponíveis** (`PrestigePerk`):

| Enum | Efeito |
|---|---|
| `BonusXP10` | +10% XP em todos os atendimentos |
| `GorjetaExtra20` | +20% gorjeta — empilha com `ClientLoyaltySystem` |
| `ReputacaoInicial` | Começa com reputação 3.5 ao invés de 2.0 |
| `AlugueMenor15` | -15% no valor do aluguel mensal |
| `DinheiroInicial500` | +BM$500 ao recomeçar |

**API de modificadores** (consumida por outros sistemas):
- `GetBonusXP()` → 1.10f ou 1f
- `GetBonusGorjeta()` → 1.20f ou 1f
- `GetFatorAluguel()` → 0.85f ou 1f
- `GetReputacaoInicial()` → 3.5f ou 2f

**Wiring:** `PlayerXPManager.AddXP` deve multiplicar por `PrestigeSystem.Instance?.GetBonusXP() ?? 1f`.

---

## Daily Challenge System

`Gameplay/DailyChallengeSystem.cs` — singleton de desafios diários com ranking fictício.

**Renovação baseada em tempo REAL** (`DateTime.Now`) — não no tempo de jogo. Desafios se renovam uma vez por dia de calendário real, independentemente de quantos dias de jogo passaram.

**Tipos de desafio** (`DailyChallengeTipo`):

| Tipo | Descrição |
|---|---|
| `AtenderNClientes` | Atender N clientes no dia |
| `FaturarNReais` | Faturar BM$N no dia |
| `FazerNPerfeitos` | Completar N serviços com rating Perfeito |
| `SemServicoRuim` | Completar N serviços seguidos sem rating ≤ Ruim |
| `AtenderConsecutivos` | Atender N clientes consecutivos sem errar |
| `GanharMaestria` | Acumular N pontos de maestria de corte no dia (ver seção "Cut Mastery System" / `CutMasterySystem.RegistrarAtendimento`) |

**Integração:** `BarbershopServiceManager` deve chamar `DailyChallengeSystem.Instance?.RegistrarAtendimento(rating, valorRecebido)` ao finalizar cada serviço.

**Persistência:** `DailyChallengeSaveWrapper` persiste desafios e progresso. O leaderboard NÃO é persistido — é regenerado com pontuações fictícias a cada sessão (design intencional para dar sensação de "ao vivo").

**UI:**
| Classe | Responsabilidade |
|---|---|
| `DailyChallengePanelUI` | Painel com duas abas: Desafios / Ranking |
| `DailyChallengeItemUI` | Card de desafio — slider de progresso + botão Coletar |
| `LeaderboardItemUI` | Linha do ranking — destaque dourado para jogador, medalhas para top 3 |

---

## Narrative Mission System

`Missions/NarrativeMissionSystem.cs` — singleton de arcos narrativos com 5 personagens fixos, cada um com 3 capítulos progressivos.

**Personagens e arcos pré-configurados:**

| ID | Nome | Backstory resumido |
|---|---|---|
| `seu_ribeiro` | Seu Ribeiro | Aposentado, 68 anos, quer reviver o Afro Clássico dos anos 70 |
| `kemi` | Kemi | Designer gráfica, 25 anos, exigente — arco de autoestima e identidade |
| `leandro_leo` | Leandro "Leo" | *(3 capítulos de progressão)* |
| `dona_conceicao` | Dona Conceição | *(3 capítulos de progressão)* |
| `gabriel` | Gabriel | 19 anos, arco de autoestima/confiança em torno do corte Shape-Up |

> IDs corretos são `leandro_leo` e `gabriel` — não `leo`/`miguel`. Configurar `narrativeCharacterId` com o ID errado falha silenciosamente (nenhum arco é vinculado).

**Desbloqueio de capítulo:** requer `nivelJogadorNecessario` + capítulo anterior concluído (`capituloAnteriorId`).

**Fluxo:** quando `ClientNPC` tem `narrativeCharacterId` configurado, ao finalizar serviço chamar:
```csharp
NarrativeMissionSystem.Instance?.RegistrarAtendimentoNarrativo(
    characterId, clienteNome, rating);
```

**Wiring na cena:** cada NPC prefab deve ter o campo `narrativeCharacterId` preenchido (ex.: `"seu_ribeiro"`).

---

## Client Loyalty System

`Progression/ClientLoyaltySystem.cs` — singleton que rastreia visitas por cliente e aplica bônus progressivos.

**Tiers** (`LoyaltyTier`):

| Tier | Visitas | Gorjeta | Paciência extra | Chance elogio |
|---|---|---|---|---|
| `Novo` | 0–1 | 1× | 0 min | 0% |
| `Conhecido` | 2–4 | 1.1× | 5 min | 5% |
| `Regular` | 5–9 | 1.25× | 10 min | 10% |
| `Fiel` | 10–19 | 1.5× | 15 min | 18% |
| `Lendario` | 20+ | 1.85× | 25 min | 30% |

**Bônus de Prestígio:** `GetMultiplicadorGorjeta` multiplica pelo `PrestigeSystem.GetBonusGorjeta()` automaticamente.

**Wiring:** `BarbershopServiceManager` deve registrar visita do cliente ao concluir serviço. `ClientNPC` deve expor `ClientId` (string única por prefab/personagem) para identificação.

---

## Finance Monthly Bills Manager

`Economy/FinanceMonthlyBillsManager.cs` — singleton de contas mensais recorrentes (aluguel, luz, internet, etc.).

**Comportamento:**
- Gera contas do mês atual em `Start()` se `autoGenerateOnStart = true`
- Monitora mudança de mês via `GameTimeSystem.OnDayChanged`
- Período de graça: `diasGracePeriod` dias após vencimento antes de aplicar multa
- Multa por atraso: `multaAtrasoPercent`% + penalidade de reputação + redução de demanda de clientes
- Crise financeira: dispara `OnCriseFinanceira` quando `FinanceManager.CurrentCash < 0` (não quando múltiplas contas estão em atraso, apesar do nome sugerir isso)

**Chave de persistência de horas trabalhadas:** `AFROBARBER_WORKED_MINUTES_{year}_{month}` (int) — rastreia minutos trabalhados por mês para cálculos de contas variáveis.

---

## Biblioteca de Cortes (Education Library)

`Education/EducationProgressManager.cs` — singleton que rastreia descoberta, favoritos e estado "novo/visto" de cada corte. **A base de dados é `ClientRequestDatabase` (`requestDatabase`), não mais uma database separada** — cada `ClientRequestData` já carrega seus próprios campos de "Biblioteca de Cortes" (`cutCategory` : `AfroCutCategory`, `decade`, `historicalSummary`, `fullHistoricalDescription`, `culturalMeaning`, `funFact`), e o corte É o pedido de serviço (`cutId` == `RequestId`).

> Arquitetura antiga (obsoleta): existia uma ScriptableObject `AfroCutDatabase`/`AfroCutInfo` separada (`ScriptableObjects/Services/Education/MainAfroCutDatabase.asset`). Essas classes foram movidas para `Scripts/_Deprecated/AfroCutDatabase.cs` e mantidas **apenas** para o asset legado não quebrar a referência de script — não usar em código novo. `MainAfroCutDatabase.asset` está órfão (nenhum código o referencia mais).

**Fluxo de desbloqueio:** `BarbershopServiceManager` chama `EducationProgressManager.Instance.UnlockCut(request.RequestId)` ao concluir qualquer serviço (ver "Post-service calls"). `UnlockCut` dispara `OnCutUnlocked`, `OnLibraryStateChanged` e mostra `EducationUnlockPopupUI`.

**Componentes:**
| Classe | Responsabilidade |
|---|---|
| `ClientRequestData` | Carrega os campos de cut lore diretamente (`cutCategory`, `decade`, `historicalSummary`, `fullHistoricalDescription`, `culturalMeaning`, `funFact`) além dos campos de serviço (`servicePrice`, `serviceTime`, `xpReward`, `difficulty`, `icon`) |
| `AfroCutCategoryUtils` | Estático — `GetDisplayName(category)`, `BuildDifficultyStars(level)`, `GetDecadeSortValue(decade)` (para ordenação por linha do tempo) |
| `EducationEncyclopediaUI` | Painel "Biblioteca de Cortes" — grade com filtro por categoria, busca por nome, ordenação por década, progresso (`X / Y descobertos`), painel de detalhes com história completa, significado cultural, fun fact, preço/tempo/XP (lidos direto do `ClientRequestData`) e botão de favoritar |
| `EducationEncyclopediaItemUI` | Card da grade — ícone, nome/década (ou "???" se bloqueado), categoria, estrelas de dificuldade, badges de bloqueado/novo/favorito |
| `EducationCategoryFilterButtonUI` | Botão de filtro instanciado dinamicamente (um por `AfroCutCategory` + "Todos"), usa `ToggleGroup` |
| `EducationUnlockPopupUI` | Popup ao desbloquear um novo corte |

> `EducationLibraryNotificationBadge` e `EducationDailyCuriosityUI` foram removidos numa limpeza de código morto — os scripts existiam mas nunca estavam conectados a nenhum Canvas da HUD (zero referência em `GameScene.unity`). `GetNewUnlockedCount()` e `GetDailyFeaturedCut()` em `EducationProgressManager` continuam existindo caso alguém queira reconstruir esses widgets.

**Wiring na cena:** registrar painel como `uiBibliotecaCortes` no `GameUIManager` (ver tabela de UI Panels). `EducationProgressManager` precisa ter apenas `requestDatabase` atribuído no Inspector (não existe mais campo `cutDatabase`).

**Geração automática do painel:** `Assets/Afrobarber/Scripts/Editor/EducationLibraryUIBuilder.cs` adiciona o item de menu `AfroBarber/UI/Construir Biblioteca de Cortes`. Selecione o Canvas principal da HUD na Hierarquia antes de rodar — o tool cria `EducationEncyclopediaItem.prefab` e `EducationCategoryFilterButton.prefab` em `Assets/Afrobarber/Prefabs/UI/Education/`, monta `Panel_BibliotecaCortes` (header, filtros, grade, painel de detalhes) já com `EducationEncyclopediaUI` totalmente referenciado, e registra o painel em `GameUIManager.uiBibliotecaCortes`/`educationEncyclopediaUI`.

---

## Cut Mastery System

`Progression/CutMasterySystem.cs` — singleton que rastreia, **por corte** (`ClientRequestData.RequestId`, os 25 cortes da Biblioteca), o quão bom o jogador está naquele corte específico. Mede progressão via XP por atendimento e 5 tiers genéricos reaproveitáveis em qualquer corte.

**Tiers** (`CutMasteryTier`):

| Tier | Nome | XP base p/ próximo (×`difficulty` 1-5) | Bônus qualidade | Redução penal. tempo | Bônus recompensa |
|---|---|---|---|---|---|
| `Aprendiz` | "Aprendiz da Navalha" | 40 | 0% | 0% | 0% |
| `Habilidoso` | "Estilista de Bairro" | 90 | +2% | +8% | +3% |
| `Especialista` | "Especialista da Cadeira" | 160 | +5% | +15% | +6% |
| `Mestre` | "Mestre do Corte" | 260 | +8% | +22% | +10% |
| `Lenda` | "Lenda do Estilo" | 0 (máx.) | +12% | +30% | +15% |

**Ganho de XP por atendimento** (`RegistrarAtendimento`, baseado em `ServiceFinalRating`):

| Rating | XP de maestria |
|---|---|
| `Horrivel` | 1 |
| `Ruim` | 3 |
| `MaisOuMenos` | 6 |
| `Bom` | 10 |
| `Maravilhoso` | 16 |
| `Perfeito` | 25 |

**Integração com o resultado do serviço** (`AdvancedServiceOutcomeResolver.Resolve`):

- `bonusQualidade` soma diretamente em `finalScore` (×5, igual a outros modificadores de qualidade)
- `reducaoPenalidadeTempo` reduz `effectiveTimePenalty`
- `bonusRecompensa` multiplica `moneyReward` e `xpReward`

Esses bônus são puramente aditivos/multiplicativos sobre os modificadores já existentes (nível do jogador, personalidade do NPC, urgência) — não alteram o comportamento quando não há maestria acumulada.

**Hook de registro:** `BarbershopServiceManager.NotificarSistemasExternos` chama `CutMasterySystem.Instance.RegistrarAtendimento(request.RequestId, rating, request.difficulty)` para todo atendimento concluído (cobre tanto o fluxo avançado quanto o fallback auto-service), e propaga o XP ganho para `DailyChallengeSystem.RegistrarGanhoMaestria(xpGanho)`.

**Evolução de tier:** ao cruzar o limiar de XP, dispara `OnMasteryTierUp(cutId, novoTier)` e, se houver, envia `mensagemEvolucao` (com o nome do corte) via `GlobalDialogueManager.AddSystemMessage`.

**Conquistas relacionadas** (`AchievementSystem`): `maestria_especialista_1`, `maestria_mestre_1`, `maestria_lenda_1`, `maestria_lenda_5`, `maestria_lenda_25` — desbloqueadas via `GetCutsAtTierOrAbove(tier)`.

**Desafio diário relacionado** (`DailyChallengeTipo.GanharMaestria`): "Evolução Contínua" — acumular N pontos de maestria no dia.

**UI:** `EducationEncyclopediaItemUI` (card) e `EducationEncyclopediaUI` (painel de detalhes) exibem o tier atual e progresso de maestria de cada corte desbloqueado.

**Wiring na cena:** adicionar `CutMasterySystem` ao `GameBootstrap` como filho (mesmo padrão de `LoanSystem`/`ClientLoyaltySystem`).

---

## Singleton Managers — Quick Reference

| Class | File | Key method / property |
|---|---|---|
| `GameBootstrap` | `Core/GameBootstrap.cs` | `IsReady`, `Progress01` |
| `GameTimeSystem` | `Core/GameTimeSystem.cs` | `AddMinutes(n)`, `IsWithinBusinessHours`, `CurrentDateTime`, `OnWorkDayStarted` |
| `BarbershopServiceManager` | `Barbershop/BarbershopServiceManager.cs` | `TryStartService(client)`, `HasActiveService`, `CurrentClient` |
| `BarberQueueSystem` | `Queue/BarberQueueSystem.cs` | `AddClientToQueue()`, `TryGetNextWaitingClient(out c)`, `OnQueueChanged` |
| `FinanceManager` | `Economy/FinanceManager.cs` | `AddMoney(amount, reason)`, `SpendMoney(amount, reason)`, `RegisterServiceIncome(request, clientName)`, `AddExpense(title, desc, FinanceMovementOrigin, int amount, DateTime)`, `OnCashChanged` |
| `InventoryManager` | `Inventory/InventoryManager.cs` | `GetUsableItemsByCategory(cat)`, `ConsumeProductUsageByUniqueId(id, n)` |
| `PlayerEnergySystem` | `Energy/PlayerEnergySystem.cs` | `CanStartService()`, `GetServiceTimeMultiplier()`, `OnEnergyChanged` |
| `PlayerXPManager` | `Progression/PlayerXPManager.cs` | `AddXP(n)`, `CurrentLevel`, `OnLevelChanged` |
| `GlobalDialogueManager` | `Dialogue/GlobalDialogueManager.cs` | `AddNpcMessage(identity, text, context)`, `AddSystemMessage(text, context)`, `OnMessageAdded` |
| `EducationProgressManager` | `Education/EducationProgressManager.cs` | `UnlockCut(cutId)`, `GetCutById(cutId)`, `IsUnlocked(cutId)`, `GetDailyFeaturedCut()`, `ToggleFavorite(cutId)`, `IsFavorite(cutId)`, `MarkAsSeen(cutId)`, `GetNewUnlockedCount()`, `OnCutUnlocked`, `OnLibraryStateChanged` |
| `BarbershopRatingManager` | `Evaluation/BarbershopRatingManager.cs` | `AddReview(rating)`, `GlobalRating`, `TotalReviews` |
| `ServiceHistorySystem` | `Reputation/ServiceHistorySystem.cs` | `AddEntry(entry)`, `RemoveEntry(entry)`, `ClearHistory()`, `Entries`, `OnHistoryChanged` — backs `uiHistoricoAtendimento`/`ServiceHistoryUI` |
| `ClientEvaluationSystem` | `Evaluation/ClientEvaluationSystem.cs` | `EvaluateService(sessionData)` |
| `BarbershopUpgradeSystem` | `Barbershop/BarbershopUpgradeSystem.cs` | `TentarComprar(def)`, `PodeComprar(def)`, `MultiplicadorAtracao`, `BonusRating` |
| `GlobalGameplayManagement` | `Gameplay/GlobalGameplayManagement.cs` | `CalculateFinalPriceForRequest(request)`, `CalculateSuggestedPriceForRequest(request)`, `GetPriceSatisfactionScore(finalPrice, suggestedPrice)`, `GetSpawnDemandMultiplierFromLastService(finalPrice, suggestedPrice)`, `GetOverworkEnergyMultiplier()`, `ApplySuggestedScheduleIfNeeded()`, `SetSuggestedBusinessHours(...)`, `SetAdjustmentForService(type, percent)`, `OnBusinessSettingsChanged`, `OnPricesChanged`, `OnScheduleChanged` |
| `CulturalEventSystem` | `Core/CulturalEventSystem.cs` | `AplicarBonusXP(n)`, `AplicarBonusDinheiro(n)`, `TemEventoAtivo`, `EventosAtivos` |
| `TutorialController` | `Core/TutorialController.cs` | `AvancarPasso()`, `PularTutorial()`, `TutorialConcluido` |
| `AchievementSystem` | `Progression/AchievementSystem.cs` | `EstaDesbloqueada(id)`, `TotalDesbloqueadas`, `OnConquistaDesbloqueada` |
| `ClientMoodSystem` | `Characters/Clients/ClientMoodSystem.cs` | `GetHumor(client)`, `GetModificadorRating(client)`, `ClienteEstaIrritado(client)` |
| `BusinessReportManager` | `Economy/BusinessReportManager.cs` | `ClientesHoje`, `ReceitaHoje`, `RatingMedioHoje`, `Historico`, `OnRelatorioGerado` |
| `LoanSystem` | `Economy/LoanSystem.cs` | `TentarPegarEmprestimo(indice)`, `TentarPagarTudo(id)`, `TotalDevido`, `TemEmprestimoAtivo`, `Emprestimos`, `OnEmprestimoTomado`, `OnEmprestimoQuitado` |
| `FinanceMonthlyBillsManager` | `Economy/FinanceMonthlyBillsManager.cs` | `MonthlyDebts`, `OnContaVencida`, `OnContaPaga`, `OnCriseFinanceira` |
| `BarberBookSystem` | `Social/BarberBookSystem.cs` | `PublicarPosAtendimento(nome, requestNome, rating, isVip)`, `Posts`, `PostsNaoLidos`, `ClientesOrganicos`, `OnNovoPost`, `OnPostViral` |
| `VIPClientSystem` | `Gameplay/VIPClientSystem.cs` | `MarcarClienteComoVip(client)`, `IsVip(client)`, `NotificarAtendimentoVip(client, rating, gorjeta)`, `VipAgendadoHoje`, `MultiplicadorPreco`, `PacienciaMinutos` |
| `WeatherSystem` | `Core/WeatherSystem.cs` | `SetClima(tipo)`, `SortearNovoClima()`, `GetEfeito(tipo)`, `ClimaAtual`, `EfeitoAtual`, `OnClimaAlterado` — auto-dirigido, ninguém mais chama `.Instance` (reage a `GameTimeSystem.OnWorkDayStarted` e empurra efeito pra `ClientSpawner`) |
| `PrestigeSystem` | `Gameplay/PrestigeSystem.cs` | `TentarPrestigiar(perk)`, `PodePrestigiar`, `NivelPrestigio`, `TemPerk(perk)`, `GetBonusXP()`, `GetBonusGorjeta()`, `GetFatorAluguel()` |
| `DailyChallengeSystem` | `Gameplay/DailyChallengeSystem.cs` | `RegistrarAtendimento(rating, valorRecebido)`, `ColetarRecompensa(desafioId)`, `DesafiosHoje`, `Leaderboard`, `PontuacaoHoje`, `OnDesafioAtualizado`, `OnDiaRenovado` |
| `NarrativeMissionSystem` | `Missions/NarrativeMissionSystem.cs` | `RegistrarAtendimentoNarrativo(characterId, clienteNome, rating)`, `GetCapituloAtivo(characterId)`, `GetProgress(characterId)`, `OnCapituloConcluido`, `OnPersonagemCompleto` |
| `MissionSystem` | `Missions/MissionSystem.cs` | `RegisterServiceCompleted(...)`, `CanClaimTier(mission)`, `ClaimCurrentTier(mission)`, `GetMissionCurrentValue(mission)`, `GetCurrentTierProgress01(mission)`, `Missions`, `History`, `Stats`, `OnMissionDataChanged` — sistema de missões por marcos/tiers, separado do `NarrativeMissionSystem`; backs `uiMissoes`/`MissionsPanelUI` |
| `ClientLoyaltySystem` | `Progression/ClientLoyaltySystem.cs` | `GetTier(clientId)`, `GetMultiplicadorGorjeta(clientId)`, `GetBonusPacienciaMinutos(clientId)`, `DeveElogiarEspontaneamente(clientId)`, `OnClienteSubiuTier` |
| `CityWaypointSystem` | `City/CityWaypointSystem.cs` | `GetRandomWaypoint(exclude)`, `WaypointCount` |
| `CutMasterySystem` | `Progression/CutMasterySystem.cs` | `RegistrarAtendimento(cutId, rating, difficulty)`, `GetTier(cutId)`, `GetTierConfig(tier)`, `GetTierConfigForCut(cutId)`, `GetXPProgressNormalized(cutId)`, `GetCutsAtTierOrAbove(tier)`, `OnMasteryTierUp`, `OnMasteryXPGained` |
| `PlayerNicknameManager` | `Core/PlayerNicknameManager.cs` | `Nickname`, `TemNicknamePersonalizado`, `SetNickname(nome)`, `OnNicknameChanged` |

> **Single finance system**: `FinanceManager` is canonical (history + cash, persisted) and the only one — there is no `PlayerWallet` anymore (it was removed; the advanced workflow calls `FinanceManager.Instance.AddCashIncome(...)` directly). Do not add a second/third financial system.
>
> **Pricing**: `GlobalGameplayManagement` is the live pricing/business-hours system (`ClientRequestData.ServicePrice` calls `GlobalGameplayManagement.Instance.CalculateFinalPriceForRequest(this)` directly). There used to be a second, competing `DynamicPricingManager` singleton whose `AplicarMargem()` was never actually called from anywhere — it was removed as dead code. Don't reintroduce a second pricing manager; extend `GlobalGameplayManagement` instead.

---

## Key Enums

**`ServiceActionType`** (all steps in advanced workflow):
`Wash, Comb, Cut, Razor, Finish, Define, Beard, Finalize`

**`ProductCategory`**:
`MaquinaDeCorte, Tesoura, Pente, Navalha, Secador, ProdutoCapilar, Laminas, Decoracao, Mobilia`

**`ServiceType`**:
`CorteDeCabelo, CorteDeCabeloEBarba, Barba, AcabamentoPezinho, DesignDeSobrancelhas, Hidratacao, Outro`

**`DialogueContextType`**:
`None, Queue, Service, City, Tutorial, CulturalEvent, Farewell, Complaint, Friendship`

**`NPCPersonality`**:
`Friendly, Irritated, Calm, Demanding, Communicative, Annoying, Shy, Playful`

**`ServiceFinalRating`**:
`Horrivel, Ruim, MaisOuMenos, Bom, Maravilhoso, Perfeito`

**`InventoryItemType`**:
`Duravel, Consumivel, CaixaComUnidades`

**`LoanStatus`**:
`Ativo, Quitado, EmAtraso`

**`PrestigePerk`**:
`BonusXP10, GorjetaExtra20, ReputacaoInicial, AlugueMenor15, DinheiroInicial500`

**`DailyChallengeTipo`**:
`AtenderNClientes, FaturarNReais, FazerNPerfeitos, SemServicoRuim, AtenderConsecutivos, GanharMaestria`

**`LoyaltyTier`**:
`Novo, Conhecido, Regular, Fiel, Lendario`

**`CutMasteryTier`**:
`Aprendiz, Habilidoso, Especialista, Mestre, Lenda`

**`AfroCutCategory`**:
`BlackPower, Fade, Braids, Dreads, Twists, FlatTop, AfroClassic, Contemporary, Traditional, Other`

**Player levels** (`PlayerXPManager` — XP para avançar ao próximo nível):
1 → "Aprendiz da Navalha" (500 XP) · 2 → "Barbeiro de Bairro" (1500) · 3 → "Profissional da Cadeira" (3500) · 4 → "Mestre do Degradê" (7000) · 5 → "Lenda AfroBarber" (max)

---

## ScriptableObject Data Layer

Add new services, products, and hair styles **without code changes** by creating and registering assets.

| ScriptableObject | `[CreateAssetMenu]` path | Key fields |
|---|---|---|
| `ProductData` | `AfroBarber/Product` | `productId`, `category`, `inventoryItemType`, `precisao`, `velocidade`, `durabilidade`, `preco` |
| `ClientRequestData` | `AfroBarber/Client/Request Data` | `id`/`requestId` (cut identity is this ID — there's no separate `afroCutId`), `serviceType`, `servicePrice`, `serviceTime`, `xpReward`, `requiredItems`, `beforeHairId`, `afterHairId`, `cutCategory`, `decade`, `historicalSummary`, `fullHistoricalDescription`, `culturalMeaning`, `funFact` |
| `ServiceRequirementData` | (inline in request) | `requirementId`, `category`, `usageType`, `amountConsumed`, `hoursConsumed` |

Asset naming convention: `Prefix_Name.asset` — e.g., `Request_BlackPower.asset`, `DB_Tesouras.asset`, `Product_MaquinaXYZ.asset`

Asset locations:
- Requests: `ScriptableObjects/Services/Requests/`
- Product databases: `ScriptableObjects/Products/Databases/`
- Master request DB: `ScriptableObjects/Services/Databases/MainClientRequestDatabase.asset`

---

## PlayerPrefs Keys

Never reuse these keys for new data:

| Key | Owner | Type |
|---|---|---|
| `AFROBARBER_FINANCE_CURRENT_CASH` | FinanceManager | int |
| `AFROBARBER_FINANCE_HISTORY` | FinanceManager | JSON |
| `AFROBARBER_FINANCE_MIGRATION_V2_DONE` | FinanceManager | flag |
| `AFROBARBER_PLAYER_MONEY` | FinanceManager (legacy migration) | int |
| `AFROBARBER_CASH_REGISTER_MONEY` | FinanceManager (legacy migration) | int |
| `AFROBARBER_INVENTORY_ITEMS` | InventoryManager | JSON |
| `AFROBARBER_PLAYER_XP` | PlayerXPManager | int |
| `AFROBARBER_PLAYER_LEVEL` | PlayerXPManager | int |
| `AFROBARBER_UNLOCKED_CUTS` | EducationProgressManager | pipe-delimited string |
| `AFROBARBER_EDUCATION_FAVORITES` | EducationProgressManager | pipe-delimited string |
| `AFROBARBER_EDUCATION_SEEN` | EducationProgressManager | pipe-delimited string |
| `AFROBARBER_GLOBAL_RATING` / `AFROBARBER_ATTENDANCE_RATING` / `AFROBARBER_STRUCTURE_RATING` / `AFROBARBER_EXPERIENCE_RATING` / `AFROBARBER_TOTAL_REVIEWS` | BarbershopRatingManager | float/int |
| `AFROBARBER_TUTORIAL_DONE` | TutorialController | flag |
| `AFROBARBER_UPGRADES_DATA` | BarbershopUpgradeSystem | JSON |
| `AFROBARBER_LOANS_DATA` | LoanSystem | JSON |
| `AFROBARBER_BARBERBOOK_DATA` | BarberBookSystem | JSON |
| `AFROBARBER_PRESTIGE_DATA` | PrestigeSystem | JSON |
| `AFROBARBER_DAILY_CHALLENGES` | DailyChallengeSystem | JSON |
| `AFROBARBER_VIP_DIAS_SEM_VIP` | VIPClientSystem | int |
| `AFROBARBER_LOYALTY_DATA` | ClientLoyaltySystem | JSON |
| `AFROBARBER_NARRATIVE_PROGRESS` | NarrativeMissionSystem | JSON |
| `AFROBARBER_ACHIEVEMENTS` | AchievementSystem | JSON |
| `AFROBARBER_BUSINESS_REPORTS` | BusinessReportManager | JSON |
| `AFROBARBER_WORKED_MINUTES_{year}_{month}` | FinanceMonthlyBillsManager | int |
| `AFROBARBER_CUT_MASTERY_DATA` | CutMasterySystem | JSON |
| `AFROBARBER_PLAYER_NICKNAME` | PlayerNicknameManager | string |

---

## UI Panels — GameUIManager

`UI/PlayerPanel/GameUIManager.cs` — singleton central de visibilidade de painéis. Mantém `currentOpenUI` e garante que apenas um painel exclusivo fique aberto por vez.

**Painéis registrados:**

| Campo | Método de abertura | Refresh chamado ao abrir |
|---|---|---|
| `uiFinanceiro` | `OpenFinanceiro()` | `FinanceUIController.RefreshUI()` |
| `uiGestao` | `OpenGestao()` | — |
| `uiInventario` | `OpenInventario()` | `InventoryUIManager.RefreshUI()` |
| `uiLoja` | `OpenLoja()` | `ShopManager.RefreshShopUI()` |
| `uiAtendimento` | `OpenAtendimento()` | — |
| `uiFila` | `OpenFila()` | `QueueUIManager.RefreshQueueUI()` |
| `uiAgenda` | `OpenAgenda()` | `AppointmentPanelUI.Refresh()` |
| `uiHistoricoAtendimento` | `OpenHistoricoAtendimento()` | `ServiceHistoryUI.RefreshHistoryUI()` |
| `uiAvaliacaoAtendimento` | `OpenAvaliacaoAtendimento()` | — |
| `uiReputacaoDetalhada` | `OpenReputacaoDetalhada()` | — |
| `uiMissoes` | `OpenMissoes()` | `MissionsPanelUI.Refresh()` |
| `uiBarberBook` | `OpenBarberBook()` | `BarberBookPanelUI.RefreshFeed()` |
| `uiEmprestimos` | `OpenEmprestimos()` | `LoanPanelUI.RefreshUI()` |
| `uiDesafiosDiarios` | `OpenDesafiosDiarios()` | `DailyChallengePanelUI.RefreshAll()` |
| `uiBibliotecaCortes` | `OpenBibliotecaCortes()` | `EducationEncyclopediaUI.RefreshList()` |

**Padrão de subscrição de eventos em painéis UI:**
- Use `bool eventosSuscritos` + método `TrySubscreverEventos()` chamado em `Start()`, `OnEnable()`, `Show()` e qualquer método de refresh — garante subscrição independente da ordem de inicialização
- Use `DessubscreverEventos()` com métodos nomeados (nunca lambdas) em `OnDisable()` — lambdas criam novas instâncias e `RemoveListener` falha silenciosamente

---

## Naming Conventions

- Scripts: `PascalCase.cs`
- Assets: `Prefix_Name.asset` / `Prefix_Name.prefab`
- Scene transform anchor points: `Point_ClientSpawn`, `Point_Entrance`, `Point_BarberChair_Walk`, `Point_BarberChair_Sit`, `Point_Cashier`, `Point_Exit`
- Waiting seat children: `ApproachPoint` (NavMesh floor), `SitPoint` (body snap)
- Animator parameters: `Speed` (float), `Sit` (bool)
- Player GameObject tag: `Player`

---

## Estado do Repositório / Débito Técnico Conhecido

- **VCS**: o projeto usa Plastic SCM (`.plastic/`) como controle de versão principal. Um repositório Git separado dentro de `Assets/Afrobarber/Scripts/.git` (espelho manual apontando para `github.com/marcelo-braga-dev/afrobarber`) existiu em algum momento, excluído do Plastic via `ignore.conf`, mas não está mais presente no checkout atual — se for recriado, trate-o como espelho manual, não como histórico confiável.
- **Testes**: `Assets/Tests/` não tinha nenhum teste até esta rodada. Um scaffold mínimo (`Assets/Tests/EditMode/`) foi criado com 2 testes de exemplo cobrindo lógica estática pura — está longe de cobertura completa.
- **Padrão `eventosSuscritos`**: seguido por poucos arquivos de UI (ver seção "UI Panels — GameUIManager"). O bug confirmado em `LoanPanelUI` (unsubscribe por lambda) foi corrigido; outros arquivos que se inscrevem em eventos sem esse padrão ainda não foram auditados/corrigidos — candidatos a regressão silenciosa de UI (inscrição duplicada a cada `OnEnable`).
- **HDRP/URP em mobile**: `Core/Mobile/MobilePerformanceBootstrap.cs` troca o quality level em Android/iOS para um perfil cujo Render Pipeline Asset é URP, enquanto a esmagadora maioria dos materiais do projeto (`Assets/Afrobarber/Materials/`) foi autorada só com shader `HDRP/Lit`. Isso é um risco real de material quebrado/rosa em build mobile — **validar em device/build real antes de shippar mobile**; se confirmado, ou se cria um set de materiais `URP/Lit` equivalente, ou se para de trocar de pipeline em mobile.
- **`BarbershopCashRegister` foi removido**: era inicializado pelo `GameBootstrap` e assinava `FinanceManager.OnCashChanged` de verdade, mas seu próprio evento de saída (`OnMoneyChanged`) não tinha nenhum consumidor — nem chamada em C#, nem listener no Inspector. Removido do código, do `GameBootstrap` e do GameObject `BarbershopManager` na cena (que mantém seus outros componentes, como `BarbershopRatingManager`, intactos). Quem alimenta a UI de dinheiro de verdade é `UI/Shop/ShopCashHeaderUI.cs` (confirmado conectado na cena) — **não** `Economy/MoneyTextBinder.cs`, que também acabou sendo removido na rodada seguinte de limpeza por estar igualmente órfão (corrige uma nota anterior deste arquivo que apontava `MoneyTextBinder` como o binder ativo).
- **Limpeza de código morto (rodada de GUID cruzado com cenas/prefabs)**: removidos 22 arquivos com zero referência em `GameScene.unity`/`MainMenuScene.unity`/qualquer `.prefab`/`.asset` — `UI/ClockAdvancePanelUI.cs`, `UI/AnalogClockUI.cs` (duplicavam `UI/HUD/GameTimeUI.cs`, esse sim ativo), `Economy/MoneyTextBinder.cs`, `Barbershop/BarberChair/BarberChairSeat.cs`, `Characters/Clients/ClientHairDefinition.cs`, `Characters/Clients/HairAnchorBinder.cs`, `Education/CutEducationPreviewUI.cs`, `Services/InventoryDiscardBrokenButton.cs`, `Transito/ModelAligner.cs`, `UI/Education/EducationDailyCuriosityUI.cs`, `UI/Education/EducationLibraryNotificationBadge.cs`, `UI/Management/BarbershopManagementStatusHUD.cs`, `UI/Management/OpenManagementButton.cs`, `UI/Inventory/InventoryToggleUI.cs`, `UI/Missions/MissionOverviewStatsUI.cs`, `Services/DefaultServiceLibrary.cs`, `Services/ServiceLoadoutBuilder.cs` (auto-loadout antigo, superado pelo fluxo atual via `ServicePlanningUI` + `client.PreparedLoadout`), e 5 scripts de debug não anexados em `Utilities/Debug/`. Se algum comportamento sumir depois disso, comece a investigação por essa lista.
- **`GlobalReputationSystem` foi removido (consolidação de reputação)**: era um segundo tracker de reputação 0-5 independente do `BarbershopRatingManager` (o canônico, usado por ~12 sistemas), alimentado só por `ServiceHistorySystem` e consumido só pelo `ClientSpawner` pra calcular demanda. Como ele começava em `0` (e a fórmula de demanda assume nota `3` como neutro), toda barbearia nova começava com demanda artificialmente baixa até a primeira review — bug que a consolidação corrigiu de brinde. `ClientSpawner` agora assina `BarbershopRatingManager.Instance.OnRatingChanged` diretamente.
- **Bug corrigido — `ClientSpawner.demandMultiplierFromPrice` compartilhado por dois sistemas**: `FinanceMonthlyBillsManager` (penalidade de conta atrasada) e a satisfação de preço do último atendimento escreviam no mesmo campo, um apagando o outro. Agora são campos separados (`demandMultiplierFromPrice` e `demandMultiplierFromLateFees`), multiplicados juntos em `RecalcularDemanda()`. O método usado pelo `FinanceMonthlyBillsManager` foi renomeado de `SetDemandMultiplierFromPricing` para `SetDemandMultiplierFromLateFees`.
- **Bug corrigido — agendamentos nunca eram marcados `Missed`**: `ClientAppointmentScheduler.ProcessAppointments` tentava ativar um agendamento a cada segundo indefinidamente; se o cliente nunca ficasse disponível, o agendamento ficava preso pra sempre em `WaitingToSpawn`. Agora, passado `missedAfterMinutes` (60min de jogo, configurável) do horário marcado sem spawnar, o agendamento é marcado `Missed` e removido da lista.
- **`AchievementSystem` está totalmente implementado mas sem nenhuma UI consumidora**: `OnConquistaDesbloqueada` não tem listener nenhum (nem C#, nem Inspector) e não existe nenhum arquivo em `UI/` mencionando conquistas — a feature roda escondida, sem popup nem tela de lista. Não corrigido nesta rodada (é trabalho de construir UI nova, não limpeza).
- **`CulturalEventSystem.AplicarBonusXP()`/`AplicarBonusDinheiro()` nunca são chamados em lugar nenhum**: eventos culturais ativam e calculam os bônus, mas nada em `FinanceManager`/`PlayerXPManager`/no fluxo de conclusão de serviço consulta esses valores — eventos culturais hoje não têm efeito real de gameplay. Decisão consciente de não conectar nesta rodada (é mudança de comportamento de maior escopo); registrado aqui pra decisão futura.
- **Eventos "fire but nobody's home" (documentados, não corrigidos)**: `BarbershopServiceManager.OnAtendimentoConcluido`, os três eventos do `NarrativeMissionSystem` (`OnCapituloDesbloqueado`/`OnCapituloConcluido`/`OnPersonagemCompleto`), `BusinessReportManager.OnRelatorioGerado`, `GlobalGameplayManagement.OnPricesChanged`/`OnScheduleChanged` são disparados mas não têm nenhum assinante hoje. Deixados como estão — são pontos de extensão baratos que uma UI futura pode consumir, não atrapalham nada funcionando vazios.
- **Investigação incompleta por limite de sessão da API**: uma varredura profunda de bugs em `ClientNPC.cs`, `BarbershopServiceManager.cs` (fluxo de recompensa/XP/rating) e no sistema de diálogo, e uma triagem de severidade dos ~21 arquivos de UI que se inscrevem em eventos sem o padrão `eventosSuscritos` (distinguindo "lambda sem unsubscribe = vazamento real" de "método nomeado sem guard = duplicação leve"), foram iniciadas mas **não terminaram** — não foram feitas, não "não encontraram nada". Candidatas a uma próxima rodada de investigação.

---

## Before Editing Any System

1. Check if a singleton manager already owns that domain — never add a second instance of any manager to a scene
2. Managers with `.Instance` must be present in the scene hierarchy before anything calls them
3. `FindFirstObjectByType<T>()` calls exist as fallbacks — prefer Inspector references for production code
4. After any service flow change, test the full 22-step client loop (spawn → seat → interact → accept → chair → plan → execute → cashier → exit → spawner frees slot)
