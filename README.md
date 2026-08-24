# AfroBarber v1

Simulação de barbearia afro-brasileira desenvolvida em Unity, com foco em atendimento, gestão, cultura negra, estética afro, educação sobre cortes, progressão do jogador, loja, inventário, sistema financeiro, fila de clientes, energia, reputação e diálogos contextuais.

---

## Visão do Projeto

AfroBarber Game é um jogo de simulação em terceira pessoa ambientado em uma barbearia com estética afro, cultura negra, identidade urbana e elementos do universo hip-hop, da ancestralidade e da estética afro-brasileira/afro-americana. O jogador assume o papel de barbeiro e gestor.

**Três pilares principais:**

**1. Simulação de barbearia** — o jogador gerencia uma barbearia funcional, recebe clientes, usa ferramentas e produtos, recebe pagamentos e compra itens para melhorar a qualidade dos atendimentos.

**2. Gameplay de gestão** — cada cliente tem pedido, requisitos de produtos, tempo estimado, recompensa e vínculos com conteúdo educativo. O fluxo envolve entrada, espera, fila, cadeira, caixa e saída.

**3. Educação e valorização cultural** — o jogo apresenta estilos de cabelo afro, suas histórias, significados culturais e relação com identidade, estética, comunidade e expressão pessoal.

---

## Loop de Gameplay

```text
1.  Cliente nasce no ponto de spawn
2.  Cliente entra na barbearia (EntrancePoint)
3.  Sistema reserva um assento disponível (WaitingAreaManager)
4.  Cliente caminha até o ApproachPoint do assento
5.  Cliente snapa no SitPoint e senta (agent desativado durante snap)
6.  Ícone de interação aparece sobre o cliente
7.  Jogador clica → ClientRequestUI.Show() exibe pedido
8.  Jogador aceita → CallForService() → BarbershopServiceManager.TryStartService()
9.  Sistema valida: energia, ferramentas, ponto da cadeira
10. Cliente libera assento e vai até BarberChairWalkPoint
11. Cliente snapa no BarberChairSitPoint (agent desativado)
12. ServicePlanningUI abre (se useAdvancedServiceWorkflow = true)
13. Jogador ordena/confirma etapas do atendimento
14. Sistema executa etapas (Wash → Cut → Finish → Finalize)
15. AdvancedServiceOutcomeResolver calcula resultado
16. Jogador recebe dinheiro, XP e avaliação
17. Cabelo final aplicado visualmente (ClientHairVisualController)
18. Conteúdo educativo desbloqueado (EducationProgressManager)
19. Produtos/ferramentas consumidos (InventoryManager)
20. Cliente vai ao caixa (CashierPoint)
21. Cliente sai da barbearia (ExitPoint)
22. ClientSpawner libera o slot para novo cliente
```

---

## Progressão do Jogador

- Dinheiro acumulado para comprar produtos e ferramentas melhores
- XP → 5 níveis (Aprendiz → Barbeiro de Bairro → Profissional → Mestre do Degradê → Lenda AfroBarber)
- Melhoria de reputação (afeta fluxo de clientes)
- Desbloqueio de 25 cortes na Biblioteca Educativa
- Maestria por corte (5 tiers, bônus de qualidade e recompensa)
- Prestige System (New Game+ com perks permanentes após nível 5)

## Decisões de Gestão

- Quais produtos comprar e manter em estoque
- Quando aceitar ou dispensar clientes (paciência limitada)
- Priorizar clientes VIP (preço 3×, paciência reduzida)
- Equilibrar qualidade, velocidade e lucro
- Controlar energia do barbeiro (cansaço reduz qualidade)
- Gerenciar empréstimos e contas mensais

---

## Tecnologias

| | |
|---|---|
| Engine | Unity 6000.3.10 |
| Render | HDRP 17.3.0 / URP 17.3.0 (coexistem — use Lit, nunca Standard) |
| Input | New Input System 1.18.0 (proibido `Input.GetKey()`) |
| Testes | Unity Test Framework 1.6.0 |
| VCS | Plastic SCM (principal) + Git (documentação) |
| UI | TextMeshPro |
| Navegação | NavMeshAgent |
| Animação | Animator (parâmetros: `Speed` float, `Sit` bool) |

### Dependências obrigatórias no Unity

Antes de configurar o projeto em outro ambiente:

- TextMeshPro importado (`Window > TextMeshPro > Import TMP Essential Resources`)
- Canvas + EventSystem na cena
- NavMesh baked no chão (obstáculos marcados corretamente)
- Tag `Player` no GameObject do jogador
- Unity 6000.3.10 (não fazer upgrade)

---

## Arquitetura Central

Sistemas comunicam via eventos C# (`UnityEvent`/`Action`) e singletons com `.Instance`. O `GameBootstrap` (`Core/GameBootstrap.cs`) orquestra a inicialização em `Start()`, ativando managers em sequência antes de liberar a UI. Todos os managers sobrevivem à troca de cena via `PersistentGameObject.MakePersistent`.

```text
ClientSpawner → ClientNPC (Wandering → GoingToBarbershop → GoingToEntrance
→ GoingToWaitingPoint → WaitingForService → GoingToBarberChair → InService
→ GoingToCashier → ReturningToCity)
```

O fluxo de serviço tem dois caminhos (controlados por `useAdvancedServiceWorkflow`):
- **Avançado** (`Services/Advanced/`): planejamento → execução passo a passo → resolução de resultado
- **Fallback** (timer simples): `autoCompleteServiceByTime = true`

---

## Estrutura dos Scripts

```text
Assets/Afrobarber/Scripts/
├── Appointment/        # Agendamentos multi-dia com detecção de Missed
├── Barbershop/         # ServiceManager, RatingManager, UpgradeSystem
│   └── Waiting/        # WaitingAreaManager, SofaSeatGroup, WaitingSeat
├── Characters/         # ClientNPC (state machine), ClientSpawner, ClientMoodSystem
│   └── Clients/        # HairVisualController, ClientServiceProfile
├── City/               # CityWaypointSystem para navegação de NPCs
├── Core/               # Bootstrap, GameTimeSystem, WeatherSystem, TutorialController
├── Dialogue/           # GlobalDialogueManager, NPCConversationBrain
├── Economy/            # FinanceManager, LoanSystem, FinanceMonthlyBillsManager
├── Education/          # Biblioteca de 25 cortes afro com lore completo
├── Energy/             # PlayerEnergySystem, BarberWorkController
├── Evaluation/         # ClientEvaluationSystem, BarbershopRatingManager
├── Gameplay/           # VIPClientSystem, PrestigeSystem, DailyChallengeSystem
├── Inventory/          # InventoryManager (duráveis, consumíveis, caixas)
├── Missions/           # NarrativeMissionSystem + MissionSystem
├── NPC/                # NPCIdentity, NPCRelationshipMemory, NPCSocialProfile
├── Progression/        # PlayerXPManager, ClientLoyaltySystem, CutMasterySystem
├── Queue/              # BarberQueueSystem, ClientQueueData
├── Reputation/         # ServiceHistorySystem
├── Services/           # Fluxo básico e avançado, ServiceToolCompatibility
├── Social/             # BarberBookSystem (rede social fictícia)
├── UI/                 # 16 painéis gerenciados por GameUIManager
├── Utilities/          # Debug helpers (ContextMenu, sem Input legado)
└── _Deprecated/        # SofaSeats.cs, ShopPurchaseHandler.cs — não referenciar
```

---

## Sistemas Principais

| Sistema | Arquivo | Responsabilidade |
|---|---|---|
| `BarbershopServiceManager` | `Barbershop/` | Orquestra o fluxo completo de atendimento |
| `ClientNPC` | `Characters/Clients/` | Máquina de estados dos clientes (10 estados) |
| `FinanceManager` | `Economy/` | Sistema financeiro canônico. Fluxo avançado usa `AddCashIncome`; fallback usa `RegisterServiceIncome` |
| `PlayerXPManager` | `Progression/` | XP e 5 níveis de progressão do jogador |
| `EducationProgressManager` | `Education/` | Biblioteca de 25 cortes afro: desbloqueio, favoritos, lore |
| `CutMasterySystem` | `Progression/` | Maestria por corte (5 tiers × 25 cortes, bônus real de qualidade/recompensa) |
| `ClientLoyaltySystem` | `Progression/` | Fidelidade de clientes (5 tiers, bônus de gorjeta e paciência) |
| `DailyChallengeSystem` | `Gameplay/` | Desafios diários (tempo real) com ranking fictício |
| `NarrativeMissionSystem` | `Missions/` | Arcos narrativos com 5 personagens fixos, 3 capítulos cada |
| `MissionSystem` | `Missions/` | Missões por marcos/tiers (⚠ só registra no fluxo fallback) |
| `BarberBookSystem` | `Social/` | Rede social simulada pós-atendimento, posts virais |
| `LoanSystem` | `Economy/` | Empréstimos com juros compostos semanais (4 opções) |
| `PrestigeSystem` | `Gameplay/` | New Game+ com 5 perks permanentes |
| `VIPClientSystem` | `Gameplay/` | Clientes VIP com preço 3× e gorjeta 2.5× |
| `WeatherSystem` | `Core/` | Clima auto-dirigido que afeta demanda e gorjetas |
| `ClientAppointmentScheduler` | `Appointment/` | Agendamentos multi-dia com detecção de Missed |
| `GameTimeSystem` | `Core/` | Tempo de jogo, dias úteis, eventos de dia/semana |
| `GlobalGameplayManagement` | `Gameplay/` | Precificação dinâmica, horários comerciais, ajustes |

---

## Configuração Crítica na Cena

Para o jogo funcionar corretamente, a cena precisa ter:

**Pontos obrigatórios (objetos vazios):**

```text
Point_ClientSpawn
Point_Entrance
Point_BarberChair_Walk     ← acessível pelo NavMesh
Point_BarberChair_Sit      ← posição exata do corpo sentado
Point_Cashier
Point_Exit
Point_PlanningInteraction
```

**Área de espera (hierarquia recomendada):**

```text
WaitingArea
└── Sofa_01
    ├── Seat_01
    │   ├── ApproachPoint   ← chão navegável (NavMesh)
    │   └── SitPoint        ← snap exato do corpo
    └── Seat_02
        ├── ApproachPoint
        └── SitPoint
```

**Managers obrigatórios na cena:**

`GameBootstrap` com os 5 managers serializados: `globalDialogueManager`, `barbershopRatingManager`, `financeManager`, `barberQueueSystem`, `appointmentScheduler`

Managers como filhos do `GameBootstrap`: `LoanSystem`, `ClientLoyaltySystem`, `CutMasterySystem`, `PlayerNicknameManager`, `ClientAppointmentScheduler`

**Assets obrigatórios:**
- `MainClientRequestDatabase.asset` (25 `ClientRequestData`)
- `ServicePriceTable.asset` referenciado em `GlobalGameplayManagement.globalPriceTable`
- Databases de produtos por categoria (`DB_MaquinasCorte.asset`, `DB_Tesouras.asset`, etc.)

---

## Bancos de Produtos

```text
ScriptableObjects/Products/Databases/
├── DB_MaquinasCorte.asset
├── DB_Tesouras.asset
├── DB_Pentes.asset
├── DB_Navalhas.asset
├── DB_Laminas.asset
├── DB_ProdutosCapilar.asset
├── DB_Secador.asset
├── DB_Mobilia.asset
└── DB_Decoracao.asset
```

---

## Troubleshooting

| Problema | Verificar |
|---|---|
| Cliente não anda | NavMesh baked; agente sobre NavMesh; destino alcançável; `agent.enabled = true` |
| Cliente senta fora do sofá | `ApproachPoint` no chão; `SitPoint` na posição exata; `snapToSeatOnArrival = true` |
| UI do pedido não abre | `ClientRequestUI.Instance` existe; Canvas + EventSystem presentes; cliente em `WaitingForService` |
| Atendimento não inicia | Sem cliente em atendimento; pedido preenchido; energia OK; `barberChairWalkPoint` configurado; itens no inventário |
| Planejamento não abre | `useAdvancedServiceWorkflow = true`; `AdvancedServiceWorkflowManager` na cena; jogador perto da cadeira |
| Produto comprado não aparece | Produto tem ID único; loja adiciona ao inventário; UI atualiza após compra |
| Dinheiro não atualiza | UI ligada ao `FinanceManager` (único sistema canônico); `OnCashChanged` e `OnFinanceDataChanged` assinados |

---

## Convenções

- Scripts: `PascalCase.cs` | Assets: `Prefix_Name.asset`
- Código, comentários e strings de UI: **português**
- Nomes de arquivo e classes: **inglês** (ex.: `CutMasterySystem`, não `SistemaDeMaestria`)
- Input: **New Input System** — proibido `Input.GetKey()` / API legada
- Materials: **HDRP Lit** ou **URP Lit** — nunca Standard
- IDs únicos para: produtos, pedidos, requisitos, cortes, clientes especiais

---

## Roadmap

**Curto prazo**
- Conectar `MissionSystem` ao fluxo avançado (`HandleAdvancedServiceFinished`)
- Aplicar `PrestigeSystem.GetBonusXP()` em `PlayerXPManager.AddXP()`
- Conectar `CulturalEventSystem` ao fluxo de atendimento
- Criar UI de conquistas (`AchievementSystem` funciona, sem tela/popup)

**Médio prazo**
- Melhorar HUD de chat com opções contextuais de gameplay
- Adicionar comentários de clientes à UI de reputação
- Personalização visual da barbearia
- Campanhas narrativas adicionais

**Longo prazo**
- Sistema de cidade ao redor da barbearia com NPCs ambulantes
- Eventos especiais temáticos (CulturalEventSystem)
- Build mobile (resolver HDRP/URP — materiais ficam rosa em URP)
- Publicação

---

## Débitos Técnicos Conhecidos

- `MissionSystem.RegisterServiceCompleted` não é chamado no fluxo avançado (só no fallback)
- `PrestigeSystem.BonusXP10` sem efeito: `PlayerXPManager.AddXP` não multiplica por `GetBonusXP()`
- `CulturalEventSystem` sem efeito real de gameplay (bônus calculados, nunca aplicados ao serviço)
- `AchievementSystem` sem UI consumidora (funciona internamente, sem popup/tela)
- HDRP/URP em mobile: materiais HDRP ficam rosa em build URP — validar em device antes de shippar
- `ClientSpawner` ainda tenta subscrever `GlobalReputationSystem` (removido) — bloco `if (Instance != null)` silencia o erro

---

## Orientação para Novos Desenvolvedores e Agentes de IA

Antes de alterar qualquer código:

1. Leia este README e o [CLAUDE.md](CLAUDE.md)
2. Identifique qual sistema será alterado — verifique se já existe um manager para esse domínio
3. Nunca crie duplicidade de singleton
4. Não altere `_Deprecated/` exceto para remoção/migração
5. Mantenha compatibilidade com `ClientNPC`, `BarbershopServiceManager`, `InventoryManager` e `ClientRequestData`
6. Teste o fluxo completo de 22 etapas após qualquer mudança no fluxo de atendimento
7. Documente novos sistemas no CLAUDE.md (APIs, PlayerPrefs keys, wiring de cena)

---

## Documentação Técnica

Consulte [CLAUDE.md](CLAUDE.md) para documentação completa: APIs de todos os sistemas, enums, PlayerPrefs keys, fluxo de serviço, wiring de cena e débito técnico detalhado.
