# NPIECS (National Planning for Infrastructures and Electric Charging Stations)

(Simon Senegas, Eric Jeannière, 2024)

The sharp increase in the number of electric vehicle (EV) that is foreseen in the near future will imply a huge effort to develop the charging infrastructure on the road network. From about 2000 charging points on the main French road network today, this number should follow an exponential curve and be multiplied by more than 10 in just a few years. However, it is necessary to plan a long time in advance since the energy connection to the grid can take a couple of years (even longer time when a new electrical source substation is needed). 
In order to convince households and transporters to the transition to EV, it is essential to ensure the ability for long distance trips for EV by sizing the right number of high power charging point along the road network. The majority of those long trips made by private vehicle, those who need a high power charge on the way, happen to be in summer time for vacation purpose. That is why we are looking on one hand at the peak period traffic for sizing the number of plugs, and on the other hand at annual average daily traffic (needed for the economic assessment).

In this document, we explain the methodology used to develop the forecasting tool built in-house under Excel. The tool is taking into account the main following parameters and their eventual foreseen changes in the coming years:

- Number of EV and their average characteristics grouped in 3 categories of cars and 5 for trucks: battery size, consumption, autonomy, charging power; sourced from car sales and fleet model data ;
- Number and characteristics of long trips: spatial and temporal distribution, with trip distance classes every 20 kilometres; sourced from national household travel surveys, and road assignment of demand matrices estimated with a transport model ;
- Behaviour of drivers related to their battery charge strategy: state of charge (SoC) at departure, SoC when arriving at charging point, number of stops for long trips, battery margin to reach destination; mainly sourced from Charging Point Operators (CPO) data.

Outputs include average charging time and energy per stop and per trip, total energy needed on the way, charging power needed on the grid, number of charging points needed, and total energy sold. The tool then allows doing the financial analysis and the cost-benefit assessment of the charging infrastructure, and is a help to define the right level of public funds needed to develop the charging infrastructure.

**More details in methodology section**
