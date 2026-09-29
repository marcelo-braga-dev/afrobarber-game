# AfroBarber — Janelas e Interface

| | |
|---|---|
| **Documento** | 03 · Janelas e interface (UI/UX) |
| **Versão** | 1.3 — 29/09/2026 |
| **Leitores** | Designers de UI/UX, artistas, desenvolvedores, testadores |
| **Relacionados** | [02 Jogabilidade](02-guia-de-jogabilidade.md) · [09 Arte e tecnologia](09-arte-audio-e-tecnologia.md) · [07 Estado atual](07-estado-atual-e-roadmap.md) |

> Catálogo de todas as telas, painéis, popups e elementos de HUD: para que servem, o que mostram, o que o jogador pode fazer e qual sistema os alimenta. Nomes técnicos entre parênteses são as classes do projeto. Marcas: ✅ ativo · 🟡 pronto, não ativo · ⚠️ com ressalva.

## Sumário
1. [Princípios da interface](#1-princípios-da-interface)
2. [Inventário de telas](#2-inventário-de-telas)
3. [Menu principal](#3-menu-principal)
4. [HUD](#4-hud)
5. [Fluxo de atendimento](#5-fluxo-de-atendimento)
6. [Painéis de gestão](#6-painéis-de-gestão)
7. [Popups e janelas auxiliares](#7-popups-e-janelas-auxiliares)
8. [Mapa painel → sistema](#8-mapa-painel--sistema)
9. [Pendências de navegação](#9-pendências-de-navegação)

---

## 1. Princípios da interface

- **Um painel principal por vez.** O `GameUIManager` mantém um painel "exclusivo" aberto; abrir outro fecha o anterior; clicar de novo fecha (toggle).
- **`Esc` fecha tudo.** No Android/Editor, `Esc` ou o botão *voltar* também abre/fecha o painel **Sair do jogo**.
- **Painéis bloqueantes travam o movimento** do personagem.
- **Notificação** é a única janela que convive com outra.
- **Atualização ao abrir:** cada painel recarrega seus dados quando aberto.
- **Tipografia:** *Londrina Solid* em títulos; famílias *Exo 2*, *Rajdhani* e *Marcellus* disponíveis no projeto.
- **Iconografia:** ícones de atributos (precisão/alvo, velocidade, durabilidade, entrega, coração, lixeira) e ícones de corte nos pedidos.
- **Idioma e moeda:** português; valores em **BM$**.
- **Cores com significado:** status da agenda, posts do BarberBook (dourado/verde/vermelho), zonas do minigame (cinza/laranja/verde/dourado).

---

## 2. Inventário de telas

| Grupo | Tela | Abertura | Status |
|---|---|---|---|
| Menu | Menu principal, Carregamento, Apelido, Opções, Créditos, Sair | Botões do menu | ✅ |
| HUD | Relógio, dinheiro, energia, XP, reputação, fila, apelido, chat, badge BarberBook | Sempre visível | ✅ |
| Atendimento | Pedido do cliente | Clique no cliente | ✅ |
| | Planejamento | Cliente sentado / botão "Iniciar Atendimento" | ✅ |
| | Minigame "Corte Perfeito" | Início do atendimento | ✅ |
| | Execução por etapas | Modo alternativo | ✅ (desligado) |
| | Avaliação do atendimento | Fim do serviço | ✅ |
| Gestão | Financeiro | Botão da HUD | ✅ |
| | Gestão da barbearia | Botão da HUD | ✅ |
| | Reputação detalhada | Botão da HUD | ✅ |
| | Histórico de atendimentos | Botão da HUD | ✅ |
| | BarberBook | Badge da HUD | ✅ |
| | Loja, Inventário, Biblioteca de Cortes, Fila, Agenda, Missões, Empréstimos, Desafios Diários | Método existe; **nenhum caminho de abertura** na cena | ⚠️ (Loja e Inventário: bloqueio crítico) |
| | Painel de Conquistas | Não está na cena | 🟡 |
| | Prestígio | Sem tela | 🟡 |
| Popups | Novo corte, Conquista, Descanso, Sair, Tutorial, Loading | Automáticos | ✅ |

---

## 3. Menu principal

| Tela | Conteúdo e ações |
|---|---|
| **Menu** | Jogar · Nome de usuário · Opções · Créditos · Sair; navegação por teclado/controle com botão padrão selecionado. |
| **Carregando pacotes** (Android) | Progresso do download de conteúdo (Play Asset Delivery); bloqueia o menu até terminar. |
| **Carregando cena** | Percentual de carregamento do jogo. |
| **Nome de usuário** | Campo de texto e Salvar. |
| **Opções** | **Áudio** (música, efeitos, som de teste ao soltar o controle), **Vídeo** (resolução e tela cheia, ocultos no celular), **Controles**; Salvar e Resetar. |
| **Créditos** | Painel informativo. |
| **Sair** | Confirmação (Sair / Cancelar). |

---

## 4. HUD

| Elemento | O que mostra | Classe |
|---|---|---|
| **Relógio e data** | Hora, dia e mês do jogo. | `GameTimeUI` |
| **Dinheiro** | Caixa em BM$. | `FinanceHUDBinder`, `ShopCashHeaderUI` |
| **Energia** | Barra e texto de contexto (bônus de recuperação, perda por dívidas, energia baixa/crítica). | `EnergyUI` |
| **XP e nível** | Título do nível, barra e XP atual/necessário. | `PlayerXPUI` |
| **Reputação** | Nota média ("4,2/5") e total de avaliações; atalho para a reputação detalhada. | `ReputationUI` |
| **Clientes esperando** | Contador da fila. | `WaitingCountUI` |
| **Apelido** | Nome do jogador. | `PlayerNicknameDisplayUI` |
| **Chat** | Mensagens de NPCs (cor por falante), do sistema e do jogador; botão de **opções de fala**. | `HudChatCardUI`, `DialogueOptionsButtonUI`, `DialoguePopupUI` |
| **Badge do BarberBook** | Posts não lidos (até "99+"); abre o feed. | `BarberBookNotificationBadge` |
| **Ícone sobre o cliente** | Cliente pronto para atendimento ou com interação. | `NPCInteractionIndicator` |
| **"Iniciar Atendimento"** | Aparece perto da cadeira com o cliente sentado aguardando. | `StartServicePlanningButtonUI` |
| **Controle na tela** (mobile) | Movimento e câmera por toque. | — |

---

## 5. Fluxo de atendimento

### 5.1 Pedido do Cliente (`ClientRequestUI`)

*Status: ✅*
- **Mostra:** nome e ícone do corte, preço, tempo, "+XP", "Dificuldade N", requisitos ⚠️ (vazios — nenhum corte tem requisitos cadastrados), título e **resumo histórico** do corte.
- **Ações:** Aceitar · Dispensar.

### 5.2 Planejamento do Atendimento (`ServicePlanningUI`)

*Status: ✅*
- **Cabeçalho:** cliente, corte e tempo estimado total (cor muda se o plano ficar longo ou curto demais).
- **Ações disponíveis:** Lavar, Pentear, Cortar, Navalha, Acabamento, Definir, Barba, Finalizar.
- **Plano:** etapas em ordem, cada uma com a imagem da ferramenta escolhida; clique na imagem para trocar; botões para **subir**, **descer** e **remover** a etapa.
- **Seleção de ferramenta** (`ServiceToolSelectionPopupUI`): só itens **compatíveis** do inventário, com atributos.
- **Avisos:** "Existe uma ação sem ferramenta…", "Monte uma sequência de ações antes de iniciar.", "Adicione a ação Finalizar para concluir o atendimento."
- **Ações:** Iniciar atendimento · Dispensar cliente.

### 5.3 Minigame "Corte Perfeito" (`MinigameCortePerfeitoController`)

*Status: ✅*
- Alvos com os ícones das etapas; anel que encolhe; cores cinza → laranja → verde → dourado.
- Contador "acertos / total", **combo** ("Combo x4!") e feedback por toque ("Perfeito!", "Ótimo!", "Bom!", "Ok", "Cedo demais!").

### 5.4 Execução por etapas (`AdvancedServiceExecutionUI`) — modo alternativo
Mostra cada etapa com ferramenta, tempo, nota e avaliação (Horrível → Perfeito) e fecha sozinha ao final.

### 5.5 Avaliação do Atendimento (`ServiceEvaluationUI`)

*Status: ✅ / ⚠️*
- Nota final **/5**, resultado, mensagem sobre o tempo ("O atendimento foi rápido", "dentro do esperado", "demorou um pouco mais").
- Notas e comentários de **Atendimento**, **Estrutura** e **Experiência**.
- ⚠️ No modo minigame a nota pode passar de 5 (ver [07](07-estado-atual-e-roadmap.md)).

---

## 6. Painéis de gestão

### 6.1 Fila (`QueueUIManager`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- Resumo e **card por cliente** (`QueueClientCardUI`): nome, pedido, tempo de espera, barra e estado de paciência (Calmo, Esperando, Impaciente, Bravo, Indo embora).
- Ordenação por **chegada** ou **urgência**.

### 6.2 Agenda (`AppointmentPanelUI`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- Agendamentos (`AppointmentItemUI`) com horário, cliente, corte e **cor por status**: Agendado, Aguardando chegada, Atrasado pela fila, Chegou, Concluído, Overbooking, Cancelado, Perdido.
- Atualização automática.

### 6.3 Loja (`ShopManager`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- **Categorias:** Máquinas de corte, Tesouras, Pentes, Navalhas, Lâminas, Produtos capilares, Secador, Mobília, Decoração.
- **Card de produto** (`ShopItemUI`): ícone, nome, preço, **Comprar** / "Comprado" e barras de **Precisão, Velocidade, Durabilidade** (ferramentas) ou **Conforto, Estética, Tempo de entrega** (móveis).
- Cabeçalho com o caixa; a compra entra no extrato.

### 6.4 Inventário (`InventoryUIManager`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- Itens agrupados por categoria.
- **Card** (`InventoryProductCardUI`): nome, status, usos restantes, vida útil estimada, barra de **condição**, atributos.
- **Ações:** Descartar ferramentas quebradas · Equipar · Colocar/Remover móveis e decoração (selo "Equipado").

### 6.5 Financeiro (`FinanceUIController`)

*Status: ✅*
- **Resumo:** caixa, dívida em aberto, dívida vencida.
- **Extrato** mensal (`FinanceMonthCardUI`, `FinanceMovementRowUI`): serviços, gorjetas, compras, contas, multas, empréstimos, recompensas; carregamento progressivo ao rolar.
- **Pagar** contas em aberto.

### 6.6 Empréstimos (`LoanPanelUI`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- "Total devendo: BM$ X" e status.
- Quatro opções (BM$ 500 / 1.000 / 2.500 / 5.000), uma ativa por vez; **Pagar tudo**.

### 6.7 Gestão da Barbearia (`BarbershopManagementUI`)

*Status: ✅*
- **Horário** de abertura e fechamento (hora e minuto) e **dias** de funcionamento.
- **Preços:** ajuste percentual por tipo de serviço (`ServicePriceManagementRowUI`), com a dica *"Preços acima do sugerido reduzem fluxo; abaixo aumentam."*
- **Resumo:** efeito na demanda e multiplicador de hora extra sobre a energia.
- **Melhorias da Barbearia** (`BarbershopUpgradeRowUI`): as 7 reformas com custo, nível mínimo, pré-requisito e Comprar.
- Salvar · Resetar.

### 6.8 Reputação detalhada (`BarbershopRatingUI`)

*Status: ✅*
Nota global e total de avaliações; notas, barras e comentários de **Atendimento**, **Estrutura** e **Experiência**.

### 6.9 Histórico de Atendimentos (`ServiceHistoryUI`)

*Status: ✅*
Dia, hora, cliente, serviço, valor recebido e nota /5 de cada atendimento.

### 6.10 Missões (`MissionsPanelUI`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- **Missões:** título, descrição, progresso ("3/5"), barra, recompensa do tier e **Resgatar**.
- **Histórico:** tiers resgatados.

### 6.11 Desafios Diários (`DailyChallengePanelUI`)

*Status: ✅ sistema · ⚠️ sem botão de abertura na cena*
- **Desafios:** 3 cards com progresso, recompensa e **Coletar**; pontuação do dia.
- **Ranking:** a barbearia do jogador (destaque dourado, apelido) contra *Barbearia do Zé, Corte & Arte, Navalha Fina, AfroStyle Studio, Raízes & Estilo, Cortes do Bairro, Estilo Livre, Arte na Navalha* e *Barbearia Ouro*; medalhas para o top 3.

### 6.12 BarberBook (`BarberBookPanelUI`)

*Status: ✅*
- **Feed** (`BarberBookPostItemUI`): **dourado** (viral), **verde** (positivo), **vermelho** (negativo); nome, corte, texto, curtidas.
- **Popup viral** e contador de **clientes orgânicos**.

### 6.13 Biblioteca de Cortes (`EducationEncyclopediaUI`)

*Status: ✅ conteúdo · ⚠️ sem caminho de abertura na cena · o tutorial e uma conquista a chamam de "Enciclopédia Afro" (nome a unificar)*

- **Filtros** (Todos, Black Power, Fade, Tranças, Dreads, Twists, Flat Top, Afro Clássico, Contemporâneo, Tradicional, Outros), **busca** e **ordenação por década**. Os filtros "Black Power" e "Outros" ficam vazios: nenhum corte está nessas categorias.
- **Progresso** "X / 24 descobertos".
- **Cards** (`EducationEncyclopediaItemUI`): ícone, nome e década (ou **"???"**), categoria, estrelas de dificuldade, selos Novo/Favorito/Bloqueado, maestria 🟡.
- **Detalhes:** história completa, significado cultural, curiosidade, preço/tempo/XP, maestria 🟡, **Favoritar**.

---

## 7. Popups e janelas auxiliares

| Janela | Função | Status |
|---|---|---|
| **Novo corte descoberto** (`EducationUnlockPopupUI`) | Anuncia um corte recém-desbloqueado. | ✅ |
| **Conquista desbloqueada** (`AchievementPopupUI`) | Fila de popups com título, descrição e recompensa. | ✅ |
| **Painel de Conquistas** (`AchievementsPanelUI`) | Lista de conquistas com progresso total. | 🟡 |
| **Apelido** (`PlayerNicknameInputUI`) | Abre na primeira execução; reabrível pela HUD. | ✅ |
| **Tutorial** (`TutorialController`) | Mensagens passo a passo com avançar e pular. | ✅ |
| **Descanso** (`RestUIController`) | "Horário de descanso" ao dormir no carro. | ✅ |
| **Sair do jogo** (`ExitGameManager`) | Confirmação de saída. | ✅ |
| **Carregamento** (`GameBootstrap`) | Loading com fade durante a inicialização. | ✅ |

---

## 8. Mapa painel → sistema

| Painel | Sistema |
|---|---|
| Pedido, Planejamento, Minigame, Avaliação | `BarbershopServiceManager`, `AdvancedServiceWorkflowManager`, `MinigameCortePerfeitoController` |
| Fila / Agenda | `BarberQueueSystem` / `ClientAppointmentScheduler` |
| Loja / Inventário | `ShopManager`, `InventoryManager`, `PlacedFurnitureManager` |
| Financeiro / Empréstimos | `FinanceManager`, `FinanceMonthlyBillsManager` / `LoanSystem` |
| Gestão | `GlobalGameplayManagement`, `GameTimeSystem`, `BarbershopUpgradeSystem` |
| Reputação / Histórico | `BarbershopRatingManager` / `ServiceHistorySystem` |
| Missões / Desafios / Conquistas | `MissionSystem` / `DailyChallengeSystem` / `AchievementSystem` |
| BarberBook | `BarberBookSystem` |
| Biblioteca | `EducationProgressManager`, `CutMasterySystem` |
| Chat | `GlobalDialogueManager`, `NPCConversationBrain`, `DialogueGameplayActionRouter` |

---

## 9. Pendências de navegação

Na `GameScene`, os botões de abertura encontrados são: **Financeiro, Gestão, Reputação detalhada, Histórico, Apelido** e o badge do **BarberBook**. Os métodos `OpenLoja`, `OpenInventario`, `OpenBibliotecaCortes`, `OpenFila`, `OpenAgenda`, `OpenMissoes`, `OpenEmprestimos` e `OpenDesafiosDiarios` existem no `GameUIManager`, mas nenhum botão, atalho ou objeto da cena ou de prefab os chama. Como o inventário começa vazio, a falta de acesso à **Loja** e ao **Inventário** impede o primeiro atendimento (ver [07 §2.1](07-estado-atual-e-roadmap.md#21-save-novo-não-consegue-atender-soft-lock)).

**Proposta de UX:** uma **barra de navegação** fixa na HUD (ícones com rótulo) com: Loja · Inventário · Fila · Agenda · Financeiro · Gestão · Missões · Desafios · Biblioteca · BarberBook · Conquistas — mais atalhos de teclado (ex.: `I` inventário, `L` loja, `M` missões) e, no celular, menu em leque. Confirmar no Editor antes de implementar.
