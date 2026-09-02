# Indústria Escolhida: Produção de Biodiesel (Transesterificação em Batelada)

## 🌐 Simulador SCADA em Operação (GitHub Pages)

Acesse o sistema supervisório e motor de inferência em tempo real diretamente pelo navegador:

👉 **[SCADA-Core Automática — Planta de Biodiesel (Grupo 5)](https://igorfantucci.github.io/Aula-Automatica---GRUPO-5/)**

O simulador web permite:
* **Diagrama P&ID interativo completo:** Desenho vetorial em SVG nativo dos 4 setores industriais, detalhando tanques, tubulações com fluxo contínuo animado, bombas, válvulas normalizadas ($\bowtie$) com indicação dinâmica de abertura/corte, e cabos de sinalização e comando ligados ao painel do CLP;
* **Injeção de falhas com 1 clique:** Bateria de testes com os 5 cenários de estresse operacional da Aula 10;
* **Manipulação contínua:** Sliders analógicos ($PT-201, TT-201, LT-201, AT-100$) com discretização por histerese ($\delta$);
* **Painel de Segurança SIS/SIL 3:** Avaliação instantânea de Permissivos de Partida e Desarmes Fail-Safe;
* **Sistema Especialista:** Encadeamento progressivo (*Forward Chaining*) processando as Cláusulas de Horn R-01 a R-08;
* **Auditoria Forense Explicável (XAI):** Reconstrução da árvore de prova causal por encadeamento regressivo (*Backward Chaining*).

---

## 🔬 O Processo Químico: Síntese de Biodiesel por Transesterificação

A produção de biodiesel na planta baseia-se na **transesterificação alcalina de triglicerídeos em regime de batelada (*batch*)**. O processo consiste na reação química entre lipídios de origem vegetal ou animal e um álcool de cadeia curta (metanol), catalisada por uma base forte, para produzir ésteres metílicos de ácidos graxos (biodiesel) e glicerol como subproduto de valor comercial agregado.

### 1. Equação Química Estequiométrica
$$\text{Triglicerídeo (Óleo Vegetal)} + 3\,\text{CH}_3\text{OH (Metanol)} \xrightarrow{\text{NaOH / KOH}} 3\,\text{Ésteres Metílicos (Biodiesel)} + \text{Glicerol (Subproduto)}$$

* **Estequiometria:** Para cada $1\text{ mol}$ de triglicerídeo processado, são necessários estequiometricamente $3\text{ mols}$ de metanol. Na prática industrial, opera-se com excesso molar de metanol (proporção típica de 6:1) para deslocar o equilíbrio químico no sentido da formação de produtos, de acordo com o Princípio de Le Chatelier.
* **Catalisador:** Hidróxido de sódio ($\text{NaOH}$) ou hidróxido de potássio ($\text{KOH}$) dissolvido no metanol no Setor 100, gerando a espécie reativa ativa: o **íon metóxido ($\text{CH}_3\text{O}^-$)**.

### 2. Aspectos Termodinâmicos e Cinéticos
* **Termodinâmica:** A reação é **levemente exotérmica** ($\Delta H^\circ \approx -15 \text{ a } -25\text{ kJ/mol}$), liberando calor à medida que os reagentes se convertem em ésteres.
* **Janela Operacional Rígida:** A temperatura da reação deve ser estritamente controlada entre **$55^\circ\text{C}$ e $60^\circ\text{C}$**:
  * *Abaixo de $55^\circ\text{C}$:* A viscosidade do óleo vegetal permanece alta, prejudicando a mistura física das fases imiscíveis e tornando a cinética química excessivamente lenta.
  * *Acima de $60^\circ\text{C}$:* A temperatura aproxima-se perigosamente do **ponto de ebulição do metanol puro ($64.7^\circ\text{C}$ à pressão atmosférica)**.
* **O Fenômeno do Runaway Térmico:** Se o aquecedor permanecer energizado ou se o resfriamento de salvaguarda falhar, a temperatura atinge $65^\circ\text{C}$. Nessa temperatura, o metanol sofre **ebulição instantânea (*flash boiling*)**, expandindo seu volume cerca de 300 vezes. Em um vaso fechado e estanque, essa vaporização súbita faz a pressão ($PT-201$) disparar para além de $2.5\text{ bar}$, criando o cenário de risco iminente de rompimento de vaso ou explosão.

---

## 🏭 Arquitetura da Planta Industrial (Topologia em 4 Setores)

A planta foi estruturada em **quatro setores operacionais segregados**, projetados segundo as normas **ISA-5.1** (simbologia e identificação de instrumentação) e **IEC 61511 / IEC 61508** (Sistemas Instrumentados de Segurança - SIS para malhas de integridade **SIL 3**):

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        PAINEL CLP / SCADA-CORE (Controlador Lógico)                    │
└───────┬────────────────────────┬────────────────────────┬──────────────────────┬───────┘
        │ Sinais / Comandos      │ Sinais / Comandos      │ Sinais / Comandos    │ Sinais
        ▼                        ▼                        ▼                      ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐   ┌──────────────────┐
│    SETOR 100     │     │    SETOR 200     │     │    SETOR 300     │   │    SETOR 400     │
│  Armazenamento & │ ──► │   Reator CSTR    │ ──► │   Decantação &   │──►│  Purificação &   │
│ Preparo Metóxido │     │ (Setor Crítico)  │     │ Separação Fases  │   │ Armazenamento    │
└──────────────────┘     └──────────────────┘     └──────────────────┘   └──────────────────┘
```

### Setor 100: Armazenamento e Preparação do Metóxido
* **Objetivo:** Recepção, armazenagem segura e homogeneização dos insumos primários da reação.
* **Equipamentos e Instrumentos:**
  * Silo de Óleo Vegetal Degomado ($TK-101$) com sensor de nível ($LT-101$), válvula de bloqueio ($XV-101$) e bomba de transferência ($P-101$);
  * Vaso de Metanol Líquido ($TK-102$) com detector de vapores de metanol na atmosfera ($AT-100$), válvula de bloqueio ($XV-102$) e bomba dosadora ($P-102$);
  * Tanque de Preparação do Metóxido ($TK-103$) com agitador mecânico de dissolução do catalisador ($AG-103$).

### Setor 200: Reação de Transesterificação (Reator CSTR R-200 — Núcleo Crítico)
* **Objetivo:** Condução da reação química em batelada com controle rigoroso de temperatura, pressão e agitação mecânica.
* **Equipamentos e Instrumentos:**
  * Reator de Batelada Encamisado ($R-200$) com jaqueta térmica para aquecimento e arrefecimento;
  * Agitador Mecânico ($AG-201$) acionado por inversor de frequência, garantindo contato íntimo entre as fases imiscíveis (óleo e metóxido);
  * Aquecedor Elétrico/Vapor ($HT-201$) na jaqueta térmica;
  * Circuito Hidráulico de Resfriamento de Emergência ($CW-201$), mantido sob pressão contínua $\ge 3.0\text{ bar}$ como salvaguarda passiva/ativa;
  * Válvulas de corte rápido: $XV-201$ (admissão de óleo), $XV-202$ (corte rápido de dosagem de metóxido) e $XV-203$ (descarga da batelada);
  * Transmissores de campo de alta integridade: Pressão ($PT-201$), Temperatura ($TT-201$) e Nível ($LT-201$).

### Setor 300: Decantação e Separação de Fases
* **Objetivo:** Separação gravitacional bifásica entre a fase leve (biodiesel bruto) e a fase pesada (glicerina bruta).
* **Equipamentos e Instrumentos:**
  * Vaso Decantador Cônico ($TK-300$), projetado para decantação estática por diferença de massa específica ($\rho_{\text{biodiesel}} \approx 0.88\text{ g/cm}^3$ vs. $\rho_{\text{glicerina}} \approx 1.26\text{ g/cm}^3$);
  * Sensor Óptico de Interface ($IT-301$) baseado em refração infravermelha, posicionado na linha divisória das duas fases;
  * Válvula de dreno inferior ($XV-301$) para purga estrita de glicerina bruta para tambores de subproduto;
  * Válvula de transbordo superior ($XV-302$) para transferência de biodiesel bruto à etapa de purificação.

### Setor 400: Purificação, Lavagem e Armazenamento Final
* **Objetivo:** Remoção de impurezas solúveis (soda residual, sabões e traços de álcool) por lavagem ácida/aquosa e estocagem do produto acabado.
* **Equipamentos e Instrumentos:**
  * Coluna de Lavagem com bicos aspersores (*sprays*) de água desmineralizada e ácido cítrico de neutralização de pH;
  * Chave de fluxo de água de lavagem ($FS-401$) para garantir fluxo contínuo antes da autorização de expedição;
  * Válvula de alimentação final ($XV-401$);
  * Tanque de Armazenamento de Biodiesel Puro ($TK-402$) com transmissor de nível ($LT-402$);
  * Bomba centrífuga de expedição final de combustível ($P-401$).

---

## ⚠️ Riscos Industriais Críticos e Filosofia de Intertravamento Fail-Safe

O sistema de automação implementado no SCADA foi formalizado para prevenir e mitigar os quatro riscos de maior severidade operacional da planta:

1. **Sobrepressão e Runaway Térmico no Reator (Regras R-01 e R-02):**
   * *Gatilho:* $PT-201 \ge 2.5\text{ bar} \land TT-201 \ge 65.0^\circ\text{C}$.
   * *Ação SIS:* Desarme instantâneo do aquecedor $HT-201$ (POP-SIS-01), fechamento forçado da válvula de corte de metóxido $XV-202$ e desligamento da bomba $P-102$ (POP-SIS-02), associado à abertura plena da água de resfriamento $CW-201$.
2. **Vazamento e Formação de Atmosfera Explosiva de Metanol (Regra R-03):**
   * *Gatilho:* $AT-100 \ge 20\text{ ppm}$ no Setor 100 (Limite de pré-alarme para LEL de 6% e limite tóxico IDLH de 300 ppm).
   * *Ação SIS:* Fechamento de $XV-102$ e $XV-202$, desenergização da bomba $P-102$ e acionamento de exaustores de emergência à prova de explosão (POP-SST-03).
3. **Cavitação Mecânica do Agitador por Nível Baixo (Regra R-06):**
   * *Gatilho:* $LT-201 \le 15\% \land AG-201\text{ ligado}$.
   * *Ação SIS:* Desarme imediato do inversor de frequência de $AG-201$ para evitar vórtice destrutivo e empenamento de pás (POP-MA-05).
4. **Perda de Biodiesel Comercial no Dreno de Subproduto (Regra R-07):**
   * *Gatilho:* Comando de purga ativo em $XV-301 \land \text{perda de interface óptica em } IT-301$.
   * *Ação SIS:* Bloqueio imediato do dreno $XV-301$ para impedir descarte acidental de éster puro (POP-SEP-02).
5. **Parada Geral de Emergência em Segurança Positiva (Fail-Safe):**
   * *Gatilho:* Botoeira física de emergência $ESD-100$ ($e_1 = 1$). O contato é eletricamente **Normalmente Fechado (NF)**: se o botão for pressionado, se o cabo romper ou se houver perda de energia, o circuito abre e a planta assume o estado seguro global (*SafeState Global*).

---

## 📦 Insumos: Matérias-Primas e Reagentes Químicos

* **Fontes de Lipídios (Base da Reação):**
  * **Óleo Vegetal (Triglicerídeos):** Matéria-prima primária (soja, girassol, canola) processada para conversão em biocombustível.
  * **Óleos Residuais ou Gordura Animal:** Fontes secundárias alternativas de lipídios compatíveis com o reator em batelada.

* **Fontes de Álcool (Reagente):**
  * **Metanol ($\text{CH}_3\text{OH}$):** Reagente de alta reatividade que reage com os triglicerídeos para liberar as cadeias de ésteres metílicos.

* **Catalisadores e Químicos Auxiliares:**
  * **Hidróxido de Sódio ($\text{NaOH}$) ou de Potássio ($\text{KOH}$):** Catalisador alcalino dissolvido no metanol para formar a solução de metóxido.
  * **Água Industrial Desmineralizada ($\text{H}_2\text{O}$):** Empregada na lavagem para extrair restos de catalisador e sabões por afinidade polar.
  * **Ácido de Neutralização (Ácido Cítrico ou Clorídrico - $\text{HCl}$):** Adicionado em doses controladas para estabilização de pH do biodiesel final.

---

## 📚 Estrutura do Repositório — Etapa 01: Lógica Formal e Sistemas Especialistas

A pasta [`etapa-01-logica`](./etapa-01-logica/) contém toda a modelagem formal, provas matemáticas de segurança e cadernos executáveis em Python/Jupyter:

| Módulo | Documento Teórico (.md) | Caderno Executável (.ipynb) | Conteúdo e Foco de Engenharia |
| :---: | :--- | :---: | :--- |
| **00** | [00 - Materias Primas e Componentes.md](./etapa-01-logica/00%20-%20Materias%20Primas%20e%20Componentes.md) | — | Descrição detalhada dos insumos, produtos, subprodutos e equipamentos por setor (100 a 400). |
| **02** | [02 - Mapeamento de variáveis de processo.md](./etapa-01-logica/02%20-%20Mapeamento%20de%20vari%C3%A1veis%20de%20processo.md) | — | Mapeamento formal de instrumentos e sensores para proposições booleanas (norma ISA-5.1). |
| **03** | [03 - Tautologias e Contradicoes.md](./etapa-01-logica/03%20-%20Tautologias%20e%20Contradicoes.md) | [03 - Tautologias e Contradicoes.ipynb](./etapa-01-logica/03%20-%20Tautologias%20e%20Contradicoes.ipynb) | Provas dedutivas de tautologias de segurança e contradições de risco operacional. |
| **04** | [04 - Logica Proposicional Conectivos e Permissivos.md](./etapa-01-logica/04%20-%20Logica%20Proposicional%20Conectivos%20e%20Permissivos.md) | [04 - Logica Proposicional Conectivos e Permissivos.ipynb](./etapa-01-logica/04%20-%20Logica%20Proposicional%20Conectivos%20e%20Permissivos.ipynb) | Conectivos lógicos, permissivos de partida (*Start Permissives*) e blocos de intertravamento. |
| **05** | [05 - Formas Normais e Otimizacao Booleana.md](./etapa-01-logica/05%20-%20Formas%20Normais%20e%20Otimizacao%20Booleana.md) | [05 - Formas Normais e Otimizacao Booleana.ipynb](./etapa-01-logica/05%20-%20Formas%20Normais%20e%20Otimizacao%20Booleana.ipynb) | Minimização por álgebra de Boole, formas canônicas FND (SOP) e FNC (POS) para CLP. |
| **06** | [06 - Quantificadores e Predicados em Redes de Sensores.md](./etapa-01-logica/06%20-%20Quantificadores%20e%20Predicados%20em%20Redes%20de%20Sensores.md) | [06 - Quantificadores e Predicados em Redes de Sensores.ipynb](./etapa-01-logica/06%20-%20Quantificadores%20e%20Predicados%20em%20Redes%20de%20Sensores.ipynb) | Lógica de predicados de 1ª ordem, quantificadores $\forall$ (FORALL) e $\exists$ (EXISTS) em redes de sensores. |
| **07** | [07 - Validade e Inferencia Logica na Seguranca de Processos.md](./etapa-01-logica/07%20-%20Validade%20e%20Inferencia%20Logica%20na%20Seguranca%20de%20Processos.md) | [07 - Validade e Inferencia Logica na Seguranca de Processos.ipynb](./etapa-01-logica/07%20-%20Validade%20e%20Inferencia%20Logica%20na%20Seguranca%20de%20Processos.ipynb) | Regras canônicas de inferência (MP, MT, SH, SD, RES, DC), falácias e prova formal de matriz ESD. |
| **08** | [08 - Base de Conhecimento e Regras de Diagnostico.md](./etapa-01-logica/08%20-%20Base%20de%20Conhecimento%20e%20Regras%20de%20Diagnostico.md) | [08 - Base de Conhecimento e Regras de Diagnostico.ipynb](./etapa-01-logica/08%20-%20Base%20de%20Conhecimento%20e%20Regras%20de%20Diagnostico.ipynb) | Sistemas especialistas (RBS), Cláusulas de Horn, priorização SIL e procedimentos POP. |
| **09** | [09 - Motor de Inferencia Forward e Backward Chaining.md](./etapa-01-logica/09%20-%20Motor%20de%20Inferencia%20Forward%20e%20Backward%20Chaining.md) | [09 - Motor de Inferencia Forward e Backward Chaining.ipynb](./etapa-01-logica/09%20-%20Motor%20de%20Inferencia%20Forward%20e%20Backward%20Chaining.ipynb) | Motores de encadeamento progressivo (*Forward*) e regressivo (*Backward*) com explicabilidade (XAI). |
| **10** | [10 - Avaliacao Modulo 1 Motor de Intertravamento e Diagnostico.md](./etapa-01-logica/10%20-%20Avaliacao%20Modulo%201%20Motor%20de%20Intertravamento%20e%20Diagnostico.md) | [10 - Avaliacao Modulo 1 Motor de Intertravamento e Diagnostico.ipynb](./etapa-01-logica/10%20-%20Avaliacao%20Modulo%201%20Motor%20de%20Intertravamento%20e%20Diagnostico.ipynb) | SCADA-Core integrado: telemetria, permissivos failsafe, matriz ESD e bateria de estresse de 5 cenários. |

---

## 🖼️ Diagrama Funcional da Planta de Biodiesel

![Diagrama Técnico de Blocos Funcionais](./etapa-01-logica/Industria_Biodiesel.jpg)

---

## 📽️ Apresentação Executiva (Slides)

Para a defesa do **Motor de Intertravamento e Diagnóstico**, a apresentação em slides está disponível em formato PDF:

* **Apresentação em PDF:** [`apresentacao_etapa_01.pdf`](./etapa-01-logica/apresentacao_etapa_01.pdf)
