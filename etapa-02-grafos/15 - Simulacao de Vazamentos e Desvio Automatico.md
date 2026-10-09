<a href="https://colab.research.google.com/github/igorfantucci/Aula-Automatica---GRUPO-5/blob/main/etapa-02-grafos/15%20-%20Simulacao%20de%20Vazamentos%20e%20Desvio%20Automatico.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# Aula 15: Simulação de Vazamentos e Desvio Automático em Malha Fechada

## 1. Fundamentos Matemáticos: Reconfiguração Dinâmica de Grafos em Tempo Real

Na operação industrial contínua e em batelada da **Planta Piloto de Produção de Biodiesel (Grupo 5)**, a infraestrutura física de escoamento e reação química está sujeita a uma variedade de perturbações e anomalias operacionais severas. Entre os eventos mais críticos de segurança de processo destacam-se a ruptura por estresse cíclico de tubulações, perda de vedação em flanges industriais, sobrepressão abrupta decorrente de reações exotérmicas descontroladas, cavitação mecânica nas motobombas e vazamentos tóxicos ou inflamáveis de reagentes químicos de alta periculosidade, como o metóxido de sódio e o metanol.

Enquanto as Aulas 11 a 14 trataram a malha hidráulica sob a premissa de uma topologia invariante no tempo $G = (V, E, W)$, os ambientes de automação em tempo real exigem que o sistema de controle supervisório (SCADA) e os Sistemas Instrumentados de Segurança (SIS, normatizados pela **IEC 61511** e **ANSI/ISA-84**) operem sobre um **Grafo Dinâmico e Reconfigurável**:

$$G_t = (V, E_t, W_t), \quad t \in \mathbb{N}$$

Onde:
* $V = \{v_1, v_2, \dots, v_n\}$ é o conjunto estático de equipamentos de processo ($|V| = n = 8$);
* $E_t \subseteq V \times V$ representa o conjunto de tubulações físicas ativas no instante de tempo $t$;
* $W_t: E_t \rightarrow \mathbb{R}^+$ é a função de pesos métricos associada aos comprimentos físicos lineares $L\text{ [m]}$ e perdas de carga no instante $t$.

---

### 1.1. Operador Formal de Punição Métrica e Isolamento Topológico

Ao ser disparado um evento de alarme por um transmissor de pressão diferencial ($\Delta P$), sensor óptico de névoa de hidrocarbonetos ou medidor de vazão mássica por efeito Coriolis indicando vazamento no segmento de duto $e_k = (u, v)$, a lógica de intertravamento do CLP executa o **Operador de Punição Métrica** $\mathcal{P}_{u,v}$:

$$\mathcal{P}_{u,v}(G_t) \implies W_{t+1}(x, y) = \begin{cases} \infty, & \text{se } (x, y) = (u, v) \\ W_t(x, y), & \text{caso contrário} \end{cases}$$

Concomitantemente, a matriz booleana de adjacência é atualizada para refletir o fechamento físico das válvulas de bloqueio rápido (tags ISA-5.1):

$$A_{\text{bin}, t+1}[u][v] = 0$$

Imediatamente após a imposição do isolamento topológico, o algoritmo de **Dijkstra** é disparado sobre o grafo reponderado $G_{t+1}$ para recalcular o caminho de menor resistência métrica entre as fontes de reagentes e o destino de decantação, garantindo a comutação automática das válvulas de contingência em frações de milissegundo.

---

### 1.2. Topologia Hidráulica e Cenário de Desvio de Contingência (Planta de Biodiesel)

O diagrama a seguir ilustra a topologia do núcleo de transesterificação da planta de biodiesel do Grupo 5, destacando o evento de injeção de falha no trecho direto do reator `R-200 -> TK-300_Decant` (válvula $XV\text{-}203$) e a rota de contingência ativada através do tanque pulmão/trocador de arrefecimento `TK-201_Pulmao`:

```mermaid
graph LR
    TK101["TK-101: Óleo Vegetal Degomado"] -->|15m - XV-101| MAN101["MAN-101: Manifold de Carga"]
    TK103["TK-103: Preparo de Metóxido"] -->|12m - XV-103| MAN101
    MAN101 -->|8m - XV-101A| P101["P-101: Bomba de Carga A"]
    MAN101 -->|10m - XV-101B| P102["P-102: Bomba de Carga B"]
    P101 -->|25m - XV-201A| R200["R-200: Reator CSTR Encamisado"]
    P102 -->|22m - XV-201B| R200
    R200 -.->|30m - XV-203 VAZAMENTO / ISOLADO| TK300["TK-300: Decantador Cônico"]
    R200 ==>|18m - XV-202B DESVIO ATIVO| TK201["TK-201: Pulmão / Arrefecimento"]
    TK201 ==>|20m - XV-204 DESVIO ATIVO| TK300

    style TK300 fill:#e1f5fe,stroke:#0288d1,stroke-width:2px;
    style R200 fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px;
    linkStyle 6 stroke:#d32f2f,stroke-width:3px,stroke-dasharray: 5 5;
    linkStyle 7 stroke:#2e7d32,stroke-width:3px;
    linkStyle 8 stroke:#2e7d32,stroke-width:3px;
```

---

### 1.3. Especificações Físicas e Instrumentação da Malha

A tabela abaixo sintetiza os parâmetros de engenharia mecânica e automação de cada duto modelado no sistema supervisório:

| Duto / Segmento $(u, v)$ | Função Operacional no Processo | Comprimento $L$ [m] | Válvula ISA-5.1 | Diâmetro $D$ [pol] | Estado Nominal | Estado sob Vazamento em XV-203 |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: |
| `TK-101_Oleo -> MAN-101` | Alimentação de Óleo Degomado | 15.0 | $XV\text{-}101$ | 4.0 | Aberta | Aberta (Fluxo Nominal) |
| `TK-103_Mix -> MAN-101` | Alimentação de Metóxido de Sódio | 12.0 | $XV\text{-}103$ | 3.0 | Aberta | Aberta (Fluxo Nominal) |
| `MAN-101 -> P-101` | Sucção da Bomba de Carga A | 8.0 | $XV\text{-}101A$ | 4.0 | Fechada (Reserva) | Fechada (Reserva) |
| `MAN-101 -> P-102` | Sucção da Bomba de Carga B | 10.0 | $XV\text{-}101B$ | 4.0 | Aberta (Principal) | Aberta (Principal) |
| `P-101 -> R-200` | Recalque da Bomba A no Reator | 25.0 | $XV\text{-}201A$ | 4.0 | Fechada | Fechada |
| `P-102 -> R-200` | Recalque da Bomba B no Reator | 22.0 | $XV\text{-}201B$ | 4.0 | Aberta | Aberta |
| `R-200 -> TK-300_Decant` | Transferência Direta pós-Reação | 30.0 | $XV\text{-}203$ | 6.0 | Aberta | **Fechada (Isolamento Emergencial)** |
| `R-200 -> TK-201_Pulmao` | Envio ao Pulmão / Arrefecimento | 18.0 | $XV\text{-}202B$ | 6.0 | Fechada | **Aberta (Comutação Automática)** |
| `TK-201_Pulmao -> TK-300_Decant` | Descarga Final no Decantador | 20.0 | $XV\text{-}204$ | 6.0 | Fechada | **Aberta (Comutação Automática)** |

---

### 1.4. Matrizes de Adjacência: Estado Nominal vs. Estado de Isolamento de Falha

Para evidenciar a mutação do grafo dinâmico, apresentam-se as matrizes ponderadas completas do sistema:

#### Matriz Ponderada Nominal $W_0$ (Valores em Metros, $\infty$ indica ausência de arco direto):
```
                  TK-101  TK-103  MAN-101   P-101   P-102   R-200  TK-201  TK-300
TK-101_Oleo    |     0.0       ∞     15.0       ∞       ∞       ∞       ∞       ∞
TK-103_Mix     |       ∞     0.0     12.0       ∞       ∞       ∞       ∞       ∞
MAN-101        |       ∞       ∞      0.0     8.0    10.0       ∞       ∞       ∞
P-101          |       ∞       ∞        ∞     0.0       ∞    25.0       ∞       ∞
P-102          |       ∞       ∞        ∞       ∞     0.0    22.0       ∞       ∞
R-200          |       ∞       ∞        ∞       ∞       ∞     0.0    18.0    30.0
TK-201_Pulmao  |       ∞       ∞        ∞       ∞       ∞       ∞     0.0    20.0
TK-300_Decant  |       ∞       ∞        ∞       ∞       ∞       ∞       ∞     0.0
```

#### Matriz Ponderada após Isolamento $W_1$ (Ruptura em `R-200 -> TK-300_Decant`):
```
                  TK-101  TK-103  MAN-101   P-101   P-102   R-200  TK-201  TK-300
TK-101_Oleo    |     0.0       ∞     15.0       ∞       ∞       ∞       ∞       ∞
TK-103_Mix     |       ∞     0.0     12.0       ∞       ∞       ∞       ∞       ∞
MAN-101        |       ∞       ∞      0.0     8.0    10.0       ∞       ∞       ∞
P-101          |       ∞       ∞        ∞     0.0       ∞    25.0       ∞       ∞
P-102          |       ∞       ∞        ∞       ∞     0.0    22.0       ∞       ∞
R-200          |       ∞       ∞        ∞       ∞       ∞     0.0    18.0       ∞  <-- Penalizado com ∞
TK-201_Pulmao  |       ∞       ∞        ∞       ∞       ∞       ∞     0.0    20.0
TK-300_Decant  |       ∞       ∞        ∞       ∞       ∞       ∞       ∞     0.0
```

---

### 1.5. Métricas de Impacto Operacional do Desvio Automático

A ativação da rota de contingência acarreta alterações quantitativas no escoamento que devem ser monitoradas pelo supervisório:

1. **Acréscimo Métrico Percentual ($\Delta L_{\text{\%}}$):**
   $$\Delta L_{\text{\%}} = \frac{L_{\text{contingência}} - L_{\text{nominal}}}{L_{\text{nominal}}} \times 100$$ = \frac{85.0 - 77.0}{77.0} \times 100 \approx +10.39\%$$
2. **Impacto na Perda de Carga Distribuída ($\Delta P$):**
   Pela formulação de Darcy-Weisbach, o acréscimo linear de $8.0\,\text{m}$ em tubulação de $6$ polegadas eleva a contrapressão em aproximadamente $0.12\,\text{bar}$, valor perfeitamente suportável pela curva característica da bomba $P\text{-}102$.
3. **Tempo Adicional de Residência Hidráulica ($\Delta \tau$):**
   Para uma vazão volumétrica de $100\,\text{m}^3/\text{h}$, o volume morto adicional contido no trecho de desvio é de aproximadamente $0.146\,\text{m}^3$, resultando em um atraso de transporte residual de apenas $5.26\,\text{segundos}$.

---

## 2. Aprofundamento Teórico

### 2.1. Grafos Dinâmicos e o Conceito de Reponderação de Arestas

Em sistemas operacionais de tempo real (RTOS) e controladores lógicos programáveis (CLPs conforme a norma **IEC 61131-3**), o gerenciamento de memória é estritamente determinístico. A alocação dinâmica de memória durante o ciclo de varredura (*scan cycle*) é expressamente proibida em softwares de missão crítica devido ao risco iminente de fragmentação de heap e estouro de pilha (*stack overflow*).

Nesse contexto, a técnica de **reponderação métrica** — atribuir $W(u, v) \leftarrow \infty$ a uma aresta isolada sem removê-la estruturalmente da memória — destaca-se como a abordagem computacional ótima:

1. **Invariância da Topologia em Memória:** Os arrays bidimensionais que armazenam as matrizes de adjacência binária e ponderada mantêm seu tamanho fixo $n \times n$. Não há necessidade de chamadas de sistema como `realloc` ou reorganização de índices tabulares (`v_to_idx`).
2. **Preservação do Registro Cadastral (Digital Twin):** O segmento físico de tubulação não deixou de existir mecanicamente na planta fabril; ele apenas encontra-se em estado de bloqueio de segurança. Manter o registro cadastral permite que o supervisório continue exibindo o trecho em vermelho intermitente na tela sinóptica, facilitando o trabalho da equipe de manutenção de campo.
3. **Consistência Algébrica com o Heap:** Como os algoritmos de menor caminho operam com filas de prioridade mínima (*min-heap*), o valor $\infty$ atua como barreira intransponível, garantindo que o arco penalizado nunca seja selecionado durante a etapa de relaxação de arestas.

---

### 2.2. Recomputar do Zero vs. Algoritmos Incrementais de Menor Caminho

A literatura clássica de Teoria dos Grafos e Otimização Combinatória oferece duas grandes classes de estratégias para lidar com alterações de pesos em grafos dinâmicos:

| Métrica de Avaliação Técnica | Recomputação Estática (Dijkstra do Zero) | Algoritmos Incrementais Dinâmicos (*Dynamic SSSP*) |
| :--- | :---: | :---: |
| **Complexidade Temporal Pior Caso** | $\mathcal{O}((|V| + |E|) \log |V|)$ | $\mathcal{O}(\|\Delta\| \log \|\Delta\|)$, onde $\Delta$ é o subgrafo afetado |
| **Complexidade Temporal Médio Caso** | $\approx 0.046\text{ ms}$ (na planta de biodiesel) | $< 0.010\text{ ms}$ (em redes locais muito grandes) |
| **Complexidade de Memória Adicional** | $\mathcal{O}(|V|)$ (vetores auxiliares transitórios) | $\mathcal{O}(|V| + |E|)$ (árvores persistentes e ponteiros DAG) |
| **Determinismo em Sistemas Embarcados** | **Máximo** (tempo previsível e sem estados ocultos) | **Médio** (sensível ao histórico de mutações do grafo) |
| **Robustez sob Falhas Múltiplas Concorrentes** | **Excelente** (reavalia toda a topologia de forma limpa) | **Baixa** (risco de estados inconsistentes em cascata) |
| **Facilidade de Depuração e Certificação SIL** | **Elevada** (código conciso, sem ponteiros complexos) | **Complexa** (difícil cobertura de testes estruturais) |

#### Algoritmos Incrementais Dinâmicos:
Estruturas desenvolvidas por Ramalingam & Reps (1996) e Frigioni et al. (2000) mantêm uma árvore de caminhos mínimos persistente (*Shortest Path Tree* - SPT). Ao aumentar o peso de uma aresta $(u, v)$, o algoritmo identifica exclusivamente os nós cuja distância ótima dependia diretamente daquele arco, recalculando apenas o subgrafo $\Delta$. 

Embora essa estratégia seja essencial para redes continentais de telecomunicações ou redes de distribuição de gás natural com $10^5$ vértices, para uma planta industrial de processamento químico ($n < 500$), a recomputação do zero com Min-Heap binário consome menos de $50\,\mu\text{s}$, oferecendo determinismo absoluto e simplificando a conformidade com as exigências de segurança funcional.

---

### 2.3. Tempo de Decisão como Métrica Crítica de Engenharia (IEC 61511 / SIS)

No relatório consolidado emitido pelo módulo `SistemaDesvioAutomatico`, registra-se com rigor a métrica de telemetria **`Tempo_Decisão_ms`**, mensurada através de temporizadores de alta resolução do sistema operacional:

$$t_{\text{decisão}} = (t_{\text{conclusão dijkstra}} - t_{\text{disparo alarme}}) \times 1000 \quad [\text{ms}]$$

Na engenharia de segurança de processos químicos, esse parâmetro é confrontado diretamente com o conceito de **Tempo de Segurança do Processo (*Process Safety Time* - PST)** estipulado pela norma internacional **IEC 61511**:

$$\text{PST} \ge t_{\text{detecção}} + t_{\text{transmissão}} + t_{\text{decisão algorítmica}} + t_{\text{atuação física}} + t_{\text{acomodação}}$$

Onde cada parcela representa:
1. **$t_{\text{detecção}}$:** Tempo de resposta dos transmissores de vazão mássica ou pressão ($100\text{ ms}$ a $300\text{ ms}$);
2. **$t_{\text{transmissão}}$:** Latência de comunicação no barramento de campo industrial (ex.: PROFINET / Foundation Fieldbus: $2\text{ ms}$ a $10\text{ ms}$);
3. **$t_{\text{decisãoalgorítmica}}$:** Tempo de execução do algoritmo de menor caminho no CLP/SCADA ($\approx 0.048\text{ ms}$);
4. **$t_{\text{atuaçãofísica}}$:** Tempo mecânico para fechamento da válvula de isolamento e abertura da válvula de desvio ($3.0\text{ s}$ a $8.0\text{ s}$ para atuadores pneumáticos rotativos de 6 polegadas);
5. **$t_{\text{acomodação}}$:** Tempo de estabilização do regime hidráulico e amortecimento do golpe de aríete (*water hammer*) ($500\text{ ms}$).

> **Conclusão Fundamental:** Como $t_{\text{decisãoalgorítmica}} \approx 0.048\text{ ms}$ representa apenas $0.0006\%$ do tempo total de fechamento mecânico das válvulas, o **gargalo de segurança do processo reside estritamente na inércia dos atuadores pneumáticos/mecânicos**, comprovando que o algoritmo de grafos opera com folga de ordens de magnitude em relação aos limites de segurança funcional.

---

### 2.4. Dinâmica Hidráulica de Fechamento de Válvulas e Golpe de Aríete

Ao se efetuar o desvio automático de um trecho pressurizado, a velocidade com que as válvulas automatizadas $XV\text{-}203$ e $XV\text{-}202B$ comutam não pode ser infinitamente rápida. De acordo com a teoria clássica de transitórios hidráulicos formulada pela **Equação de Joukowsky (Allievi)**:

$$\Delta P_{\text{golpe}} = \rho \cdot a \cdot \Delta v$$

Onde:
* $\Delta P_{\text{golpe}}$ é a sobrepressão transitória gerada no interior da tubulação [$\text{Pa}$];
* $\rho$ é a massa específica da mistura reagente (aproximadamente $880\text{ kg/m}^3$ para o biodiesel a $60^\circ\text{C}$);
* $a$ é a celeridade da onda de pressão acústica no duto de aço carbono [$\text{m/s}$] (tipicamente $1.100\text{ m/s}$);
* $\Delta v$ é a variação da velocidade linear média do fluido escoando na linha [$\text{m/s}$].

Se a válvula de bloqueio fechar em um intervalo inferior ao tempo de trânsito da onda de pressão de ida e volta ($T_{\text{crítico}} = \frac{2L}{a} \approx \frac{2 \times 30}{1.100} = 0.054\text{ s}$), a sobrepressão atinge seu pico máximo, podendo romper soldas e conexões de instrumentação. Por conseguinte, enquanto o algoritmo decide a nova rota em $0.048\text{ ms}$, os atuadores são calibrados para realizar rampas suaves de desaceleração de $3\text{ a } 5\text{ segundos}$.

---

### 2.5. Conectividade, Pontes (*Bridges*) e Vértices de Articulação

A injeção de falhas evidencia propriedades estruturais profundas da malha hidráulica:

* **Ponte (*Bridge*):** Uma aresta $e = (u, v) \in E$ é uma ponte se e somente se sua remoção estrita aumenta o número de componentes conexos do grafo:
  $$c(G \setminus \{e\}) > c(G)$$
  Em termos topológicos fundamentais: **uma aresta é uma ponte se e somente se ela não pertence a nenhum ciclo simples do grafo**.
* **Vértice de Articulação (*Cut Vertex*):** Um equipamento de processo $v \in V$ cuja eliminação decompõe a planta em subsistemas desconectados.

#### Mapeamento de Vulnerabilidades na Planta de Biodiesel:
* **Duto `R-200 -> TK-300_Decant` (XV-203): NÃO É PONTE.**  
  Pertence ao ciclo formado por $\{R\text{-}200, TK\text{-}201\_\text{Pulmao}, TK\text{-}300\_\text{Decant}\}$. Sua perda permite desvio pelo caminho alternativo com acréscimo de apenas $8.0\,\text{m}$ de tubulação.
* **Duto `TK-101_Oleo -> MAN-101` (XV-101): É PONTE CRÍTICA.**  
  Não existe alimentação redundante para o óleo vegetal. Sua ruptura desconecta permanentemente o nó de carga, forçando o desligamento da planta.
* **Manifold `MAN-101` e Reator `R-200`: SÃO VÉRTICES DE ARTICULAÇÃO.**  
  Sua inoperância divide a planta entre a etapa de estocagem de reagentes e a etapa de separação final.

#### Algoritmo de Tarjan para Detecção de Pontes (1974):
Baseado em uma única travessia em profundidade (DFS) com tempo $\mathcal{O}(|V| + |E|)$, calcula para cada vértice $u$ o seu tempo de descoberta $d[u]$ e o menor tempo de descoberta alcançável por arestas de retorno $low[u]$. Uma aresta $(u, v)$ é uma ponte se e somente se:

$$low[v] > d[u]$$

Essa rotina é executada durante o comissionamento da planta para apontar à engenharia onde são obrigatórias tubulações redundantes de segurança.

---

### 2.6. Resiliência de Rede: k-Conectividade e Teorema de Menger

A tolerância a falhas de uma planta é formalizada pelo conceito de **$k$-aresta-conectividade**:

> **Definição:** Um grafo $G$ é $k$-aresta-conexo se a desconexão de quaisquer dois vértices exige a supressão de no mínimo $k$ arestas simultâneas.

Pelo célebre **Teorema de Menger (1927)**:
$$\text{O número máximo de caminhos disjuntos em arestas entre } s \text{ e } t \text{ é igual ao tamanho do corte mínimo de arestas que desconecta } s \text{ de } t.$$

Na malha do Grupo 5:
* Entre `MAN-101` e `R-200`, existem 2 caminhos disjuntos em arestas (via `P-101` e via `P-102`);
* Entre `R-200` e `TK-300_Decant`, existem 2 caminhos disjuntos em arestas (direto de 30m e via pulmão de 38m).

Consequentemente, esses subsistemas operacionais são **$2$-aresta-conexos**, garantindo conformidade com o **critério de contingência simples ($N-1$)**: qualquer falha única de tubulação é absorvida sem interrupção do processo produtivo.

---

## 3. Exemplo Resolvido

### Cenário 1: Ruptura Simples na Tubulação de Saída do Reator (Válvula XV-203)

**Enunciado:**  
Durante a descarga contínua de uma batelada de ésteres a $60^\circ\text{C}$, o sensor de vazamento acusa ruptura na linha direta `R-200 -> TK-300_Decant` (comprimento nominal de $30.0\,\text{m}$, válvula ISA $XV\text{-}203$). O sistema supervisório dispara a rotina `tratar_evento_vazamento`. Demonstre analiticamente os cálculos executados pelo motor de roteamento.

**Resolução Passo a Passo:**

1. **Isolamento Topológico Inicial:**  
   O SCADA atualiza as matrizes do grafo na memória:
   $$W(\text{R-200}, \text{TK-300Decant}) \leftarrow \infty, \quad A_{\text{bin}}[\text{R-200}][\text{TK-300Decant}] \leftarrow 0$$
2. **Rota Nominal Anterior:**  
   $$\text{TK-101Oleo} \xrightarrow{15\text{m}} \text{MAN-101} \xrightarrow{10\text{m}} \text{P-102} \xrightarrow{22\text{m}} \text{R-200} \xrightarrow{30\text{m}} \text{TK-300Decant} \quad (L = 77.0\,\text{m})$$
3. **Execução de Dijkstra sob Reponderação:**  
   * O algoritmo parte de `TK-101_Oleo` ($d = 0.0$), atinge `MAN-101` ($d = 15.0$) e seleciona a motobomba `P-102` ($d = 25.0$).
   * Atinge o reator `R-200` com custo acumulado:
     $$d[\text{R-200}] = 25.0 + 22.0 = 47.0\,\text{m}$$
   * Ao inspecionar os vizinhos de `R-200`:
     * Para `TK-300_Decant`: peso é $\infty$ $\implies$ não ocorre relaxação.
     * Para `TK-201_Pulmao`: peso é $18.0\,\text{m}$ $\implies$ relaxa com $d = 47.0 + 18.0 = 65.0\,\text{m}$.
   * A partir de `TK-201_Pulmao`, relaxa `TK-300_Decant` via duto de arrefecimento ($20.0\,\text{m}$):
     $$d[\text{TK-300Decant}] = 65.0 + 20.0 = 85.0\,\text{m}$$
4. **Resultado Consolidado:**  
   * **Nova Rota Ativa:**  
     $$\text{TK-101Oleo} \rightarrow \text{MAN-101} \rightarrow \text{P-102} \rightarrow \text{R-200} \rightarrow \text{TK-201Pulmao} \rightarrow \text{TK-300Decant}$$
   * **Comprimento Total:** $85.0\,\text{m}$ (acréscimo de $8.0\,\text{m}$ em relação à nominal).
   * **Tempo de Decisão:** $t_{\text{decisão}} \approx 0.0480\,\text{ms}$.

---

### Cenário 2: Rastreamento Completo da Fila Min-Heap na Contingência

A tabela a seguir documenta detalhadamente as transições internas do Min-Heap binário e do vetor de predecessores ao longo de cada iteração da busca por menor caminho durante a comutação de emergência:

| Iteração | Vértice Extraído ($u$) | Distância $d[u]$ | Vizinhos Inspecionados ($v$) | Peso $W(u,v)$ | Nova Distância Proposta | Ação de Relaxação | Estado do Min-Heap |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :--- |
| **0** | — (Início) | — | — | — | — | $d[\text{TK-101}] = 0.0$ | `[(0.0, 'TK-101_Oleo')]` |
| **1** | `TK-101_Oleo` | $0.0$ | `MAN-101` | $15.0$ | $15.0$ | Relaxa (`pred=TK-101`) | `[(15.0, 'MAN-101')]` |
| **2** | `MAN-101` | $15.0$ | `P-101`, `P-102` | $8.0, 10.0$ | $23.0, 25.0$ | Relaxa ambos | `[(23.0, 'P-101'), (25.0, 'P-102')]` |
| **3** | `P-101` | $23.0$ | `R-200` | $25.0$ | $48.0$ | Relaxa (`pred=P-101`) | `[(25.0, 'P-102'), (48.0, 'R-200')]` |
| **4** | `P-102` | $25.0$ | `R-200` | $22.0$ | $47.0$ | **Re-relaxa!** ($47 < 48$) | `[(47.0, 'R-200'), (48.0, 'R-200')]` |
| **5** | `R-200` | $47.0$ | `TK-300`, `TK-201` | $\infty, 18.0$ | $\infty, 65.0$ | Relaxa só `TK-201` | `[(48.0, 'R-200'), (65.0, 'TK-201')]` |
| **6** | `R-200` (dupl.) | $48.0$ | — | — | — | Descarta ($48 > 47$) | `[(65.0, 'TK-201_Pulmao')]` |
| **7** | `TK-201_Pulmao` | $65.0$ | `TK-300_Decant` | $20.0$ | $85.0$ | Relaxa (`pred=TK-201`) | `[(85.0, 'TK-300_Decant')]` |
| **8** | `TK-300_Decant` | $85.0$ | — (Destino) | — | — | **Alvo Atingido! Fim.** | `[]` |

---

### Cenário 3: Falha Dupla Simultânea e Intertravamento de Segurança (ESD)

**Enunciado:**  
Suponha que, enquanto a linha direta `R-200 -> TK-300_Decant` encontra-se isolada, ocorra uma sobrepressão violenta rompendo também a linha `TK-201_Pulmao -> TK-300_Decant` (válvula $XV\text{-}204$). Qual a resposta do algoritmo de Dijkstra?

**Resolução:**  
* Ambas as arestas convergentes para `TK-300_Decant` passam a ter peso $\infty$.
* O grau de entrada efetivo do decantador torna-se nulo: $\deg^-(\text{TK-300Decant}) = 0$.
* O algoritmo de Dijkstra esgota a fila de prioridades Min-Heap sem relaxar o vértice de destino, finalizando com:
  $$d[\text{TK-300Decant}] = \infty, \quad \text{pred}[\text{TK-300Decant}] = \text{None}, \quad \text{caminho} = [\,]$$
* **Ação do SIS:** O sistema identifica formalmente a perda irreversível de conectividade com o decantador, aborta a batelada, desliga imediatamente as bombas `P-101` e `P-102` e aciona o sistema de parada de emergência (ESD - *Emergency Shutdown*).

---

## 4. Relação com as Próximas Aulas do Módulo de Grafos

A modelagem de reconfiguração dinâmica em tempo real desenvolvida nesta Aula 15 conecta-se diretamente com os temas subsequentes da disciplina:

1. **Aula 16 (Circuitos Eulerianos e Inspeção de Infraestrutura):**  
   Os dutos isolados nesta aula passam a ser o foco das rotas prioritárias de manutenção autônoma. O robô de inspeção deve planejar um percurso euleriano para inspecionar e certificar a integridade física de todas as linhas de transferência antes de sua liberação operacional pelo SCADA.
2. **Aula 17 (Problemas Hamiltonianos e Roteamento de Amostragem):**  
   A coleta autônoma de amostras laboratoriais para análise de acidez e teor de ésteres metílicos percorre os nós principais da planta visitando cada vaso de processo exatamente uma vez (Ciclo Hamiltoniano / TSP).
3. **Aula 18 (Árvore Geradora Mínima e Infraestrutura Elétrica):**  
   O cabeamento de comunicação industrial e alimentação dos atuadores das válvulas $XV$ é otimizado através de algoritmos de MST (Kruskal e Prim), minimizando os custos de instalação na planta fabril.

---

## 5. Glossário Normativo e Técnico

* **ESD (*Emergency Shutdown*):** Sistema de parada de emergência projetado para desenergizar e isolar processos industriais sob risco iminente de dano severo.
* **IEC 61511:** Padrão internacional de segurança funcional para o setor de indústrias de processo utilizando sistemas instrumentados de segurança (SIS).
* **Min-Heap:** Estrutura de dados em árvore binária na qual o elemento de menor valor ocupa a raiz, garantindo extração de mínimo em $\mathcal{O}(\log n)$.
* **P&ID (*Piping and Instrumentation Diagram*):** Diagrama técnico que ilustra todas as tubulações, instrumentos, atuadores e vasos de processo de uma planta industrial.
* **PST (*Process Safety Time*):** Intervalo de tempo entre a ocorrência de uma falha de processo e o momento em que um evento perigoso se consuma caso nenhuma ação de segurança seja tomada.
* **SIL (*Safety Integrity Level*):** Nível de integridade de segurança (de 1 a 4) definido pela norma IEC 61508/61511 com base na probabilidade de falha sob demanda (PFD).

---

## 6. Entregável da Aula 15

* **Módulo `SistemaDesvioAutomatico` em Python:** Sistema integrado de isolamento emergencial de dutos com reponderação instantânea de matrizes de custo, cálculo reativo de caminhos mínimos via `RoteadorDijkstra` e monitoramento de tempos de resposta computacional (`Tempo_Decisão_ms`).
* **Caderno Jupyter Executável:** Arquivo [`15 - Simulacao de Vazamentos e Desvio Automatico.ipynb`](./15%20-%20Simulacao%20de%20Vazamentos%20e%20Desvio%20Automatico.ipynb) com simulação de estresse de injeção de falhas, validação de rota substituta de $85.0\,\text{m}$ e asserções automatizadas.
