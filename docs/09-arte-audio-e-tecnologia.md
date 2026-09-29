# AfroBarber — Arte, Áudio e Tecnologia

| | |
|---|---|
| **Documento** | 09 · Direção de arte, áudio, tecnologia e produção de conteúdo |
| **Versão** | 1.3 — 29/09/2026 |
| **Leitores** | Artistas, músicos, desenvolvedores, avaliadores técnicos |
| **Relacionados** | [03 Interface](03-janelas-e-interface.md) · [07 Estado atual](07-estado-atual-e-roadmap.md) · [08 Editais](08-referencia-para-editais.md) · [CLAUDE.md](../CLAUDE.md) |

> Descreve o que existe hoje em arte, áudio e tecnologia, e as diretrizes para a produção autoral. Itens marcados *(diretriz)* são orientações de produção, não conteúdo já pronto.

## Sumário
1. [Direção de arte](#1-direção-de-arte)
2. [Personagens](#2-personagens)
3. [Cabelos](#3-cabelos)
4. [Animação](#4-animação)
5. [Cenários, objetos e iluminação](#5-cenários-objetos-e-iluminação)
6. [Áudio](#6-áudio)
7. [Tecnologia](#7-tecnologia)
8. [Salvamento](#8-salvamento)
9. [Desempenho e plataformas](#9-desempenho-e-plataformas)
10. [Produção de conteúdo sem programação](#10-produção-de-conteúdo-sem-programação)
11. [Qualidade e testes](#11-qualidade-e-testes)

---

## 1. Direção de arte

**Estilo atual:** 3D com iluminação e materiais realistas (HDRP, texturas PBR em 2K), câmera em terceira pessoa próxima ao personagem, céus pintados que mudam com a hora.

**Diretrizes** *(diretriz)*
- **Estética afro-urbana brasileira:** barbearia de bairro com referências de hip-hop, samba, baile e arte afro-brasileira (lambe-lambes, grafite, painéis, tecidos de estampas africanas).
- **Representação:** diversidade de tons de pele, texturas capilares (do 1 ao 4C), idades, gêneros e corpos; nada de caricatura.
- **Leitura clara do cabelo:** o cabelo é o "produto" do jogo — deve ser o elemento de maior detalhe e contraste nos personagens.
- **Cor com significado:** dourado para excelência (Perfeito, viral, destaque do jogador); verde para positivo; vermelho para negativo; laranja para atenção.
- **Referências culturais com consultoria:** máscaras, tambores e símbolos africanos usados como decoração devem ter origem identificada e contexto respeitoso.

---

## 2. Personagens

| Grupo | Situação atual |
|---|---|
| **Barbeiro (jogador)** | Modelo "Barbeiro" com controlador de terceira pessoa, animações de andar, correr, pular, desmaiar e levantar. |
| **Clientes** | **15 prefabs** (entre eles Rapper, Wolf Shirt, Homem de Terno, Punk Woman, Adriana, Joaquim, Francisco, Robson, Lewis, Thales, Adan) com componentes de NPC: identidade, personalidade, memória, paciência, cabelo. |
| **Origem dos modelos** | Mistura de modelos de terceiros (Sketchfab, Ready Player Me) e modelos gerados por IA — licenças a confirmar. |
| **Personagens narrativos** | Sem modelo próprio ainda (Seu Ribeiro, Kemi, Gabriel, Dona Conceição e Leo, além dos netos Caio e Theo). |

**Necessidades de produção** *(diretriz)*: barbeiro(a) autoral com opções de aparência; ~12 clientes genéricos autorais; 5 personagens narrativos + 2 crianças; variações de roupa por clima e estação.

---

## 3. Cabelos

- **Modelos esculpidos** de cabelos afro em produção (malhas com texturas de cor, normal, opacidade e máscara especular), incluindo variações como *Black Short Curl*, *Brown Kink*, *Curly*, *Long Wavy Afro*, *Small Brown Curly*, *Thick Curly*.
- **Sistema:** cada cliente tem um `ClientHairVisualController` com um cabelo padrão e uma lista "pedido → cabelo". Ao fim do serviço, o cabelo do corte feito é ativado.
- **Situação:** nenhuma variante está configurada nos clientes; a troca visual ainda não aparece (ver [07](07-estado-atual-e-roadmap.md)).
- **Ícones de pedido:** 9 ícones prontos (Black Power, Burst Fade, Caesar Cut, High Fade, Line Up, Low Fade, Shape-Up, Skin Fade, Temple Fade) para 24 cortes.

**Necessidades de produção** *(diretriz)*: 24 cabelos "depois" (um por corte) compatíveis com as cabeças dos clientes; cabelos "antes" (natural/crescido); 24 ícones de corte no mesmo estilo; ilustrações para a Biblioteca.

---

## 4. Animação

23 animações de personagem, incluindo: respiração e espera em pé (*Breathing Idle*, *Neutral Idle*), início de caminhada e caminhada, pulo, sentar e levantar (*Stand/Sit*), quatro variações de espera sentado (braços cruzados, movendo a cabeça, movendo as pernas), desmaio (*Dying Backwards*), deitado (*Laying Pose*) e levantar do chão (*Getting Up*).

Parâmetros do Animator: `Speed` (velocidade), `Sit` (sentado), gatilhos de desmaio e de levantar. Nos clientes, o corpo é "encaixado" no assento (`SitPoint`) com o agente de navegação desligado.

**Necessidades de produção** *(diretriz)*: animações do barbeiro trabalhando (máquina, tesoura, navalha, pente, lavar), reações do cliente ao espelho (satisfeito/insatisfeito), conversa e gestos na fila.

---

## 5. Cenários, objetos e iluminação

| Elemento | Situação |
|---|---|
| **Barbearia** | Interior montado com pacote de cenário de barbearia (terceiros) e móveis próprios configurados (sofás, mesas, bancada). |
| **Bairro** | Cidade com terrenos, ruas, rotas de trânsito e pontos de passagem de moradores. |
| **Veículos** | Fusca Dourado, Opala, Jeep e Fox, com rodas animadas. |
| **Decoração cultural** | Máscaras africanas (incluindo máscara Chokwe) e tambor djembê escaneado. |
| **Mobília da loja** | Cadeiras Almofadada e Clássica, Mesa de Bilhar, Fliperama Pac-Man. |
| **Reformas** | Objetos das 7 reformas ainda não modelados/posicionados. |
| **Céu e luz** | Skyboxes por período (manhã, pôr do sol, noite), controlador de céu por hora e materiais que respondem ao horário (cor e emissão). |
| **Materiais** | Texturas PBR 2K (tijolo, metal) e shaders Lit — nunca Standard. |

---

## 6. Áudio

| Elemento | Situação |
|---|---|
| **Efeitos de serviço** | Gravações de máquina de corte, máquina de barba, tesoura, navalha e pente. |
| **Efeitos de interface** | Pacote de sons casuais (licença incluída no pacote). |
| **Música** | Playlist (`MusicManager`) com três faixas instrumentais (*Elijah_K — Cairo, Gorilla, Ice Cream*); licença a confirmar. |
| **Mixagem** | Volumes separados de música e efeitos nas Opções, com som de teste. |

**Diretrizes para a trilha original** *(diretriz)*
- Gêneros: afrobeat, soul, samba de roda, hip-hop boom bap, baile charme e samba-rock; faixas instrumentais em loop para a barbearia e temas próprios para eventos (Carnaval, Consciência Negra).
- Música diegética: o rádio da barbearia (reforma *Som Ambiente*) muda a playlist.
- Ambiência do bairro por hora e clima (chuva, trânsito, conversa na calçada).
- Vozes: sem dublagem prevista; possível "fala" não verbal curta por personagem.

---

## 7. Tecnologia

| Item | Uso |
|---|---|
| **Unity 6** (6000.3.10) | Motor; versão fixa do projeto. |
| **HDRP 17.3** / **URP 17.3** | Renderização de alta qualidade no PC; perfil URP no Android. |
| **Input System 1.18** | Teclado, mouse, gamepad e toque. |
| **AI Navigation (NavMesh)** | Caminhada de clientes e moradores. |
| **Cinemachine 3**, **Timeline**, **Shader Graph**, **Terrain Tools** | Câmeras, sequências, materiais e terreno. |
| **Addressables + Play Asset Delivery** | Pacotes de conteúdo baixados no Android. |
| **TextMeshPro / uGUI** | Interface. |
| **Unity Test Framework 1.6** | Testes automatizados. |

**Arquitetura**
- ~**206 scripts C#** (~33 mil linhas), organizados por domínio (atendimento, clientes, economia, progressão, educação, social, diálogo, cidade, UI).
- **Gerenciadores únicos** (singletons) por domínio que conversam por **eventos**; inicialização ordenada por um `GameBootstrap` com tela de carregamento.
- **Dados em ScriptableObjects:** cortes, produtos, bancos de produtos, missões, reformas, tabela de preços, perfis de agendamento.
- **Máquina de estados** do cliente: passeando → indo à barbearia → entrada → espera → sentado aguardando → indo à cadeira → em atendimento → caixa → volta à cidade.
- Documentação técnica completa de APIs em [CLAUDE.md](../CLAUDE.md).

---

## 8. Salvamento

Salvamento automático no armazenamento local do aparelho (PlayerPrefs), por sistema:

| Dados salvos | Formato |
|---|---|
| Caixa e extrato financeiro | Número e JSON |
| Inventário | JSON |
| XP e nível | Números |
| Biblioteca: desbloqueados, favoritos, vistos | Listas |
| Reputação (global e 3 subnotas) e total de avaliações | Números |
| Reformas, conquistas | Listas de IDs |
| Empréstimos, BarberBook, prestígio, desafios diários, fidelidade, narrativa, relatórios, maestria, missões | JSON |
| Horários, dias e ajustes de preço | Números |
| Mobília colocada, ferramenta escolhida por corte | JSON / texto |
| Horas trabalhadas por mês, dias sem desmaio | Números |
| Memória de diálogo de cada NPC | JSON |
| Apelido, tutorial concluído | Texto / marcador |

Não salvos: **data e hora do jogo** (toda sessão recomeça em 10/04/2026, 8h — bloqueio crítico, ver [07 §2.2](07-estado-atual-e-roadmap.md#22-a-data-do-jogo-não-é-salva)), agenda de clientes e ranking dos desafios (regerado por sessão). Lista completa de chaves em [CLAUDE.md](../CLAUDE.md#playerprefs-keys).

---

## 9. Desempenho e plataformas

- **PC:** HDRP com qualidade alta.
- **Android:** `MobilePerformanceBootstrap` troca o nível de qualidade para um perfil URP; configurações de vídeo ocultas no celular; pacotes de conteúdo por Play Asset Delivery; identificador `br.com.afrobarber.game`.
- **Risco conhecido:** a maioria dos materiais usa shader HDRP, que não renderiza em URP — é preciso converter materiais e validar em aparelhos de entrada.
- **Metas de desempenho** *(diretriz)*: 30 fps estáveis em celular intermediário; carregamento inicial abaixo de 30 s; download base abaixo de 150 MB.

---

## 10. Produção de conteúdo sem programação

| Para adicionar… | Como |
|---|---|
| **Corte** | Criar um *Client Request Data* (nome, tipo de serviço, preço, tempo, XP, dificuldade, categoria, década, história, significado, curiosidade, ícone, requisitos), registrá-lo no banco principal de pedidos e configurar o cabelo nos clientes. |
| **Produto** | Criar um *Product* (ID, categoria, tipo de item, atributos, preço) e incluí-lo no banco da categoria. |
| **Missão** | Criar um asset de missão (métrica, frequência, tiers e recompensas, janela de datas opcional). |
| **Reforma** | Criar uma definição de reforma (custo, nível, pré-requisito, bônus, objeto da cena) e incluí-la na lista do sistema de reformas. |
| **Preço de serviço** | Editar a tabela central de preços. |
| **Evento cultural** | Incluir entrada no sistema de eventos (mês, dias, bônus, mensagem, cortes) — hoje configurado no código. |
| **Personagem narrativo** | Incluir personagem e capítulos no sistema de narrativa e informar o ID no cliente — hoje configurado no código. |

---

## 11. Qualidade e testes

- **28 arquivos de testes automatizados** (modo Editor) com **~357 casos**, cobrindo cálculo de resultado do atendimento, avaliação por etapas, compatibilidade de ferramentas, planejamento, tabela de preços, gestão global, finanças, contas mensais, empréstimos, XP, maestria, fidelidade, VIP, prestígio, desafios diários, missões, narrativa, BarberBook, reputação, clima, tempo de jogo, produtos e inventário.
- Fluxo de validação manual: o ciclo completo de 22 etapas do cliente (ver [README do projeto](../README.md#loop-de-gameplay)).
- Recomendações *(diretriz)*: testes de integração em cena (PlayMode) para o ciclo de atendimento; playtests com público a cada marco.
