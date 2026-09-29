# AfroBarber — Estado Atual e Roadmap

| | |
|---|---|
| **Documento** | 07 · Estado da build, pendências e roadmap |
| **Versão** | 1.3 — 29/09/2026 |
| **Leitores** | Produção, desenvolvimento, design, proponente de editais |
| **Base do levantamento** | Código (`Assets/Afrobarber/Scripts`), assets (`ScriptableObjects`, prefabs) e `GameScene.unity` em 29/09/2026 |

## Sumário
1. [Resumo executivo](#1-resumo-executivo)
2. [Bloqueios críticos](#2-bloqueios-críticos)
3. [Matriz de funcionalidades](#3-matriz-de-funcionalidades)
4. [Regras projetadas ainda fora do cálculo](#4-regras-projetadas-ainda-fora-do-cálculo)
5. [Inconsistências de conteúdo e design](#5-inconsistências-de-conteúdo-e-design)
6. [Balanceamento](#6-balanceamento)
7. [Precisão cultural e propriedade intelectual](#7-precisão-cultural-e-propriedade-intelectual)
8. [Pendências de interface](#8-pendências-de-interface)
9. [Plano de ação priorizado](#9-plano-de-ação-priorizado)
10. [Roadmap de produto](#10-roadmap-de-produto)
11. [Riscos técnicos](#11-riscos-técnicos)
12. [Método do levantamento](#12-método-do-levantamento)

Legenda: ✅ ativo · 🟡 pronto, não ativo · 🧩 regra projetada, fora do cálculo · ⚠️ com ressalva · 🔲 planejado.

---

## 1. Resumo executivo

**Estágio:** pré-alfa (versão 0.1.0). O ciclo de atendimento funciona, mas **um save novo não consegue atender clientes sem intervenção no Editor** (ver §2).

**O que já funciona de ponta a ponta:** o ciclo completo de atendimento (agenda → espera → pedido → planejamento → minigame → avaliação → pagamento → saída), bairro vivo com moradores e trânsito, dia/noite, clima, energia e desmaio, economia (caixa, extrato, contas, multas, empréstimos, loja, mobília, reformas com bônus), desbloqueio dos 24 cortes, missões, desafios diários com ranking, conquistas, BarberBook, conversa contextual com memória, tutorial e salvamento automático.

**O que está programado mas não ligado (🟡):** maestria por corte, clientes VIP, arcos narrativos, prestígio, painel de conquistas, clientes espontâneos, relatório diário e os botões de 8 painéis na HUD (incluindo Loja, Inventário e Biblioteca).

**Regras configuradas mas fora do cálculo (🧩):** bônus de fidelidade, efeitos de humor e personalidade na nota, gorjeta por clima, perks de aluguel e gorjeta, desgaste e consumo de itens, troca visual de cabelo, aparição física das reformas.

**Principais ajustes de design:** preço único por tipo de serviço (o preço de cada corte não é usado; Turbante sai por 0), escala de nota do minigame inflada, inventário inicial vazio com caixa de BM$ 100, conquistas inalcançáveis e IDs errados nos eventos culturais.

**Bloqueios críticos:** sem acesso a Loja/Inventário com inventário inicial vazio; data do jogo não salva; contas de abril nascem vencidas; paciência e agenda correm rápido demais em tempo real.

**Leitura:** a base de sistemas é ampla e sólida; o trabalho que falta é sobretudo de **integração, conteúdo de configuração e balanceamento**, não de novos sistemas.

---

## 2. Bloqueios críticos

Verificados no código e na `GameScene` em 29/09/2026. Precisam ser resolvidos antes de qualquer teste com público.

### 2.1 Save novo não consegue atender (soft-lock)
- O jogador começa **sem itens** e o planejamento exige uma ferramenta compatível por etapa.
- **Nenhum botão, atalho ou objeto** da `GameScene` abre a **Loja** ou o **Inventário** (`OpenLoja`/`OpenInventario` não têm chamadores; o painel da loja começa desativado). A **Biblioteca** também não tem caminho de abertura.
- **Consequência:** sem ferramentas não há plano; sem plano não há atendimento.
- **Correção:** barra de navegação na HUD **e** kit inicial no inventário.

### 2.2 A data do jogo não é salva
- O `GameTimeSystem` não grava nada: toda sessão recomeça em **10/04/2026, 8h**.
- Com expedientes de ~3 minutos reais, sessões curtas **nunca viram o mês**: aluguel de maio, juros semanais, eventos de fevereiro, julho, setembro e novembro e a missão anual "Consciência em Ação" podem nunca acontecer.
- Sistemas que guardam datas (contas, empréstimos, VIP, relatórios) passam a conviver com um relógio que volta no tempo.
- **Correção:** salvar data e hora (ou `TotalMinutesElapsed`) e restaurar no carregamento.

### 2.3 Contas de abril nascem vencidas
- Na primeira sessão (10/04), as contas de abril são geradas com vencimentos nos dias 5, 10 e 12. O **aluguel já nasce 5 dias atrasado**. Total: BM$ 700 + 220 + 90 = **BM$ 1.010**, contra um caixa de BM$ 100.
- A verificação diária só age quando o atraso é **exatamente 1 dia** (aviso) ou **exatamente 3 dias** (multa). Por isso:
  - o aluguel de abril (6 dias de atraso na primeira virada) **nunca recebe aviso nem multa**, mas conta como dívida vencida;
  - com as três contas abertas a partir de 13/04, a dívida vencida passa de BM$ 1.000 e **começa a drenar energia**;
  - como a data volta a 10/04 a cada sessão (§2.2), **avisos, multas e perda de reputação de luz e água podem se repetir a cada sessão**.
- Pagar uma conta **não restaura a demanda** nem mostra "pago!": `NotificarContaPaga` não tem chamador.
- A "−0,1 de reputação" é, na prática, **uma avaliação a mais com nota (média − 0,1)**: o efeito real é bem menor que 0,1.
- **Correção:** começar o jogo no início do mês, ou gerar as contas a partir do mês seguinte; usar `>=` nas verificações de atraso; chamar `NotificarContaPaga` ao pagar; salvar a data.

### 2.4 Paciência e agenda rápidas demais
- A paciência, a espera e a agenda usam o relógio do jogo, que corre a **3 minutos por segundo** e **nunca pausa** (nem com painéis abertos; `SetPause` não é chamado).

| Parâmetro | Em minutos de jogo | Em tempo real |
|---|---|---|
| Paciência padrão | 90 min | **30 s** |
| Paciência VIP | 6 min | **2 s** |
| Bônus de fidelidade | +5 a +25 min | +1,7 a +8,3 s |
| Intervalo de agendamento (cena) | ~5 min | **~1,7 s** (limitado a 2 por hora de jogo = 1 a cada ~20 s) |
| Atraso até "Perdido" | 60 min | 20 s |
| Duração de um corte (10–60 min) | somada ao relógio de uma vez | a fila "envelhece" 10–60 min instantaneamente |

- **Correção:** pausar o tempo com painéis bloqueantes abertos e durante o planejamento; medir a paciência em tempo real ou multiplicar os valores; revisar o intervalo de agendamento.

---

## 3. Matriz de funcionalidades

| Área | Funcionalidade | Status | Observação |
|---|---|---|---|
| **Menu** | Menu, opções, apelido, créditos, sair, carregamento (PAD no Android) | ✅ | |
| **Controle** | Personagem em 3ª pessoa, câmera orbital, sprint, pulo | ✅ | |
| | Interação e minigame | ✅ / ⚠️ | Teclado, mouse e toque; **gamepad não interage nem joga o minigame** |
| **Mundo** | Moradores por waypoints, trânsito, dia/noite, clima por estação | ✅ | 15 prefabs de clientes |
| **Tempo** | Relógio, horário comercial, dias úteis, avanço durante o serviço | ✅ / ⚠️ | 3 min de jogo por segundo; nunca pausa (§2.4) |
| **Clientes** | Agenda automática, atrasos em cascata, "Perdido" | ✅ | |
| | Clientes espontâneos (walk-in) | 🟡 | `ClientSpawner.autoSpawn = false` |
| | Espera, fila, paciência, mensagens de humor | ✅ | |
| | Cliente vai embora ao esgotar a paciência | 🔲 | Estado "Indo embora" só visual |
| | Clientes VIP | 🟡 | Nada chama `MarcarClienteComoVip` |
| **Atendimento** | Pedido, aceitar/dispensar | ✅ / ⚠️ | Requisitos vazios; Turbante com preço 0 |
| | Planejamento com ferramentas compatíveis | ✅ | |
| | Minigame "Corte Perfeito" | ✅ / ⚠️ | Escala de nota inflada (§5) |
| | Execução por etapas (modo alternativo) | ✅ | Desligado (`useMinigame = true`) |
| | Avaliação, dinheiro, gorjeta, XP, reputação | ✅ / ⚠️ | |
| | Troca visual do cabelo | 🧩 | Nenhuma variante de cabelo configurada nos clientes |
| | Consumo e desgaste de itens | 🧩 | Dependem de `requiredItems`, vazio nos 24 cortes |
| **Conversa** | Chat, falas por contexto e personalidade, memória, confiança/tensão | ✅ | |
| | "Fechar para novos clientes" | 🧩 | Só gera mensagem |
| **Energia** | Energia, fadiga, recuperação por distância, desmaio, descanso no carro | ✅ | |
| **Economia** | Caixa, extrato, empréstimos, crise | ✅ | |
| | Contas mensais | ⚠️ | Nascem vencidas; verificação por igualdade; pagamento não restaura demanda (§2.3) |
| | Preço por corte | ⚠️ | Tabela por tipo de serviço sobrepõe o preço do corte |
| | Loja, inventário, mobília colocável | ✅ | Painéis sem botão na HUD (§2.1, §8) |
| | Gestão: horários, dias, ajuste de preço, reformas | ✅ | |
| | Reformas — bônus | ✅ | |
| | Reformas — objetos na cena | 🧩 | Nenhum dos 7 objetos existe na `GameScene` |
| **Progressão** | Níveis e multiplicadores de XP | ✅ | |
| | Biblioteca de Cortes — desbloqueio e conteúdo | ✅ | |
| | Biblioteca de Cortes — abertura do painel | ⚠️ | Nenhum caminho de abertura na cena |
| | Maestria por corte | 🟡 | `CutMasterySystem` fora da `GameScene` |
| | Fidelidade — contagem e consulta | ✅ | |
| | Fidelidade — bônus | 🧩 | §4 |
| | Reputação e subnotas | ✅ / ⚠️ | |
| | Missões (8) | ✅ | |
| | Desafios diários e ranking | ✅ | |
| | Conquistas — popup | ✅ / ⚠️ | §5 |
| | Conquistas — painel | 🟡 | `AchievementsPanelUI` fora da cena |
| | Arcos narrativos | 🟡 | Nenhum NPC com `narrativeCharacterId` |
| | Prestígio | 🟡 | Sem tela que chame `TentarPrestigiar` |
| | Eventos culturais — bônus | ✅ | |
| | Eventos culturais — desbloqueio de cortes | ⚠️ | IDs errados |
| **Social** | BarberBook, posts virais, clientes orgânicos | ✅ | Viral depende de VIP 🟡 |
| **Onboarding** | Tutorial, apelido | ✅ | |
| **Dados** | Salvamento automático | ✅ / ⚠️ | **Data do jogo não é salva** (§2.2); agenda também não |
| | Relatório diário | 🟡 | Sem tela |
| **Plataforma** | PC | ✅ | |
| | Android | ⚠️ | Materiais HDRP em perfil URP — validar em aparelho |

---

## 4. Regras projetadas ainda fora do cálculo

Valores já configurados que nenhum sistema consome hoje:

| Regra | Valor configurado | Onde está | Onde deveria entrar |
|---|---|---|---|
| Preço VIP | × 3 | `VIPClientSystem.AplicarMultiplicadorPreco` | Receita do atendimento |
| Gorjeta VIP | × 2,5 | `VIPClientSystem.AplicarMultiplicadorGorjeta` | Gorjeta |
| Gorjeta por fidelidade | × 1,10 a × 1,85 | `ClientLoyaltySystem.GetMultiplicadorGorjeta` | Gorjeta (hoje só aparece na fala de consulta) |
| Paciência extra por fidelidade | +5 a +25 min | `ClientLoyaltySystem.GetBonusPacienciaMinutos` | Entrada na fila |
| Elogio espontâneo | 5% a 30% | `ClientLoyaltySystem.DeveElogiarEspontaneamente` | Chat após o atendimento |
| Humor na nota | +0,2 a −0,6 | `ClientMoodSystem.GetModificadorRating` | Nota final |
| Gorjeta e humor por clima | × 1,0 a × 1,35; 30% a 75% | `WeatherEffect` | Gorjeta; humor inicial do cliente |
| Perk *AlugueMenor15* | × 0,85 | `PrestigeSystem.GetFatorAluguel` | Conta de aluguel |
| Perk *GorjetaExtra20* | × 1,2 | Via fidelidade | Gorjeta |
| Penalidade de qualidade por energia | −0,25 a −1,4 | `PlayerEnergySystem.GetServiceQualityPenalty` | Nota (hoje só no fluxo automático antigo) |
| Nível, maestria, personalidade, urgência, ferramenta | Diversos | `AdvancedServiceOutcomeResolver` | Nota — hoje só no modo por etapas, desligado |

---

## 5. Inconsistências de conteúdo e design

| # | Problema | Impacto | Sugestão | Prioridade |
|---|---|---|---|---|
| 1 | **Preço por tipo de serviço:** com `overrideRequestPriceTable = true`, todos os cortes do tipo "Corte de Cabelo" custam BM$ 35 e os acabamentos BM$ 15; o preço de referência de cada corte (BM$ 20–80) é ignorado. | Cortes difíceis não pagam mais; progressão econômica achatada. | Usar o preço do corte como base e a tabela como ajuste por tipo. | Alta |
| 2 | **Turbante** (tipo "Outro") não tem entrada na tabela: preço **0** no pedido; no minigame usa reserva de BM$ 50. | Inconsistência visível. | Criar entrada ou aplicar o item 1. | Alta |
| 3 | **Nota do minigame = fator × 5:** Horrível 2,25 · Ruim 3,25 · Mais ou Menos 4,25 · **Bom 5,0** · Maravilhoso 5,75 · Perfeito 6,5. | Reputação inflada; "Mãos de Ouro" (≥ 4,8) e combos (≥ 3,5) quase automáticos; "Nota Máxima" trivial; avaliação pode exibir "6,5/5". | Mapear resultado → 0,5 / 1,5 / 2,5 / 3,5 / 4,3 / 5,0. | Alta |
| 4 | **Minigame ignora ferramentas e contexto:** só o timing conta. | Comprar ferramentas melhores não muda o resultado. | Combinar timing com qualidade das ferramentas e contexto do cliente. | Alta |
| 5 | **Inventário inicial vazio** e caixa de BM$ 100: o plano padrão com produto capilar custa ≥ BM$ 195. | Primeiros minutos confusos. | Dar um kit inicial (pente, tesoura, creme básico). | Alta |
| 6 | **Requisitos vazios** nos 24 cortes. | Pedido sem requisitos; nenhum consumo nem desgaste. | Preencher `requiredItems` por corte. | Alta |
| 7 | **Cabelos não configurados:** 0 variantes nos clientes, embora existam ~220 arquivos de arte de cabelo. | O corte não aparece no cliente. | Configurar `ClientHairVisualController` por prefab. | Média |
| 8 | **IDs errados nos eventos culturais** (`black_power_classico`, `afro_natural_50s`, `conk_alisado`…); os reais usam o prefixo `req_`. | Cortes não liberados; IDs fantasmas inflam a contagem da Biblioteca e podem liberar "Mestre da Enciclopédia" antes da hora. | Corrigir para os `RequestId` reais. | Média |
| 9 | Conquista **"Barbearia Premium"** exige 10 reformas (existem 7). | Inalcançável. | Mudar para 7. | Baixa |
| 10 | Conquista **"Mestre Supremo AfroBarber"** exige 25 cortes Lenda (existem 24). | Inalcançável. | Mudar para 24 / "todos". | Baixa |
| 11 | **Secador** à venda sem etapa compatível. | Compra inútil. | Criar etapa "Secar" ou incluir em Finalizar/Definir. | Baixa |
| 12 | Itens caros com atributos piores (FadeX Control, UrbanBlade Pro). | Escolha confusa. | Reordenar preços por atributos. | Baixa |
| 13 | XP de High Top Fade (dif. 4) e Jheri Curl (dif. 3) = 10. | Cortes difíceis pouco atraentes. | Normalizar XP ≈ 6–8 × dificuldade. | Baixa |
| 14 | "Dia Impecável" e "Sequência de Ouro" medem quase a mesma coisa. | Pouca variedade. | Trocar um por "atender N cortes diferentes". | Baixa |
| 15 | Agenda gera cliente a cada ~5 min de jogo na cena (código: 90). | Fila enche rápido. | Validar em playtest. | Média |
| 16 | Cliente não abandona a fila. | Espera longa sem custo real. | Sair com avaliação negativa ao esgotar. | Média |

---

## 6. Balanceamento

| # | Observação | Números | Sugestão |
|---|---|---|---|
| B1 | **Turbante paga mais que os outros cortes** no minigame: sem entrada na tabela, cai na reserva de BM$ 50. | Dif. 1, 10 min → BM$ 50; demais cortes → BM$ 35. | Resolver com o preço por corte (§5 #1). |
| B2 | **Etapas não servem a 9 dos 24 estilos.** O plano Lavar → Cortar → Acabamento → Finalizar com máquina ou tesoura não descreve tranças, locs, twists, puff e turbante. | Box Braids, Cornrows, Tranças Nagô, Dreads/Locs, Sisterlocks, Twists, Twist Out, Coily Puff, Turbante. | Novas ações (Trançar, Torcer, Enrolar, Amarrar, Secar) e, talvez, um tipo de serviço "Penteado/Trança". |
| B3 | **Conquistas dominam o XP.** | Conquistas: 17.500 XP. Nível 1→5: 12.500 XP. Um corte: 10–40 XP. Com a nota inflada, o nível 2 chega em ~10 clientes. Os 500 XP de "Lenda AfroBarber" chegam quando não há mais nível. | Reduzir XP das conquistas ou aumentar a curva; trocar XP da conquista de nível 5 por outra recompensa. |
| B4 | **Maestria total é inviável.** | Lenda nos 24 cortes = 550 × Σ dificuldades (75) = **41.250 pontos** ≈ 1.650 atendimentos Perfeitos. | Reduzir o multiplicador por dificuldade ou os limiares. |
| B5 | **Meta semanal alta demais.** | "Semana dos Campeões" tier 3 = BM$ 10.000/semana. No teto de 2 agendamentos por hora de jogo (+ overbooking) × 9 h × 5 dias ≈ 90–110 clientes × ~BM$ 51 ≈ **BM$ 5.600**. | Ajustar tiers para BM$ 1.500 / 3.000 / 5.000. |
| B6 | **XP de cortes desalinhado.** | Twist Out (dif. 3) = 30 XP = Skin Fade (dif. 5); High Top Fade (dif. 4) e Jheri Curl (dif. 3) = 10 XP. | XP ≈ 6–8 × dificuldade. |
| B7 | **Produtos desalinhados.** | Cremes 1 (BM$ 145, média 19) vs. Cremes 2 (BM$ 149, média 31); FadeX Control e UrbanBlade Pro piores que GoldenEdge Supreme. | Preço proporcional à média de atributos. |
| B8 | **Ícones e filtros.** | 2 dos 9 ícones de pedido (High Fade, Low Fade) não correspondem a cortes; os filtros "Black Power" e "Outros" da Biblioteca ficam sempre vazios (nenhum corte nessas categorias). | Criar os cortes ou remover ícones/filtros; mover Black Power Clássico para a categoria Black Power. |
| B9 | **Sem trava de corte por nível.** | A agenda sorteia qualquer um dos cortes; um Sisterlocks (dif. 5) pode chegar no primeiro dia. | Liberar cortes por nível ou por reputação. |
| B10 | **Eventos sobrepostos.** | Em 20/11, "Mês" (× 1,5) e "Dia" (× 2,0) não se somam: vale o **maior** bônus de cada tipo. | Documentado; decidir se é o comportamento desejado. |
| B11 | **Desafio "Dia Impecável".** | Conta atendimentos não ruins do dia; um Ruim zera o progresso exibido, mas o próximo atendimento restaura a contagem acumulada. | Definir se é "5 no dia" ou "5 seguidos" e alinhar o código. |

---

## 7. Precisão cultural e propriedade intelectual

O texto dos cortes em [06](06-universo-cultural.md) é o mesmo que o jogador lê no jogo; correções precisam ser feitas **nos assets `Request_*.asset`** e depois no documento. Recomenda-se revisão por consultoria em cultura afro-brasileira.

### 7.1 Conteúdo cultural a corrigir ou confirmar
| Onde | Texto atual | Correção proposta |
|---|---|---|
| Cornrows (curiosidade) | "Em 2018, uma lei na Califórnia…" | **CROWN Act** da Califórnia: assinado em **2019**, em vigor em 2020. |
| Tranças Nagô (curiosidade) | Tranças como mapas de fuga "em algumas culturas africanas" | Tradição oral **afro-colombiana** (San Basilio de Palenque, Benkos Biohó); usar "segundo relatos". |
| Conk (curiosidade) | Malcolm X chama o conk de "o primeiro grande passo na destruição da própria autoestima" | Frase original: *"my first really big step toward self-degradation"* ("meu primeiro grande passo rumo à autodegradação"). Indicar tradução. |
| Box Braids (curiosidade) | "Janet Jackson braids" | O apelido difundido é **"Poetic Justice braids"**. |
| Moicano Afro (curiosidade) | Afropunk "incluindo São Paulo" | A edição brasileira é o **Afropunk Bahia**, em Salvador — confirmar. |
| Corte César | Dificuldade 4; "Will Smith popularizou" | Rever dificuldade e atribuição com consultoria. |
| Evento de 25/09 | "Dia Nacional do Turbante" | Não foi identificada lei federal; confirmar a fonte ou chamar de "Dia do Turbante" (data celebrada por movimentos). |
| Evento de julho | "Mês da Cultura Afro-Latino-Americana" | Usar **25 de julho — Dia Nacional de Tereza de Benguela e da Mulher Negra** (Lei 12.987/2014) e o "Julho das Pretas". |
| Carnaval | Fixo em fevereiro | O Carnaval cai em março em alguns anos (ex.: 2025, 2030); calcular pela data da Páscoa. |
| Consciência Negra | — | Citar a Lei 14.759/2023 (feriado nacional em 20/11). |
| Nome da Biblioteca | Tutorial e conquista dizem "Enciclopédia Afro"; a interface diz "Biblioteca de Cortes" | Unificar o nome. |

### 7.2 Propriedade intelectual
| Item | Risco | Ação |
|---|---|---|
| "Fliperama **Pac-Man**" | Marca da Bandai Namco | Renomear ("Fliperama Retrô") e trocar o visual. |
| **Jeep**, Opala, Fusca e demais veículos | Marcas e desenho industrial | Nomes e modelos genéricos. |
| Fontes Exo 2, Rajdhani, Marcellus, Londrina Solid | Licença a registrar (em geral SIL OFL) | Listar nos créditos com a licença. |
| Máscaras africanas (incl. Chokwe) e djembê escaneados | Licença e sensibilidade cultural | Confirmar origem e licença; contextualizar. |
| Modelos Ready Player Me e gerados por IA | Termos de uso; disponibilidade do serviço | Confirmar termos e continuidade do serviço; substituir por modelos autorais. |

---

## 8. Pendências de interface

Botões de abertura encontrados na cena: **Financeiro, Gestão, Reputação detalhada, Histórico, Apelido** e o badge do **BarberBook**. Sem caminho de abertura: **Loja, Inventário, Biblioteca de Cortes, Fila, Agenda, Missões, Empréstimos, Desafios Diários** (os métodos `Open*` existem, mas nada os chama). Loja e Inventário são **bloqueio crítico** (§2.1). Também faltam: **painel de Conquistas**, **tela de Prestígio** e **tela de Relatório**. Proposta de barra de navegação em [03 §9](03-janelas-e-interface.md#9-pendências-de-navegação). Conferir no Editor antes de implementar.

---

## 9. Plano de ação priorizado

Esforço: **P** (horas) · **M** (1–3 dias) · **G** (1+ semana).

### P0-A — bloqueios (sem isso o jogo não se sustenta)
| Ação | Esforço | Ref. |
|---|---|---|
| Barra de navegação da HUD (Loja, Inventário, Biblioteca e demais painéis) | M | §2.1 |
| Kit inicial no inventário (pente, tesoura, creme básico) | P | §2.1 |
| Salvar e restaurar a data/hora do jogo | P | §2.2 |
| Contas: início no começo do mês ou contas a partir do mês seguinte; verificação de atraso com `>=`; chamar `NotificarContaPaga` | P | §2.3 |
| Pausar o tempo com painéis bloqueantes e no planejamento; recalibrar paciência e agenda para tempo real | M | §2.4 |

### P0-B — coerência para testes com público
| Ação | Esforço | Ref. |
|---|---|---|
| Escala de nota do minigame (0–5) | P | §5 #3 |
| Preço por corte + entrada para o Turbante | P | §5 #1–2 |
| Requisitos (`requiredItems`) dos 24 cortes → ativa consumo e desgaste | M | §5 #6 |
| `CutMasterySystem` na `GameScene` | P | §3 |
| Correções culturais nos assets dos cortes e nos eventos | P | §7.1 |

### P1 — ativar o conteúdo já escrito
| Ação | Esforço |
|---|---|
| Atribuir os 5 personagens narrativos a clientes com visual próprio | M |
| Spawn de VIP + aplicar preço e gorjeta VIP | P |
| Aplicar bônus de fidelidade, humor e clima | M |
| Combinar minigame com qualidade de ferramentas e contexto | M |
| Corrigir IDs dos eventos culturais e as duas conquistas | P |
| Painel de Conquistas e tela de Prestígio | M |
| Configurar cabelos antes/depois nos 15 clientes | G |
| Objetos visuais das 7 reformas | G |

### P2 — polimento
| Ação | Esforço |
|---|---|
| Cliente vai embora ao esgotar a paciência | P |
| Etapa "Secar" | P |
| Rebalancear preços de ferramentas e XP de cortes | P |
| Tela de relatório diário | M |
| Validar Android em aparelho real | M |

---

## 10. Roadmap de produto

### Médio prazo
- 🔲 Tela de relatório diário e mensal.
- 🔲 Comentários de clientes na reputação detalhada.
- 🔲 Personalização visual da barbearia (cores, pisos, quadros, layout).
- 🔲 Modo foto e compartilhamento do corte (integração com BarberBook).
- 🔲 Cancelamento de agendamento pelo jogador.
- 🔲 Serviços de barba, sobrancelha e hidratação (tipos já existem).
- 🔲 Mais arcos narrativos ligados a datas culturais.
- 🔲 Recursos de acessibilidade (ver [08 §9](08-referencia-para-editais.md#9-acessibilidade)).

### Longo prazo
- 🔲 Funcionários: segundo barbeiro ou trancista, dois atendimentos simultâneos.
- 🔲 Expansão do bairro: lojas, eventos de rua (roda de samba, batalha de rima, feira afro), interiores visitáveis.
- 🔲 Segunda barbearia / franquia.
- 🔲 Campeonato de barbearia.
- 🔲 Personalização do personagem.
- 🔲 Ranking online e desafios comunitários.
- 🔲 Modo educativo com Biblioteca navegável, imagens e áudio.
- 🔲 Localização (inglês, espanhol, francês).
- 🔲 Publicação (Google Play; PC).

---

## 11. Riscos técnicos

| Risco | Detalhe | Mitigação |
|---|---|---|
| HDRP × URP no Android | Maioria dos materiais é `HDRP/Lit`; o perfil Android usa URP | Conversão de materiais; testes em aparelho |
| Salvamento em PlayerPrefs | Sem slots nem nuvem; limpar dados apaga tudo | Save em arquivo JSON versionado |
| Testes só de lógica | 28 arquivos, ~357 casos EditMode; nenhum teste do ciclo em cena | Testes PlayMode do atendimento completo |
| Recursos de terceiros | Licenças a confirmar (ver [08 §15](08-referencia-para-editais.md#15-direitos-autorais-e-recursos-de-terceiros)) | Substituir por arte e áudio autorais |
| Cenas duplicadas | `_Recovery/*.unity` e `URP_Demo.unity` referenciam os mesmos scripts | Limpar ou isolar |

---

## 12. Método do levantamento

- Leitura do código-fonte dos sistemas (206 scripts C#) e de seus consumidores (quem chama cada método).
- Leitura dos assets de conteúdo (24 cortes, 8 missões, 41 produtos, 7 reformas, tabela de preços).
- Inspeção da `GameScene.unity` e dos prefabs: presença de componentes, valores sobrescritos no Inspector e ligações de botões.
- Pontos marcados "conferir no Editor" dependem de verificação visual no Unity.
