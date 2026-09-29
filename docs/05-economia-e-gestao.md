# AfroBarber — Economia e Gestão

| | |
|---|---|
| **Documento** | 05 · Economia, preços, demanda e catálogo |
| **Versão** | 1.3 — 29/09/2026 |
| **Leitores** | Designers de sistemas, balanceamento, produção |
| **Relacionados** | [02 Jogabilidade](02-guia-de-jogabilidade.md) · [04 Progressão](04-progressao-e-objetivos.md) · [07 Estado atual](07-estado-atual-e-roadmap.md) |

> Marcas: ✅ ativo · 🟡 pronto, não ativo · 🧩 regra projetada, ainda fora do cálculo · ⚠️ com ressalva — ver [legenda](README.md#2-legenda-de-status).

## Sumário
1. [Visão geral da economia](#1-visão-geral-da-economia)
2. [Caixa e extrato](#2-caixa-e-extrato)
3. [Receita de um atendimento](#3-receita-de-um-atendimento)
4. [Preços](#4-preços)
5. [Demanda de clientes](#5-demanda-de-clientes)
6. [Horário e hora extra](#6-horário-e-hora-extra)
7. [Contas mensais](#7-contas-mensais)
8. [Empréstimos](#8-empréstimos)
9. [Loja, itens e desgaste](#9-loja-itens-e-desgaste)
10. [Catálogo de produtos](#10-catálogo-de-produtos)
11. [Mobília e personalização](#11-mobília-e-personalização)
12. [Relatórios](#12-relatórios)
13. [Ciclo econômico e balanceamento](#13-ciclo-econômico-e-balanceamento)

---

## 1. Visão geral da economia

```text
         ┌───────────── ENTRADAS ─────────────┐        ┌────────────── SAÍDAS ──────────────┐
         │ Atendimentos (preço × resultado)   │        │ Loja (ferramentas, produtos, móveis)│
         │ Gorjetas                           │        │ Reformas                            │
         │ Bônus de evento cultural           │  CAIXA │ Aluguel, luz, água                  │
         │ Missões, desafios, conquistas,     │ ─────► │ Multas (contas e empréstimo)        │
         │ capítulos narrativos               │        │ Parcelas e juros de empréstimo      │
         │ Empréstimos                        │        │                                     │
         └────────────────────────────────────┘        └─────────────────────────────────────┘
                ▲                                                      │
                └──── reputação, preço, clima, reformas → DEMANDA ◄────┘
```

A economia busca uma tensão constante: **o aluguel chega todo mês**, o cansaço limita quantos clientes cabem num dia e o preço afeta quantos clientes vêm.

---

## 2. Caixa e extrato

*Status: ✅*

- Moeda: **BM$**. Um único caixa (`FinanceManager`) guarda saldo e **extrato completo**, salvos automaticamente.
- **Caixa inicial:** BM$ 100 na cena (padrão do código: BM$ 1.000). Após prestígio: BM$ 200 (BM$ 700 com a perk *DinheiroInicial500*).
- O caixa **pode ficar negativo**, o que dispara a **crise financeira** (§7).
- O extrato registra cada movimentação com título, descrição, origem e data, agrupado por mês.

---

## 3. Receita de um atendimento

### 3.1 Modo minigame (atual)

*Status: ✅*
```text
Receita = Preço cobrado × fator do resultado + Gorjeta
Fator:   Horrível 0,45 · Ruim 0,65 · Mais ou Menos 0,85 · Bom 1,00 · Maravilhoso 1,15 · Perfeito 1,30
Gorjeta: 15% do preço cobrado em Maravilhoso ou Perfeito
XP     = XP do corte × o mesmo fator
```
Sobre o total incidem os bônus de **evento cultural** (dinheiro) e, no XP, reformas, prestígio e evento.

### 3.2 Modo por etapas (alternativo)
```text
Receita = Preço × (0,55 a 1,40 conforme a nota 0–5) × urgência × maestria + gorjeta por personalidade
```

### 3.3 Bônus previstos, ainda fora do cálculo

*Status: 🧩*
| Bônus | Valor configurado |
|---|---|
| Cliente VIP — preço | × 3 |
| Cliente VIP — gorjeta | × 2,5 |
| Fidelidade — gorjeta | × 1,10 a × 1,85 |
| Clima — gorjeta | × 1,00 a × 1,35 |
| Perk *GorjetaExtra20* | × 1,2 |

---

## 4. Preços

### 4.1 Como o preço é calculado

*Status: ✅ / ⚠️*
Hoje o preço cobrado vem da **tabela central por tipo de serviço** (`MainServicePriceTable`), com o ajuste percentual definido pelo jogador na Gestão:

```text
Preço cobrado = Preço da tabela para o tipo de serviço × (1 + ajuste % da Gestão)
```

| Tipo de serviço | Preço de tabela (BM$) | Cortes que usam |
|---|---|---|
| Corte de Cabelo | 35 | 21 cortes |
| Corte de Cabelo e Barba | 55 | — |
| Barba | 25 | — |
| Acabamento/Pezinho | 15 | Line Up, Shape-Up |
| Design de Sobrancelhas | 20 | — |
| Hidratação | 40 | — |
| Outro | — (sem entrada) | Turbante |

⚠️ Consequências atuais:
- Cada corte tem um **preço de referência** próprio (BM$ 20 a 80 — ver [06](06-universo-cultural.md#21-tabela-resumo)), mas ele **não é usado**: um Sisterlocks e um Coily Puff custam o mesmo.
- O **Turbante** aparece com preço **0** no pedido (no minigame recebe um valor reserva de BM$ 50).

**Direção recomendada:** usar o preço de referência do corte como base e a tabela como ajuste por tipo de serviço (ver [07](07-estado-atual-e-roadmap.md)).

### 4.2 Preço sugerido

*Status: ✅*
```text
Preço sugerido = Preço de tabela × multiplicador da reputação (0,75 a 1,45)
```
Reputação alta permite cobrar mais sem perder clientes.

### 4.3 Satisfação com o preço

*Status: ✅*
Após cada atendimento, o preço cobrado é comparado ao sugerido:

| Situação | Efeito na demanda |
|---|---|
| Acima do sugerido | Demanda × (1 − 0,15 × % acima), mínimo × 0,35 |
| Igual | × 1,00 |
| Abaixo do sugerido | Demanda × (1 + 0,10 × % abaixo), máximo × 1,6 |

---

## 5. Demanda de clientes

*Status: ✅*

A frequência de novos clientes é o produto de cinco fatores:

```text
Demanda = Preço × Contas em atraso × Reputação × Reformas × Clima
```

| Fator | Faixa | Origem |
|---|---|---|
| Preço | 0,35 – 1,6 | Preço cobrado vs. sugerido (§4.3) |
| Contas em atraso | −20% enquanto houver conta atrasada | Contas mensais (§7) |
| Reputação | 0,5 – 1,8 (3,0 = neutro) | Nota média |
| Reformas | até ≈ 2,5 | Atração das reformas |
| Clima | 0,25 – 1,25 | Clima do dia |

Posts positivos no BarberBook também geram **clientes orgânicos** na virada do dia.

---

## 6. Horário e hora extra

*Status: ✅*

- Horário sugerido: **10h–19h, terça a sábado** (aplicado ao iniciar).
- Na **Gestão**, o jogador define abertura, fechamento e dias de funcionamento.
- **Hora extra:** cada hora além do horário sugerido aumenta em **8%** o gasto de energia.
- As horas trabalhadas no mês **encarecem a luz e a água do mês seguinte**.

---

## 7. Contas mensais

*Status: ✅*

| Conta | Vencimento | Valor (cena) | Cálculo |
|---|---|---|---|
| **Aluguel** | Dia 5 | BM$ 700 | Fixo (padrão do código: BM$ 1.800); perk *AlugueMenor15* 🧩 |
| **Energia elétrica** | Dia 10 | BM$ 220 + BM$ 7/hora | Horas trabalhadas no mês anterior |
| **Água** | Dia 12 | BM$ 90 + BM$ 3/hora | Horas trabalhadas no mês anterior |

⚠️ **Situação atual** (detalhes em [07 §2.3](07-estado-atual-e-roadmap.md#23-contas-de-abril-nascem-vencidas)):
- as contas de abril nascem no primeiro dia (10/04), com o **aluguel já vencido** — BM$ 1.010 contra BM$ 100 de caixa;
- aviso e multa só disparam com atraso de **exatamente** 1 e 3 dias; o aluguel de abril, que nasce com 5, nunca é multado, mas pesa na dívida vencida;
- como a data não é salva, avisos e multas podem se repetir a cada sessão.

**Exemplo:** 22 dias × 9 h = 198 h → energia BM$ 1.606 · água BM$ 684 · total do mês com aluguel ≈ **BM$ 2.990**.

**Atrasos**
| Momento | Consequência |
|---|---|
| 1 dia após o vencimento | Aviso no chat: *"… venceu ontem! Pague logo para evitar multa."* |
| 3 dias de atraso | Multa de **5%**, uma avaliação com nota (média − 0,1) e **−20% de demanda** |
| Dívida vencida > BM$ 1.000 | Perda diária de energia, crescente com os dias de atraso |
| Caixa negativo | **CRISE FINANCEIRA:** *"Seu saldo está negativo. Atenda mais clientes ou considere um empréstimo…"* |
| Pagamento | Previsto: mensagem *"'Aluguel' pago!"* e restauração da demanda — hoje não acontecem (a notificação de pagamento não é chamada) ⚠️ |

---

## 8. Empréstimos

*Status: ✅*

Um empréstimo ativo por vez.

| Opção | Valor | Juros por semana | Prazo | Saldo no prazo, sem pagamentos* |
|---|---|---|---|---|
| 1 | BM$ 500 | 5% | 4 semanas | ≈ BM$ 608 |
| 2 | BM$ 1.000 | 8% | 6 semanas | ≈ BM$ 1.587 |
| 3 | BM$ 2.500 | 12% | 10 semanas | ≈ BM$ 7.765 |
| 4 | BM$ 5.000 | 18% | 16 semanas | ≈ BM$ 70.645 |

\* Juros compostos semanais; os valores mostram por que empréstimos longos e caros exigem pagamento antecipado.

- Vencido o prazo: status **Em Atraso**, **multa de 3%** do saldo e penalidade de reputação.
- **Pagar tudo** quita o saldo do empréstimo ativo (debita só o devido).

---

## 9. Loja, itens e desgaste

### 9.1 Tipos de item
| Tipo | Como se gasta | Exemplos |
|---|---|---|
| **Durável** | Horas de uso; a condição cai até quebrar | Máquinas, tesouras, pentes, navalhas, secador, móveis |
| **Consumível** | Usos por atendimento | Cremes e produtos capilares |
| **Caixa com unidades** | Unidades | Lâminas |

### 9.2 Atributos
Ferramentas têm **Precisão**, **Velocidade** e **Durabilidade** (0–100); móveis e decoração têm **Conforto**, **Estética** e **Tempo de entrega**. Raridades previstas: Comum, Raro, Épico, Lendário.

### 9.3 Uso no jogo
| Regra | Status |
|---|---|
| Ter ferramenta compatível para cada etapa do plano | ✅ |
| Escolha de ferramenta lembrada por corte | ✅ |
| Descartar ferramentas quebradas | ✅ |
| Qualidade da ferramenta influenciar a nota | 🧩 (só no modo por etapas) |
| Consumo de produtos e desgaste por atendimento | 🧩 — depende dos requisitos dos cortes, ainda vazios |

---

## 10. Catálogo de produtos

**Máquinas de corte** — etapas Cortar e Barba
| Produto | Preço | Precisão | Velocidade | Durabilidade | Média |
|---|---|---|---|---|---|
| BasicCut Starter | 99 | 5 | 12 | 40 | 19 |
| TrimLite Home | 129 | 21 | 43 | 21 | 28 |
| CutPro 200 | 350 | 35 | 28 | 47 | 37 |
| PowerCut Elite | 399 | 60 | 49 | 38 | 49 |
| FadeMaster X | 430 | 70 | 84 | 71 | 75 |
| GoldenEdge Supreme | 499 | 87 | 94 | 65 | 82 |
| FadeX Control | 699 | 47 | 61 | 26 | 45 ⚠️ |
| UrbanBlade Pro | 850 | 53 | 34 | 67 | 51 ⚠️ |

**Tesouras** — Cortar e Barba
| Produto | Preço | Precisão | Velocidade | Durabilidade | Média |
|---|---|---|---|---|---|
| EdgeFlow X | 35 | 7 | 21 | 14 | 14 |
| BlackShear Pro | 42 | 21 | 16 | 34 | 24 |
| FadeCut Prime | 49 | 46 | 31 | 25 | 34 |
| SteelLine V | 65 | 59 | 75 | 34 | 56 |
| SharpCore Elite | 140 | 68 | 85 | 55 | 69 |
| BladeCraft Neo | 189 | 93 | 94 | 64 | 84 |

**Pentes** — Pentear, Acabamento, Definir, Finalizar
| Produto | Preço | Precisão | Velocidade | Durabilidade | Média |
|---|---|---|---|---|---|
| GripComb X | 15 | 6 | 9 | 5 | 7 |
| FadeLine Pro | 19 | 11 | 5 | 9 | 8 |
| WaveMaster | 24 | 12 | 18 | 6 | 12 |
| StyleFlow V | 32 | 23 | 17 | 25 | 22 |
| PrecisionComb Elite | 39 | 34 | 27 | 34 | 32 |
| CurlMaster X | 42 | 36 | 29 | 47 | 37 |
| BladeComb V | 49 | 30 | 45 | 36 | 37 |
| AfroPick Pro | 69 | 53 | 47 | 60 | 53 |
| SharpLine Comb | 75 | 71 | 63 | 40 | 58 |
| FlowPick Elite | 79 | 72 | 58 | 65 | 65 |
| UrbanComb X | 83 | 85 | 78 | 65 | 76 |
| TextureMaster Pro | 99 | 91 | 65 | 87 | 81 |

**Navalhas** — Navalha e Barba
| Produto | Preço | Precisão | Velocidade | Durabilidade | Média |
|---|---|---|---|---|---|
| Navalha 1 | 50 | 10 | 19 | 10 | 13 |
| Navalha 2 | 89 | 35 | 31 | 41 | 36 |
| Navalha 3 | 149 | 54 | 65 | 43 | 54 |
| Navalha 4 | 349 | 86 | 72 | 78 | 79 |

**Lâminas** — Navalha · *Lâminas 1*: BM$ 29 (8 / 23 / 15), caixa com unidades.

**Produtos capilares** — Lavar, Acabamento, Definir, Finalizar (consumíveis)
| Produto | Preço | Precisão | Velocidade | Durabilidade | Média |
|---|---|---|---|---|---|
| Cremes 1 | 145 | 13 | 25 | 18 | 19 |
| Cremes 2 | 149 | 30 | 25 | 37 | 31 |
| Cremes 3 | 199 | 47 | 45 | 65 | 52 |
| Cremes 4 | 240 | 61 | 75 | 54 | 63 |
| Cremes 5 | 490 | 89 | 80 | 75 | 81 |

**Secador** — *Secador 1*: BM$ 399 (16 / 30 / 23). ⚠️ Nenhuma etapa aceita secador hoje.

**Mobília**
| Item | Preço |
|---|---|
| Cadeira de barbeiro Almofadada | 800 |
| Cadeira de barbeiro Clássica | 1.200 |
| Mesa de Bilhar | 1.190 |
| Fliperama Pac-Man | 1.600 |

**Decoração** — categoria preparada na loja, alimentada por uma biblioteca padrão de móveis e decoração.

**Kit para começar.** O jogador começa **sem itens** no inventário.
- **Plano mínimo** (Cortar → Finalizar): tesoura + pente — a partir de **BM$ 50** (EdgeFlow X + GripComb X).
- **Plano padrão** (Lavar → Cortar → Acabamento → Finalizar): acrescenta um produto capilar — a partir de **BM$ 195**. ⚠️ Acima do caixa inicial de BM$ 100.

---

## 11. Mobília e personalização

*Status: ✅*

- Móveis e decoração comprados vão ao Inventário e podem ser **colocados** ou **removidos** em pontos da barbearia (até 12 itens); a disposição é salva.
- Cada móvel da loja tem um ponto definido na cena; as cadeiras compradas substituem a cadeira padrão.
- As **reformas** dão bônus permanentes ✅; a aparição física na cena está prevista 🧩 (ver [04 §10](04-progressao-e-objetivos.md#10-reformas-da-barbearia)).

---

## 12. Relatórios

*Status: 🟡*

O `BusinessReportManager` registra por dia **clientes atendidos**, **receita** e **nota média**, com histórico — base para uma futura tela de relatório.

---

## 13. Ciclo econômico e balanceamento

### 13.1 Ciclo típico
```text
Início     Caixa de BM$ 100: comprar tesoura + pente (BM$ 50); plano mínimo; juntar para o creme.
           Contas do 1º mês (≈ BM$ 1.010) — hoje nascem vencidas; o desenho pretendido é começar no início do mês.
Semana 2   Primeiras reformas baratas (Espelhos, Som Ambiente). Preço no sugerido.
Mês 1      Luz e água crescem com as horas trabalhadas. Empréstimo pequeno se necessário.
Meses 2–3  Ferramentas Pro, Cadeira Premium, Decoração Afro. Reputação ↑ → preço sugerido ↑.
Mês 4+     Recepção e Ar-Condicionado; metas semanais de faturamento.
```

### 13.2 Referências de balanceamento
| Grandeza | Valor atual |
|---|---|
| Receita de um corte "Bom" | BM$ 35 (preço de tabela) |
| Receita de um corte "Perfeito" | ≈ BM$ 51 (35 × 1,30 + 15% de gorjeta) |
| Clientes por expediente | limitado pela agenda, pela duração dos cortes e pela energia |
| Custo fixo mensal | ≈ BM$ 1.000 a 3.000 (aluguel + contas variáveis) |
| Reformas completas | BM$ 6.800 |

### 13.3 Pontos de atenção
- Preço único por tipo de serviço reduz o interesse por cortes difíceis (§4.1); o Turbante, o mais fácil, recebe a reserva de BM$ 50 no minigame — mais que os BM$ 35 dos outros cortes.
- Meta "Semana dos Campeões" tier 3 (BM$ 10.000) acima do teto semanal estimado (~BM$ 5.600).
- Consumo e desgaste inativos eliminam custos recorrentes de estoque (§9.3).
- Itens caros com atributos piores (FadeX Control, UrbanBlade Pro).
- Inventário inicial vazio e caixa de BM$ 100 não cobrem o plano padrão com produto capilar.
- Empréstimo de BM$ 5.000 a 18% por semana cresce ~14× em 16 semanas — é punitivo por design, mas vale sinalizar ao jogador.

Detalhes e sugestões em [07](07-estado-atual-e-roadmap.md).
