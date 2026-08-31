# Aula 09: Motores de Inferência — Encadeamento para Frente e para Trás (Forward e Backward Chaining)
## Processo: Planta Industrial de Produção de Biodiesel (Transesterificação em Batelada — Grupo 5)

---

## 1. Fundamentos Matemáticos: Algoritmos de Inferência em Lógica de Produção

Um **Motor de Inferência (*Inference Engine*)** é o algoritmo formal responsável por aplicar as regras da base de conhecimento ($\mathcal{R}$) sobre os fatos ativos ($\mathcal{F}$) para produzir novas deduções ou provar hipóteses de segurança operacional em tempo de execução.

### 1.1. Encadeamento para Frente (*Forward Chaining* — Data-Driven / Bottom-Up)
* **Princípio:** Inicia com os **fatos conhecidos** (telemetria em tempo real dos sensores ISA-5.1) e dispara todas as regras cujos antecedentes são verdadeiros (*Modus Ponens* sucessivo), adicionando os consequentes à memória de trabalho até alcançar um ponto fixo (*Fixed Point*).
* **Resolução de Conflitos:** Ordenação determinística baseada na **prioridade de segurança** (1 a 10) e na **especificidade** (cardinalidade dos antecedentes).
* **Aplicação na Planta de Biodiesel:** Diagnóstico imediato de eventos como *Runaway* térmico no reator $\text{R-200}$, vazamento de vapores inflamáveis de metanol no Setor 100 e disparo automático em cascata de intertravamentos de corte (*Trip* de válvulas e aquecedores).

### 1.2. Encadeamento para Trás (*Backward Chaining* — Goal-Driven / Top-Down)
* **Princípio:** Inicia com uma **hipótese ou meta** de segurança (ex.: *"A válvula de dosagem de metóxido XV-202 deve ser desarmada?"* ou *"Houve contaminação no dreno de glicerina?"*) e busca recursivamente provar as submetas necessárias a partir dos fatos ativos ou de outras regras.
* **Explicabilidade e Auditoria Forense:** Constrói a **Árvore de Prova Dedutiva (*Proof Tree / Audit Trail*)**, evidenciando explicitamente a cadeia causal de falhas para análise pós-evento.

---

## 2. Base de Conhecimento Especialista da Planta de Biodiesel (Grupo 5)

| ID | Prioridade | Severidade | SE (Antecedentes) | ENTÃO (Consequente) | Diagnóstico | Procedimento POP |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **R-01** | 10 | CRÍTICA | $t_{\text{alta}} \land p_1$ | `REACAO_RUNAWAY_REATOR` | Exotermia Descontrolada e Sobrepressão no Reator R-200 | POP-SIS-01: Desarme total imediato, corte de alimentação e resfriamento de emergência |
| **R-02** | 10 | CRÍTICA | `REACAO_RUNAWAY_REATOR` $\land v_{\text{in\_mix}}$ | `TRIP_CORTE_METOXIDO` | Fechamento de Emergência da Válvula de Metóxido XV-202 | POP-SIS-02: Fechar imediatamente XV-202 e bloquear dosagem de catalisador |
| **R-03** | 10 | CRÍTICA | `REACAO_RUNAWAY_REATOR` $\land h_1$ | `TRIP_DESLIGA_AQUECIMENTO` | Desarme Forçado do Sistema de Aquecimento HT-201 | POP-SIS-03: Desligar resistências térmicas HT-201 para evitar explosão |
| **R-04** | 9 | CRÍTICA | $h_1 \land \neg r_1$ | `RISCO_CHOQUE_TERMICO` | Aquecimento Ligado sem Resfriamento de Emergência (Violação Regra R1) | POP-HT-01: Desarmar HT-201 e verificar pressurização da linha CW-201 |
| **R-05** | 9 | CRÍTICA | $g_{\text{alm}} \land m_{\text{mix}}$ | `RISCO_IGNICAO_METOXIDO` | Vapores Inflamáveis de Metanol com Misturador AG-103 Ligado (Violação Regra R7) | POP-SST-04: Desligar AG-103, acionar exaustão de emergência e alarme de evacuação |
| **R-06** | 8 | ALTA | $v_{\text{glic}} \land \neg i_{\text{glic}}$ | `CONTAMINACAO_DRENO_GLICERINA` | Válvula de Dreno XV-301 Aberta sem Detecção de Interface (Violação Regra R6) | POP-SEP-02: Fechar XV-301 imediatamente para evitar descarte indevido de biodiesel |
| **R-07** | 7 | ALTA | $b_{\text{final}} \land \neg f_{\text{lav}}$ | `RISCO_CAVITACAO_LAVAGEM_INCOMPLETA` | Bomba Final P-401 Ligada sem Fluxo de Água de Lavagem (Violação Regra R8) | POP-PUR-01: Desligar bomba P-401 e restabelecer linha de água de lavagem FS-401 |
| **R-08** | 8 | ALTA | $b_{\text{final}} \land l_{\text{fim}}$ | `TRIP_TRANSBORDAMENTO_FINAL` | Bomba P-401 Ligada com Tanque de Armazenamento Cheio | POP-STG-03: Desligar bomba P-401 e fechar válvula de entrada XV-401 |
| **R-09** | 10 | CRÍTICA | $e_1$ | `PARADA_EMERGENCIA_GERAL` | Acionamento do Botão de Parada de Emergência ESD-100 | POP-ESD-01: Desarme geral de segurança e travamento de todos os atuadores |
| **R-10** | 10 | CRÍTICA | `PARADA_EMERGENCIA_GERAL` $\land m_{\text{reator}}$ | `TRIP_AGITADOR_REATOR` | Desarme Forçado do Agitador AG-201 por Emergência Ativa (Violação Regra R5) | POP-ESD-02: Cortar alimentação do inversor do agitador AG-201 |

---

## 3. Entregável da Aula 09

* **Notebook Jupyter:** [`09 - Motor de Inferencia Forward e Backward Chaining.ipynb`](file:///c:/Users/Vitória/Documents/GitHub/Aula-Automatica---GRUPO-5/etapa-01-logica/09%20-%20Motor%20de%20Inferencia%20Forward%20e%20Backward%20Chaining.ipynb) contendo a modelagem orientada a objetos das classes `RegraProducao`, `BaseConhecimentoBiodiesel` e `MotorInferenciaBiodiesel`, simulação com múltiplos cenários industriais e verificação de integridade com 100% de asserções atendidas.
