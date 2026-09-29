# AfroBarber — Progressão e Objetivos

| | |
|---|---|
| **Documento** | 04 · Progressão, metas e recompensas |
| **Versão** | 1.4 — 29/09/2026 |
| **Leitores** | Game designers, roteiristas, balanceamento |
| **Relacionados** | [02 Jogabilidade](02-guia-de-jogabilidade.md) · [05 Economia](05-economia-e-gestao.md) · [06 Universo cultural](06-universo-cultural.md) · [07 Estado atual](07-estado-atual-e-roadmap.md) |

> Marcas: ✅ ativo · 🟡 pronto, não ativo · 🧩 regra projetada, ainda fora do cálculo · ⚠️ com ressalva — ver [legenda](README.md#2-legenda-de-status).

## Sumário
1. [Camadas de progressão](#1-camadas-de-progressão)
2. [Nível do barbeiro](#2-nível-do-barbeiro)
3. [Biblioteca de Cortes e maestria](#3-biblioteca-de-cortes-e-maestria)
4. [Fidelidade de clientes](#4-fidelidade-de-clientes)
5. [Reputação](#5-reputação)
6. [Arcos narrativos](#6-arcos-narrativos)
7. [Missões](#7-missões)
8. [Desafios diários](#8-desafios-diários)
9. [Conquistas](#9-conquistas)
10. [Reformas da barbearia](#10-reformas-da-barbearia)
11. [Eventos culturais](#11-eventos-culturais)
12. [Prestígio](#12-prestígio)
13. [Curva de progressão por fase](#13-curva-de-progressão-por-fase)

---

## 1. Camadas de progressão

| Camada | Escopo | Recompensa | Onde ver | Status |
|---|---|---|---|---|
| **Nível do barbeiro** | Conta | Título; libera reformas, VIPs, capítulos e prestígio | HUD | ✅ |
| **Biblioteca de Cortes** | 24 cortes | Conteúdo cultural; conquistas | Biblioteca | ✅ |
| **Maestria por corte** | Cada corte | Qualidade, tempo e recompensa naquele corte | Biblioteca | 🟡 |
| **Fidelidade** | Cada cliente | Gorjeta, paciência, elogios | Conversa | ✅ contagem · 🧩 bônus |
| **Reputação** | Barbearia | Demanda, preço sugerido, VIPs | HUD / Reputação | ✅ |
| **Missões** | 8 metas em 3 tiers | BM$ + XP | Missões | ✅ |
| **Desafios diários** | 3 por dia real | BM$ + XP + ranking | Desafios | ✅ |
| **Conquistas** | 29 marcos | XP + BM$ | Popup | ✅ |
| **Arcos narrativos** | 5 × 3 capítulos | História, BM$, XP | Chat | 🟡 |
| **Reformas** | 7 melhorias | Bônus permanentes | Gestão | ✅ |
| **Prestígio** | Até 3× | Perks permanentes | — | 🟡 |

---

## 2. Nível do barbeiro

XP vem de atendimentos, missões, desafios, conquistas e capítulos narrativos.

| Nível | Título | XP para o próximo nível |
|---|---|---|
| 1 | Aprendiz da Navalha | 500 |
| 2 | Barbeiro de Bairro | 1.500 |
| 3 | Profissional da Cadeira | 3.500 |
| 4 | Mestre do Degradê | 7.000 |
| 5 | **Lenda AfroBarber** | — (máximo) |

**O que o nível libera:** reformas (níveis 1 a 3) ✅ · clientes VIP (nível 2) 🟡 · capítulos narrativos (níveis 1 a 4) 🟡 · prestígio (nível 5) 🟡.

**Efeito na execução** 🧩 (modo por etapas): +5% de qualidade e −5% de penalidade de tempo por nível acima do 1.

**Multiplicadores de XP**, aplicados em sequência a todo XP recebido:

| Ordem | Fonte | Valor | Status |
|---|---|---|---|
| 1 | Reformas | Som Ambiente × 1,10 · Decoração Afro × 1,05 · Recepção × 1,15 (acumulam) | ✅ |
| 2 | Perk de prestígio *BonusXP10* | × 1,10 | ✅ (depende do prestígio 🟡) |
| 3 | Evento cultural ativo | × 1,2 a × 2,0 | ✅ |
| — | Cliente Comunicativo · atendimento urgente | × 1,5 · × 1,1 | 🧩 |

---

## 3. Biblioteca de Cortes e maestria

### 3.1 Descoberta

*Status: ✅*
O corte feito pela primeira vez é **desbloqueado** na Biblioteca (popup "Novo corte descoberto"). Cortes ainda não feitos aparecem como **"???"**. O jogador marca **favoritos** e vê os **novos**. Conteúdo completo em [06](06-universo-cultural.md).

### 3.2 Maestria por corte

*Status: 🟡*
Cada corte tem sua barra de maestria. Todo atendimento dá XP de maestria conforme o resultado:

| Resultado | Horrível | Ruim | Mais ou Menos | Bom | Maravilhoso | Perfeito |
|---|---|---|---|---|---|---|
| XP de maestria | 1 | 3 | 6 | 10 | 16 | 25 |

O XP para subir de tier é o **valor base × dificuldade do corte** (1–5): dominar um Sisterlocks exige 5× mais que um Coily Puff.

| Tier | Nome | XP base | Qualidade | Penalidade de tempo | Recompensa |
|---|---|---|---|---|---|
| 1 | Aprendiz da Navalha | 40 | — | — | — |
| 2 | Estilista de Bairro | 90 | +2% | −8% | +3% |
| 3 | Especialista da Cadeira | 160 | +5% | −15% | +6% |
| 4 | Mestre do Corte | 260 | +8% | −22% | +10% |
| 5 | **Lenda do Estilo** | máximo | +12% | −30% | +15% |

Subir de tier gera mensagem no chat e alimenta 5 conquistas e o desafio "Evolução Contínua". Os bônus de qualidade e tempo valem no modo por etapas.

**Escala total:** levar os 24 cortes a Lenda exige 550 × 75 (soma das dificuldades) = **41.250 pontos** — cerca de 1.650 atendimentos Perfeitos. Ajuste recomendado em [07 §6](07-estado-atual-e-roadmap.md#6-balanceamento).

---

## 4. Fidelidade de clientes

Cada cliente tem um identificador único e o jogo conta suas visitas ✅. Os bônus por nível são 🧩 (configurados, ainda não aplicados):

| Nível | Visitas | Gorjeta | Paciência extra | Chance de elogio espontâneo |
|---|---|---|---|---|
| Novo | 0–1 | × 1,00 | — | — |
| Conhecido | 2–4 | × 1,10 | +5 min | 5% |
| Regular | 5–9 | × 1,25 | +10 min | 10% |
| Fiel | 10–19 | × 1,50 | +15 min | 18% |
| **Lendário** | 20+ | × 1,85 | +25 min | 30% |

A fala *"Deixa eu ver aqui... quantas vezes você já veio?"* revela o nível e o bônus do cliente ✅. A perk *GorjetaExtra20* multiplica a gorjeta de fidelidade por mais 1,2 🧩.

---

## 5. Reputação

*Status: ✅*

- Nota média **0–5**, começando em **3,0**, com subnotas de **Atendimento**, **Estrutura** e **Experiência**.
- Cada atendimento soma uma avaliação; reformas acrescentam **bônus de nota**.
- **Efeitos:** demanda de clientes (de × 0,5 a × 1,8; 3,0 é neutro), frequência de agendamentos, **preço sugerido** (de × 0,75 a × 1,45) e requisito dos VIPs (≥ 3,5 🟡).
- **Penalidades:** conta mensal em atraso registra uma avaliação com nota (média − 0,1) — o efeito real é menor que 0,1; empréstimo em atraso também penaliza.
- ⚠️ No modo minigame, a nota de um atendimento "Bom" já conta como 5,0, o que infla a reputação (ver [07](07-estado-atual-e-roadmap.md)).

---

## 6. Arcos narrativos

*Status: 🟡*

Cinco moradores têm histórias em **3 capítulos**. Cada capítulo exige um número de **visitas** do personagem, um **nível mínimo** do jogador e uma quantidade de **atendimentos** no capítulo; capítulos que pagam dinheiro exigem resultado **Bom ou melhor**. Ao concluir, o desfecho aparece no chat e o prêmio é pago. O roteiro e a lógica estão prontos; falta atribuir os personagens a clientes da cena.

### Seu Ribeiro — *o legado da barbearia do bairro* (Afro Clássico)
| Cap. | Título | Requisitos | Objetivo | Recompensa |
|---|---|---|---|---|
| 1 | O Velho dos Velhos | 1 visita · nível 1 | Primeiro atendimento ("seja paciente, ele vai falar muito") | 30 XP |
| 2 | A História do Bairro | 3 visitas · nível 2 | Mais 2 atendimentos — ele traz a foto da barbearia do Geraldo, que ficava **no mesmo lugar** | BM$ 200 · 60 XP |
| 3 | O Legado Vivo | 6 visitas · nível 3 | Primeiro corte do neto **Caio** (12 anos) | BM$ 500 · 150 XP |

> *"Seu Ribeiro abraça o neto enquanto os dois olham no espelho. 'A tradição continua.' — Você ganhou um cliente fiel eterno."*

### Kemi — *autoestima e identidade* (Box Braids)
| Cap. | Título | Requisitos | Objetivo | Recompensa |
|---|---|---|---|---|
| 1 | A Cliente Difícil | 1 visita · nível 1 | Resultado Bom ou melhor | 25 XP |
| 2 | Além da Aparência | 4 visitas · nível 2 | 3 atendimentos de qualidade — ela conta que cresceu odiando o cabelo por causa da escola | BM$ 300 · 80 XP |
| 3 | O Projeto Cultural | 7 visitas · nível 3 | 1 atendimento **Perfeito**, fotografado para a exposição dela | BM$ 400 · 120 XP |

> *"A exposição de Kemi traz 3 novos clientes… Sua barbearia virou ponto cultural do bairro."*

### Gabriel — *ascensão social e autenticidade* (Shape-Up)
| Cap. | Título | Requisitos | Objetivo | Recompensa |
|---|---|---|---|---|
| 1 | A Grande Entrevista | 1 visita · nível 1 | Shape-Up **Maravilhoso ou Perfeito** para a entrevista de estágio | 40 XP |
| 2 | Dois Mundos | 4 visitas · nível 2 | Conversar e atender mais 2 vezes — colegas criticam o cabelo dele | BM$ 250 · 70 XP |
| 3 | Raízes que Sustentam | 7 visitas · nível 4 | Último atendimento — ele foi promovido | BM$ 600 · 180 XP |

> *"O envelope contém uma carta de recomendação para um prêmio local de 'Negócio Cultural do Bairro'."*

### Dona Conceição — *educação e memória* (Corte César)
| Cap. | Título | Requisitos | Objetivo | Recompensa |
|---|---|---|---|---|
| 1 | A Senhora dos Saberes | 1 visita · nível 1 | Cortar o cabelo do neto **Theo** (10 anos) | 25 XP |
| 2 | Memória e Resistência | 4 visitas · nível 2 | Receber avó e neto mais 3 vezes — ela nota os painéis educativos | BM$ 300 · 80 XP |
| 3 | A Roda de Conversa | 8 visitas · nível 3 | Dia do evento: **5 clientes Bom ou melhor** | BM$ 500 · 200 XP |

> *"O evento traz 8 novos clientes… Dona Conceição escreve sobre sua barbearia num jornal local."*

### Leandro "Leo" — *arte e criação* (Afro com Degradê)
| Cap. | Título | Requisitos | Objetivo | Recompensa |
|---|---|---|---|---|
| 1 | O Artista Inquieto | 1 visita · nível 1 | Primeiro Afro com Degradê para destravar a música | 30 XP |
| 2 | Música e Memória | 4 visitas · nível 2 | 2 atendimentos durante a composição — sample de samba de roda da avó | BM$ 200 · 65 XP |
| 3 | O Show | 7 visitas · nível 3 | Corte **Perfeito** para o show de lançamento | BM$ 450 · 130 XP |

> *"As fotos do show circulam nas redes com a hashtag #AfroBarber… 5 novos clientes chegam mencionando o show dele."*

**Total da narrativa:** 15 capítulos · BM$ 3.700 · 1.285 XP.

---

## 7. Missões

*Status: ✅*

Metas por **tiers**: ao atingir um degrau, o jogador **resgata** a recompensa no painel e passa ao próximo. Há missões permanentes, diárias, semanais e anuais.

| Missão | Frequência | Métrica | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|---|---|
| **Dia Produtivo** — *"Todo grande barbeiro começa o dia com um objetivo."* | Diária | Clientes atendidos no dia | 3 → BM$ 200 + 10 XP | 5 → BM$ 500 + 25 XP | 8 → BM$ 1.000 + 50 XP |
| **Mãos à Obra** — *"Barbeiro bom não para."* | Diária | Minutos de atendimento no dia | 30 → BM$ 150 + 8 XP | 60 → BM$ 400 + 20 XP | 120 → BM$ 800 + 40 XP |
| **Semana dos Campeões** — *"Mostre que sua barbearia é um negócio sério."* | Semanal | Faturamento na semana | BM$ 2.000 → BM$ 500 + 20 XP | BM$ 5.000 → BM$ 1.500 + 50 XP | BM$ 10.000 → BM$ 3.000 + 100 XP |
| **Artesão do Black Power** — *"Domine esse corte icônico."* | Permanente | Cortes Black Power | 5 → BM$ 300 + 30 XP | 15 → BM$ 800 + 75 XP | 30 → BM$ 2.000 + 200 XP |
| **Nota Máxima** — *"A reputação da sua barbearia fala por você."* | Permanente | Nota média | 3,5 → BM$ 800 + 25 XP | 4,2 → BM$ 2.000 + 60 XP | 4,8 → BM$ 5.000 + 150 XP |
| **Sequência Perfeita** — *"Prove que sua maestria não foi sorte."* | Permanente | Perfeitos seguidos | 3 → BM$ 600 + 40 XP | 5 → BM$ 1.500 + 100 XP | 10 → BM$ 4.000 + 250 XP |
| **Disposição de Ferro** — *"Barbeiro bom cuida do próprio corpo."* | Permanente | Dias seguidos sem desmaiar | 3 → BM$ 400 + 20 XP | 7 → BM$ 1.000 + 50 XP | 15 → BM$ 3.000 + 120 XP |
| **Consciência em Ação** — *"Cada corte afro é um ato de celebração da identidade."* | Anual (novembro) | Clientes em novembro | 10 → BM$ 1.000 + 50 XP | 30 → BM$ 3.000 + 150 XP | 60 → BM$ 8.000 + 400 XP |

O sistema também aceita missões em **janela de evento** e por **corte específico**, criadas apenas como dados.

---

## 8. Desafios diários

*Status: ✅*

- **3 por dia**, sorteados entre 6 tipos e renovados pelo **relógio real**.
- Metas e recompensas escalam com o **nível** (N) do jogador.
- Ao concluir, aviso no chat; a recompensa é **coletada** no painel.

| Desafio | Objetivo | Meta | Recompensa |
|---|---|---|---|
| **Mãos que Trabalham** | Atender clientes | 3 + N | BM$ 80·N + 15·N XP |
| **Caixa Forte** | Faturar | BM$ 400·N | BM$ 120·N + 20·N XP |
| **Artesão do Dia** | Atendimentos Perfeitos | 2 + N/2 | BM$ 180·N + 35·N XP |
| **Dia Impecável** | Atendimentos do dia sem Ruim ou Horrível ⚠️ | 5 | BM$ 250·N + 45·N XP |
| **Sequência de Ouro** | Atendimentos seguidos sem Ruim ou Horrível | 4 | BM$ 200·N + 30·N XP |
| **Evolução Contínua** 🟡 | Pontos de maestria | 20 + 8·N | BM$ 100·N + 25·N XP |

⚠️ "Dia Impecável": um resultado Ruim zera o progresso exibido, mas o atendimento seguinte restaura a contagem acumulada do dia — regra a definir (ver [07 §6](07-estado-atual-e-roadmap.md#6-balanceamento)).

**Ranking:** a barbearia do jogador, com o apelido, disputa com 9 barbearias fictícias; a pontuação delas é regerada a cada sessão para dar sensação de "ao vivo".

---

## 9. Conquistas

*Status: ✅*

29 conquistas; cada uma dispara popup e paga XP e/ou BM$. "Bom atendimento" = nota ≥ 3,5.

| Grupo | Conquista | Condição | Recompensa |
|---|---|---|---|
| **Atendimentos** | Primeiro Corte | 1 atendimento | 50 XP |
| | 10 Clientes | 10 | 100 XP |
| | Barbeiro Movimentado | 50 | 250 XP + BM$ 200 |
| | Centenário | 100 | 500 XP + BM$ 500 |
| | Lenda da Barbearia | 500 | 2.000 XP + BM$ 2.000 |
| **Qualidade** | Mãos de Ouro | Nota ≥ 4,8 num atendimento | 200 XP |
| | Aquecendo | 3 bons atendimentos seguidos | 100 XP |
| | Em Chamas | 5 seguidos | 300 XP |
| | Imbatível | 10 seguidos | 600 XP + BM$ 300 |
| | Fenômeno | 20 seguidos | 1.500 XP + BM$ 1.000 |
| **Nível** | Barbeiro de Bairro | Nível 2 | — |
| | Profissional da Cadeira | Nível 3 | BM$ 500 |
| | Mestre do Degradê | Nível 4 | BM$ 1.000 |
| | Lenda AfroBarber | Nível 5 | 500 XP + BM$ 3.000 |
| **Receita** | Primeiro Milhar | BM$ 1.000 faturados | 100 XP |
| | Dez Mil! | BM$ 10.000 | 300 XP + BM$ 500 |
| | Empresário de Sucesso | BM$ 50.000 | 800 XP + BM$ 2.000 |
| | Império AfroBarber | BM$ 100.000 | 2.000 XP + BM$ 5.000 |
| **Reformas** | Primeira Reforma | 1 reforma | 100 XP |
| | Barbearia Modernizada | 5 reformas | 300 XP + BM$ 300 |
| | Barbearia Premium ⚠️ | 10 reformas (existem 7) | 600 XP + BM$ 1.000 |
| **Cultura** | Historiador Iniciante | 1 corte na Biblioteca | 50 XP |
| | Guardião da Cultura | 10 cortes | 300 XP |
| | Mestre da Enciclopédia Afro | 24 cortes | 1.000 XP + BM$ 2.000 |
| **Maestria** 🟡 | Mão Afiada | Especialista em 1 corte | 150 XP |
| | Mão de Mestre | Mestre em 1 corte | 400 XP + BM$ 300 |
| | Lenda Viva | Lenda do Estilo em 1 corte | 800 XP + BM$ 800 |
| | Multi-Talentoso | Lenda em 5 cortes | 1.500 XP + BM$ 1.500 |
| | Mestre Supremo AfroBarber ⚠️ | Lenda em 25 cortes (existem 24) | 3.000 XP + BM$ 5.000 |

Soma das recompensas: **17.500 XP** e **BM$ 26.900** — mais XP do que os 12.500 necessários para ir do nível 1 ao 5 (ver [07 §6](07-estado-atual-e-roadmap.md#6-balanceamento)). Ressalvas (conquistas inalcançáveis, conquistas facilitadas pela nota inflada, contagem da Biblioteca afetada por IDs de eventos) em [07 §5](07-estado-atual-e-roadmap.md#5-inconsistências-de-conteúdo-e-design).

---

## 10. Reformas da barbearia

*Status: ✅*

Compradas na **Gestão**; algumas exigem a anterior. Os bônus são permanentes. A aparição física na cena está prevista (cada reforma aponta para um objeto), mas esses objetos ainda não existem na `GameScene` 🧩.

| Reforma | Custo (BM$) | Nível | Requer | Atração | Nota | Energia | XP | Descrição |
|---|---|---|---|---|---|---|---|---|
| **Espelhos Profissionais** | 400 | 1 | — | × 1,15 | +0,05 | — | — | Espelhos de corpo inteiro com moldura trabalhada. |
| **Som Ambiente** | 500 | 1 | — | × 1,05 | — | −5% | × 1,10 | Caixas com playlist de **afrobeat e soul**. |
| **Iluminação LED** | 600 | 1 | Espelhos | × 1,10 | +0,08 | — | — | LED branco frio e arandelas que eliminam sombras. |
| **Cadeira Premium** | 800 | 2 | — | × 1,10 | +0,10 | — | — | Cadeira hidráulica reclinável de couro sintético. |
| **Decoração Afro** | 1.000 | 2 | Iluminação | × 1,20 | +0,12 | — | × 1,05 | **Painéis com arte afro-brasileira**, plantas e referências culturais. |
| **Ar-Condicionado** | 1.500 | 3 | Cadeira Premium | × 1,15 | +0,05 | −15% | — | O barbeiro cansa menos. |
| **Área de Recepção** | 2.000 | 3 | Decoração Afro | × 1,25 | +0,10 | — | × 1,15 | Sofá, revistas e atendente virtual. |

**Árvore:** Espelhos → Iluminação → Decoração Afro → Recepção · Cadeira Premium → Ar-Condicionado · Som Ambiente (independente).
**Total:** BM$ 6.800; com todas, a atração soma ≈ × 2,5 e o XP ≈ × 1,33.

---

## 11. Eventos culturais

*Status: ✅ / ⚠️*

Ativados pela data do jogo e anunciados no chat. Os bônus funcionam ✅; o desbloqueio automático de cortes usa identificadores errados ⚠️ (ver [07](07-estado-atual-e-roadmap.md)).

| Evento | Quando | XP | Dinheiro | Cortes em destaque |
|---|---|---|---|---|
| Carnaval | Fevereiro (fixo no jogo; a data real varia) | × 1,3 | × 1,4 | Moicano Afro, Burst Fade, Afro Degradê |
| Julho afro-latino-americano (nome em revisão — sugestão: 25/07, Tereza de Benguela) | Julho | × 1,2 | × 1,1 | Corte César, Cornrows, Box Braids |
| Dia do Turbante (fonte a confirmar) | 25/09 | × 1,5 | × 1,3 | Turbante |
| Mês da Consciência Negra | Novembro | × 1,5 | × 1,2 | Black Power Clássico, Tranças Nagô, Cornrows, Turbante |
| Dia da Consciência Negra | 20/11 | **× 2,0** | × 1,5 | Dreads/Locs, Afro Natural, Conk |

- Quando dois eventos coincidem (ex.: 20/11 dentro de novembro), vale o **maior** bônus de cada tipo — não se somam.
- ⚠️ Como a data do jogo não é salva, eventos fora de abril só ocorrem em sessões longas ([07 §2.2](07-estado-atual-e-roadmap.md#22-a-data-do-jogo-não-é-salva)).
- Nomes e datas em revisão cultural ([07 §7.1](07-estado-atual-e-roadmap.md#71-conteúdo-cultural-a-corrigir-ou-confirmar)).

Contexto cultural de cada data em [06 §3](06-universo-cultural.md#3-calendário-cultural).

---

## 12. Prestígio

*Status: 🟡*

- **Requisito:** nível 5; até **3 prestígios**. A lógica está pronta; falta a tela de escolha.
- **Reinicia:** XP e nível; caixa volta a BM$ 200.
- **Mantém:** nível de prestígio, perks e histórico; a reputação só é redefinida com a perk *ReputacaoInicial*.
- **Escolha de 1 perk por prestígio:**

| Perk | Efeito | Status |
|---|---|---|
| **BonusXP10** | +10% de XP em tudo | ✅ |
| **GorjetaExtra20** | +20% de gorjeta (via fidelidade) | 🧩 |
| **ReputacaoInicial** | Reputação redefinida para 3,5 | ✅ |
| **AlugueMenor15** | −15% no aluguel | 🧩 |
| **DinheiroInicial500** | +BM$ 500 ao recomeçar (BM$ 700) | ✅ |

---

## 13. Curva de progressão por fase

> Curva **pretendida**. Hoje não há trava de corte por nível: a agenda sorteia qualquer um dos 24 cortes desde o primeiro dia.

| Fase | Nível | Foco do jogador | Conteúdo típico |
|---|---|---|---|
| **Abertura** | 1 | Aprender o fluxo; kit básico de ferramentas; pagar o primeiro aluguel | Coily Puff, Afro Natural, Turbante, Line Up, Shape-Up; Espelhos e Som Ambiente |
| **Consolidação** | 2 | Fidelizar, manter contas em dia, melhorar ferramentas | Black Power, Twists, Temple Fade; Iluminação, Cadeira Premium, Decoração Afro |
| **Profissionalização** | 3 | Cortes difíceis, reformas caras, missões semanais | Box Braids, Cornrows, Tranças Nagô, Locs, Burst Fade; Ar-Condicionado, Recepção |
| **Maestria** | 4 | Consistência (Sequência Perfeita, Nota Máxima 4,8) | Skin Fade, Sisterlocks; capítulos finais |
| **Lenda** | 5 | Completar Biblioteca e conquistas; prestigiar | Todo o catálogo |
