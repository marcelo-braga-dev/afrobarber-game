# AfroBarber v1

Simulação de barbearia afro-brasileira desenvolvida em Unity, combinando gestão de atendimento, progressão de negócio e educação cultural centrada na estética afro e na identidade negra.

## Tecnologias

- **Unity 6000.3.10**
- **HDRP 17.3.0** / **URP 17.3.0** (coexistem — use HDRP Lit ou URP Lit, nunca Standard)
- **New Input System 1.18.0** (nunca `Input.GetKey()` legado)
- **Unity Test Framework 1.6.0**
- **Plastic SCM** (controle de versão principal)

## Arquitetura Central

Os sistemas se comunicam via eventos C# (`UnityEvent`/`Action`) e singletons com `.Instance`. O `GameBootstrap` (`Core/GameBootstrap.cs`) orquestra a inicialização em `Start()`, ativando managers em sequência antes de liberar a UI. Todos os managers sobrevivem a troca de cena via `PersistentGameObject.MakePersistent`.

### Ciclo completo de atendimento

```text
ClientSpawner → ClientNPC (Wandering → GoingToBarbershop → GoingToEntrance
→ GoingToWaitingPoint → WaitingForService → GoingToBarberChair → InService
→ GoingToCashier → ReturningToCity)
```

O fluxo de serviço tem dois caminhos controlados por `useAdvancedServiceWorkflow`:
- **Avançado** (`Services/Advanced/`): planejamento → execução passo a passo → resolução de resultado
- **Fallback** (timer simples): `autoCompleteServiceByTime = true`

## Estrutura dos Scripts

```text
Assets/Afrobarber/Scripts/
├── Appointment/        # Agendamentos multi-dia com detecção de Missed
├── Barbershop/         # ServiceManager, RatingManager, UpgradeSystem, WaitingArea
├── Characters/         # ClientNPC (state machine), ClientSpawner, ClientMoodSystem
├── City/               # CityWaypointSystem para navegação de NPCs
├── Core/               # Bootstrap, GameTimeSystem, WeatherSystem, TutorialController
├── Dialogue/           # GlobalDialogueManager, NPCConversationBrain, NPCDialogueMemory
├── Economy/            # FinanceManager, LoanSystem, FinanceMonthlyBillsManager
├── Education/          # Biblioteca de 25 cortes afro com lore completo
├── Energy/             # PlayerEnergySystem (fadiga afeta qualidade)
├── Evaluation/         # ClientEvaluationSystem, BarbershopRatingManager
├── Gameplay/           # VIPClientSystem, PrestigeSystem, DailyChallengeSystem
├── Inventory/          # InventoryManager (produtos duráveis, consumíveis, caixas)
├── Missions/           # NarrativeMissionSystem (arcos) + MissionSystem (marcos/tiers)
├── Progression/        # PlayerXPManager, ClientLoyaltySystem, CutMasterySystem
├── Queue/              # BarberQueueSystem
├── Reputation/         # ServiceHistorySystem
├── Services/           # Fluxo de serviço básico e avançado, compatibilidade de ferramentas
├── Social/             # BarberBookSystem (rede social fictícia pós-atendimento)
├── UI/                 # 16 painéis gerenciados por GameUIManager
└── Utilities/          # Debug helpers (ContextMenu, sem Input legado)
```

## Sistemas Principais

| Sistema | Arquivo | Responsabilidade |
|---|---|---|
| `BarbershopServiceManager` | `Barbershop/` | Orquestra o fluxo completo de atendimento |
| `ClientNPC` | `Characters/Clients/` | Máquina de estados dos clientes (10 estados) |
| `FinanceManager` | `Economy/` | Sistema financeiro canônico (único). `AddCashIncome` no fluxo avançado; `RegisterServiceIncome` no fallback |
| `PlayerXPManager` | `Progression/` | XP e 5 níveis de progressão do jogador |
| `EducationProgressManager` | `Education/` | Biblioteca de 25 cortes afro: desbloqueio, favoritos, lore |
| `CutMasterySystem` | `Progression/` | Maestria por corte (5 tiers × 25 cortes, bônus de qualidade/recompensa) |
| `ClientLoyaltySystem` | `Progression/` | Fidelidade de clientes (5 tiers, bônus de gorjeta e paciência) |
| `DailyChallengeSystem` | `Gameplay/` | Desafios diários (tempo real) com ranking fictício |
| `NarrativeMissionSystem` | `Missions/` | Arcos narrativos com 5 personagens fixos, 3 capítulos cada |
| `MissionSystem` | `Missions/` | Missões por marcos/tiers (só registra no fluxo fallback — débito técnico) |
| `BarberBookSystem` | `Social/` | Rede social simulada pós-atendimento, posts virais |
| `LoanSystem` | `Economy/` | Empréstimos com juros compostos semanais (4 opções) |
| `PrestigeSystem` | `Gameplay/` | New Game+ com 5 perks permanentes |
| `VIPClientSystem` | `Gameplay/` | Clientes VIP com preço 3× e gorjeta 2.5× |
| `WeatherSystem` | `Core/` | Clima auto-dirigido que afeta demanda e gorjetas |
| `ClientAppointmentScheduler` | `Appointment/` | Agendamentos multi-dia com detecção automática de Missed |
| `GameTimeSystem` | `Core/` | Tempo de jogo, dias úteis, eventos de dia/semana |
| `GlobalGameplayManagement` | `Gameplay/` | Precificação dinâmica, horários comerciais, ajustes de serviço |

## Configuração Crítica na Cena

Para o jogo funcionar corretamente, a cena precisa ter:

1. **GameBootstrap** com os 5 managers serializados: `globalDialogueManager`, `barbershopRatingManager`, `financeManager`, `barberQueueSystem`, `appointmentScheduler`
2. **NavMesh baked** com pontos de spawn, entrada, cadeira e saída dentro do mesh
3. **WaitingAreaManager** com cadeiras tendo filhos `ApproachPoint` (NavMesh) e `SitPoint` (snap exato)
4. **ClientSpawner** com `cityResidents[]`, `playerTransform`, `waitingAreaManager`, `entrancePoint`, `exitPoint`
5. **ServicePriceTable** asset referenciado em `GlobalGameplayManagement.globalPriceTable`
6. **MainClientRequestDatabase** com os 25 `ClientRequestData` da Biblioteca de Cortes

## Convenções

- Scripts: `PascalCase.cs` | Assets: `Prefix_Name.asset`
- Código, comentários e strings de UI: **português**
- Nomes de arquivo e classes: **inglês** (ex.: `CutMasterySystem`, não `SistemaDeMaestria`)
- Input: **New Input System** apenas — proibido `Input.GetKey()` / API legada
- Materials: **HDRP Lit** ou **URP Lit** — nunca Standard

## Débitos Técnicos Conhecidos

- `MissionSystem.RegisterServiceCompleted` não é chamado no fluxo avançado (só no fallback)
- `PrestigeSystem.BonusXP10` sem efeito: `PlayerXPManager.AddXP` não multiplica por `GetBonusXP()`
- `CulturalEventSystem` não tem efeito real de gameplay (bônus calculados, nunca aplicados)
- `AchievementSystem` sem UI consumidora (funciona internamente, sem popup/tela)
- HDRP/URP em mobile: materiais HDRP ficam rosa em build URP — validar em device antes de shippar

## Documentação Técnica

Consulte [CLAUDE.md](CLAUDE.md) para documentação completa: APIs de todos os sistemas, enums, PlayerPrefs keys, fluxo de serviço, wiring de cena e débito técnico detalhado.
