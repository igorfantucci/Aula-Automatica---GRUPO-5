<a href="https://colab.research.google.com/github/igorfantucci/Aula-Automatica---GRUPO-5/blob/main/etapa-02-grafos/12%20-%20Matrizes%20de%20Incidencia%20Adjacencia%20e%20Custos.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# Aula 12: Matrizes de Incidência, Adjacência e Balanço de Massa Matricial

## 1. Fundamentos Matemáticos: A Matriz de Incidência Vértice-Aresta ($B$)

Na engenharia de automação e controle aplicada a plantas químicas de batelada contínua, a modelagem de balanço estequiométrico e distribuição de vazões volumétricas exige uma representação matemática que vincule diretamente os **equipamentos de processo (vértices)** aos **dutos de interligação (arestas)**. Essa formulação é estabelecida pela **Matriz de Incidência Vértice-Aresta**.

Seja um dígrafo $G = (V, E)$ representativo da malha de tubulações, contendo $|V| = n$ nós e $|E| = m$ trechos orientados. A **Matriz de Incidência** $B \in \{-1, 0, 1\}^{n \times m}$ é formalmente definida por:

$$B[i, j] = \begin{cases} -1, & \text{se a tubulação } e_j \text{ parte do nó } v_i \text{ (equipamento de montante / origem)} \\ +1, & \text{se a tubulação } e_j \text{ converge para o nó } v_i \text{ (equipamento de jusante / destino)} \\ 0, & \text{se o equipamento } v_i \text{ não tem conexão direta com a tubulação } e_j \end{cases}$$

### Propriedades Formais Fundamentais:

1. **Soma por Coluna Rigorosamente Nula:**  
   Para toda coluna $j \in \{1, 2, \dots, m\}$:
   $$\sum_{i=1}^n B[i, j] = 0$$
2. **Equação Matricial de Balanço de Massa em Regime Permanente:**  
   Seja $\vec{Q} \in \mathbb{R}^m$ o vetor das vazões volumétricas instantâneas em cada duto de processo ($\text{m}^3/\text{h}$) e $\vec{S} \in \mathbb{R}^n$ o vetor de taxas líquidas de geração/acumulação em cada equipamento:
   $$B \cdot \vec{Q} = \vec{S}$$

---

## 2. Aprofundamento Teórico

### 2.1. Interpretação Física da Propriedade da Soma Nula (Kirchhoff Hidráulico)

A propriedade $\sum_{i=1}^n B[i,j] = 0$ traduz algebricamente o **Princípio da Conservação da Massa** formulado em rede discreta. Como cada segmento físico de tubulação conecta um único ponto de partida a um único ponto de chegada, qualquer volume infinitesimal de fluido admitido na entrada de um duto deve ser descarregado em sua extremidade oposta (em regime incompressível e estanque).

Essa identidade representa a versão matricial da **Primeira Lei de Kirchhoff (Lei dos Nós)** aplicada a fluidos em redes hidráulicas. Se em algum momento o algoritmo do supervisório detectar que $\sum_{i=1}^n B[i, j] \neq 0$, isso acusa uma incoerência cadastral crítica no banco de dados topológico (aresta com extremidades indefinidas ou duplicadas).

---

### 2.2. Matriz de Incidência vs. Matriz de Adjacência: Comparativo de Engenharia

| Aspecto de Engenharia | Matriz de Incidência $B$ | Matriz de Adjacência Ponderada $A$ |
| :--- | :--- | :--- |
| **Dimensão Algébrica** | $n \times m$ (equipamentos $\times$ tubulações) | $n \times n$ (equipamentos $\times$ equipamentos) |
| **Espaço de Valores** | Discreto: $\{-1, 0, +1\}$ (topologia pura) | Contínuo: Pesos reais $L\text{ [m]} \in \mathbb{R}^+$ e $\infty$ |
| **Finalidade Primária no SCADA** | Balanço de massa volumétrico, integridade de rede, análise de cortes e conservação de fluxo | Roteamento dinâmico de caminho mínimo, algoritmos de menor custo (Dijkstra) |
| **Relação Algébrica Notável** | O produto $B B^T$ gera o **Laplaciano do Grafo** ($L = D - A_{\text{sim}}$) | Potências $A^k$ informam o número de passeios de comprimento $k$ entre os nós |

A conexão formal entre as matrizes é dada pelo **Laplaciano da Rede**:

$$L = B \cdot B^T$$

Onde os autovalores de $L$ determinam a conectividade algébrica (número de Fiedler), fornecendo uma métrica global da robustez da planta fabril contra desconexões acidentais.

---

### 2.3. Posto (*Rank*) da Matriz de Incidência e o Espaço Fundamental de Ciclos

Para uma malha industrial conexa com $n$ equipamentos e $m$ tubulações, o posto (*rank*) da matriz de incidência $B$ sobre $\mathbb{R}$ é dado exatamente por:

$$\text{Posto}(B) = n - 1$$

Pelo **Teorema do Núcleo e da Imagem (Rank-Nullity Theorem)**, a dimensão do espaço nulo à direita (*espaço de ciclos* ou *núcleo* de $B$) é calculada por:

$$\dim(\ker(B)) = m - \text{Posto}(B) = m - (n - 1) = m - n + 1$$

Fisicamente, essa dimensão quantifica o **número de ciclos fundamentais independentes** presentes no grafo hidráulico não-dirigido subjacente.

**Aplicação Direta na Planta de Biodiesel (Grupo 5):**  
Para nossa arquitetura básica ($n = 8$ nós e $m = 9$ dutos):

$$\dim(\ker(B)) = 9 - 8 + 1 = 2$$

Os **dois ciclos independentes** do projeto correspondem exatamente às redundâncias operacionais projetadas para garantir disponibilidade contínua (tolerância a falhas):
1. **Ciclo 1 (Redundância de Bombeamento):** Formado pelas linhas paralelas de alimentação do reator via bomba principal `P-101` e bomba reserva `P-102` (`MAN-101 -> P-101 -> R-200` vs. `MAN-101 -> P-102 -> R-200`).
2. **Ciclo 2 (Redundância de Descarga e Resfriamento):** Formado pelo trajeto direto de descarregamento do reator para decantação `R-200 -> TK-300_Decant` em paralelo com a rota alternativa de segurança pelo vaso pulmão de arrefecimento `R-200 -> TK-201_Pulmao -> TK-300_Decant`.

---

### 2.4. Balanço de Massa Matricial em Regime Permanente ($B \cdot \vec{Q} = \vec{S}$)

A equação matricial $B \cdot \vec{Q} = \vec{S}$ governa o comportamento estático das correntes de fluido:
* $\vec{Q} = [Q_1, Q_2, \dots, Q_m]^T \in \mathbb{R}^m$ contém as vazões volumétricas em cada segmento de duto.
* $\vec{S} = [S_1, S_2, \dots, S_n]^T \in \mathbb{R}^n$ indica o balanço líquido em cada nó:
  - $S_i < 0$: O equipamento é uma **fonte externa** líquida (está suprindo fluido para a rede);
  - $S_i = 0$: O equipamento é um **nó interno conservativo** (regime permanente estrito, sem acúmulo);
  - $S_i > 0$: O equipamento é um **sumidouro de recepção** (está retendo e acumulando a batelada).

Na operação nominal de carga do reator da planta de biodiesel (admissão de $50\,\text{m}^3/\text{h}$ de óleo vegetal e $70\,\text{m}^3/\text{h}$ de solução de metóxido catalisado, totalizando $120\,\text{m}^3/\text{h}$ de vazão de processo com a bomba `P-101` em carga e rota direta de descarga ativa):

$$\vec{Q} = [50.0,\, 70.0,\, 120.0,\, 0.0,\, 120.0,\, 0.0,\, 120.0,\, 0.0,\, 0.0]^T\text{ [m}^3\text{/h]}$$

O cálculo do produto matricial resulta em:

$$\vec{S} = B \cdot \vec{Q} = \begin{bmatrix} -50.0 \\ -70.0 \\ 0.0 \\ 0.0 \\ 0.0 \\ 0.0 \\ 0.0 \\ +120.0 \end{bmatrix} \begin{matrix} \text{TK-101\_Oleo (Fonte)} \\ \text{TK-103\_Mix (Fonte)} \\ \text{MAN-101 (Conservativo)} \\ \text{P-101 (Conservativo)} \\ \text{P-102 (Standby)} \\ \text{R-200 (Conservativo)} \\ \text{TK-201\_Pulmao (Conservativo)} \\ \text{TK-300\_Decant (Sumidouro)} \end{matrix}$$

A conservação global manifesta-se no somatório nulo das entradas de $\vec{S}$: $\sum_{i=1}^8 S_i = -50 - 70 + 120 = 0$.

---

### 2.5. Matriz de Custos e Roteamento de Fluidos

A **Matriz de Adjacência Ponderada (Custos)** $W \in (\mathbb{R}^+ \cup \{\infty\})^{n \times n}$ complementa a matriz de incidência estrutural: enquanto $B$ garante a integridade volumétrica e física das correntes, a matriz $W$ parametriza o algoritmo de menor caminho (Dijkstra, Aula 14).

Em contraste com algoritmos de busca estática, a matriz $W$ em um SCADA moderno é dinâmica: caso um transmissor acuse vazamento de metanol ou sobrepressão em uma linha, o custo associado àquele duto na matriz é instantaneamente modificado para $\infty$, forçando o recálculo do vetor de vazões sem paralisar a produção do biodiesel.

---

## 3. Exemplo Resolvido

**Problema:**  
Considere a tubulação de alta pressão $e_5$ (`P-101 -> R-200`), correspondente ao recalque da bomba principal de óleo/metóxido para o reator de transesterificação. Demonstre a formação da 5ª coluna da Matriz de Incidência $B$, confirme a propriedade da soma nula e explique o significado físico da conservação no nó do reator $R-200$ quando operando com vazão $Q_5 = 120\,\text{m}^3/\text{h}$ e descarga direta $Q_7 = 120\,\text{m}^3/\text{h}$.

**Resolução Passo a Passo:**  
1. **Mapeamento de Índices dos Nós:**
   - Origem da tubulação $e_5$: Nó `P-101` (Índice linha $i = 3$, considerando base zero: 0:`TK-101`, 1:`TK-103`, 2:`MAN-101`, 3:`P-101`, 4:`P-102`, 5:`R-200`, 6:`TK-201`, 7:`TK-300`).
   - Destino da tubulação $e_5$: Nó `R-200` (Índice linha $i = 5$).
2. **Definição dos Elementos da Coluna $j = 5$:**  
   $$B[3, 5] = -1 \quad (\text{duto sai de P-101})$$  
   $$B[5, 5] = +1 \quad (\text{duto entra em R-200})$$  
   $$B[i, 5] = 0 \quad (\forall i \notin \{3, 5\})$$
3. **Verificação da Soma por Coluna:**  
   $$\sum_{i=0}^7 B[i, 5] = 0 + 0 + 0 + (-1) + 0 + (+1) + 0 + 0 = 0 \quad \checkmark$$
4. **Balanço Físico no Reator `R-200`:**  
   O elemento $S_{\text{R-200}}$ é calculado pela 6ª linha de $B$ multiplicada pelo vetor $\vec{Q}$:  
   $$S_{\text{R-200}} = \sum_{k=1}^9 B[5, k] \cdot Q_k = B[5, 5] \cdot Q_5 + B[5, 6] \cdot Q_6 + B[5, 7] \cdot Q_7 + B[5, 8] \cdot Q_8$$  
   Substituindo os coeficientes ($+1$ para admissões $e_5, e_6$ e $-1$ para descargas $e_7, e_8$):  
   $$S_{\text{R-200}} = (+1) \cdot (120) + (+1) \cdot (0) + (-1) \cdot (120) + (-1) \cdot (0) = 120 - 120 = 0.0\,\text{m}^3/\text{h}$$  
   O valor $0.0$ comprova matematicamente que o volume de fluido no interior do reator em regime permanente permanece estritamente constante, prevenindo transbordamento ($LSH-201$) ou esvaziamento prematuro.

---

## 4. Atividades de Investigação

1. **Detecção de Vazamentos por Balanço Matricial Não-Conservativo:** Suponha que um rompimento parcial de gaxeta na motobomba `P-101` provoque um vazamento contínuo de $10\,\text{m}^3/\text{h}$ no duto $e_5$. A vazão que efetivamente chega ao reator passa a ser de apenas $110\,\text{m}^3/\text{h}$, enquanto o medidor de descarga do reator acusa $Q_7 = 120\,\text{m}^3/\text{h}$. Calcule o novo valor de $S_{\text{R-200}}$ e explique como o SCADA utiliza esse desbalanceamento residual para disparar o alarme de emergência.
2. **Matriz Laplaciana e Robustez Algébrica:** Demonstre algebricamente que o produto $L = B \cdot B^T$ gera a matriz Laplaciana para o dígrafo não-direcionado subjacente. Calcule o traço $\text{tr}(L)$ e mostre que ele equivale ao dobro do número de tubulações ($2 \cdot |E|$).
3. **Injeção de Reciclo e Expansão do Espaço de Ciclos:** Caso seja implantada uma linha de recirculação de biodiesel imperfeito retornando do decantador `TK-300_Decant` para o coletor `MAN-101` ($e_{10}$), recalcule a dimensão do espaço de ciclos $\dim(\ker(B))$. Identifique fisicamente o novo ciclo formado.
4. **Complexidade Assintótica de Algoritmos de Roteamento:** Compare a complexidade computacional para calcular a matriz completa de menores distâncias entre todos os pares de equipamentos ($n \times n$) utilizando:
   - (a) O algoritmo de Dijkstra executado $n$ vezes com fila de prioridade Min-Heap;
   - (b) O algoritmo de Floyd-Warshall baseado em programação dinâmica.  
   Para uma refinaria ampliada com $n = 500$ equipamentos e $m = 1200$ tubulações, qual das abordagens oferece menor tempo de computação? Justifique numericamente.

---

## 5. Entregável da Aula 12

* **Módulo Matricial de Balanço de Massa em Python:** Implementação orientada a objetos da classe `CalculadorIncidencia`, contendo métodos estáticos para montagem automática de $B \in \{-1, 0, 1\}^{n \times m}$, conferência de consistência topológica $\sum B[:, j] = 0$ e cálculo do produto linear $B \cdot \vec{Q} = \vec{S}$.
* **Caderno Jupyter Executável:** Arquivo [`12 - Matrizes de Incidencia Adjacencia e Custos.ipynb`](./12%20-%20Matrizes%20de%20Incidencia%20Adjacencia%20e%20Custos.ipynb) com a simulação do regime de carregamento nominal e validação das leis de conservação.
