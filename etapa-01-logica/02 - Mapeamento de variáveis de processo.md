# Mapeamento de Variáveis de Processo para Proposições Lógicas (ISA-5.1)

## Setor 100: Armazenamento e Preparação do Metóxido

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **LT-101** | Chave de Nível | Nível do Tanque de Óleo | $l_{oleo}$ | Nível de óleo **suficiente** para uma batelada |
| **P-101** | Motor / Bomba | Bomba de Óleo Vegetal | $b_{oleo}$ | Bomba de óleo LIGADA |
| **XV-101** | Válvula On/Off | Saída de Óleo | $v_{oleo}$ | Válvula de saída de óleo ABERTA |
| **LT-102** | Chave de Nível | Nível do Tanque de Metanol | $l_{met}$ | Nível de metanol **suficiente** para uma batelada |
| **P-102** | Motor / Bomba | Bomba de Metanol | $b_{met}$ | Bomba de metanol LIGADA |
| **XV-102** | Válvula de Corte | Saída de Metanol | $v_{met}$ | Válvula de segurança de metanol ABERTA |
| **LT-103** | Transmissor de Nível | Nível Tanque de Metóxido | $l_{mix}$ | Nível de metóxido atingiu o setpoint da receita |
| **AG-103** | Contator / Motor | Misturador de Metóxido | $m_{mix}$ | Agitador do metóxido LIGADO |
| **AT-100** | Detector de Gás | Vapores Inflamáveis | $g_{alm}$ | **ALARME:** Concentração de gás acima do limite |
| **FS-102** | Sensor / Chave de Fluxo | Fluxo de Metanol | $f_{met}$ | Fluxo de alimentação de metanol CONFIRMADO |

## Setor 200: Reação (Reator CSTR)

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **LT-201** | Transmissor de Nível | Nível do Reator | $l_{reator}$ | Nível atingiu o volume total da batelada |
| **LSH-201** | Chave de Nível Alto | Transbordamento | $l_{alto}$ | **ALARME:** Nível acima do limite de segurança |
| **LSL-201** | Chave de Nível Baixo | Nível Mínimo do Reator | $l_{baixo}$ | **ALARME:** Nível de líquido insuficiente para agitação |
| **PT-201** | Transmissor de Pressão | Pressão no Reator | $p_1$ | **ALARME:** Sobrepressão no vaso do reator ($\ge 2.5\text{ bar}$) |
| **TT-201** | Transmissor Temp. | Temperatura de Processo | $t_{proc}$ | Temperatura atingiu o setpoint da reação |
| **TSH-201** | Chave de Temp. Alta | Temperatura Crítica | $t_{alta}$ | **ALARME:** Temperatura excede o limite seguro |
| **AG-201** | Inversor / Motor | Agitador do Reator | $m_{reator}$ | Agitador do reator LIGADO |
| **HT-201** | Atuador Térmico | Aquecimento do Reator | $h_{1}$ | Sistema de aquecimento AUTORIZADO/LIGADO |
| **CW-201** | Circuito Hidráulico | Resfriamento Emergência | $r_{1}$ | Sistema de emergência DISPONÍVEL (Pressão OK) |
| **CW-202** | Circuito Hidráulico | Resfriamento Processo | $c_{proc}$ | Resfriamento de fim de reação LIGADO |
| **XV-201** | Válvula On/Off | Entrada de Óleo | $v_{in\_oleo}$ | Válvula de entrada de óleo no reator ABERTA |
| **XV-202** | Válvula On/Off | Entrada de Metóxido | $v_{in\_mix}$ | Válvula de entrada de metóxido no reator ABERTA |
| **XV-203** | Válvula On/Off | Saída do Reator | $v_{out\_r}$ | Válvula de esvaziamento do reator ABERTA |

## Setor 300: Decantação e Separação

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **LT-301** | Transmissor de Nível | Nível do Decantador | $l_{dec}$ | Decantador cheio (pronto para repouso) |
| **IT-301** | Sensor de Interface | Divisão de Fases | $i_{glic}$ | Interface Glicerina/Biodiesel detectada no fundo |
| **XV-301** | Válvula Proporcional | Dreno de Glicerina | $v_{glic}$ | Válvula de retirada de glicerina ABERTA |
| **XV-302** | Válvula On/Off | Saída Biodiesel Bruto | $v_{bruto}$ | Válvula de transferência para purificação ABERTA |

## Setor 400: Purificação e Armazenamento Final

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **FS-401** | Chave de Fluxo | Água de Lavagem | $f_{lav}$ | Fluxo de água de lavagem PRESENTE |
| **XV-401** | Válvula On/Off | Entrada Tanque Final | $v_{final}$ | Válvula do tanque de armazenamento ABERTA |
| **LT-402** | Transmissor de Nível | Nível Tanque Final | $l_{fim}$ | **ALARME:** Tanque de produto acabado CHEIO |
| **P-401** | Motor / Bomba | Transf. Biodiesel Puro | $b_{final}$ | Bomba de transferência final LIGADA |

---

## Dispositivos Globais e Sistema Instrumentado de Segurança (SIS / ESD)

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **ESD-100** | Botoeira / Chave ESD | Parada de Emergência Geral | $e_1$ | **EMERGÊNCIA:** Botoeira acionada no painel (Safe State Global) |

---

## Tabela de Correspondência e Notações Complementares (Mnemônica vs. Compacta)

Para assegurar total consistência entre a notação mnemônica (Aulas 02, 04, 08, 09 e 10) e a notação compacta/numérica adotada nas deduções e provas formais (Aulas 03 e 07), estabelece-se a seguinte equivalência:

| Tag Instrumento | Notação Mnemônica | Notação Compacta / Prova | Descrição do Sinal |
| :--- | :---: | :---: | :--- |
| **HT-201** | $h_1$ | $h_1$ | Aquecedor do reator ligado |
| **CW-201** | $r_1$ | $r_1$ | Resfriamento de emergência disponível |
| **PT-201** | $p_1$ | $p_1$ | Sobrepressão no vaso do reator |
| **TSH-201** | $t_{\text{alta}}$ | $t_1$ | Alarme de temperatura crítica no reator |
| **LSH-201** | $l_{\text{alto}}$ | $l_{high}$ | Alarme de nível alto / transbordamento |
| **LSL-201** | $l_{\text{baixo}}$ | $l_{low}$ | Alarme de nível baixo no reator |
| **ESD-100** | $e_1$ | $e_1$ | Parada geral de emergência ativada |
| **XV-202** | $v_{\text{in\_mix}}$ | $v_2$ | Válvula de alimentação de metóxido aberta |
| **AG-201** | $m_{\text{reator}}$ | $m_2$ | Agitador do reator em rotação |
| **XV-301** | $v_{\text{glic}}$ | $v_3$ | Válvula de dreno de glicerina aberta |
| **IT-301** | $i_{\text{glic}}$ | $l_3$ | Interface de glicerina/biodiesel detectada |
| **AG-103** | $m_{\text{mix}}$ | $m_1$ | Misturador de metóxido ligado |
| **AT-100** | $g_{\text{alm}}$ | $g_1$ | Alarme de detecção de gás inflamável |
| **P-401** | $b_{\text{final}}$ | $b_1$ | Bomba de transferência final ligada |
| **FS-401** | $f_{\text{lav}}$ | $f_1$ | Fluxo de água de lavagem presente |

---

## Diagrama Técnico de Blocos Funcionais da Planta

![Diagrama Técnico da Planta de Biodiesel](./Industria_Biodiesel.jpg)

<img width="1408" height="768" alt="Gemini_Generated_Image_lw1a21lw1a21lw1a" src="https://github.com/user-attachments/assets/56be4ad7-533a-4c4a-99ce-232f6df7f6b9" />





