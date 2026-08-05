# Definição da Planta
---
​## Planta de Produção de Biodiesel
​Esta é uma escolha clássica e muito didática. Envolve misturar óleo vegetal com um álcool (metanol) e um catalisador dentro de um reator.
​Lógica Formal (Intertravamentos): O metanol é altamente inflamável. Você pode provar por tautologia que a válvula de aquecimento do reator jamais abrirá se a válvula de resfriamento de emergência estiver com defeito.
​Grafos (Otimização): Se a bomba principal de transferência de óleo quebrar, o algoritmo de Dijkstra pode acionar automaticamente um caminho alternativo de tubulações (bypass) para não parar a produção.
​Relações (Grafcet): O processo é em batelada (lotes). É perfeito para modelar em estados sequenciais: Encher -> Aquecer -> Reagir -> Decantar -> Esvaziar.
