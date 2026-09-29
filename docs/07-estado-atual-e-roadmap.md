# AfroBarber — Estado Atual e Roadmap

| | |
|---|---|
| **Documento** | 07 · Estado da build, pendências e roadmap |
| **Versão** | 1.2 — 29/09/2026 |
| **Leitores** | Produção, desenvolvimento, design, proponente de editais |
| **Base do levantamento** | Código (`Assets/Afrobarber/Scripts`), assets (`ScriptableObjects`, prefabs) e `GameScene.unity` em 29/09/2026 |

## Sumário
1. [Resumo executivo](#1-resumo-executivo)
2. [Matriz de funcionalidades](#2-matriz-de-funcionalidades)
3. [Regras projetadas ainda fora do cálculo](#3-regras-projetadas-ainda-fora-do-cálculo)
4. [Inconsistências de conteúdo e design](#4-inconsistências-de-conteúdo-e-design)
5. [Pendências de interface](#5-pendências-de-interface)
6. [Plano de ação priorizado](#6-plano-de-ação-priorizado)
7. [Roadmap de produto](#7-roadmap-de-produto)
8. [Riscos técnicos](#8-riscos-técnicos)
9. [Método do levantamento](#9-método-do-levantamento)

Legenda: ✅ ativo · 🟡 pronto, não ativo · 🧩 regra projetada, fora do cálculo · ⚠️ com ressalva · 🔲 planejado.

---

## 1. Resumo executivo

**Estágio:** pré-alfa jogável (versão 0.1.0).

**O que já funciona de ponta a ponta:** o ciclo completo de atendimento (agenda → espera → pedido → planejamento → minigame → avaliação → pagamento → saída), bairro vivo com moradores e trânsito, dia/noite, clima, energia e desmaio, economia (caixa, extrato, contas, multas, empréstimos, loja, mobília, reformas com bônus), Biblioteca com os 24 cortes, missões, desafios diários com ranking, conquistas, BarberBook, conversa contextual com memória, tutorial e salvamento automático.

**O que está programado mas não ligado (🟡):** maestria por corte, clientes VIP, arcos narrativos, prestígio, painel de conquistas, clientes espontâneos, relatório diário e 7 botões de painéis na HUD.

**Regras configuradas mas fora do cálculo (🧩):** bônus de fidelidade, efeitos de humor e personalidade na nota, gorjeta por clima, perks de aluguel e gorjeta, desgaste e consumo de itens, troca visual de cabelo, aparição física das reformas.

**Principais ajustes de design:** preço único por tipo de serviço (o preço de cada corte não é usado; Turbante sai por 0), escala de nota do minigame inflada, inventário inicial vazio com caixa de BM$ 100, conquistas inalcançáveis e IDs errados nos eventos culturais.

**Leitura:** a base de sistemas é ampla e sólida; o trabalho que falta é sobretudo de **integração, conteúdo de configuração e balanceamento**, não de novos sistemas.

---

## 2. Matriz de funcionalidades

| Área | Funcionalidade | Status | Observação |
|---|---|---|---|
| **Menu** | Menu, opções, apelido, créditos, sair, carregamento (PAD no Android) | ✅ | |
| **Controle** | Personagem em 3ª pessoa, câmera orbital, sprint, pulo, interação | ✅ | |
| **Mundo** | Moradores por waypoints, trânsito, dia/noite, clima por estação | ✅ | 15 prefabs de clientes |
| **Tempo** | Relógio, horário comercial, dias úteis, avanço durante o serviço | ✅ | 3 min de jogo por segundo |
| **Clientes** | Agenda automática, atrasos em cascata, "Perdido" | ✅ | |
| | Clientes espontâneos (walk-in) | 🟡 | `ClientSpawner.autoSpawn = false` |
| | Espera, fila, paciência, mensagens de humor | ✅ | |
| | Cliente vai embora ao esgotar a paciência | 🔲 | Estado "Indo embora" só visual |
| | Clientes VIP | 🟡 | Nada chama `MarcarClienteComoVip` |
| **Atendimento** | Pedido, aceitar/dispensar | ✅ / ⚠️ | Requisitos vazios; Turbante com preço 0 |
| | Planejamento com ferramentas compatíveis | ✅ | |
| | Minigame "Corte Perfeito" | ✅ / ⚠️ | Escala de nota inflada (§4) |
| | Execução por etapas (modo alternativo) | ✅ | Desligado (`useMinigame = true`) |
| | Avaliação, dinheiro, gorjeta, XP, reputação | ✅ / ⚠️ | |
| | Troca visual do cabelo | 🧩 | Nenhuma variante de cabelo configurada nos clientes |
| | Consumo e desgaste de itens | 🧩 | Dependem de `requiredItems`, vazio nos 24 cortes |
| **Conversa** | Chat, falas por contexto e personalidade, memória, confiança/tensão | ✅ | |
| | "Fechar para novos clientes" | 🧩 | Só gera mensagem |
| **Energia** | Energia, fadiga, recuperação por distância, desmaio, descanso no carro | ✅ | |
| **Economia** | Caixa, extrato, contas, multas, crise, empréstimos | ✅ | |
| | Preço por corte | ⚠️ | Tabela por tipo de serviço sobrepõe o preço do corte |
| | Loja, inventário, mobília colocável | ✅ | Painéis sem botão na HUD (§5) |
| | Gestão: horários, dias, ajuste de preço, reformas | ✅ | |
| | Reformas — bônus | ✅ | |
| | Reformas — objetos na cena | 🧩 | Nenhum dos 7 objetos existe na `GameScene` |
| **Progressão** | Níveis e multiplicadores de XP | ✅ | |
| | Biblioteca de Cortes | ✅ | |
| | Maestria por corte | 🟡 | `CutMasterySystem` fora da `GameScene` |
| | Fidelidade — contagem e consulta | ✅ | |
| | Fidelidade — bônus | 🧩 | §3 |
| | Reputação e subnotas | ✅ / ⚠️ | |
| | Missões (8) | ✅ | |
| | Desafios diários e ranking | ✅ | |
| | Conquistas — popup | ✅ / ⚠️ | §4 |
| | Conquistas — painel | 🟡 | `AchievementsPanelUI` fora da cena |
| | Arcos narrativos | 🟡 | Nenhum NPC com `narrativeCharacterId` |
| | Prestígio | 🟡 | Sem tela que chame `TentarPrestigiar` |
| | Eventos culturais — bônus | ✅ | |
| | Eventos culturais — desbloqueio de cortes | ⚠️ | IDs errados |
| **Social** | BarberBook, posts virais, clientes orgânicos | ✅ | Viral depende de VIP 🟡 |
| **Onboarding** | Tutorial, apelido | ✅ | |
| **Dados** | Salvamento automático | ✅ | Agenda não é salva |
| | Relatório diário | 🟡 | Sem tela |
| **Plataforma** | PC | ✅ | |
| | Android | ⚠️ | Materiais HDRP em perfil URP — validar em aparelho |

---

## 3. Regras projetadas ainda fora do cálculo

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

## 4. Inconsistências de conteúdo e design

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

## 5. Pendências de interface

Botões de abertura encontrados na cena: **Financeiro, Gestão, Reputação detalhada, Histórico, Apelido** e o badge do **BarberBook**. Sem botão: **Loja, Inventário, Fila, Agenda, Missões, Empréstimos, Desafios Diários** (os métodos `Open*` existem). Também faltam: **painel de Conquistas**, **tela de Prestígio** e **tela de Relatório**. Proposta de barra de navegação em [03 §9](03-janelas-e-interface.md#9-pendências-de-navegação). Conferir no Editor antes de implementar.

---

## 6. Plano de ação priorizado

Esforço: **P** (horas) · **M** (1–3 dias) · **G** (1+ semana).

### P0 — tornar o jogo coerente para testes com público
| Ação | Esforço |
|---|---|
| Barra de navegação da HUD para todos os painéis | M |
| Kit inicial no inventário | P |
| Escala de nota do minigame (0–5) | P |
| Preço por corte + entrada para o Turbante | P |
| Requisitos (`requiredItems`) dos 24 cortes → ativa consumo e desgaste | M |
| `CutMasterySystem` na `GameScene` | P |

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

## 7. Roadmap de produto

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

## 8. Riscos técnicos

| Risco | Detalhe | Mitigação |
|---|---|---|
| HDRP × URP no Android | Maioria dos materiais é `HDRP/Lit`; o perfil Android usa URP | Conversão de materiais; testes em aparelho |
| Salvamento em PlayerPrefs | Sem slots nem nuvem; limpar dados apaga tudo | Save em arquivo JSON versionado |
| Testes só de lógica | 28 arquivos, ~357 casos EditMode; nenhum teste do ciclo em cena | Testes PlayMode do atendimento completo |
| Recursos de terceiros | Licenças a confirmar (ver [08 §15](08-referencia-para-editais.md#15-direitos-autorais-e-recursos-de-terceiros)) | Substituir por arte e áudio autorais |
| Cenas duplicadas | `_Recovery/*.unity` e `URP_Demo.unity` referenciam os mesmos scripts | Limpar ou isolar |

---

## 9. Método do levantamento

- Leitura do código-fonte dos sistemas (206 scripts C#) e de seus consumidores (quem chama cada método).
- Leitura dos assets de conteúdo (24 cortes, 8 missões, 41 produtos, 7 reformas, tabela de preços).
- Inspeção da `GameScene.unity` e dos prefabs: presença de componentes, valores sobrescritos no Inspector e ligações de botões.
- Pontos marcados "conferir no Editor" dependem de verificação visual no Unity.
