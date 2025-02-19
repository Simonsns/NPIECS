# Methodology

This section describes the inputs and outputs of the “microscopic” (or bottom-up) model, used to quantify the electrical power required for each area and axis of the road network. The general methodology is summarized in the flow chart in the figure below. The model is made up of two modules, symbolized by the black and red frames. 
The model predicts the itineraries of individuals and then calculates for each area used the energy consumed and the power required to be supplied. Each module is independent: 

- The module calculating the shortest paths, exporting and filtering them is supported by a modulus of the MARVeLL traffic model, coded in C++.
- The module calculating energy and total power for each area (initially coded in Python, then in C++ to improve calculation times).

This choice of architecture was motivated by the difficulty of developing such programs rapidly in C++. As data analysis and management are much more accessible in Python, the intermediate route simplification module was initially coded in Python, even though the final architecture integrates all the most intensive functions in C++ to save computing time.
