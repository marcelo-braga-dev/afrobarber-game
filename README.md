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

> Status de cada etapa e das regras em [docs/07-estado-atual-e-roadmap.md](docs/07-estado-atual-e-roadmap.md). Antes de testar num save novo, veja os **bloqueios críticos** ([§2](docs/07-estado-atual-e-roadmap.md#2-bloqueios-críticos)): sem botão de Loja/Inventário e inventário inicial vazio, data do jogo não salva, contas iniciais vencidas e paciência de ~30 s reais.

```text
1.  Cliente nasce no ponto de spawn
2.  Cliente entra na barbearia (EntrancePoint)
3.  Sistema reserva um assento disponível (WaitingAreaManager)
4.  Cliente caminha até o ApproachPoint do assento
5.  Cliente snapa no SitPoint e senta (agent desativado durante snap)
6.  Ícone de interação aparece sobre o cliente
7.  Jogador clica → ClientRequestUI.Show() exibe pedido
8.  Jogador aceita → CallForService() → BarbershopServiceManager.TryStartService()
9.  Sistema valida: nenhum atendimento ativo, pedido, energia ≥ 10, ponto da cadeira (ferramentas NÃO são validadas aqui)
10. Cliente libera assento e vai até BarberChairWalkPoint
11. Cliente snapa no BarberChairSitPoint (agent desativado)
12. ServicePlanningUI abre (se useAdvancedServiceWorkflow = true)
13. Jogador monta etapas e escolhe ferramentas (exige ferramenta compatível por etapa)
14. Execução: minigame "Corte Perfeito" (useMinigame = true, padrão) OU execução por etapas
15. Resultado: fator do minigame (padrão) OU AdvancedServiceOutcomeResolver (por etapas)
16. Jogador recebe dinheiro, XP e avaliação
17. Cabelo final aplicado visualmente (ClientHairVisualController — sem variantes configuradas hoje)
18. Conteúdo educativo desbloqueado (EducationProgressManager)
19. Produtos/ferramentas consumidos (InventoryManager — depende de requiredItems, vazio nos 24 cortes)
20. Cliente vai ao caixa (CashierPoint)
21. Cliente sai da barbearia (ExitPoint)
22. ClientSpawner libera o slot para novo cliente
```

---

## Progressão do Jogador

- Dinheiro acumulado para comprar produtos e ferramentas melhores
- XP → 5 níveis (Aprendiz → Barbeiro de Bairro → Profissional → Mestre do Degradê → Lenda AfroBarber)
- Melhoria de reputação (afeta fluxo de clientes)
- Desbloqueio de 24 cortes na Biblioteca Educativa
- Maestria por corte (5 tiers, bônus de qualidade e recompensa) — `CutMasterySystem` ainda fora da cena
- Prestige System (New Game+ com perks permanentes após nível 5) — sem tela ainda

## Decisões de Gestão

- Quais produtos comprar e manter em estoque
- Quando aceitar ou dispensar clientes (paciência limitada)
- Priorizar clientes VIP (preço 3×, paciência reduzida) — VIP ainda não é spawnado; multiplicadores não aplicados
- Equilibrar qualidade, velocidade e lucro
- Controlar energia do barbeiro (cansaço aumenta o tempo do serviço e bloqueia atendimento abaixo de 10)
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

O fluxo de serviço tem três caminhos:
- **Minigame** (padrão, `useMinigame = true`): planejamento → minigame "Corte Perfeito" → resultado por timing
- **Avançado por etapas** (`Services/Advanced/`, `useMinigame = false`): planejamento → execução passo a passo → `AdvancedServiceOutcomeResolver`
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
├── Education/          # Biblioteca de 24 cortes afro com lore completo
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
| `EducationProgressManager` | `Education/` | Biblioteca de 24 cortes afro: desbloqueio, favoritos, lore |
| `CutMasterySystem` | `Progression/` | Maestria por corte (5 tiers × 24 cortes; bônus só no modo por etapas) — 🟡 fora da `GameScene` |
| `ClientLoyaltySystem` | `Progression/` | Fidelidade de clientes (5 tiers); contagem ativa, bônus de gorjeta/paciência 🧩 não aplicados |
| `DailyChallengeSystem` | `Gameplay/` | Desafios diários (tempo real) com ranking fictício |
| `NarrativeMissionSystem` | `Missions/` | Arcos narrativos com 5 personagens fixos, 3 capítulos cada |
| `MissionSystem` | `Missions/` | Missões por marcos/tiers (registra em todos os fluxos via `NotificarSistemasExternos`) |
| `BarberBookSystem` | `Social/` | Rede social simulada pós-atendimento, posts virais |
| `LoanSystem` | `Economy/` | Empréstimos com juros compostos semanais (4 opções) |
| `PrestigeSystem` | `Gameplay/` | New Game+ com 5 perks permanentes |
| `VIPClientSystem` | `Gameplay/` | Clientes VIP (preço 3×, gorjeta 2,5×) — 🟡 sem spawn; multiplicadores não aplicados |
| `WeatherSystem` | `Core/` | Clima auto-dirigido; afeta a demanda (gorjeta por clima 🧩 não aplicada) |
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
- `MainClientRequestDatabase.asset` (24 `ClientRequestData`)
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
| Atendimento não inicia | Sem cliente em atendimento; pedido preenchido; energia ≥ 10; `barberChairWalkPoint` configurado. Para o plano iniciar: ferramenta compatível em cada etapa (inventário) e etapa "Finalizar" |
| Planejamento não abre | `useAdvancedServiceWorkflow = true`; `AdvancedServiceWorkflowManager` na cena; jogador perto da cadeira |
| Produto comprado não aparece | Produto tem ID único; loja adiciona ao inventário; UI atualiza após compra |
| Dinheiro não atualiza | UI ligada ao `FinanceManager` (único sistema canônico); `OnCashChanged` e `OnFinanceDataChanged` assinados |

---

## Convenções

- Scripts: `PascalCase.cs` | Assets: `Prefix_Name.asset`
- Comentários, strings de UI e muitos nomes de métodos e campos: **português**
- Nomes de arquivo e classes: **inglês** (ex.: `CutMasterySystem`, não `SistemaDeMaestria`)
- Input: **New Input System** — proibido `Input.GetKey()` / API legada
- Materials: **HDRP Lit** ou **URP Lit** — nunca Standard
- IDs únicos para: produtos, pedidos, requisitos, cortes, clientes especiais

---

## Roadmap

Roadmap completo e matriz de funcionalidades em [docs/07-estado-atual-e-roadmap.md](docs/07-estado-atual-e-roadmap.md).

**Curto prazo**
- Colocar `CutMasterySystem` na `GameScene` (maestria hoje não roda)
- Spawnar clientes VIP (`MarcarClienteComoVip` não é chamado) e preencher `narrativeCharacterId` nos NPCs dos arcos
- Barra de botões da HUD para todos os painéis + painel de Conquistas + tela de Prestígio
- Corrigir IDs de cortes nos eventos culturais, conquistas inalcançáveis e escala de nota do minigame

**Médio prazo**
- Relatório de negócio, comentários de clientes na reputação, personalização visual da barbearia, mais arcos narrativos

**Longo prazo**
- Funcionários, expansão do bairro, campeonatos, build mobile validada (HDRP/URP), publicação

---

## Débitos Técnicos Conhecidos

Lista detalhada em [docs/07-estado-atual-e-roadmap.md](docs/07-estado-atual-e-roadmap.md#5-inconsistências-de-conteúdo-e-design). Principais:

- `CutMasterySystem`, `AchievementsPanelUI` ausentes da `GameScene`; VIP e arcos narrativos sem gatilho na cena; Prestígio sem UI
- Eventos culturais desbloqueiam IDs de corte inexistentes (falta prefixo `req_`)
- Conquistas `melhorias_10` e `maestria_lenda_25` inalcançáveis (7 reformas, 24 cortes)
- Minigame: nota `fatorRating × 5` infla a reputação e ignora ferramentas/maestria
- HDRP/URP em mobile: materiais HDRP ficam rosa em build URP — validar em device antes de shippar
- `npc_dialogue_memory_{npcId}` (NPCDialogueMemory) não segue a convenção `AFROBARBER_` nas chaves de PlayerPrefs — exceção documentada

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

## Documentação de Design e Jogo

A pasta [docs/](docs/README.md) é o documento de design completo do jogo (10 documentos): visão geral, guia de jogabilidade, janelas e interface, progressão e objetivos, economia, universo cultural (os 24 cortes), estado atual e roadmap, referência para editais de fomento, arte/áudio/tecnologia e glossário.

## Documentação Técnica

Consulte [CLAUDE.md](CLAUDE.md) para documentação completa: APIs de todos os sistemas, enums, PlayerPrefs keys, fluxo de serviço, wiring de cena e débito técnico detalhado.
