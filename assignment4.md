# Model Proposal for _Globalization and Inequality of (Many) Nations_

_Luis Espinoza_

* Course ID: CMPLXSYS 530,
* Course Title: Computer Modeling of Complex Systems
* Term: Winter, 2019



&nbsp; 
### Goal 
*****
The goal of this project is to, first, build an agent-based model (ABM) that replicates Krugman and Venables' 1995 paper
"Globalization and the Inequality of Nations", and, second, to extend it to include more than two economies.

&nbsp;  
### Justification
****
Although the equation-based model (EBM) in the original paper can be solved (it has 10 unknowns and 10 equations), 
the analytical study of the equilibrium is, in the words of the authors, "algebraically complex".
For this reason, the original paper uses numerical simulations instead of the traditional closed-form solutions.
However, their paper (1) modeled a world with only two countries and (2) conducted simulations using arbitrary parameter values.

With this in mind, I plan to (1) extend their model to any (_N_) number of countries, (2) to explore the input-output space for 
some parameters, (3) to run simulations changing not only the trade costs but the populations to account for international migration.

&nbsp; 
### Main Micro-level Processes and Macro-level Dynamics of Interest
****
The model would work at two different levels. The first level is what happens *within* each country.
Here, I won't be using an ABM approach, but a system of 5 equations (per country) that summarizes the utility and profit maximization 
by consumers and firms, respectively, and the equilibrium conditions in the markets.
The second level is the *international* setting. This is where I will conduct exogenous changes on the bilateral trade costs 
among countries.

I am interested in two macro-level dynamics:
* the geographical pattern of industrialization as trade costs decrease/population migrate,
* the utility dynamics of industrializing and de-industrializing countries as trade cost decrease/population migrate,

&nbsp; 
## Model Outline
****
### 1) Environment
In this model, the environment represents the topography of the "world".
This will be captured by the variable **trade costs**, a mapping from all relevant dimensions that affect trade among countries 
(geographical distance, cultural affinity, common language, transportation and communication technology, etc.) into the real line.
Moreover, they will be of the *iceberg* type, i.e. they will represent the proportion of the quantity shipped relative to the quantity
received (the difference being "lost" on the way).
Each pair of countries *i*,*j* will have an associated trade cost (*t<sub>ij</sub>*).
Additionally, I will assume that trade costs are *symmetrical* (*t<sub>ij</sub>=t<sub>ji</sub>*), and that when a country buys from 
itself, there are no trade costs. i.e. *t<sub>ii</sub>=1*. Hence for *N* countries, there will be *N(N-1)/2* trade cost values.

As a first approximation, I am going to abstract from different country sizes, so each country will occupy exactly one cell on the grid.
In this setting, the total trade costs between any two countries will be equal to the sum of the trade costs of all the cells in the
trade route, which will be the shortest-path between them.

* _Boundary conditions:_ they will wrap West-East, but not North-South (can't cross the poles).
* _Dimensionality:_ 2D.
* _List of environment-owned variables:_  trade costs.
* _List of environment-owned methods/procedures:_ trade costs will decrease exogenously and assymetrically in the map.

```python
# Include first pass of the code you are thinking of using to construct your environment
# This may be a set of "patches-own" variables and a command in the "setup" procedure, a list, an array, or Class constructor
# Feel free to include any patch methods/procedures you have. Filling in with pseudocode is ok! 
# NOTE: If using Netlogo, remove "python" from the markdown at the top of this section to get a generic code block
```

&nbsp; 
### 2) Agents
The agents in this model will be the **countries**, which are fixed in certain cells and cannot move.
 Each country *i* will be characterized by one exogenous variable and five endogenous variables:
 
* _List of agent-owned variables:_
  * Exogenous: population, *L<sub>i</sub>*
  * Endogenous:
      * Manufacturing price index, *Q<sub>i</sub>*
      * Wages, *w<sub>i</sub>*
      * Total expenditure in manufactured goods, *E<sub>i</sub>*
      * Price of representative manufacturing variety, *p<sub>ii</sub>*
      * Number of manufacturing firms/varieties, *n<sub>i</sub>
      
* _List of agent-owned methods/procedures:_ Implicitly, each agent hosts a set of workers-consumers and firms, each of which maximizes
its utility or profit, respectively, taking prices, wage, and **trade costs** as given. Moreover, these agents interact in free markets,
where the equilibrium prices and wage are determined. Instead of modelling these processes, I work with five equations that together
characterize the equilibrium in each country:
  * Determination of the manufacturing price index.
  * Equilibrium in the labor market.
  * Determination of total expenditure in manufactured goods.
  * Optimal manufacturing pricing for the representative manufacturing firm.
  * Zero-profit condition.

```python
# Include first pass of the code you are thinking of using to construct your agents
# This may be a set of "turtle-own" variables and a command in the "setup" procedure, a list, an array, or Class constructor
# Feel free to include any agent methods/procedures you have so far. Filling in with pseudocode is ok! 
# NOTE: If using Netlogo, remove "python" from the markdown at the top of this section to get a generic code block
```

&nbsp; 
### 3) Action and Interaction 
**_Interaction Topology_**
All countries can potentially interact with any other country in the world. Hence, the interaction topology is the whole world.
However, some countries will not trade with others due to large trade costs between them.
 
**_Action Sequence_**
1. Step 1: The set of bilateral trade costs and/or the set of country populations are exogenously updated.
2. Step 2: Each country *simultaneously* updates its five endogenous variables using the five equilibrium conditions.
This process is iterated until the world system reaches a new world equilibrium.

&nbsp; 
### 4) Model Parameters and Initialization
_Describe and list any global parameters you will be applying in your model._

_Describe how your model will be initialized_

_Provide a high level, step-by-step description of your schedule during each "tick" of the model_

&nbsp; 
### 5) Assessment and Outcome Measures
_What quantitative metrics and/or qualitative features will you use to assess your model outcomes?_

&nbsp; 
### 6) Parameter Sweep

_What parameters are you most interested in sweeping through? What value ranges do you expect to look at for your analysis?_