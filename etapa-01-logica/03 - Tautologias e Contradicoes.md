# Tautologias e Contradições — Planta de Biodiesel

## 1. Definições Formais Utilizadas

Antes de validar as regras da planta, fixam-se as três classificações possíveis para uma proposição composta, com base na sua tabela-verdade:

* **Tautologia:** proposição que resulta **Verdadeiro** para *todas* as combinações de valores-verdade de suas variáveis. Representa uma regra estruturalmente segura, válida em qualquer estado da planta.
* **Contradição:** proposição que resulta **Falso** para *todas* as combinações de valores-verdade. Representa um estado fisicamente/logicamente impossível de ocorrer, se as regras de intertravamento forem respeitadas.
* **Contingência:** proposição cujo valor-verdade depende da combinação das variáveis (verdadeira em alguns casos, falsa em outros). É o tipo mais comum de sinal de processo (ex.: $p_1$ isoladamente é uma contingência — pode ou não haver sobrepressão).

Equivalências lógicas usadas nas provas:

| Nome | Equivalência |
| --- | --- |
| Condicional | $A \rightarrow B \equiv \neg A \lor B$ |
| De Morgan (∧) | $\neg(A \land B) \equiv \neg A \lor \neg B$ |
| De Morgan (∨) | $\neg(A \lor B) \equiv \neg A \land \neg B$ |
| Distributiva | $A \land (B \lor C) \equiv (A \land B) \lor (A \land C)$ |
| Complemento | $A \land \neg A \equiv \text{Falso}$ ; $A \lor \neg A \equiv \text{Verdadeiro}$ |
| Absorção do Falso | $\text{Falso} \land A \equiv \text{Falso}$ ; $\text{Falso} \lor A \equiv A$ |

---

## 2. Regras de Intertravamento e Permissivos (Base das Provas)

| Ref. | Regra | Equação |
| --- | --- | --- |
| R1 | Aquecimento exige resfriamento de emergência disponível | $h_1 \rightarrow r_1$ |
| R2 | Evento crítico do reator | $F \equiv p_1 \lor t_1 \lor l_{high} \lor e_1$ |
| R3 | Trip: evento crítico fecha metóxido e desliga aquecimento | $F \rightarrow (\neg v_2 \land \neg h_1)$ |
| R4 | Permissivo de abertura de metóxido | $v_2 \rightarrow (\neg p_1 \land \neg t_1 \land \neg l_{high} \land m_2)$ |
| R5 | Desligamento do agitador do reator | $(l_{low} \lor e_1) \rightarrow \neg m_2$ |
| R6 | Permissivo de dreno de glicerina | $v_3 \rightarrow (l_3 \land \neg e_1)$ |
| R7 | Permissivo de partida do misturador de metóxido | $m_1 \rightarrow (\neg g_1 \land \neg e_1)$ |
| R8 | Bloqueio de transferência final de biodiesel | $b_1 \rightarrow (\neg e_1 \land f_1)$ |

---

## 3. Prova de Tautologia 1 — Segurança contra Sobrepressão com Metóxido Aberto

**Afirmação:** "Nunca é verdade que haja sobrepressão ($p_1$) com a válvula de metóxido aberta ($v_2$) *e* a regra R4 esteja sendo respeitada." Isto é, a proposição abaixo é uma **tautologia** (sempre Verdadeira):

$$T_1 \equiv \big[v_2 \rightarrow (\neg p_1 \land \dots)\big] \rightarrow \neg(p_1 \land v_2)$$

**Prova por equivalências:**

Da regra R4, extrai-se a implicação parcial relevante:
$$v_2 \rightarrow \neg p_1$$

Aplicando o condicional:
$$v_2 \rightarrow \neg p_1 \equiv \neg v_2 \lor \neg p_1$$

Isso é exatamente a negação de $(p_1 \land v_2)$, por De Morgan:
$$\neg(p_1 \land v_2) \equiv \neg p_1 \lor \neg v_2$$

Logo:
$$\big(v_2 \rightarrow \neg p_1\big) \equiv \neg(p_1 \land v_2)$$

**Tabela-verdade (confirmação exaustiva):**

| $p_1$ | $v_2$ | $v_2 \rightarrow \neg p_1$ | $\neg(p_1 \land v_2)$ | Iguais? |
| :---: | :---: | :---: | :---: | :---: |
| F | F | V | V | ✔ |
| F | V | V | V | ✔ |
| V | F | V | V | ✔ |
| V | V | F | F | ✔ |

As duas colunas coincidem em todas as linhas → a equivalência é uma **tautologia**: sempre que a regra R4 é implementada corretamente, o estado de risco $(p_1 \land v_2)$ é logicamente impossível.

---

## 4. Prova de Tautologia 2 — Trip Sempre Fecha o Metóxido em Evento Crítico

**Afirmação:** "Se ocorrer qualquer evento crítico, o metóxido estará fechado" é consequência lógica direta de R2 e R3.

$$T_2 \equiv (p_1 \lor t_1 \lor l_{high} \lor e_1) \rightarrow \neg v_2$$

Partindo de R3: $F \rightarrow (\neg v_2 \land \neg h_1)$, e de R2: $F \equiv p_1 \lor t_1 \lor l_{high} \lor e_1$, por substituição direta:

$$(p_1 \lor t_1 \lor l_{high} \lor e_1) \rightarrow (\neg v_2 \land \neg h_1)$$

Como $(\neg v_2 \land \neg h_1) \rightarrow \neg v_2$ é sempre verdadeiro (simplificação conjuntiva), por transitividade do condicional:

$$(p_1 \lor t_1 \lor l_{high} \lor e_1) \rightarrow \neg v_2 \equiv \text{Verdadeiro (tautologia)}$$

Isso comprova que **qualquer** uma das quatro condições de falha, isoladamente, já é suficiente para garantir o fechamento do metóxido — não existe combinação das quatro variáveis que resulte em $v_2$ aberto sob evento crítico.

---

## 5. Prova de Contradição 1 — Agitador Ligado durante Emergência

**Afirmação:** o estado "parada de emergência ativa **e** agitador do reator ligado" é uma **contradição**, se R5 for respeitada.

$$S_1 \equiv e_1 \land m_2$$

Da regra R5 (particularizando para $e_1$): $e_1 \rightarrow \neg m_2 \equiv \neg e_1 \lor \neg m_2$

Combinando com o estado testado:
$$(e_1 \land m_2) \land (\neg e_1 \lor \neg m_2)$$

Distribuindo:
$$\big[(e_1 \land m_2) \land \neg e_1\big] \lor \big[(e_1 \land m_2) \land \neg m_2\big]$$
$$(\text{Falso} \land m_2) \lor (e_1 \land \text{Falso})$$
$$\text{Falso} \lor \text{Falso} \equiv \text{FALSO}$$

**Tabela-verdade (confirmação exaustiva):**

| $e_1$ | $m_2$ | $e_1 \rightarrow \neg m_2$ | $S_1 \equiv e_1 \land m_2$ | $S_1 \land (e_1 \rightarrow \neg m_2)$ |
| :---: | :---: | :---: | :---: | :---: |
| F | F | V | F | F |
| F | V | V | F | F |
| V | F | V | F | F |
| V | V | F | V | F |

A última coluna é Falso em todas as linhas → **contradição**: com R5 ativa, é impossível o agitador estar ligado durante uma emergência.

---

## 6. Prova de Contradição 2 — Dreno de Glicerina Aberto sem Interface Detectada

**Afirmação:** "válvula de dreno de glicerina aberta **sem** a interface de separação detectada" é um estado contraditório, dado R6.

$$S_2 \equiv v_3 \land \neg l_3$$

Da regra R6: $v_3 \rightarrow (l_3 \land \neg e_1)$, logo $v_3 \rightarrow l_3$ (simplificação conjuntiva) $\equiv \neg v_3 \lor l_3$.

Combinando:
$$(v_3 \land \neg l_3) \land (\neg v_3 \lor l_3)$$
$$\big[(v_3 \land \neg l_3) \land \neg v_3\big] \lor \big[(v_3 \land \neg l_3) \land l_3\big]$$
$$(\text{Falso} \land \neg l_3) \lor (v_3 \land \text{Falso})$$
$$\text{Falso} \lor \text{Falso} \equiv \text{FALSO}$$

Portanto, respeitada a regra R6, a planta nunca drena glicerina sem que a interface de separação tenha sido efetivamente detectada — evitando o descarte de biodiesel junto com a glicerina.

---

## 7. Contraexemplo — Uma Regra que NÃO é Tautologia (Contingência)

Para reforçar o contraste, observa-se que a proposição isolada do permissivo de partida do misturador,

$$m_1 \rightarrow (\neg g_1 \land \neg e_1)$$

**não** é logicamente equivalente a uma tautologia se avaliada sem vínculo com a implementação real — ou seja, o valor-verdade de $m_1$, $g_1$ e $e_1$ depende do estado real dos sensores a cada instante. Trata-se de uma **contingência**: verdadeira quando a regra é respeitada operacionalmente ($m_1$ só liga com $\neg g_1 \land \neg e_1$), mas potencialmente falsa caso o intertravamento não esteja implementado no CLP. Isso justifica por que R7 precisa ser **codificada como lógica de bloqueio real** no controlador, e não apenas descrita em documento — só a implementação transforma a regra operacional em uma tautologia garantida do sistema.

---

## 8. Síntese

| Prova | Tipo | Resultado |
| --- | --- | --- |
| T1 — Sobrepressão x Metóxido aberto | Tautologia | Estado de risco logicamente impossível sob R4 |
| T2 — Evento crítico → metóxido fechado | Tautologia | Trip cobre as 4 condições de falha sem exceção |
| S1 — Agitador ligado em emergência | Contradição | Estado inatingível sob R5 |
| S2 — Dreno aberto sem interface | Contradição | Estado inatingível sob R6 |
| Contraexemplo — Partida do misturador | Contingência | Depende da implementação correta de R7 no CLP |

*(Detalhamento superficial/didático — aprofundamento formal, com tabelas-verdade completas de todas as regras (R1–R8) e formas normais, previsto para as Aulas 03 e 05 do módulo.)*
