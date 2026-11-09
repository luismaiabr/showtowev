Problema: estou fazendo uma disciplina de Projeto e Análise de Algoritmos na faculdade. Sugira problemas de cobertura, no domínio de quantitative finance (quant), que sejam resolvíveis com o algoritmo Primal-Dual. Eu quero fazer uma apresentação sobre isso, resolvendo um problema real, e ilustrando com recursos visuais que eu irei programar com Angular. 

A aplicação do método **Primal-Dual** (especialmente como algoritmo de aproximação para problemas NP-difíceis baseados em relaxação de Programação Linear) encaixa-se muito bem em finanças quantitativas quando modelamos **custos discretos**, **seleção de ativos** ou **infraestrutura de execução**.

Como você vai construir uma interface visual interativa em **Angular**, o segredo para uma apresentação de alto impacto em *Projeto e Análise de Algoritmos (PAA)* é explorar a mecânica visual do método: **variáveis duais crescendo continuamente ("enchendo como baldes") até que uma restrição dual fique justa (*tight*), congelando variáveis e ativando a escolha primal.**

---

## 1. Cobertura de Fatores de Risco com Custo Mínimo (*Weighted Set Cover*)

**O Problema Quant:**
Imagine uma mesa proprietária ou um fundo quantitativo exposto a um conjunto de choques de mercado ou fatores de risco discretos (ex: alta nos juros DI, pico de volatilidade implícita no dólar, choque em commodities, queda no índice B3). Para proteger a carteira (*hedging*), o algoritmo deve selecionar um subconjunto de derivativos (opções de diferentes *strikes*, contratos futuros WIN/WDO, ETFs). Cada instrumento cobre um subconjunto específico de cenários de risco e possui um custo fixo de montagem/spread ($c_j$). O objetivo é **cobrir todos os cenários de estresse com o menor custo total de prêmio/execução**.

* **Modelagem Primal (Minimização):**

$$\min \sum_{j \in S} c_j x_j \quad \text{sujeito a} \quad \sum_{j: e \in S_j} x_j \ge 1, \quad \forall e \in U, \quad x_j \in \{0,1\}$$

Onde $U$ é o universo de cenários de risco e $S_j$ são os riscos cobertos pelo derivativo $j$.

> **Nota explicativa sobre a restrição ($\sum_{j: e \in S_j} x_j \ge 1$):** Para cada risco $e$ pertencente ao universo $U$, deve existir pelo menos um contrato $S_j$ ativo ($x_j = 1$) que o contemple. Em outras palavras, ao iterar sobre cada risco $e$, a soma das variáveis de decisão $x_j$ de todos os contratos que cobrem esse risco específico deve ser maior ou igual a $1$, assegurando que toda a carteira de riscos esteja devidamente protegida.
* **Modelagem Dual (Maximização — "Empacotamento de Preços de Sombra"):**

$$\max \sum_{e \in U} y_e \quad \text{sujeito a} \quad \sum_{e \in S_j} y_e \le c_j, \quad \forall j \in S, \quad y_e \ge 0$$



Aqui, $y_e$ representa o "preço sombra" (*shadow price*) ou o quanto estamos dispostos a pagar individualmente para neutralizar o risco $e$.
* **Fator de Aproximação:** Garantia teórica de $O(f)$, onde $f$ é o número máximo de instrumentos que cobrem um mesmo risco.

> **Adendo: o que significa Primal e Dual na prática?**
>
> O Primal e o Dual são **duas faces da mesma moeda**: duas maneiras diferentes de enxergar e resolver exatamente o mesmo problema econômico ou computacional.
>
> | Perspectiva | Primal (O Problema Original) | Dual (O Problema Espelho) |
> | --- | --- | --- |
> | **Ponto de vista** | Físico / Decisório (alocação de recursos) | Econômico / Valoração (preços e orçamentos) |
> | **Pergunta central** | "Quais derivativos comprar para cobrir todos os riscos gastando o mínimo possível?" | "Quanto vale a neutralização de cada risco para maximizar o orçamento sem ultrapassar o custo dos derivativos?" |
> | **Ação** | **Minimizar** custos ($\min \sum c_j x_j$) | **Maximizar** valor/preços-sombra ($\max \sum y_e$) |
> | **Variáveis** | $x_j \in \{0,1\}$ (comprar ou não o derivativo $j$) | $y_e \ge 0$ (orçamento ou preço-sombra do risco $e$) |
> | **Restrições** | Cada risco precisa de cobertura: $\sum_{j: e \in S_j} x_j \ge 1$ | Ninguém paga mais que o preço de mercado: $\sum_{e \in S_j} y_e \le c_j$ |
>
> **Por que isso importa para o algoritmo:** pela dualidade fraca, qualquer solução dual viável é uma **cota inferior** para o ótimo ($\sum y_e \le OPT$). Assim, o algoritmo Primal-Dual constrói simultaneamente uma solução primal (a carteira de derivativos) e uma dual (os preços-sombra), e usa a dual para certificar a qualidade da primal.

**Visualização em Angular:**

* **Painel Esquerdo (Grafo Bipartido):** Nós à esquerda são os Cenários de Risco ($e$); nós à direita são os Contratos/Opções ($S_j$) com uma barra de capacidade equivalente ao seu custo $c_j$.
* **Animação do Algoritmo:** Um *slider* de "Passo a Passo" (ou *Play/Pause*). Ao avançar, os nós de risco descobertos aumentam seu valor $y_e$ simultaneamente. As barras dos derivativos vão enchendo conforme a soma dos $y_e$ conectados a eles cresce.
* **Gatilho Primal-Dual:** Assim que a barra de um derivativo atinge 100% do seu custo ($\sum y_e = c_j$), ela pisca em verde (restrição *tight*), o derivativo é comprado ($x_j = 1$), e todos os riscos que ele cobre são "congelados" e marcados como protegidos.

---

## 2. Seleção de Liquidez e Cobertura de Arbitragem (*Vertex Cover* em Grafos de Correlação)

**O Problema Quant:**
Em estratégias de *Statistical Arbitrage* (Pairs Trading) ou monitoramento de *Order Books* correlacionados, modelamos o mercado como um grafo $G = (V, E)$. Os vértices $V$ são os ativos (com custo $w_v$ representando taxas de subscrição de *market data* L3, margem exigida ou *slippage* médio) e as arestas $E$ conectam pares com cointegração estatisticamente significativa ou dependência de liquidez. Para monitorar ou travar todas as relações de arbitragem do grafo, precisamos selecionar um subconjunto de ativos-chave tal que **toda relação de cointegração tenha pelo menos um de seus ativos monitorado/posicionado na carteira base**.

* **Modelagem Primal:**

$$\min \sum_{v \in V} w_v x_v \quad \text{sujeito a} \quad x_u + x_v \ge 1, \quad \forall (u,v) \in E$$


* **Modelagem Dual:**

$$\max \sum_{e \in E} y_e \quad \text{sujeito a} \quad \sum_{e \in \delta(v)} y_e \le w_v, \quad \forall v \in V$$



Onde $y_e$ é o "orçamento de risco/arbitragem" alocado na aresta $e = (u,v)$.
* **Fator de Aproximação:** Algoritmo **2-aproximado** exato e fácil de provar no quadro durante a apresentação ($\text{Custo Primal} \le 2 \times \text{Custo Dual} \le 2 \times \text{OPT}$).

**Visualização em Angular:**

* **Componente Visual:** Um grafo de força (*force-directed graph* usando D3.js ou Cytoscape.js integrado ao Angular) onde os nós são tickers (ex: PETR4, VALE3, ITUB4, WIN) e as arestas são os pares cointegrados.
* **Interatividade:** O usuário pode definir um *threshold* de correlação (ex: $\rho > 0.80$) que cria ou remove arestas dinamicamente.
* **Execução Visual:** Durante a execução do Primal-Dual, as arestas descobertas ficam mais espessas (aumento de $y_e$) transferindo "pressão" para os dois vértices das pontas. Quando a soma das arestas incidentes satura o peso $w_v$ de um ativo, o nó muda de cor, entra no portfólio e desativa as arestas adjacentes.

---

## 3. Infraestrutura de Execução e *Smart Order Routing* (*Facility Location* / *Hitting Set*)

**O Problema Quant:**
Uma infraestrutura de *High-Frequency Trading* (HFT) ou um sistema distribuído de captura de dados (*tick-by-tick*) precisa decidir em quais *data centers* / *matching engines* (ex: B3 Puma, CME, Nasdaq, servidores Cloud específicos) vai alugar *co-location* ou manter instâncias dedicadas (custo fixo de abertura $f_i$), além de considerar o custo de latência/roteamento $c_{ij}$ para executar ordens de diferentes estratégias ou clientes $j$.

* **Por que brilha em PAA:** O algoritmo Primal-Dual de **Jain e Vazirani** para o *Uncapacitated Facility Location Problem (UFLP)* é um dos algoritmos mais elegantes da literatura de aproximação (garantia de **1.861-aproximação** ou **3-aproximação** na versão clássica).
* **Dinâmica Dual:** Cada estratégia/ordem $j$ aumenta sua disposição a pagar ($\alpha_j$). Quando $\alpha_j$ supera a latência/custo de conexão $c_{ij}$ até o servidor $i$, a estratégia começa a contribuir com $\beta_{ij}$ para pagar o custo fixo de instalação $f_i$ daquele servidor. Há ainda uma **fase de limpeza (*pruning*)** no final para garantir que duas estratégias conectadas não paguem pelo mesmo servidor de forma redundante.

**Visualização em Angular:**

* **Dashboard de Rede:** Mostra os nós de estratégias de um lado e os servidores/*brokers* de execução do outro.
* **Duas Fases Visuais Claras:**
1. **Fase de Crescimento Dual:** Ondas ou raios de alcance crescendo a partir das ordens ($\alpha_j$) até tocarem os servidores e começarem a financiar a abertura deles.
2. **Fase de Poda Primal (*Reverse Delete*):** O algoritmo abre servidores provisoriamente e depois desliga os redundantes, redirecionando o fluxo de ordens — visualmente excelente para explicar por que a etapa de limpeza é necessária para manter a folga complementar relaxada (*relaxed complementary slackness*).



---

## Comparativo para a sua Escolha

| Critério | 1. Hedge de Riscos (*Set Cover*) | 2. Grafo de Arbitragem (*Vertex Cover*) | 3. Roteamento HFT (*Facility Location*) |
| --- | --- | --- | --- |
| **Dificuldade Matemática (Prova)** | Média ($f$-aproximação) | Fácil (**2-aproximação** limpa) | Alta (Duas variáveis duais + Poda) |
| **Apelo Financeiro (Quant)** | Muito Alto (Gestão de Risco / Opções) | Alto (StatArb / Cointegração) | Médio-Alto (Microestrutura / Infra) |
| **Impacto Visual no Angular** | Barras de capacidade + Matriz/Bipartido | Grafo de rede interativo muito fluido | Animação em 2 fases (Expansão + Poda) |

---

## Dica de Ouro para a Arquitetura no Angular

Para deixar a apresentação didática para a turma e para o professor de PAA, estruture a tela do seu app Angular em **3 colunas sincronizadas**:

1. **Coluna 1 (Mercado / Grafo Primal):** O grafo interativo ou matriz de cobertura mostrando quais ativos/contratos foram selecionados ($x_j \in \{0, 1\}$).
2. **Coluna 2 (Termômetro Dual):** Barras de progresso representando as restrições duais ($\sum y_e \le c_j$). Isso materializa o conceito abstrato de "folga complementar" (*Complementary Slackness*): o público **vê** que uma variável primal $x_j$ só vira $1$ quando a barra da restrição dual correspondente bate em $100\%$.
3. **Coluna 3 (Cota de Aproximação em Tempo Real):** Um gráfico ou placar comparando:
* **Soma Dual Atual ($\sum y_e$):** Que serve como *Lower Bound* (cota inferior para o ótimo teórico $OPT$).
* **Custo Primal Acumulado ($\sum c_j x_j$):** O custo real da carteira montada.
* **Razão ($\text{Primal} / \text{Dual}$):** Mostrando ao vivo que essa razão nunca ultrapassa o fator teórico de aproximação (ex: nunca passa de $2.0$ no *Vertex Cover*). Isso conecta a implementação diretamente com os teoremas de análise de algoritmos.