# Planta de Produção de Biodiesel

Este repositório abrigará o desenvolvimento do projeto SCADA-Core Automática, cujo objetivo será a estruturação de um motor computacional de supervisão, controle e diagnóstico para uma planta automatizada de produção de biodiesel. O processo produtivo abordado baseia-se na transesterificação em batelada, onde óleo vegetal, metanol e um catalisador alcalino reagem sob condições térmicas estritas. A infraestrutura física engloba áreas classificadas de tancagem, preparo exotérmico de metóxido, reação principal em reator CSTR, decantação gravimétrica e purificação final. Devido aos riscos de inflamabilidade e sobrepressão, a operação demandará alta confiabilidade na execução sequencial das etapas e na segurança dos atuadores.

Neste contexto, o projeto integrará os fundamentos da matemática discreta à engenharia de controle e automação para projetar o núcleo lógico do sistema SCADA. A lógica formal (proposicional e de predicados) será aplicada para modelar e validar matematicamente os intertravamentos de segurança, prevenindo falhas críticas. A teoria dos grafos será utilizada para o roteamento dinâmico de fluidos e logística de amostragem, permitindo o recalculo de rotas perante bloqueios nos manifolds. O gerenciamento de diagnósticos empregará estruturas de árvores para categorizar e suprimir cascatas de alarmes secundários, isolando a causa-raiz das anomalias. Adicionalmente, a teoria das relações será empregada para estruturar a matriz de transição de estados do processo sequencial e os níveis de permissão da interface homem-máquina. A aplicação destas metodologias estabelecerá uma arquitetura determinística e robusta para operações químicas de risco contínuo.

## Imagem da Planta:
<img width="1920" height="1080" alt="planta1" src="https://github.com/user-attachments/assets/f617efc2-1aab-48eb-9439-54cc7fb2ee7e" />



## Funcionamento Detalhado

# Processo de Produção de Biodiesel — SCADA-Core Automática

## 1. Descrição do processo

O projeto SCADA-Core Automática tem como objetivo desenvolver um sistema computacional de supervisão, controle e diagnóstico aplicado a uma planta automatizada de produção de biodiesel.

O processo produtivo é realizado em **batelada**, no qual uma quantidade determinada de óleo vegetal, metanol e catalisador é processada sequencialmente para produzir biodiesel. Devido à presença de substâncias inflamáveis e às condições térmicas do processo, o sistema deve possuir intertravamentos de segurança e uma sequência de operação bem definida.

O fluxo geral do processo é:

**Armazenamento → Preparação do metóxido → Enchimento do reator → Aquecimento → Reação → Resfriamento → Decantação → Separação → Purificação → Armazenamento do biodiesel.**

---

## 2. Armazenamento das matérias-primas

O processo começa com o armazenamento das matérias-primas.

O **óleo vegetal** é armazenado em um tanque equipado com sensores de nível e temperatura, bomba de transferência e válvulas de controle.

O **metanol** é armazenado em um tanque separado. Como é uma substância altamente inflamável, sua transferência deve ser realizada somente quando todas as condições de segurança estiverem satisfeitas.

O **catalisador alcalino** também é armazenado separadamente e posteriormente dosado de acordo com a receita da batelada.

Antes do início do processo, o SCADA-Core verifica os permissivos necessários, como nível dos tanques, disponibilidade das bombas, posição das válvulas e condições de segurança.

---

## 3. Preparação do metóxido

A primeira etapa de preparação consiste na combinação do metanol com o catalisador, formando o metóxido que será utilizado na reação.

A sequência pode ser representada por:

1. Verificar disponibilidade de metanol;
2. Abrir a válvula de transferência;
3. Acionar a bomba;
4. Transferir a quantidade especificada;
5. Adicionar o catalisador;
6. Realizar a mistura;
7. Confirmar a preparação;
8. Liberar o metóxido para o reator.

Essa etapa possui intertravamentos devido ao risco associado ao metanol.

---

## 4. Enchimento do reator

Após a preparação das matérias-primas, inicia-se o carregamento do reator.

Primeiramente, o óleo vegetal é transferido para o reator. O sistema aciona a bomba e abre a válvula correspondente.

O nível do reator é monitorado continuamente.

Quando o nível programado é atingido:

**Bomba de óleo → DESLIGADA**

**Válvula de óleo → FECHADA**

Em seguida, o metóxido é transferido para o reator.

Quando a quantidade necessária é atingida, a transferência é interrompida.

O processo só pode avançar para a próxima etapa quando os sensores confirmarem que o carregamento foi concluído corretamente.

---

## 5. Aquecimento do reator

Após o carregamento, inicia-se o aquecimento do reator.

O sistema monitora a temperatura através de um sensor instalado no equipamento.

O aquecimento pode ser realizado por uma resistência, camisa de aquecimento ou circulação de fluido térmico, dependendo da configuração da planta.

O controlador compara continuamente a temperatura medida com a temperatura desejada:

**Erro = Temperatura desejada − Temperatura medida**

O controle pode ser realizado por um controlador PID.

---

## 6. Intertravamento de segurança

Uma das principais aplicações da lógica formal no projeto é o controle dos intertravamentos.

O sistema de aquecimento não deve ser autorizado caso o sistema de resfriamento de emergência esteja indisponível.

Podemos representar:

**H → R**

Onde:

- **H** = aquecimento autorizado;
- **R** = sistema de resfriamento de emergência disponível.

A condição equivalente de bloqueio é:

**¬R → ¬H**

Portanto, caso o sistema de resfriamento de emergência apresente uma falha, o SCADA-Core deverá bloquear o acionamento do aquecimento.

Esse mecanismo evita que uma falha em um sistema de segurança permita a operação de um equipamento potencialmente perigoso.

---

## 7. Reação de transesterificação

Quando o reator atinge a condição adequada de temperatura, inicia-se a reação de transesterificação.

De forma simplificada:

**Óleo + Metanol → Biodiesel + Glicerina**

Durante essa etapa, o agitador mantém os componentes misturados para favorecer o contato entre as substâncias.

O SCADA-Core monitora:

- temperatura;
- tempo de reação;
- nível;
- pressão;
- velocidade de agitação;
- estado das válvulas;
- estado das bombas;
- alarmes de segurança.

A reação permanece ativa até que o tempo de processo especificado seja atingido.

---

## 8. Controle de temperatura

Durante a reação, a temperatura deve permanecer dentro da faixa estabelecida para o processo.

O controlador PID recebe a temperatura medida e compara com o setpoint.

Caso a temperatura esteja abaixo do valor desejado, o sistema aumenta a ação de aquecimento.

Caso esteja acima do valor desejado, a ação de aquecimento é reduzida.

Se a temperatura ultrapassar um limite de segurança, o sistema deve executar a estratégia de emergência definida no projeto.

---

## 9. Resfriamento

Ao final da reação, o aquecimento é desligado e inicia-se o resfriamento.

O sistema aciona o circuito de resfriamento e acompanha a temperatura.

Quando a temperatura atingir a condição especificada, o sistema encerra essa etapa e libera a transferência da mistura para o tanque de decantação.

---

## 10. Decantação

Após o resfriamento, a mistura é encaminhada para um tanque de decantação.

Nesse tanque ocorre uma separação por gravidade.

A fase mais leve é composta principalmente pelo biodiesel, enquanto a fase mais pesada contém principalmente glicerina e outros componentes.

Durante essa etapa, o sistema mantém o tanque em repouso durante o período programado.

---

## 11. Separação da glicerina

Depois do período de decantação, a fase inferior é retirada através de uma válvula localizada na parte inferior do tanque.

O controle pode utilizar sensores de nível, sensores de interface ou temporização.

Quando a quantidade desejada de glicerina é retirada, a válvula é fechada.

O biodiesel permanece no tanque para seguir para a etapa de purificação.

---

## 12. Purificação do biodiesel

Após a separação, o biodiesel ainda pode apresentar impurezas, como resíduos de catalisador, metanol, glicerina e água.

Por isso, o produto pode passar por etapas de purificação.

Dependendo da tecnologia adotada, podem ser utilizadas etapas como:

**Lavagem → Separação → Secagem → Filtragem**

O objetivo é obter um biodiesel com as características especificadas para o processo.

---

## 13. Armazenamento do biodiesel

Após a purificação, o biodiesel é transferido para o tanque de produto.

A bomba de transferência é acionada e as válvulas correspondentes são abertas.

O nível do tanque é monitorado durante toda a transferência.

Quando a operação termina:

**Bomba → DESLIGADA**

**Válvulas → FECHADAS**

A batelada é então considerada concluída.

---

# 14. Sequência de controle do processo

A sequência principal pode ser representada através de uma máquina de estados:

```text
S0 - INÍCIO
       ↓
S1 - VERIFICAR PERMISSIVOS
       ↓
S2 - ENCHER REATOR COM ÓLEO
       ↓
S3 - ADICIONAR METÓXIDO
       ↓
S4 - AQUECER
       ↓
S5 - REALIZAR REAÇÃO
       ↓
S6 - RESFRIAR
       ↓
S7 - TRANSFERIR PARA DECANTAÇÃO
       ↓
S8 - DECANTAR
       ↓
S9 - SEPARAR GLICERINA
       ↓
S10 - PURIFICAR
       ↓
S11 - TRANSFERIR BIODIESEL
       ↓
S12 - FINALIZAR BATELADA
```

Essa sequência pode ser implementada posteriormente utilizando GRAFCET.

---

# 15. Aplicação da teoria dos grafos

A teoria dos grafos será utilizada para representar a rede de tubulações e os caminhos disponíveis para transferência dos fluidos.

Cada equipamento pode ser representado como um vértice:

```text
Tanque → Bomba → Válvula → Reator
```

As tubulações representam as conexões entre esses equipamentos.

Caso uma tubulação ou equipamento fique indisponível, o sistema pode procurar uma rota alternativa.

Por exemplo:

```text
Tanque ─────X───── Reator
   │
   └──→ Bypass ───→ Reator
```

O algoritmo de **Dijkstra** pode ser utilizado para determinar o caminho de menor custo entre dois pontos.

Dessa forma, o sistema pode realizar um roteamento alternativo sem necessariamente interromper todo o processo.

---

# 16. Diagnóstico através de árvores

As árvores serão utilizadas para organizar as possíveis causas de falhas.

Por exemplo, para um alarme de temperatura elevada:

```text
TEMPERATURA ALTA
       │
       ├── Sensor
       │
       ├── Aquecimento
       │
       ├── Controlador
       │
       └── Resfriamento
```

O SCADA-Core pode analisar essas possibilidades e procurar a causa-raiz.

Isso permite evitar uma grande quantidade de alarmes secundários.

Por exemplo:

```text
Falha na válvula de resfriamento
             ↓
Temperatura elevada
             ↓
Intertravamento acionado
             ↓
Aquecimento desligado
```

Nesse caso, o sistema pode identificar a falha da válvula como a causa principal.

---

# 17. Aplicação da teoria das relações

As relações podem representar as transições entre os estados do processo.

Por exemplo:

```text
S1 → S2
```

pode representar a passagem da verificação de permissivos para o enchimento do reator.

Essa transição somente ocorre quando a condição necessária for verdadeira.

Exemplo:

**S1 → S2 se o nível necessário estiver disponível.**

Outro exemplo:

**S4 → S5 se a temperatura atingir o valor de processo.**

E:

**S5 → S6 quando o tempo de reação for atingido.**

Assim, as relações permitem representar matematicamente o comportamento sequencial da planta.

---

# 18. Integração com o SCADA-Core

O SCADA-Core será responsável por integrar todas essas metodologias.

A arquitetura conceitual é:

```text
             SENSORES
                 ↓
           SCADA-CORE
                 ↓
       ┌─────────┼─────────┐
       ↓         ↓         ↓
   LÓGICA     GRAFOS    DIAGNÓSTICO
   FORMAL                 ÁRVORES
       ↓         ↓         ↓
       └─────────┼─────────┘
                 ↓
       MÁQUINA DE ESTADOS
             / GRAFCET
                 ↓
               CLP
                 ↓
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     BOMBAS    VÁLVULAS  AQUECIMENTO
                 ↓
               PLANTA
```

O sistema, portanto, não será apenas responsável pela supervisão da planta, mas também pela tomada de decisões lógicas relacionadas à segurança, sequência de operação, diagnóstico e roteamento.

---

# 19. Funcionamento completo

O funcionamento geral da planta pode ser resumido da seguinte forma:

```text
INÍCIO
  ↓
Verificar condições de segurança
  ↓
Preparar metóxido
  ↓
Transferir óleo
  ↓
Transferir metóxido
  ↓
Verificar carregamento
  ↓
Iniciar agitação
  ↓
Aquecer
  ↓
Atingir temperatura de processo
  ↓
Realizar transesterificação
  ↓
Controlar temperatura e tempo
  ↓
Resfriar
  ↓
Transferir para decantador
  ↓
Decantar
  ↓
Separar glicerina
  ↓
Purificar biodiesel
  ↓
Transferir para tanque final
  ↓
Finalizar batelada
```

A principal função do SCADA-Core será garantir que essa sequência seja executada de maneira **determinística, segura e supervisionada**, impedindo transições indevidas, identificando falhas e permitindo o diagnóstico das condições anormais.

Dessa forma, o projeto integra conceitos de **Matemática Discreta, Automação Industrial, Controle de Processos e Sistemas Supervisórios** em uma única aplicação.


# Malha de controle do processo
<img width="1536" height="1024" alt="1523cd44-3c4d-4cdc-9a3a-a7728ef68286" src="https://github.com/user-attachments/assets/c2f47d93-0785-449d-a177-3682025b3f6e" />

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

## Setor 200: Reação (Reator CSTR)

| Tag Instrumento | Tipo de Dispositivo | Variável Física | Proposição Lógica | Descrição do Estado 1 (Verdadeiro) |
| :--- | :--- | :--- | :--- | :--- |
| **LT-201** | Transmissor de Nível | Nível do Reator | $l_{reator}$ | Nível atingiu o volume total da batelada |
| **LSH-201** | Chave de Nível Alto | Transbordamento | $l_{alto}$ | **ALARME:** Nível acima do limite de segurança |
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
