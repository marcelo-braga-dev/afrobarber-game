# Documentação do AfroBarber

| | |
|---|---|
| **Projeto** | AfroBarber — simulador de barbearia afro-brasileira |
| **Versão do jogo** | 0.1.0 (pré-alfa) |
| **Versão da documentação** | 1.4 — 29/09/2026 |
| **Motor** | Unity 6 (6000.3.10) |
| **Idioma** | Português do Brasil |

Conjunto de documentos de design (GDD), referência de jogo e apoio institucional do **AfroBarber**. Foi escrito para que qualquer pessoa — jogador, designer, artista, educador, parceiro, avaliador de edital, investidor, desenvolvedor — ou uma IA consiga entender o jogo por completo: proposta, universo, regras, telas, conteúdo, estado atual e futuro.

Todos os números foram levantados diretamente do projeto (código, assets e `GameScene`). Quando a configuração da cena difere do padrão do código, os dois valores são informados.

---

## 1. Mapa da documentação

| # | Documento | Conteúdo | Leitores principais |
|---|---|---|---|
| 01 | [Visão geral do jogo](01-visao-geral-do-jogo.md) | Conceito, pilares, fantasia, mundo, personagens, diferenciais, público, objetivos | Todos |
| 02 | [Guia de jogabilidade](02-guia-de-jogabilidade.md) | Controles, tempo, dia de trabalho, atendimento passo a passo, minigame, clientes, clima, energia | Designers, jogadores, testadores |
| 03 | [Janelas e interface](03-janelas-e-interface.md) | Cada tela, painel, popup e elemento de HUD | Designers de UI, artistas, desenvolvedores |
| 04 | [Progressão e objetivos](04-progressao-e-objetivos.md) | Níveis, maestria, fidelidade, reputação, narrativa, missões, desafios, conquistas, reformas, prestígio | Designers, roteiristas |
| 05 | [Economia e gestão](05-economia-e-gestao.md) | Caixa, preços, demanda, contas, empréstimos, loja e catálogo de produtos | Designers de sistemas, balanceamento |
| 06 | [Universo cultural](06-universo-cultural.md) | Os 24 cortes (história, significado, curiosidade), calendário cultural, BarberBook, voz | Educadores, roteiristas, consultores culturais |
| 07 | [Estado atual e roadmap](07-estado-atual-e-roadmap.md) | Matriz de funcionalidades, regras não aplicadas, inconsistências, checklist, roadmap | Produção, desenvolvimento |
| 08 | [Referência para editais](08-referencia-para-editais.md) | Textos prontos, justificativa, objetivos, metas, orçamento, acessibilidade, checklist de inscrição | Proponente, produção cultural |
| 09 | [Arte, áudio e tecnologia](09-arte-audio-e-tecnologia.md) | Direção de arte, personagens, cabelos, animação, tipografia, áudio, arquitetura, dados, salvamento, desempenho | Artistas, desenvolvedores, avaliadores técnicos |
| 10 | [Glossário](10-glossario.md) | Termos do jogo, da barbearia afro, culturais e técnicos | Todos |
| 11 | [Proposta — Edital PNAB 008/2026 (Ribeirão Preto)](11-proposta-edital-008-2026-rp.md) | Planejamento da inscrição na Modalidade I (R$ 30 mil): regras, pendências, campos A–N, equipe, cronograma, orçamento | Proponente |
| 12 | [Projeto "Cada Corte, uma História" — Edital PNAB 008/2026](12-projeto-edital-008-2026.md) | Texto do projeto pronto para o formulário: campos A–N, cronograma (Anexo 3), planilha (Anexo 4), portfólio e checklist | Proponente |

### Por onde começar

| Se você quer… | Leia |
|---|---|
| Entender o jogo em 5 minutos | 01 |
| Saber como se joga | 02 → 03 |
| Balancear ou mexer nas regras | 02 → 04 → 05 → 07 |
| Conhecer o conteúdo cultural | 06 |
| Inscrever o projeto em um edital | 08 (com apoio de 01, 06 e 07) |
| Produzir arte, áudio ou código | 09 → 07 → [CLAUDE.md](../CLAUDE.md) |

---

## 2. Legenda de status

Os documentos descrevem o jogo **como projetado**, mas sinalizam o que já funciona na build atual:

| Marca | Significado |
|---|---|
| ✅ **Ativo** | Funciona na `GameScene` atual. |
| 🟡 **Pronto, não ativo** | Programado, mas ainda não ligado na cena (falta configuração, conteúdo ou botão). |
| 🧩 **Regra projetada** | A regra existe (valores configurados), mas ainda não entra no cálculo do jogo. |
| ⚠️ **Com ressalva** | Funciona, com bug ou inconsistência conhecida. |
| 🔲 **Planejado** | Ideia ou item de roadmap. |

Detalhes de cada item em [07 — Estado atual e roadmap](07-estado-atual-e-roadmap.md).

> **Atenção:** há quatro **bloqueios críticos** na build atual — sem acesso à Loja e ao Inventário com inventário inicial vazio, data do jogo não salva, contas iniciais vencidas e paciência de ~30 s reais. Ver [07 §2](07-estado-atual-e-roadmap.md#2-bloqueios-críticos).

---

## 3. Números-chave

| Item | Quantidade |
|---|---|
| Cortes na Biblioteca | **24** (8 categorias; da era ancestral a 2010s) |
| Níveis do barbeiro | 5 (Aprendiz da Navalha → Lenda AfroBarber) |
| Tiers de maestria por corte | 5 (Aprendiz da Navalha → Lenda do Estilo) |
| Tiers de fidelidade | 5 (Novo → Lendário) |
| Personagens narrativos | 5, com 3 capítulos cada (15 capítulos) |
| Missões | 8, com 3 tiers cada |
| Tipos de desafio diário | 6 (3 sorteados por dia real) |
| Conquistas | 29 |
| Reformas da barbearia | 7 |
| Produtos na loja | 41 (ferramentas, produtos e móveis) |
| Eventos culturais | 5 |
| Climas | 4 |
| Personalidades de cliente | 8 |
| Prefabs de clientes | 15 |
| Opções de empréstimo | 4 |
| Perks de prestígio | 5 (até 3 prestígios) |
| Código | ~206 scripts C#, ~33 mil linhas |
| Testes automatizados | 28 arquivos, ~357 casos |

---

## 4. Documentação técnica relacionada

- [README do projeto](../README.md) — visão técnica, loop de 22 etapas, estrutura de pastas, configuração de cena, troubleshooting.
- [CLAUDE.md](../CLAUDE.md) — APIs de todos os sistemas, enums, chaves de salvamento, wiring de cena, débito técnico (para desenvolvedores e agentes de IA).
- [plano-correcoes-2026-08.md](plano-correcoes-2026-08.md) — histórico das rodadas de correção de código.

---

## 5. Histórico da documentação

| Versão | Data | Alterações |
|---|---|---|
| 1.0 | 29/09/2026 | Criação dos documentos 01 a 07 a partir de análise do código, assets e cena. |
| 1.1 | 29/09/2026 | Revisão geral; marcação de funções em integração; documento 08 (editais). |
| 1.2 | 29/09/2026 | Análise profunda das regras: modificadores não aplicados, desgaste inativo, visual de cabelo não configurado, reputação inicial 3,0; padronização (cabeçalho, sumário, legenda de status); documentos 09 (arte, áudio e tecnologia) e 10 (glossário). |
| 1.3 | 29/09/2026 | Incorporação de revisão externa, verificada no código: bloqueios críticos (acesso à Loja/Inventário/Biblioteca, data não salva, contas iniciais vencidas e verificação por igualdade, paciência em tempo real); seções de balanceamento e de precisão cultural/propriedade intelectual no 07; correções culturais no 06 (CROWN Act, Palenque, Malcolm X, Poetic Justice, Afropunk, datas); textos do 08 sem promessas além da build; README raiz, CLAUDE.md e plano de correções alinhados ao 07. |
| 1.4 | 29/09/2026 | Ajustes da segunda revisão externa: status ❌ das perks GorjetaExtra20 e AlugueMenor15 no CLAUDE.md; bloqueios no topo do Roadmap e dos Débitos do README; Loja/Inventário como ⚠️ na matriz do 07; resumo do 07 diz "programado, jogável com intervenção no Editor"; duplicatas entre 07 §5 e §6 removidas; etapa 1 do cronograma do 08 começa pelos bloqueios; nomes de eventos atualizados no 04; aviso sobre pagamento de contas no 02; glossário com "Enciclopédia Afro". |

## 6. Como manter esta documentação

1. Ao mudar uma regra no código ou um valor no Inspector, atualize o documento correspondente e o status em 07.
2. Ao ligar uma função 🟡 ou 🧩, troque a marca para ✅ em todos os documentos (busque pelo nome da função).
3. Ao adicionar conteúdo (corte, missão, produto, personagem), atualize as tabelas de 04, 05 ou 06 e os números-chave acima.
4. Registre a mudança no histórico (seção 5).
