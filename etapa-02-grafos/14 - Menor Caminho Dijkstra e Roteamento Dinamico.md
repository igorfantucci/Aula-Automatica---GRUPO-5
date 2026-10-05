<a href="https://colab.research.google.com/github/igorfantucci/Aula-Automatica---GRUPO-5/blob/main/etapa-02-grafos/14%20-%20Menor%20Caminho%20Dijkstra%20e%20Roteamento%20Dinamico.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# Aula 14: Menor Caminho — Algoritmo de Dijkstra e Roteamento Dinâmico de Fluidos

## 1. Fundamentos Matemáticos: Algoritmo Guloso de Dijkstra em Redes Hidráulicas

Na engenharia de automação e controle aplicada ao processamento químico contínuo e em batelada, selecionar a rota de transferência de fluidos envolve critérios conflitantes: enquanto a Aula 13 priorizou o **menor número de manobras de válvulas** (confiabilidade mecânica via BFS), a Aula 14 foca na **otimização métrica de transporte** (menor comprimento físico linear, menor tempo de trânsito e menor perda de carga hidráulica).

Dado um dígrafo ponderado $G = (V, E, W)$, onde:
* $V$ é o conjunto de unidades e equipamentos de processo ($|V| = n$);
* $E$ é o conjunto de tubulações orientadas ($|E| = m$);
* $W: E \rightarrow \mathbb{R}^+$ é a função de pesos estritamente não-negativos, correspondente ao **comprimento físico linear de cada tubulação $L\text{ [m]}$**.

O **Problema do Caminho Mínimo de Origem Única (SSSP - *Single-Source Shortest Path*)** consiste em determinar, para um vértice fonte $s \in V$, o caminho orientado $P = (s = v_0, v_1, \dots, v_k = t)$ até um destino $t$ que minimize o somatório dos pesos:

$$\delta(s, t) = \min_{P} \sum_{i=1}^k W(v_{i-1}, v_i)$$

O **Algoritmo de Dijkstra** resolve esse problema com complexidade temporal $\mathcal{O}((|V| + |E|) \log |V|)$ utilizando uma fila de prioridade baseada em **Min-Heap binário**.

### Topologia Hidráulica Ponderada (Planta de Biodiesel - Grupo 5):

```mermaid
graph LR
    TK101["TK-101: Óleo Vegetal Degomado"] -->|15m - XV-101| MAN101["MAN-101: Manifold de Carga"]
    TK103["TK-103: Preparo de Metóxido"] -->|12m - XV-103| MAN101
    MAN101 -->|8m - XV-101A| P101["P-101: Bomba de Carga A"]
    MAN101 -->|10m - XV-101B| P102["P-102: Bomba de Carga B"]
    P101 -->|25m - XV-201A| R200["R-200: Reator CSTR Encamisado"]
    P102 -->|22m - XV-201B| R200
    R200 -->|30m - XV-203| TK300["TK-300: Decantador Cônico de Fases"]
    R200 -->|18m - XV-202B| TK201["TK-201: Pulmão / Arrefecimento"]
    TK201 -->|20m - XV-204| TK300
```

### Justificativa Termo-Hidráulica da Minimização de Distância no Biodiesel:
1. **Redução da Perda de Carga Distribuída ($\Delta P_f$):**  
   Pela Equação de Darcy-Weisbach:
   $$\Delta P_f = f \cdot \frac{L}{D} \cdot \frac{\rho v^2}{2}$$
   Minimizar o comprimento linear $L$ reduz diretamente a contrapressão sobre as motobombas `P-101` e `P-102`, economizando energia elétrica de bombeamento e prevenindo cavitação.
2. **Preservação Térmica dos Reagentes:**  
   O óleo vegetal degomado necessita ser mantido a aproximadamente $55^\circ\text{C}$ a $60^\circ\text{C}$ para viabilizar a cinética de transesterificação. Trajetos mais longos acarretam maior perda convectiva de calor ao longo da tubulação isolada, exigindo maior sobrecarga no trocador térmico $HT-201$.
3. **Minimização do Volume Morto (*Holdup* Líquido):**  
   Comprimentos menores retêm menor volume de mistura reagente retida no duto após o corte das bombas, reduzindo desperdícios na drenagem e facilidade de limpeza CIP (*Clean-In-Place*).

---

## 2. Aprofundamento Teórico

### 2.1. O Princípio de Relaxação de Arestas (*Edge Relaxation*)

O núcleo do Algoritmo de Dijkstra repousa sobre a operação elementar de **relaxação**: para cada aresta dirigida $(u, v) \in E$ com custo métrico $w(u, v) = L$, verifica-se se o trajeto que atinge o nó $v$ passando previamente por $u$ é estritamente mais curto do que a melhor estimativa de distância provisória atualmente conhecida para $v$ ($d[v]$):

$$\text{se } d[u] + w(u, v) < d[v] \implies \begin{cases} d[v] \leftarrow d[u] + w(u, v) \\ \text{pred}[v] \leftarrow u \end{cases}$$

Ao atualizar $d[v]$, atualiza-se também o apontador de predecessor $\text{pred}[v]$, registrando a rota ótima em formação. A sistemática repetição dessa relaxação, guiada pela extração do vértice de menor distância provisória ainda não finalizado no heap, caracteriza a **estratégia gulosa (*greedy strategy*)** de Dijkstra.

---

### 2.2. Por Que a Estratégia Gulosa Funciona (Esboço de Prova e Limitação de Pesos)

A corretude matemática de Dijkstra decorre diretamente da hipótese de **pesos estritamente não-negativos ($w(u, v) \ge 0$)**:

* **Hipótese de Indução:** Todo vértice $u$ extraído do Min-Heap já possui sua distância provisória consolidada igual à distância mínima real: $d[u] = \delta(s, u)$.
* **Passo Indutivo:** Suponha por absurdo que, ao extrair o vértice $u$ com o menor valor de $d[u]$ entre os nós abertos no heap, existisse outro caminho ainda não descoberto mais curto até $u$. Tal caminho precisaria cruzar a fronteira entre os nós finalizados e os não-finalizados através de uma aresta $(x, y)$, onde $x$ já foi finalizado e $y$ permanece no heap. Como todos os pesos são não-negativos ($w \ge 0$), temos que:
  $$d[y] \le \delta(s, u) < d[u]$$
  Contudo, isso contradiz frontalmente a seleção gulosa do heap, que elegeu $u$ exatamente por possuir a menor distância provisória entre todos os elementos presentes na fila. Logo, $d[u] = \delta(s, u)$. $\blacksquare$

#### Por Que Pesos Negativos Quebram o Algoritmo de Dijkstra:
Se existisse um trecho com peso negativo (por exemplo, um modelo hipotético onde uma reação exotérmica gerasse um "crédito" energético negativo no duto), uma rota posterior mais longa em número de nós poderia subitamente reduzir a distância de nós já marcados como finalizados, violando a invariante indutiva. 

Para grafos com pesos negativos sem ciclos negativos, a alternativa clássica é o **Algoritmo de Bellman-Ford**, cuja complexidade $\mathcal{O}(|V| \cdot |E|)$ é assintoticamente superior. Na engenharia de processos, tubulações físicas têm comprimentos estritamente positivos ($L > 0$), garantindo a aplicabilidade universal de Dijkstra.

---

### 2.3. Análise de Complexidade Detalhada e Estruturas de Dados

A eficiência do algoritmo de Dijkstra depende essencialmente da estrutura de dados adotada para gerenciar a fila de prioridade:

| Estrutura da Fila de Prioridade | Extração do Mínimo | Atualização de Chave (Relaxação) | Complexidade Temporal Total | Adequação Industrial |
| :--- | :---: | :---: | :---: | :--- |
| **Busca Linear em Array** | $\mathcal{O}(|V|)$ | $\mathcal{O}(1)$ | $\mathcal{O}(|V|^2)$ | Simples, porém ineficiente para redes com centenas de nós. |
| **Heap Binário (`heapq`)** | $\mathcal{O}(\log |V|)$ | $\mathcal{O}(\log |V|)$ | $\mathcal{O}((|V| + |E|) \log |V|)$ | **Padrão recomendado para plantas SCADA e CLPs.** |
| **Heap de Fibonacci** | $\mathcal{O}(\log |V|)$ | $\mathcal{O}(1)$ amortizado | $\mathcal{O}(|E| + |V| \log |V|)$ | Teórica; sobrecarga de constantes oculta torna inviável na prática. |

Para a rede básica da planta de biodiesel ($|V| = 8$, $|E| = 9$), a execução com heap binário em Python consome menos de 0.1 ms de CPU, viabilizando recálculos de rota a cada ciclo de varredura do CLP ($10\text{ ms}$).

---

### 2.4. Reconstrução do Caminho Ótimo por Retropropagação (*Backtracking*)

O algoritmo não duplica nem transporta vetores de rotas dentro dos nós do heap (o que consumiria espaço excessivo de memória); ele apenas atualiza o dicionário de antecessores:

$$\text{pred}: V \rightarrow V \cup \{\text{None}\}$$

Concluída a busca, o caminho ótimo é reconstruído de jusante para montante: partindo do nó de destino $t$, segue-se iterativamente $\text{pred}[t], \text{pred}[\text{pred}[t]], \dots$ até atingir a fonte de origem $s$. A lista resultante é então invertida em tempo linear $\mathcal{O}(|V|)$.

---

### 2.5. Roteamento Dinâmico Reativo sob Falhas

O método `calcular_menor_caminho` da classe `RoteadorDijkstra` aceita o argumento `bloqueios: Optional[Set[str]]`. 

Quando um transmissor de pressão detecta sobrepressão no reator ($PT-201 \ge 2.5\text{ bar}$) ou um sensor acusa desarme de bomba por sobrecarga elétrica no contator de $P-102$, o identificador do equipamento entra no conjunto de bloqueios. Do ponto de vista de Teoria dos Grafos, isso equivale a atribuir **peso infinito ($\infty$)** a todas as arestas incidentes àquele nó, forçando o algoritmo a eleger automaticamente a rota alternativa ótima remanescente.

---

## 3. Exemplo Resolvido: Roteamento Nominal e Comparação Gulosa na Planta de Biodiesel

**Problema:**  
O SCADA deve calcular a rota ótima de bombeamento de óleo vegetal do tanque `TK-101_Oleo` até o decantador `TK-300_Decant`. Demonstre a execução manual passo a passo de Dijkstra e explique por que a escolha gulosa local no manifold não prejudica a obtenção do ótimo global.

**Resolução Passo a Passo:**

1. **Inicialização:**  
   $$d[\text{TK-101\_Oleo}] = 0.0, \quad d[v] = \infty \quad (\forall v \neq \text{TK-101\_Oleo})$$  
   Heap Min: `[(0.0, 'TK-101_Oleo')]`, $\text{pred} = \{\text{todos None}\}$.

2. **Iteração 1:**  
   Retira `TK-101_Oleo` ($d = 0.0$). Relaxa vizinho `MAN-101`:  
   $$d[\text{MAN-101}] = 0.0 + 15.0 = 15.0, \quad \text{pred}[\text{MAN-101}] = \text{'TK-101\_Oleo'}$$  
   Heap Min: `[(15.0, 'MAN-101')]`.

3. **Iteração 2:**  
   Retira `MAN-101` ($d = 15.0$). Relaxa vizinhos:
   * Para `P-101`: $d[\text{P-101}] = 15.0 + 8.0 = 23.0$, $\text{pred}[\text{P-101}] = \text{'MAN-101'}$;
   * Para `P-102`: $d[\text{P-102}] = 15.0 + 10.0 = 25.0$, $\text{pred}[\text{P-102}] = \text{'MAN-101'}$.  
   Heap Min: `[(23.0, 'P-101'), (25.0, 'P-102')]`.

4. **Iteração 3:**  
   Retira `P-101` ($d = 23.0$, o menor elemento do heap). Relaxa vizinho `R-200`:  
   $$d[\text{R-200}] = 23.0 + 25.0 = 48.0, \quad \text{pred}[\text{R-200}] = \text{'P-101'}$$  
   Heap Min: `[(25.0, 'P-102'), (48.0, 'R-200')]`.

5. **Iteração 4 (O Ponto Crítico da Escolha Global):**  
   Retira `P-102` ($d = 25.0$). Relaxa vizinho `R-200`:  
   A nova distância proposta passando por `P-102` é:  
   $$\text{nova\_d} = 25.0 + 22.0 = 47.0\,\text{m}$$  
   Como $47.0 < 48.0$ (distância via `P-101`), ocorre uma **nova relaxação bem-sucedida**!  
   $$d[\text{R-200}] \leftarrow 47.0, \quad \text{pred}[\text{R-200}] \leftarrow \text{'P-102'}$$  
   Heap Min: `[(47.0, 'R-200'), (48.0, 'R-200')]`.

6. **Iteração 5:**  
   Retira `R-200` ($d = 47.0$). Relaxa vizinhos:
   * Para `TK-300_Decant`: $d = 47.0 + 30.0 = 77.0\,\text{m}$, $\text{pred} = \text{'R-200'}$;
   * Para `TK-201_Pulmao`: $d = 47.0 + 18.0 = 65.0\,\text{m}$, $\text{pred} = \text{'R-200'}$.  
   Heap Min: `[(48.0, 'R-200'), (65.0, 'TK-201_Pulmao'), (77.0, 'TK-300_Decant')]`.

7. **Iteração 6 e 7:**  
   * Retira `R-200` desatualizado ($48.0 > 47.0 \rightarrow$ descartado).
   * Retira `TK-201_Pulmao` ($d = 65.0$). Relaxa `TK-300_Decant`:  
     $$\text{nova\_d} = 65.0 + 20.0 = 85.0\,\text{m} > 77.0\,\text{m} \quad (\text{não relaxa!})$$
   * Retira `TK-300_Decant` ($d = 77.0\,\text{m}$). Como é o destino, a busca finaliza!

8. **Reconstrução do Caminho:**  
   `TK-300_Decant` $\leftarrow$ `R-200` $\leftarrow$ `P-102` $\leftarrow$ `MAN-101` $\leftarrow$ `TK-101_Oleo`.  
   Invertendo:  
   $$\text{TK-101\_Oleo} \rightarrow \text{MAN-101} \rightarrow \text{P-102} \rightarrow \text{R-200} \rightarrow \text{TK-300\_Decant} \quad (L = 77.0\,\text{m})$$

> **Insight de Engenharia:** Mesmo a bomba `P-101` estando mais próxima do manifold ($8\,\text{m}$ contra $10\,\text{m}$ de `P-102`), o trecho de recalque de `P-102` até o reator é mais curto ($22\,\text{m}$ contra $25\,\text{m}$), resultando em um ganho líquido total de $1\,\text{metro}$ ($32\,\text{m}$ contra $33\,\text{m}$). O algoritmo de Dijkstra identifica com precisão essa vantagem global!

---

## 4. Atividades de Investigação

1. **Roteamento Ótimo do Reagente Metóxido:**  
   Execute o algoritmo de Dijkstra partindo de `TK-103_Mix` até `TK-300_Decant`. Determine a rota ótima obtida, seu custo métrico total em metros e confirme se ela também prioriza a motobomba `P-102`.
2. **Comutação Dinâmica em Falha Elétrica de Bomba:**  
   Bloqueie a bomba `P-102` (`bloqueios={"P-102"}`) simulando atuação de relé térmico de proteção de motor. Execute o algoritmo para o óleo vegetal e comprove que a rota se altera automaticamente para `P-101` com comprimento total de $78.0\,\text{m}$.
3. **Desvio Obrigatório por Arrefecimento Térmico:**  
   Suponha que, devido a uma batelada com excesso de calor residual ($TT-201 = 62^\circ\text{C}$), a válvula direta $XV-203$ seja mantida fechada, exigindo que o fluido passe compulsoriamente pelo tanque pulmão de arrefecimento `TK-201_Pulmao`. Simule esse cenário na malha e calcule o novo comprimento métrico total e o tempo adicional de trânsito para uma vazão nominal de $120\,\text{m}^3/\text{h}$ em duto de 6 polegadas.
4. **Análise Comparativa de Desempenho (Dijkstra vs. Floyd-Warshall):**  
   Considere uma refinaria de biodiesel expandida com $n = 300$ tanques/reatores e $m = 850$ tubulações. Para obter a matriz completa de menores distâncias entre todos os pares de equipamentos ($300 \times 300$):
   * Calcule o custo operacional teórico de executar $n$ vezes o Dijkstra com heap binário: $\mathcal{O}(n \cdot (n + m) \log n)$.
   * Calcule o custo de executar o algoritmo de Floyd-Warshall: $\mathcal{O}(n^3)$.  
   Qual método apresenta menor número de operações computacionais? Justifique com os cálculos numéricos.

---

## 5. Entregável da Aula 14

* **Módulo `RoteadorDijkstra` em Python:** Implementação robusta orientada a objetos do algoritmo de Dijkstra com Min-Heap binário (`heapq`), reconstrução ótima por vetor de predecessores e comutação dinâmica tolerante a falhas (`bloqueios`).
* **Caderno Jupyter Executável:** Arquivo [`14 - Menor Caminho Dijkstra e Roteamento Dinamico.ipynb`](./14%20-%20Menor%20Caminho%20Dijkstra%20e%20Roteamento%20Dinamico.ipynb) com validação das rotas nominais, testes de contingência e asserções matemáticas automáticas.
