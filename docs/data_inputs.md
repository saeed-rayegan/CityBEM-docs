
---

# ✅ **3. `data_inputs.md`**

```markdown
# Input Data

CityBEM V.2 uses lightweight **Citywide Building Data Arrays** to efficiently represent thousands of buildings across a city.

## 1. Geometry Data
- Building height  
- Footprint area  
- Surface orientation  
- Roof tilt & azimuth  
- STL-based triangular meshes (optional for 3D shading)

## 2. Thermal Properties
- Wall/roof U-values  
- Window-to-wall ratios (WWR)  
- Thermal mass  
- Infiltration & ventilation rates  

## 3. Building Usage Parameters
Derived from archetypes:
- Occupancy density  
- Internal gains (sensible/latent)  
- Equipment and lighting loads  
- HVAC schedules  
- Setpoints  

## 4. Weather Data
CityBEM uses:
- Dry bulb temperature  
- Direct normal irradiance (DNI)  
- Diffuse horizontal irradiance (DHI)  
- Wind speed and direction  

Weather files can be:
- Typical Meteorological Year (TMY)
- EnergyPlus EPW
- CityFFD microclimate-corrected fields

## 5. Microclimate (optional)
From CityFFD:
- Local air temperature  
- Turbulence intensity  
- Wind speed at façade height  
- Heat transfer coefficient adjustments  

## 6. PV System Inputs (optional)
- Panel layout geometry  
- Module temperature model coefficients  
- PV module type (mono/poly/crystalline)  
- Tilt, azimuth, spacing  