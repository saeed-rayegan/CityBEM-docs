# Rooftop PV Model

CityBEM V.2 integrates a transient rooftop photovoltaic (PV) model suitable for large-scale urban analysis.

This module is based on the methodology introduced in *Rayegan et al. (2024), Energy & Buildings 324*, which evaluates building-level energy self-sufficiency across a city.

## 1. Key Features
- Hourly PV power simulation  
- Cell temperature model (NOCT-based)  
- Integration with shading engine  
- Support for tilted and horizontal modules  
- Net-metering and grid interaction modeling  

## 2. PV Power Model
The output power is given by:

$$P = A \cdot G_{\text{eff}} \cdot \eta(T)$$

where:
- $A$: Active panel area
- $G_{\text{eff}}$: Effective irradiance after shading
- $\eta(T)$: Temperature-corrected efficiency

The temperature correction is:

$$\eta(T) = \eta_{\text{STC}} \left[1 - \gamma (T_c - 25)\right]$$

The effective irradiance after shading is:

$$G_{\text{eff}} = G_{\text{beam}} (1 - S) + G_{\text{diffuse}}$$

where $S$ is the shading factor.

## 4. PV Array Layout
CityBEM supports:
- Uniform tilted arrays (5–15° for flat roofs)  
- Module spacing based on azimuth and tilt  
- Custom layouts from user geometry  

## 5. Outputs
- Hourly PV production  
- Annual energy yield  
- Building-level self-sufficiency index  