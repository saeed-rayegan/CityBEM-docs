# Microclimate Co-Simulation

CityBEM V.2 can be coupled with microclimate, detailed CFD or surrogate, solvers to improve prediction of wind-dependent and temperature-dependent heat transfer.

## 1. Why Microclimate?
Urban conditions differ significantly from airport weather stations.  
CityFFD provides:
- Local air temperature  
- Wind speed at façade height  
- Turbulence intensity  
- Convective heat transfer coefficients  

## 2. Workflow
1. Run CityFFD for the target neighborhood  
2. Export hourly microclimate data (CSV)  
3. Map fields to each building surface  
4. Adjust:
   - Convective coefficients  
   - Surface heat flux  
   - PV module cooling  
5. Run CityBEM with corrected boundary conditions  

## 3. Benefits
- More accurate heat-loss estimation  
- Better PV temperature predictions  
- Enhanced UHI analysis capability  

## 4. Outputs
- Microclimate-enhanced building loads  
- Temperature deviations from baseline  