# AfroBarber — Guia de Jogabilidade

| | |
|---|---|
| **Documento** | 02 · Guia de jogabilidade (regras e fluxo) |
| **Versão** | 1.3 — 29/09/2026 |
| **Leitores** | Game designers, testadores, jogadores, desenvolvedores |
| **Relacionados** | [03 Interface](03-janelas-e-interface.md) · [04 Progressão](04-progressao-e-objetivos.md) · [05 Economia](05-economia-e-gestao.md) · [07 Estado atual](07-estado-atual-e-roadmap.md) |

> Valores são os padrões do projeto em 29/09/2026 (código e `GameScene`). Marcas de status: ✅ ativo · 🟡 pronto, não ativo · 🧩 regra projetada, ainda fora do cálculo · ⚠️ com ressalva — ver [legenda](README.md#2-legenda-de-status).

## Sumário
1. [Primeiros passos](#1-primeiros-passos)
2. [Controles](#2-controles)
3. [O tempo no jogo](#3-o-tempo-no-jogo)
4. [Um dia de trabalho](#4-um-dia-de-trabalho)
5. [O atendimento, passo a passo](#5-o-atendimento-passo-a-passo)
6. [Os dois modos de execução](#6-os-dois-modos-de-execução)
7. [Clientes](#7-clientes)
8. [Conversa e relacionamento](#8-conversa-e-relacionamento)
9. [Clima](#9-clima)
10. [Energia, fadiga, descanso e desmaio](#10-energia-fadiga-descanso-e-desmaio)
11. [Fim do dia e virada de dia](#11-fim-do-dia-e-virada-de-dia)
12. [Salvamento](#12-salvamento)
13. [Dicas de estratégia](#13-dicas-de-estratégia)

---

## 1. Primeiros passos

### 1.1 Menu principal
| Botão | Função |
|---|---|
| **Jogar** | Carrega o jogo com barra de progresso. No Android, aguarda antes o download dos pacotes de conteúdo (Play Asset Delivery); o botão fica bloqueado até terminar. |
| **Nome de usuário** | Define o **apelido** do jogador, usado no chat e no ranking. |
| **Opções** | **Áudio** (música, efeitos, som de teste), **Vídeo** (resolução e tela cheia; ocultos no celular), **Controles**. |
| **Créditos** | Créditos do projeto. |
| **Sair** | Confirmação de saída. |

`Esc` fecha diálogos ou volta ao menu principal.

### 1.2 Primeira sessão
- Sem apelido salvo, o painel de **nickname** abre sozinho (padrão "Jogador"). ✅
- **Tutorial** em 6 passos, que pode ser pulado e não se repete depois de concluído: ✅
  1. **Boas-vindas** — "Você acaba de abrir sua própria barbearia especializada em cortes afro…"
  2. **Primeiro cliente** — esperar o cliente sentar e clicar nele.
  3. **Aceitar serviço** — conferir as ferramentas no inventário e aceitar.
  4. **Executar serviço** — escolher as ferramentas certas para cada etapa (lavar, cortar, finalizar).
  5. **Receber pagamento** — o cliente vai ao caixa; serviço bem feito rende dinheiro, XP e conteúdo histórico.
  6. **Enciclopédia** — "Você desbloqueou seu primeiro conteúdo na Enciclopédia Afro! … Conhecimento é poder!"
- **Estado inicial:** BM$ 100 (cena; padrão do código BM$ 1.000) · inventário vazio · nível 1, *Aprendiz da Navalha* · reputação 3,0 · 10/04/2026, 8h.
- ⚠️ **Contas já vencidas:** as contas de abril são geradas no início, com o aluguel (dia 5) já atrasado — total de BM$ 1.010 contra BM$ 100 de caixa (ver [07 §2.3](07-estado-atual-e-roadmap.md#23-contas-de-abril-nascem-vencidas)).
- ⚠️ **Acesso à Loja:** hoje não há botão para abrir Loja e Inventário, então um save novo não consegue comprar ferramentas (ver [07 §2.1](07-estado-atual-e-roadmap.md#21-save-novo-não-consegue-atender-soft-lock)).

---

## 2. Controles

| Ação | Teclado e mouse | Gamepad | Toque (Android) |
|---|---|---|---|
| Andar | `W A S D` | Analógico esquerdo | Controle na tela |
| Correr (sprint) | `Shift` esquerdo | Pressionar analógico esquerdo | — |
| Pular | `Espaço` | Botão Sul (A/✕) | — |
| Girar a câmera | Setas `← → ↑ ↓` | Analógico direito | Controle na tela |
| Interagir (cliente, cadeira, carro) | `E` ou clique esquerdo no objeto | — ⚠️ | Tocar no objeto |
| Minigame | Clique no alvo | — ⚠️ | Toque no alvo |
| Fechar painéis | `Esc` | — | Botão fechar |

- **Câmera:** orbital atrás do personagem (4,5 m andando, 5,8 m no sprint), realinha sozinha após ~1,5 s de caminhada, desvia de paredes e amplia o campo de visão no sprint.
- **Gamepad:** move o personagem, a câmera, corre e pula, mas **ainda não interage** com clientes nem joga o minigame. ⚠️
- **Personagem:** andar 2,5 · correr 5,5 · sprint 8 m/s; pulo com tolerância (*coyote time* e *jump buffer*); sobe rampas suavemente. Painéis bloqueantes travam o movimento.

---

## 3. O tempo no jogo

| Parâmetro | Valor |
|---|---|
| Velocidade | **3 minutos de jogo por segundo real** (1 hora de jogo ≈ 20 s) |
| Horário de funcionamento sugerido | **10h às 19h, terça a sábado** (aplicado ao iniciar; alterável na Gestão) |
| Hora de recolher | 22h |
| Duração de um expediente | ~3 minutos reais |

- Relógio e data ficam na HUD; céu, luz e névoa acompanham a hora.
- ⚠️ **O tempo nunca pausa**, nem com painéis abertos ou durante o planejamento.
- ⚠️ **A data não é salva:** toda sessão recomeça em 10/04/2026, 8h (ver [07 §2.2](07-estado-atual-e-roadmap.md#22-a-data-do-jogo-não-é-salva)).
- Durante o atendimento, o **tempo do serviço é somado ao relógio** (e cresce com o cansaço — ver §10).
- No fechamento, clientes que ainda estão na barbearia são dispensados.

---

## 4. Um dia de trabalho

### 4.1 Antes de abrir
- **Inventário:** o jogo começa sem itens; é preciso ter ferramentas **compatíveis** com as etapas do corte (máquina ou tesoura para cortar, pente, navalha, produto capilar…), senão o plano não pode ser iniciado.
- **Agenda:** quem vem e a que horas.
- **Clima:** anunciado no chat; indica o movimento do dia.
- **Energia:** se estiver baixa, descanse antes (§10).

### 4.2 Chegada dos clientes

*Status: ✅*
Os clientes são moradores da cidade que chegam pela **agenda automática**:
- novos agendamentos são gerados continuamente — na cena atual a cada ~5 min de jogo, cerca de 1,7 s reais (padrão do código: 90 min), com frequência maior quando a reputação é alta;
- podem ser marcados com até **3 dias** de antecedência (30% de chance); o limite é de 2 agendamentos por hora, que o **overbooking** (25% de chance) pode ultrapassar;
- no horário, o morador interrompe a caminhada e vai até a barbearia.

Ao entrar, o cliente **reserva um assento livre**, caminha até ele, senta, entra na **fila** e exibe um **ícone de interação**.

Com a fila cheia, os agendamentos seguintes **atrasam em cascata** (+8 min por pessoa esperando, até 45 min). Se o cliente não for ativado até **60 min** após o horário, o agendamento vira **"Perdido"** e ele envia uma mensagem.

> Clientes espontâneos (sem agendamento) existem no código, mas estão desligados na cena. 🟡

### 4.3 Paciência e humor
Paciência padrão: **90 minutos de jogo** (VIP: 6 min 🟡; fiéis: +5 a +25 min 🧩).

> ⚠️ Como o relógio corre a 3 minutos por segundo e não pausa, **90 minutos de paciência equivalem a 30 segundos reais** (VIP: 2 segundos). A recalibração está no plano de ação ([07 §2.4](07-estado-atual-e-roadmap.md#24-paciência-e-agenda-rápidas-demais)).

A mesma barra gera duas leituras, com limites diferentes — por isso o rótulo "Impaciente" aparece em faixas distintas na fila e no humor:

**Estado na fila** ✅ — exibido no card da fila

| Paciência gasta | Estado |
|---|---|
| < 25% | Calmo |
| 25–50% | Esperando |
| 50–75% | Impaciente |
| 75–95% | Bravo |
| ≥ 95% | Indo embora |

**Humor** — o cliente reclama no chat a cada mudança ✅; o efeito na nota é 🧩

| Paciência gasta | Humor | Mensagem | Efeito na nota 🧩 |
|---|---|---|---|
| < 35% | Tranquilo | — | +0,2 |
| 35–65% | Ansioso | "… está ficando ansioso na fila de espera..." | −0,1 |
| 65–85% | Impaciente | "… está impaciente. Quanto tempo ainda?" | −0,3 |
| ≥ 85% | Irritado | "… está muito irritado com a espera! Atenda logo!" | −0,6 |

**Urgência** 🧩 (modo por etapas): atender com ≤ 15% da paciência gasta dá +20% dinheiro e +10% XP; com ≥ 85% gasta, −10% dinheiro.

> Hoje o cliente **não abandona** a fila quando a paciência acaba; o estado "Indo embora" é visual. ⚠️

---

## 5. O atendimento, passo a passo

### Passo 1 — Ler o pedido

*Status: ✅*
Clique no cliente sentado. A janela **Pedido do Cliente** mostra: nome e ícone do corte, **preço** (BM$), **tempo estimado**, **+XP**, **dificuldade (1–5)**, **requisitos** e o **resumo histórico** do corte. Ações: **Aceitar** ou **Dispensar**.

> A lista de requisitos aparece vazia porque nenhum corte tem requisitos cadastrados ainda. ⚠️

### Passo 2 — Aceitar

*Status: ✅*
O jogo exige que:
- não haja outro cliente em atendimento (**um por vez**);
- o barbeiro não esteja desmaiado e tenha **≥ 10 de energia**.

O cliente libera o assento, caminha até a cadeira e senta.

### Passo 3 — Planejar

*Status: ✅*
Com o cliente sentado, abre o **Planejamento do Atendimento** (ou surge o aviso *"… está na cadeira. Aproxime-se e clique em 'Iniciar Atendimento'"*).

O jogo sugere a sequência **Lavar → Cortar → Acabamento → Finalizar**. Etapas podem ser adicionadas, **movidas para cima ou para baixo** e **removidas**; só "Finalizar" é obrigatória. Cada etapa precisa de uma ferramenta compatível do inventário:

| Etapa | Ferramentas compatíveis |
|---|---|
| Lavar | Produto capilar |
| Pentear | Pente |
| Cortar | Máquina de corte, Tesoura |
| Navalha | Navalha, Lâminas |
| Acabamento | Produto capilar, Pente |
| Definir | Produto capilar, Pente |
| Barba | Navalha, Máquina de corte, Tesoura |
| Finalizar | Produto capilar, Pente |

- A escolha de ferramenta é **lembrada por corte** para a próxima vez.
- O tempo estimado muda de cor se o plano ficar muito longo ou curto.
- Avisos bloqueiam o início: etapa sem ferramenta, plano vazio, falta de "Finalizar".
- O cliente também pode ser **dispensado** daqui.

### Passo 4 — Executar: minigame "Corte Perfeito"

*Status: ✅*
Os **ícones das etapas do plano** surgem como **alvos**, um a cada ~1,4 s. Cada alvo tem um **anel que encolhe** em ~2,2 s; a cor indica o momento: **cinza** → **laranja** → **verde** → **dourado pulsante**. Clique ou toque no momento certo.

| Momento do toque | Pontos | Feedback |
|---|---|---|
| Zona perfeita | 100 | "Perfeito!" |
| Muito perto | 80 | "Ótimo!" |
| Perto | 60 | "Bom!" |
| Razoável | 35 | "Ok" |
| Cedo demais | 15 | "Cedo demais!" |
| Deixou expirar | 0 | — |

- Acertos de 60+ pontos em sequência formam **combo** ("Combo x3!").
- Quantidade de alvos = **5 + (dificuldade − 1)**: 5 alvos na dificuldade 1, 9 na dificuldade 5.

### Passo 5 — Resultado

*Status: ✅ / ⚠️*

| % dos pontos possíveis | Resultado | Dinheiro e XP | Gorjeta |
|---|---|---|---|
| ≥ 90% | **Perfeito** | × 1,30 | 15% do preço |
| ≥ 75% | **Maravilhoso** | × 1,15 | 15% do preço |
| ≥ 55% | **Bom** | × 1,00 | — |
| ≥ 35% | **Mais ou Menos** | × 0,85 | — |
| ≥ 15% | **Ruim** | × 0,65 | — |
| < 15% | **Horrível** | × 0,45 | — |

Em seguida:
- abre a **Avaliação** (nota /5, resultado, comentários de Atendimento, Estrutura e Experiência, comentário sobre o tempo) ✅;
- entram **dinheiro + gorjeta** (com bônus de evento cultural) e **XP** (com bônus de reformas, prestígio e evento) ✅;
- a nota entra na **reputação** ⚠️ — a escala do minigame está inflada: "Bom" conta como 5,0 (ver [07 §5](07-estado-atual-e-roadmap.md#5-inconsistências-de-conteúdo-e-design));
- o corte é **desbloqueado na Biblioteca** ✅;
- são notificados: fidelidade ✅, maestria 🟡, BarberBook ✅, VIP 🟡, desafios diários ✅, arcos narrativos 🟡, missões ✅, contas vencidas ✅;
- **troca visual do cabelo** 🧩 — sistema pronto, sem cabelos configurados nos clientes;
- **consumo de produtos e desgaste de ferramentas** 🧩 — depende dos requisitos dos cortes, ainda vazios.

### Passo 6 — Caixa e saída

*Status: ✅*
O cliente vai ao **caixa**, paga, sai e volta a passear pela cidade. O assento é liberado.

> O ciclo técnico completo (do spawn à liberação do slot) tem 22 etapas — ver [README do projeto](../README.md#loop-de-gameplay).

---

## 6. Os dois modos de execução

O projeto tem dois modos de execução do corte, escolhidos por configuração (`useMinigame`):

| | **Minigame** (padrão atual) ✅ | **Execução por etapas** (alternativo) |
|---|---|---|
| O que decide a nota | Só o timing do jogador | Qualidade e compatibilidade das ferramentas, tempo, personalidade, humor, nível, maestria, urgência |
| Ferramentas | Obrigatórias para montar o plano | Definem a qualidade de cada etapa (ferramenta incompatível: −1,5) |
| Gorjeta | 15% do preço em Maravilhoso/Perfeito | Por personalidade (ver §7.2) |
| Nível do jogador | Não influencia | +5% qualidade e −5% penalidade de tempo por nível |

A direção de design recomendada é **combinar os dois**: timing do jogador + qualidade das ferramentas + contexto do cliente (ver [07](07-estado-atual-e-roadmap.md)).

**Faixas de nota do modo por etapas (0–5):** Horrível < 0,9 · Ruim < 1,8 · Mais ou Menos < 2,8 · Bom < 3,8 · Maravilhoso < 4,6 · Perfeito ≥ 4,6. A qualidade de cada ferramenta é a média de precisão, velocidade e durabilidade.

---

## 7. Clientes

### 7.1 Tipos
| Tipo | Como surge | Particularidades | Status |
|---|---|---|---|
| **Comum** | Agenda automática | Personalidade, humor e paciência próprios | ✅ |
| **Fiel** | Volta várias vezes | Visitas contadas em 5 níveis; bônus de gorjeta, paciência e elogio | ✅ / 🧩 |
| **VIP** | Até 1 a cada 3 dias; 25% de chance/dia; exige nota ≥ 3,5 e nível ≥ 2 | Paga 3×, gorjeta 2,5×, paciência de 6 min; post pode viralizar | 🟡 |
| **Narrativo** | Seu Ribeiro, Kemi, Gabriel, Dona Conceição, Leo | Capítulos de história com objetivo | 🟡 |

### 7.2 Personalidades
Cada personalidade muda as **opções de conversa** ✅ e, no modo por etapas, a nota e a gorjeta 🧩:

| Personalidade | Efeito na nota e no tempo | Gorjeta |
|---|---|---|
| **Amigável** | +5% qualidade; −30% penalidade de tempo | 10–25% (nota ≥ 3) |
| **Calmo** | −15% penalidade de tempo | 5% (nota ≥ 4) |
| **Comunicativo** | **+50% XP** (divulga o barbeiro) | 15–30% (nota ≥ 3,5) |
| **Exigente** | −10% qualidade; +15% penalidade de tempo | 8% (nota ≥ 4,5) |
| **Irritado** | −15% qualidade; +30% penalidade de tempo | — |
| **Chato** | Qualidade reduzida de 0 a 40% ao acaso | — |
| **Tímido** | −50% penalidade de tempo | 5–10% (nota ≥ 3) |
| **Brincalhão** | Qualidade ±20% ao acaso | 0–35% ao acaso |

---

## 8. Conversa e relacionamento

*Status: ✅*

O **chat da HUD** mostra, em tempo real: falas espontâneas dos clientes (entre si e sobre o bairro), reclamações de espera, mensagens do sistema (clima, contas, conquistas, eventos) e as falas do jogador, com o apelido.

O jogador escolhe **opções de fala** conforme o contexto:

| Contexto | Exemplos | Efeito |
|---|---|---|
| **Fila** | "Próximo da fila" · "Já vou te chamar, segura só um pouco" · "Foi mal pela demora, vou organizar aqui" · "Hoje não vou atender mais ninguém" | Chamar o próximo ✅; pedir paciência e desculpar-se alteram a relação ✅; fechar para novos clientes 🧩 (só gera mensagem) |
| **Atendimento** | "Qual estilo você prefere?" · "Vou priorizar seu acabamento" · "Começar atendimento" · "Deixa eu ver aqui... quantas vezes você já veio?" | Iniciar ✅; **revelar o nível de fidelidade** do cliente e o bônus de gorjeta ✅ |
| **Cidade e cultura** | "E esse movimento do bairro hoje?" · "Vai colar no evento cultural mais tarde?" | Conversa social ✅ |
| **Narrativa** | "E aí, como anda aquela história que você me contou?" | Com personagens de arco 🟡 |
| **Despedida** | "Valeu por vir, volta sempre!" · "Você já é cliente de confiança aqui, parceiro." | Encerrar ✅ |
| **Reclamação** | "Peço desculpas, vamos resolver isso direitinho." · "Relaxa, vou resolver contigo agora" | Acalmar ✅ |
| **Por personalidade** | Piada (Brincalhão) · "Capricho é meu sobrenome" (Exigente) · "Vamos no seu ritmo" (Tímido) · "Me conta as novidades do bairro" (Comunicativo) · "Vamos focar no corte, beleza?" (Chato) | Tom adequado a cada pessoa ✅ |

Cada NPC guarda **memória** das conversas e dos atendimentos, com níveis de **confiança** e **tensão** que definem seu humor e podem ser citados depois.

---

## 9. Clima

*Status: ✅*

Sorteado no início de cada expediente, com probabilidade por estação (hemisfério sul) e anunciado no chat.

| Clima | Demanda ✅ | Gorjeta 🧩 | Humor positivo 🧩 | Mensagem |
|---|---|---|---|---|
| **Ensolarado** | × 1,25 | × 1,10 | 75% | "O movimento está ótimo — as pessoas adoram sair quando faz sol." |
| **Nublado** | × 1,00 | × 1,00 | 50% | "Fluxo normal de clientes." |
| **Chuvoso** | × 0,60 | × 1,20 | 40% | "Menos clientes, mas quem enfrenta a chuva… costuma dar gorjeta melhor." |
| **Tempestade** | × 0,25 | × 1,35 | 30% | "Quase ninguém sai de casa. Os poucos que aparecem são clientes leais." |

| Estação | Ensolarado | Nublado | Chuvoso | Tempestade |
|---|---|---|---|---|
| Verão | 35% | 35% | 20% | 10% |
| Outono | 25% | 40% | 25% | 10% |
| Inverno | 20% | 35% | 30% | 15% |
| Primavera | 30% | 35% | 25% | 10% |

---

## 10. Energia, fadiga, descanso e desmaio

*Status: ✅*

O barbeiro tem **Energia (0–100)** e **Fadiga acumulada (0–100)**, exibidas na HUD com textos de contexto ("Bônus de recuperação 2.0x", "Perda de disposição por dívidas em atraso").

**O que cansa**
- estar dentro da barbearia (desgaste contínuo), mais com a **fila cheia**;
- cada atendimento;
- ficar na barbearia **depois do horário** (× 2,2) ou na rua após as 22h (× 1,4);
- **hora extra**: cada hora além do horário sugerido aumenta o gasto em 8%;
- **dívidas vencidas acima de BM$ 1.000**: perda diária que cresce com os dias de atraso.

**O que recupera**
- **sair da barbearia**: após ~15 min fora, a energia volta; quanto **mais longe** (faixas de 10, 25, 50 e 90 m), maior o bônus, até **2,5×**;
- **dormir no carro**: fora do horário comercial, interaja com o carro; o tempo avança até o próximo expediente ("Horário de descanso"). Dentro do horário, o descanso é bloqueado;
- após as 22h, a recuperação na rua é bloqueada.

**Efeitos da energia baixa**

| Energia | Tempo do serviço ✅ | Penalidade de qualidade 🧩 |
|---|---|---|
| ≤ 55% | +10% | −0,25 |
| ≤ 35% | +20% | −0,6 |
| ≤ 20% | +30% | −1,0 |
| ≤ 10% | +45% | −1,4 |

Com menos de 10 de energia, **não é possível iniciar atendimento**.

**Desmaio:** com energia zerada, o personagem **cai no chão** (animação), fica deitado 10–30 s reais, perde **20–45 min** de jogo e levanta com 10–20 de energia. O desmaio zera a sequência da missão *Disposição de Ferro*.

---

## 11. Fim do dia e virada de dia

- No horário de fechamento, a barbearia fecha e os clientes restantes saem.
- Dormir no carro leva ao próximo expediente.
- Na virada do dia: novo clima; verificação de contas; juros semanais de empréstimo; expiração de posts e conversão de posts positivos em **clientes orgânicos**; ativação de eventos culturais; renovação de missões diárias; registro do relatório diário.
- Os **desafios diários** renovam pelo **relógio real**, não pelo dia do jogo.

---

## 12. Salvamento

*Status: ✅*

O progresso é salvo **automaticamente e continuamente** no aparelho (exceto a **data do jogo** ⚠️ e a agenda): caixa e extrato, inventário, XP e nível, Biblioteca (desbloqueios, favoritos, vistos), reputação, reformas, empréstimos, BarberBook, prestígio, desafios, fidelidade, narrativa, conquistas, relatórios, maestria, missões, horários e preços, mobília, escolhas de ferramenta, memória dos NPCs e apelido. Não há slots de salvamento; a agenda de clientes não é salva entre sessões. Detalhes em [09](09-arte-audio-e-tecnologia.md#8-salvamento).

---

## 13. Dicas de estratégia

1. **Monte o kit básico.** Você começa sem itens: compre ao menos tesoura (ou máquina) e pente para o plano mínimo *Cortar → Finalizar*; depois, um produto capilar para o plano completo.
2. **Treine o timing.** No modo atual, o resultado depende só da precisão no minigame; combos rendem mais.
3. **Não deixe a fila crescer.** Ela cansa o barbeiro e irrita os clientes.
4. **Caminhe para recuperar.** Uma volta longa pelo bairro recupera energia até 2,5× mais rápido.
5. **Pague as contas em dia.** Atraso gera multa de 5%, reduz a reputação, corta 20% da demanda e drena energia.
6. **Preço acima do sugerido espanta clientes; abaixo, atrai.**
7. **Dias de sol são movimentados; de tempestade, vazios** — planeje compras e descanso.
8. **Aproveite as datas culturais:** novembro dá bônus de XP e dinheiro, e o dia 20 dobra o XP.
