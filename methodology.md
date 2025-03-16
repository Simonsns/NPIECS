# Methodology

The model is divided into two parts: 
- a macroscopic part to control the overall need for recharging on the national road network. This part will not be developed here.
- Another part, called the “microscopic model”, is designed to localize the charging requirement along the national road network.

This section describes the inputs and outputs of the “microscopic” (or bottom-up) model, in order to quantify the electrical power required for each area and axis of the NHS. The general methodology is summarized in the flow chart in the figure below. The model is made up of two modules, symbolized by the black and red frames. 
The model predicts the itineraries of individuals and then calculates for each area used the energy consumed and the power required to be supplied. Each module is independent: 

- The module calculating the shortest paths, exporting and filtering them is supported by a modulus of the MARVeLL traffic model, coded in C++.
- The module calculating energy and total power for each area (initially coded in Python, then in C++ to improve calculation times).

This choice of architecture was motivated by the difficulty of rapidly developing such programs in C++. As data analysis and management are much more accessible in Python, the intermediate route simplification module was first coded in Python, even though the final architecture integrates all the most intensive functions in C++ to save computing time.

![](npiecs_illustrations/flowchart_global.png "Model illustration (flowchart in french)").

### Route management

Modeling the shortest routes in the sense of generalized cost, i.e. taking into account the cost of the time spent on the journey as well as the true monetary cost: 
$$C_g = P + V_tT$$

Where 
- $C_g$ : generalized cost of the journey
- $P$ : the monetary price of the journey.
- $V_t$: the value of time. This value corresponds to the value given to each minute spent in transport. It is evaluated at around $10.7€_{2015}$ by the socio-economic reference framework for transport.

The routes taken into account are those passing through the various rest areas, distinguishing between rest areas and service areas. The shortest 
shortest paths for each Origin-Destination (OD) using a hierarchical contraction algorithm.
The data used to build, calibrate and validate the model comes from the Enquête Nationale Transports et Déplacements (ENTD), 
the Transport de Marchandises (TRM) survey and annual average daily traffic (TMJA) count data. Finally, only origin-destinations passing through at least one area of the 
our perimeter, so as not to increase the calculation time of the second module.

### Hierarchical contraction algorithm

The shortest-path algorithm used to achieve significant improvements in computation time is the hierarchical contraction algorithm.

We can define a directed graph $G = (V,E,\omega)$, with $V$ the set of ordered vertices, $E$ the set of arcs, and $\omega$ the weight function associated with the arcs.


#### Vertex contraction

The contraction of any vertex consists in removing it from the graph while preserving the minimum distance between the other vertices. To do this, consider all pairs $(u,w)$ such that :

$$ (u, v) \in E ; (v, w) \in E $$ 

If $d_G(u, w) > \omega(u, v) + \omega(v, w)$, then we add a new arc $(u, w)$ with :

$$\omega(u, w) = \omega(u, v) + \omega(v, w)$$

We now have all the minimum distances preserved for the shortest path calculation, with a new number of vertices $V'$. By successively contracting all the vertices in the order defined initially, a new simplified graph $G'$ appears, with only the minimum distances.

This hierarchical structure, in which each arc respects the order, speeds up shortest-path queries. During the query phase, a bidirectional search (or directed search) traverses only the “ascending” arcs, considerably reducing the number of nodes explored.

#### Path simplification

A path is made up of a displacement origin, the series of identifiers of the arcs taken, and the associated destination. 

![](npiecs_illustrations/trajet.png "Travel structure (french)").


Distances in km are attached to the identifiers of the arcs making up the paths.  The next step is to condense this path in order to simply calculate the cumulative distance between each area. 

![](npiecs_illustrations/segmentation.png "Travel segmentation (french)").


In the modeled network, the areas are more or less in the middle of the arcs. For simplicity's sake, we'll assume that the areas are positioned in the middle of the section on which they are located, i.e. we'll add **half the length of the section** to the distance before and after the area (the small error made is cancelled out by the symmetry of journeys in both directions on average over the year). 

### Choix des aires d'arrêts pour recharge

For each class of vehicle, it is assumed that drivers set off with 100% battery power, and that they drive as far as possible before stopping to recharge if their effective range does not allow them to reach their destination. This somewhat frustrating method potentially involves stopping only a few km before the destination, but this is partially offset by the symmetrization of the OD matrix into annual averages.  In addition, the multiplicity of ODs (around 3,000 zones in mainland France) should offset the risk of over-concentrating recharging in certain areas. In any case, we consider that the results of the model at the scale of a single area are not at all reliable, as many other parameters come into play (the price of recharging, the services offered, etc.). The results of the microscopic model only make sense when aggregated at the level of large sections (between major road junctions).

To go into more detail, the calculation algorithm can be simplified as follows. 

![](npiecs_illustrations/flowchart.png "Travel segmentation (french)").

The algorithm runs through all the ODs one by one for each vehicle class, as follows:
- The user starts from origin O with his battery fully charged.
- If the user can reach his destination D without recharging, he will not recharge during his journey.
- If the user can't reach his destination, he travels along the OD and stops when he absolutely must recharge (i.e. before his battery drops below 12%). It then continues on its way until it reaches its destination or another recharging area.
- The list of stopping points is saved.

If only one stop is required, the energy recharged is calculated to reach the set % battery at destination parameter (around 30%). The case with several recharging stops requires additional calculations to determine determine how the loads are distributed.

### Individual behavior towards multi-charging journeys 

Each individual will behave differently when it comes to charging his or her vehicle during a journey, particularly when the journey requires multiple recharges. Although reality suggests a normal distribution of behaviors, we simply calculate the average of the two extreme behaviors: minimum VS maximum charging strategy. In both cases, the list of stopping points is the same (see description of the method in the previous section), and the total energy recharged is the same: it's simply a question of taking into account different possibilities for distributing the energy recharged between the various stops.
In the case of minimum recharging, the user recharges exactly what is needed to reach the next recharging point calculated at the start of his journey (i.e. the one closest to his destination). This behavior optimizes charging times, but entails a higher risk of running out of fuel (even if the SoC min battery threshold is still exceeded). In the case of maximum charging, it simply charges up to 100% (i.e. with greater risk aversion and a longer charging time). In the end, for each stop area, we take the average charge between these two extreme cases. 
In all cases, at the last charging stop, recharging is calculated to reach the fixed % battery at destination parameter (around 30%).

- At each charging stop, the value of the energy charged and the associated charging time are recorded.
- The total energy charged at an area is the weighted sum of all ODs travelled (if the user has charged at the area studied), distinguishing between vehicle classes.


