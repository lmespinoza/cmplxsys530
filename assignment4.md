# Model Proposal for _Globalization and Inequality of (Many) Nations_

_Luis Espinoza_

* Course ID: CMPLXSYS 530,
* Course Title: Computer Modeling of Complex Systems
* Term: Winter, 2019

&nbsp; 
### Goal 
*****
The goal of this project is to, first, build an agent-based model (ABM) that replicates Krugman and Venables' 1995 paper "Globalization and the Inequality of Nations", and, second, to extend it to include more than two economies.

&nbsp;  
### Justification
****
Although the equation-based model (EBM) in the original paper can be solved (it has 10 unknowns and 10 equations), the analytical study of the equilibrium is, in the words of the authors, "algebraically complex". For this reason, the original paper uses numerical simulations instead of the traditional closed-form solutions. However, their paper (1) modeled a world with only two countries and (2) conducted simulations using arbitrary parameter values. 
With this in mind, I plan to (1) extend their model to any (_N_) number of countries, (2) to explore the input-output space for some parameters, (3) to run simulations changing not only the trade costs but the populations to account for international migration.

&nbsp; 
### Main Micro-level Processes and Macro-level Dynamics of Interest
****
The model would work at two different levels. The first level is what happens *within* each country. Here, I won't be using an ABM approach, but a system of 5 equations (per country) that summarizes the utility and profit maximization by consumers and firms, respectively, and the equilibrium conditions in the markets. The second level is the *international* setting. This is where I will conduct exogenous changes on the bilateral trade costs among countries.

I am interested in two macro-level dynamics:
* the geographical pattern of industrialization as trade costs decrease/population migrate,
* the utility dynamics of industrializing and de-industrializing countries as trade cost decrease/population migrate,

&nbsp; 
## Model Outline
****
### 1) Environment
In this model, the environment represents the topography of the "world". This will be captured by the variable **trade costs**, a mapping from all relevant dimensions that affect trade among countries (geographical distance, cultural affinity, common language, transportation and communication technology, etc.) into the real line. Moreover, they will be of the *iceberg* type, i.e. they will represent the proportion of the quantity shipped relative to the quantity received (the difference being "lost" on the way). Each pair of countries *i*,*j* will have an associated trade cost (*t<sub>ij</sub>*). Additionally, I will assume that trade costs are *symmetrical* (*t<sub>ij</sub>=t<sub>ji</sub>*), and that when a country buys from itself, there are no trade costs. i.e. *t<sub>ii</sub>=1*. Hence for *N* countries, there will be *N(N-1)/2* trade cost values.

As a first approximation, I am going to abstract from different country sizes, so each country will occupy exactly one cell on the grid. In this setting, there are many ways to define trade costs. One way is to first assign a "cell trade cost" to each cell on the grid, and then define the bilateral trade cost as the sum of the cell-trade costs along the "trade route" (shortest-path) between the two countries.

A simpler way would be to assume that bilateral trade costs are proportional to the Euclidian distance between two countries. This alternative method requires to create a *N* by *N* matrix of distances between pairs of countries. Since the original paper considered "3" to be a high bilateral trade cost value, I initialize the simulations with bilateral trade costs that are at least as high as that number (except for the values on the diagonal, in which case the trade cost is 1).

* _Boundary conditions:_ they will wrap West-East, but not North-South (can't cross the poles).
* _Dimensionality:_ 2D.
* _List of environment-owned variables:_  trade costs, **t<sub>ij</sub>>=1** for all *i,j=1,...,N*
* _List of environment-owned methods/procedures:_ trade costs will decrease exogenously and assymetrically in the map.

```python
# After creating the environment, I will locate the *N* countries on the grid randomly.
# Then, the bilateral trade costs will be initially proportional to their location.
def initialize():
    global env, costs, nextcosts
  
    # Environment: world map
    env = zeros([w, w])
    
    # Environment: initial trade costs        
    costs = np.empty([n,n]) # empty nxn matrix of trade costs
    for i in xrange(n):
        for j in xrange(n):
            if x != y:
                c[i,j] = 3 + np.sqrt((agent[i].x-agent[j].x)**2 + (agent[i].y-agent[j].y)**2)
            else:
                c[i,j] = 1
```

&nbsp; 
### 2) Agents
The agents in this model will be the **countries**, which are fixed in certain cells and cannot move. Each country *i* will be characterized by one exogenous variable and five endogenous variables:
 
* _List of agent-owned variables:_
  * Exogenous: population, *L<sub>i</sub>*
  * Endogenous:
      * Manufacturing price index, *Q<sub>i</sub>*
      * Wages, *w<sub>i</sub>*
      * Total expenditure in manufactured goods, *E<sub>i</sub>*
      * Price of representative manufacturing variety, *p<sub>ii</sub>*
      * Number of manufacturing firms/varieties, *n<sub>i</sub>*
      
* _List of agent-owned methods/procedures:_ Implicitly, each agent hosts a set of workers-consumers and firms, each of which maximizes its utility or profit, respectively, taking prices, wage, and **trade costs** as given. Moreover, these agents interact in free markets, where the equilibrium prices and wage are determined. Instead of modelling these processes, I work with five equations that together characterize the equilibrium in each country:
  * Determination of the manufacturing price index.
  * Equilibrium in the labor market.
  * Determination of total expenditure in manufactured goods.
  * Optimal manufacturing pricing for the representative manufacturing firm.
  * Zero-profit condition.

Unfortunately, I need to run the solver even for initializing the simulation. Since I don't know how to do that yet, I am leaving the initial values of the agent's characteristics blank ("???").

```python
def initialize():
    global agents
    
    # Agents: countries
    agents = [None] * n # Creates a n-vector with empty values
    for i in xrange(n):
        ag = agent()
        ag.x = randint(w) # starts at a random position
        ag.y = randint(w)
        ag.q = ???  # Price index for manufactures (Q)
        ag.newq = ag.q
        ag.w = ??? # Equilibrium wage (w)
        ag.neww = ag.w
        ag.e = ??? # Total expenditure on manufactured goods (E)
        ag.newe = ag.e
        ag.exp = ??? # Manufacturing exports (x)
        ag.newexp = ag.exp
        ag.dom = ??? # Manufacturing domestic sales (y)
        ag.newdom = ag.dom
        ag.p = ??? # Optimal manufacturing representative price (p)
        ag.newp = ag.p
        ag.n = ??? # Number of manufacturing varieties/firms (n)
        ag.newn = ag.n
        ag.l = 27000000 # Populations (L) = Avg country-population in 2000
        agents[i] = ag
```

&nbsp; 
### 3) Action and Interaction 
**_Interaction Topology_**
All countries can potentially interact with any other country in the world. Hence, the interaction topology is the whole world.
 
**_Action Sequence_**
1. Step 1: The set of bilateral trade costs and/or the set of country populations are exogenously updated.
2. Step 2: Each country *simultaneously* updates its five endogenous variables using the five equilibrium conditions. Since countries' endogeous variables are interrelated, I will use a nonlinear solver at each period.

&nbsp; 
### 4) Model Parameters and Initialization
The mathematical model has four parameters:

1. sigma = the elasticity of substitution in *consumption* among the different varieties of manufactured goods.
2. gamma = the elasticity of substitution in *consumption* between agricultural goods and manufactured goods.
3. mu = the elasticity of substitution in *production* between labor and manufactured intermediate goods.
4. beta = the marginal cost of an additional unit of production

Additionally, there are two more global parameters. First, I would like to let the user to decide how many countries to work with (to a total of 100), so the Python program also uses the numbers of countries *N* as a global parameter. Second, I will se *w* as the parameter of the world size.

```python
###############################################################################
########################## PARAMETERS #########################################
###############################################################################
# The equatorial circumference of Earth is 24,901 miles (40,075 km)
# The meridional circumference of Earth is 24,860 miles (40,008 km)
# Width/Hight = 24901/24860 ~ 1
n = 3 # number of agents
w = 100 # number of rows/columns in spatial array
maxsteps = 15000 # When the program ends
seed(10)

# From Krugman and Venables (1995):
sigma = 5
gamma = 0.6
mu = 0.5
beta = 1 # original paper doesn't specify

class agent:
    pass
```

Taking the previous sections together, the model will be initialized as follows:
```python
###############################################################################
########################## SETTING UP INITIAL VALUES ##########################
###############################################################################
def initialize(n):
    global agents, env, costs, nextcosts, time
    
    # Time ticker
    time = 0

   # Environment: world map
    env = zeros([w, w])
   
    # Agents: countries
    agents = [None] * n # Creates a n-vector with empty values
    for i in xrange(n):
        ag = agent()
        ag.x = randint(w) # starts at a random position
        ag.y = randint(w)
        ag.q = ???  # Price index for manufactures (Q)
        ag.newq = ag.q
        ag.w = ??? # Equilibrium wage (w)
        ag.neww = ag.w
        ag.e = ??? # Total expenditure on manufactured goods (E)
        ag.newe = ag.e
        ag.exp = ??? # Manufacturing exports (x)
        ag.newexp = ag.exp
        ag.dom = ??? # Manufacturing domestic sales (y)
        ag.newdom = ag.dom
        ag.p = ??? # Optimal manufacturing representative price (p)
        ag.newp = ag.p
        ag.n = ??? # Number of manufacturing varieties/firms (n)
        ag.newn = ag.n
        ag.l = 27000000 # Populations (L) = Avg country-population in 2000
        agents[i] = ag

    # Environment: initial trade costs        
    costs = np.empty([n,n]) # empty nxn matrix of trade costs
    for i in xrange(n):
        for j in xrange(n):
            if x != y:
                c[i,j] = 3 + np.sqrt((agent[i].x-agent[j].x)**2 + (agent[i].y-agent[j].y)**2)
            else:
                c[i,j] = 1
```

Each "tick" in the model will consist of the following steps:
1. (Exogenous) new matrix of bilateral trade costs.
2. Use the Python solver to find the new (static) equilibrium.
3. Save the utility of each country and their productive structure.

&nbsp; 
### 5) Assessment and Outcome Measures
There are two main features to keep track of:
1. The global distribution of manufacturing across countries in the world.
2. The global distribution of utility (payoff of the representative consumer-worker) across countries in the world.

&nbsp; 
### 6) Parameter Sweep
I am interested in playing with the four model parameters in greek letters.
