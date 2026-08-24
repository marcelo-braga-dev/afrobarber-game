# AfroBarber v1

Jogo de simulação de barbearia afro-brasileira desenvolvido em Unity, com foco em progressão de negócio, cultura afro e gestão de clientes.

## Tecnologias

- **Unity 6000.3.10**
- **HDRP 17.3.0** / **URP 17.3.0**
- **New Input System 1.18.0**
- **Unity Test Framework 1.6.0**
- **Plastic SCM** (controle de versão principal)

## Estrutura dos Scripts

```text
Assets/Afrobarber/Scripts/
├── Appointment/        # Sistema de agendamentos
├── Barbershop/         # Gerenciamento da barbearia e fila
├── Characters/         # NPCs (clientes, spawner)
├── City/               # Waypoints e movimentação na cidade
├── Core/               # Bootstrap, tempo, clima, tutorial
├── Dialogue/           # Sistema de diálogo global
├── Economy/            # Finanças, empréstimos, contas mensais
├── Education/          # Biblioteca de Cortes
├── Energy/             # Sistema de energia do jogador
├── Evaluation/         # Avaliação de serviços e reputação
├── Gameplay/           # VIP, prestígio, desafios diários
├── Inventory/          # Inventário de produtos e ferramentas
├── Missions/           # Missões narrativas e por marcos
├── Progression/        # XP, fidelidade, maestria por corte
├── Queue/              # Fila de clientes
├── Reputation/         # Histórico de atendimentos
├── Services/           # Fluxo de serviço (básico e avançado)
├── Social/             # BarberBook (rede social fictícia)
├── UI/                 # Todos os painéis e componentes de UI
└── Utilities/          # Ferramentas de debug
```

## Sistemas Principais

| Sistema | Descrição |
|---|---|
| `BarbershopServiceManager` | Orquestra o fluxo completo de atendimento |
| `ClientNPC` | Máquina de estados dos clientes (Wandering → InService → ...) |
| `FinanceManager` | Sistema financeiro canônico (único) |
| `PlayerXPManager` | Progressão do jogador (5 níveis) |
| `EducationProgressManager` | Biblioteca de 25 cortes afro com lore |
| `CutMasterySystem` | Maestria por corte (5 tiers por corte) |
| `ClientLoyaltySystem` | Fidelidade de clientes (5 tiers) |
| `DailyChallengeSystem` | Desafios diários com ranking fictício |
| `NarrativeMissionSystem` | Arcos narrativos com 5 personagens fixos |
| `BarberBookSystem` | Rede social simulada pós-atendimento |
| `LoanSystem` | Empréstimos com juros compostos semanais |
| `PrestigeSystem` | New Game+ com perks permanentes |

## Documentação Técnica

Consulte [CLAUDE.md](CLAUDE.md) para documentação completa de arquitetura, APIs, convenções e débito técnico conhecido.

## Idioma

Todo o código, comentários, strings de UI e nomes de assets estão em **português**.
