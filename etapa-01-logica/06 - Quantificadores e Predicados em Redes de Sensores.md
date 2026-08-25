## 1. Fundamentos Matemáticos: Lógica de Primeira Ordem (FOL)

Enquanto a lógica proposicional trata sentenças atômicas indivisíveis, a **Lógica de Predicados** permite parametrizar propriedades sobre domínios e conjuntos finitos de ativos industriais. Na engenharia de controle, isso é fundamental para lidar com malhas de instrumentos redundantes e redes de sensores distribuídos.

*   **Predicado $P(x)$:** Função booleana $P: U \rightarrow \{0, 1\}$ onde $U$ é o universo de discurso (ex: conjunto de tanques $T$, conjunto de sensores de gás $S$).
*   **Quantificador Universal ($\forall x \in U : P(x)$):** "Para todo $x$ em $U$, $P(x)$ é Verdadeiro".
    *   Expansão em domínio finito $U = \{x_1, x_2, ..., x_n\}$: $\forall x P(x) \equiv P(x_1) \land P(x_2) \land ... \land P(x_n)$.
*   **Quantificador Existencial ($\exists x \in U : P(x)$):** "Existe ao menos um $x$ em $U$ tal que $P(x)$ é Verdadeiro".
    *   Expansão em domínio finito: $\exists x P(x) \equiv P(x_1) \lor P(x_2) \lor ... \lor P(x_n)$.

---
