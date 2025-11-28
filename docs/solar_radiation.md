# City Solar Radiation Modeling

CityBEM V.2 incorporates a highly efficient **3D ray-tracing solar radiation model** to compute direct beam shading in dense urban environments.

## 1. Overview
The model:
- Reads STL-based geometry  
- Breaks each building surface into triangles  
- Computes sun vectors hourly  
- Performs ray–triangle intersection  
- Determines shaded vs. unshaded fractions  

This approach is adapted from *Rayegan et al. (2026), Renewable Energy 256*, which demonstrates high accuracy compared to EnergyPlus and full ray-tracing engines.

## 2. Ray-Tracing Methodology
For each hour:
1. Compute sun position  
2. Cast ray from centroid of each triangle  
3. Check intersection with all other triangles  
4. If intersection distance < sun-ray distance → surface is shaded  
5. Compute shaded fraction and irradiance reduction  

## 3. Computational Benefits
- Handles **50,000+ surfaces**  
- Vectorized operations  
- Automatic grouping of triangles  
- Fast enough for city-wide PV simulations  

## 4. Outputs
- Hourly shading factor per surface  
- Shaded/unshaded irradiance components  
- Surface view factors (optional)

See **Rooftop PV Model** for coupling with PV arrays.