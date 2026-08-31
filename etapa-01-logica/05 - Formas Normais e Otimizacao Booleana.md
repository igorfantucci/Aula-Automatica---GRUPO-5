<a href="https://colab.research.google.com/github/igorfantucci/Aula-Automatica---GRUPO-5/blob/main/etapa-01-logica/05%20-%20Formas%20Normais%20e%20Otimizacao%20Booleana.ipynb" target="_parent"><img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/></a>

# Aula 05: Formas Normais e Otimização Booleana
**Projeto:** Automação do Processo de Produção de Biodiesel (Transesterificação)  
**Módulo:** Representação Canonica e Minimização de Algoritmos de Controle

---

## 1. Forma Normal Disjuntiva (FND / Soma de Produtos - SOP)

Na automação do processo de transesterificação, a **Forma Normal Disjuntiva (FND)** representa a condição de acionamento do atuador do reator por meio de uma **Soma de Produtos (SOP)**. Ela é construída pela disjunção ($\lor$) de mintermos, onde cada mintermo representa uma combinação de variáveis que satisfaz a condição de liberação da reação ou segurança do processo.

Considere as variáveis de processo do sistema de dosagem/mistura:
* $A$: Fluxo de Óleo Vegetal OK ($1 = \text{OK}$)
* $B$: Solução de Metóxido ($\text{CH}_3\text{OH} + \text{NaOH}/\text{KOH}$) Pronta e Dosada ($1 = \text{OK}$)
* $C$: Temperatura do Reator na Faixa Adequada ($55^\circ\text{C} - 60^\circ\text{C}$) ($1 = \text{OK}$)
* $D$: Nível Crítico do Tanque de Lavagem atingido ($1 = \text{Crítico/Cheio}$)

A lógica de habilitação do agitador principal do reator de transesterificação em FND estrita é dada pela disjunção dos mintermos onde o processo é seguro ($V_{\text{reator}} = 1$):

$$V_{\text{reator}_{\text{FND}}} = (A \land B \land C \land \neg D) \lor (A \land B \land \neg C \land \neg D) \lor (A \land \neg B \land C \land \neg D)$$

---

## 2. Forma Normal Conjuntiva (FNC / Produto de Somas - POS)

A **Forma Normal Conjuntiva (FNC)** representa os requisitos de travamento de segurança (*Interlocks*) e neutralização do reator via **Produto de Somas (POS)**. Ela é estruturada como uma conjunção ($\land$) de cláusulas disjuntivas (maxtermos). Na FNC, qualquer cláusula falsa ($\bot$) interrompe imediatamente a operação, garantindo a parsimônia no circuito de travamento do CLP.

Considerando as mesmas variáveis do setor de reação e purificação:

$$V_{\text{reator}_{\text{FNC}}} = (A \lor B \lor C \lor \neg D) \land (A \lor B \lor \neg C \lor \neg D) \land (A \lor \neg B \lor C \lor \neg D) \land (\neg A \lor B \lor C \lor \neg D)$$

---

## 3. Álgebra Booleana e Leis de Simplificação

Para a otimização dos circuitos digitais e da lógica ladder do CLP responsável pela dosagem dos reagentes e lavagem do biodiesel bruto, aplicam-se os axiomas e lemas da Álgebra de Boole conforme a tabela a seguir:

| Lei Lógica | Identidade Booleana | Aplicação na Linha de Biodiesel |
| :--- | :--- | :--- |
| **Idempotência** | $p \land p \equiv p$, $p \lor p \equiv p$ | Eliminação de leituras redundantes de redundância de sensores de vazão de metanol. |
| **Absorção** | $p \lor (p \land q) \equiv p$ | Simplificação da lógica do tanque de ácido cítrico/HCl se a interrupção geral já estiver ativa. |
| **Elemento Inverso** | $p \land \neg p \equiv \bot$, $p \lor \neg p \equiv \top$ | Impedimento físico de abertura simultânea de válvulas de água industrial e saída de glicerina. |
| **Leis de De Morgan** | $\neg(p \land q) \equiv \neg p \lor \neg q$ | Conversão da lógica de alarme de parada (interrupção por falta de $\text{NaOH}$ **ou** falta de óleo). |
| **Consenso** | $(p \land q) \lor (\neg p \land r) \lor (q \land r) \equiv (p \land q) \lor (\neg p \land r)$ | Eliminação de *glitches* na transição entre alimentações (óleo vegetal bruto vs. óleos residuais/gordura animal). |

---

## 4. Aplicação na Automação: Otimização de Lógica

### 4.1. Análise do Sistema de Reação e Purificação
Considere a válvula solenoide $V_{\text{reagente}}$ responsável por liberar a dosagem de metóxido de sódio/potássio no reator contendo os triglicerídeos.

**Variáveis do Processo:**
* $A$: Pressão no Reator OK
* $B$: Temperatura do Reator OK ($55^\circ\text{C}-60^\circ\text{C}$)
* $C$: Válvula de Entrada de Óleo Aberta
* $D$: Parada de Emergência Ativada ($1 = \text{Emergência}$)

A lógica bruta (não simplificada) para acionamento da dosagem de reagentes é descrita como:

$$V_{\text{reagente}} = (\neg D \land A \land B \land C) \lor (\neg D \land A \land B \land \neg C) \lor (\neg D \land A \land \neg B \land C)$$

### 4.2. Minimização Passo a Passo por Álgebra Booleana

1. **Fatoração por Distribuição:**
   Agrupando os termos $(\neg D \land A \land B \land C)$ e $(\neg D \land A \land B \land \neg C)$:
   $$(\neg D \land A \land B) \land (C \lor \neg C)$$

2. **Aplicação do Elemento Inverso ($C \lor \neg C \equiv \top$):**
   $$(\neg D \land A \land B) \land 1 \equiv \neg D \land A \land B$$

3. **Recombinação com o Termo Remanescente:**
   $$V_{\text{reagente}} = (\neg D \land A \land B) \lor (\neg D \land A \land \neg B \land C)$$

4. **Fatoração de $(\neg D \land A)$:**
   $$V_{\text{reagente}} = (\neg D \land A) \land [B \lor (\neg B \land C)]$$

5. **Aplicação da Distributiva/Absorção Estendida $[B \lor (\neg B \land C) \equiv B \lor C]$:**
   $$V_{\text{otim}} = \neg D \land A \land (B \lor C)$$

### 4.3. Ganho Prático em Engenharia de Automação
* **Redução de Complexidade:** A expressão passa de **11 operadores lógicos** para apenas **3 operadores lógicos**.
* **Impacto no CLP:** Menor tempo de varredura (*scan time*), menor ocupação de memória e facilidade na implementação do diagrama ladder e blocos de função FBD.
* **Segurança:** O termo $\neg D \land A$ atua como o *interlock* mestre obrigatório (ausência de emergência e pressão adequada) antes de avaliar a temperatura ou entrada de matéria-prima.

---

## 5. Entregável da Aula 05

* **Notebook Jupyter Interativo:** [`05 - Formas Normais e Otimizacao Booleana.ipynb`](./05%20-%20Formas%20Normais%20e%20Otimizacao%20Booleana.ipynb) com suporte ao Google Colab, contendo a modelagem das formas normais (FND e FNC), simplificação booleana passo a passo e rotinas de validação comparativa.
