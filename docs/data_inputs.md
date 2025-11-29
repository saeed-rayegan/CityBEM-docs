# CityBEM UBEM Workflow

---

**CityBEM V2** transforms multiple data sources into unified **citywide building data arrays** through a structured and scalable workflow.  
The process is organized into the following core components:

---

<div align="center">
  <img src="/assets/citybem_workflow.png" alt="CityBEM Workflow Diagram" width="100%">
</div>

---

!!! info "1. Geospatial Matching (QGIS)"
    🗺️ Integrates GIS building footprints with attributes such as height, land-use, construction year, and parcel data.

---

!!! info "2. Archetype Assignment"
    🏢 Assigns thermal, material, occupancy, and HVAC characteristics to each building through archetype mapping.

---

!!! info "3. 3D Geometry Processing"
    📐 Computes surface geometry (area, tilt, azimuth, façade grouping, roof type) required for thermal and solar modeling.

---

!!! info "4. Weather Data Preparation"
    🌦️ Converts hourly meteorological data to the simulation time step and prepares dynamic boundary conditions.

---

!!! info "5. Citywide Building Data Arrays"
    🧩 Builds optimized internal arrays for storing building attributes and driving the transient heat/mass balance solver.

---

Together, these components form a cohesive workflow enabling CityBEM V2 to perform **large-scale, high-resolution UBEM simulations** with both computational efficiency and physical fidelity.

---

# 📥 CityBEM V2 — Input Data Specification

CityBEM V2 operates using a structured collection of input files that define the full configuration of a city-scale building energy simulation.  
These files specify the **building inventory**, **archetype definitions**, **material and thermal properties**, **meteorological conditions**, and **simulation control parameters** required by the framework.

All input files must follow a consistent formatting standard to ensure reliable parsing and reproducibility:

- Mostly text format (`.txt`)
- Tab-separated values (recommended for clarity and compatibility)
- UTF-8 encoding
- A single header row followed by data records

This page provides a detailed description of each required input file, including its purpose, data structure, parameter definitions, and the consistency constraints enforced throughout the CityBEM workflow.

---

# 1. Building Stock Data

## 1.1 `Input_City_scale_building_info.txt`

<div align="center">
  <img src="../assets/input_building_info.png" alt="CityBEM Building Info Example" width="80%">
</div>

### Purpose

This file defines the **core metadata** for every building in the simulation domain. It acts as the **central index** that links:
- GIS/OSM geometry
- Archetype assignments (by year and usage type)
- Geospatial location for solar and weather processing

Each row corresponds to **one building**.

### Typical Columns and Their Roles

| Column | Description |
|--------|-------------|
| `building_id` | Unique internal ID used by CityBEM (integer or string). This ID must be consistent across all input and output files referencing buildings. |
| `building_osm_id` | External GIS or OpenStreetMap identifier used for traceability and GIS post-processing. |
| `year_of_construction` | Construction year or representative vintage. Used to map the building to a **construction-year archetype** (thermal envelope properties, infiltration, etc.). |
| `usage_type_code` | Code that links the building to a **usage-type archetype** (occupancy, internal loads, schedules, HVAC). Must match IDs defined in `Input_City_scale_archetype_usage_type.txt`. |
| `longitude`, `latitude` | Geographic coordinates (WGS84). Used for solar position and potential external GIS operations. |

Depending on the case, additional columns can be included (e.g., number of floors, floor area), but the **minimum set** must at least allow:
- Unambiguous building identification
- Mapping to year and usage archetypes
- Providing a location in space (for solar and microclimate coupling)

### How CityBEM Uses This File

- Reads all building rows and constructs an internal **building list/array**.
- For each building:
  - Uses `year_of_construction` to select envelope properties from the **year archetype** file.
  - Uses `usage_type_code` to select operational and HVAC properties from the **usage-type archetype** file.
  - Links the `building_id` to geometric entities in the STL file based on the same ID or consistent naming.

**Consistency requirement:**  
All buildings referenced in other files (e.g., result selection, PV configuration, etc.) must use the same `building_id` defined here.

---

# 2. Archetype Libraries

Archetypes define **typical properties** for groups of buildings, instead of specifying every parameter per building. CityBEM V2 uses archetypes to keep simulations scalable at the **city level**.

The following files define different archetype dimensions:

- Construction year (envelope, infiltration)
- Usage type (internal loads, schedules, HVAC)
- Material composition (slabs, walls, construction materials)
- Energy sources (for operational GHG calculations)
- Internal heat gains

---

## 2.1 Construction Year Archetype  
### `Input_City_scale_archetype_year.txt`

<div align="center">
  <img src="../assets/input_archetype_year.png" alt="Year-based Archetypes Example" width="80%">
</div>

### Purpose

This file defines **construction-year-based archetypes**, which capture how building envelopes and infiltration rates differ across vintages (e.g., pre-1945, 1945–1975, post-2000).

Each row corresponds to a **construction year class**, usually represented by:
- A year interval (e.g., 1945–1975) or
- A representative year (e.g., 1975)

### Main Data Blocks

1. **Infiltration (ACH)**  
   - Typical air change rates (ACH) for:
     - Winter conditions
     - Summer conditions
   - These values are used to estimate **air leakage** and its impact on heating/cooling loads.
   - ACH values are later converted to mass/volumetric flow rates based on floor area and ceiling height.

2. **Envelope Properties**  
   For each year archetype, the file includes parameters such as:

   - **U-values** (W/m²·K) for:
     - Roof
     - External walls
     - Ground floor/slab
     - Windows
   - **Solar Heat Gain Coefficient (SHGC)** for glazing  
   - **Material properties** (effective values for simplified envelope layers):
     - Heat capacity (Cp, J/kg·K)
     - Density (ρ, kg/m³)
     - Effective layer thickness (m)
   - **Radiative properties**:
     - Shortwave absorptance (α) for walls and roofs
     - Longwave emissivity (ε)

These parameters directly influence:
- Heat transfer through the envelope
- Solar absorption on opaque surfaces
- Thermal mass effects during transient simulations

### How CityBEM Uses This File

- For each building, the `year_of_construction` from `Input_City_scale_building_info.txt` is mapped to a **year archetype row**.
- The envelope and infiltration values are assigned to all surfaces and zones of that building.
- These properties are used in:
  - Surface heat balance equations
  - Zone energy balance
  - Calculation of heating/cooling demand

---

## 2.2 Usage-Type Archetype  
### `Input_City_scale_archetype_usage_type.txt`

<div align="center">
  <img src="../assets/input_usage_type.png" alt="Usage Type Archetype Example" width="80%">
</div>

### Purpose

This file defines **operational and internal-load characteristics** for each building usage type, such as:
- Residential
- Office
- Retail
- Educational
- Healthcare
- Etc.

Each row corresponds to a **usage-type archetype** referenced by `usage_type_code` in `Input_City_scale_building_info.txt`.

### Key Parameters and Their Roles

Typical columns include:

- **Geometric/structural parameters**
  - Floor range (minimum and maximum number of floors where this usage type applies)
  - Floor height `Hf` (m), used to compute zone volume and number of thermal zones

- **Occupancy and schedules**
  - Occupancy schedule ID (linking to schedule definitions used for internal gains)
  - Maximum occupancy density (person/m²)
  - Sensible and latent heat gains per person (W/person)
  - Activity levels may be defined or referenced elsewhere, depending on implementation

- **HVAC parameters**
  - Heating and cooling setpoint temperatures for:
    - Occupied periods
    - Unoccupied periods (setback/set-up)
  - HVAC system type or operation mode (e.g., heating-only, cooling-only, full HVAC)
  - HVAC efficiencies:
    - Seasonal performance factor (SPF) for heat pumps
    - Coefficient of performance (COP) for chillers or cooling systems

- **Envelope-related operational parameters**
  - Window-to-wall ratio (WWR) for typical façade design per usage type
  - Heated floor area ratio (fraction of building floor area that is conditioned/heated)

### How CityBEM Uses This File

- For each building, CityBEM:
  - Reads its `usage_type_code`
  - Assigns the corresponding internal loads, schedules, HVAC setpoints, and system efficiencies
- These archetypes determine:
  - Internal gains (occupants, lighting, appliances) as a function of time
  - The target indoor temperature for the HVAC control
  - Whether a zone is actively heated/cooled or allowed to float

This file is central to **operational behavior** and has a strong influence on:
- Annual energy demand
- Peak loads
- Comfort-related variables (indoor temperature, humidity)

---

## 2.3 Slab Material Archetype  
### `Input_City_scale_archetype_slabs_material.txt`

<div align="center">
  <img src="../assets/input_slabs_material.png" alt="Slab Material Archetype Example" width="80%">
</div>

### Purpose

This file defines **simplified multi-layer slab constructions** (e.g., floor slabs, intermediate slabs) used to represent the thermal mass of horizontal building elements.

Each row represents a **slab type**, which may be linked to specific usage types or construction-year categories depending on your workflow.

### Typical Parameters

- Total slab thickness (m)
- Layer sequence and thicknesses (e.g.):
  - Concrete
  - Insulation
  - Gypsum board
  - Wood floor
- Optional effective thermal properties if present:
  - Conductivity (W/m·K)
  - Volumetric heat capacity (J/m³·K)

### How CityBEM Uses This File

- Converts layer definitions into:
  - Overall thermal resistance / capacitance
  - Equivalent thermal mass representation
- Applies slab properties in:
  - Heat balance equations for floor surfaces
  - Transient storage of energy in the building mass

---

## 2.4 External Wall Material Archetype  
### `Input_City_scale_archetype_external_wall_material.txt`

<div align="center">
  <img src="../assets/input_external_wall_material.png" alt="External Wall Material Archetype Example" width="80%">
</div>

### Purpose

This file defines **simplified external wall constructions** for typical wall types used in the building stock.

Each row corresponds to a **wall material configuration**, which may be associated with specific year/usage archetypes.

### Typical Parameters

- Number of layers and their material type (e.g., brick, insulation, concrete, sheathing)
- Thickness of each layer (m)
- Optional effective properties (if not provided in the year archetype):
  - Thermal conductivity (W/m·K)
  - Density (kg/m³)
  - Specific heat (J/kg·K)

### How CityBEM Uses This File

- Builds an equivalent 1D conduction model for wall elements.
- Evaluates:
  - Transient heat conduction through walls
  - Thermal mass buffering effects
- Works in combination with the **year archetype** file:
  - The year archetype provides effective U-values and high-level properties.
  - This material file refines the internal representation if detailed mass modeling is required.

---

## 2.5 Internal Heat Gains (IHG)  
### `Input_City_scale_archetype_IHG.txt`

<div align="center">
  <img src="../assets/input_IHG.png" alt="Internal Heat Gains Example" width="80%">
</div>

### Purpose

This file specifies **detailed internal heat gain profiles** for each usage type or sub-case (e.g., residential weekday vs. weekend, office, school, etc.).

Each row usually represents an **IHG case** that can be linked to a usage-type archetype.

### Typical Parameters

- **Occupancy-related**
  - Occupancy schedules (fraction of peak occupancy over time)
  - Sensible heat gain per person (W/person)
  - Latent heat gain per person (W/person)

- **Appliances and plug loads**
  - Equipment schedules (fraction of maximum load)
  - Maximum equipment power density (W/m²)

- **Lighting**
  - Lighting schedules
  - Lighting power density (W/m²)
  - Radiative vs. convective fraction of lighting gains (if considered)

### How CityBEM Uses This File

- Reads schedule IDs and mapping definitions to generate **time-varying internal loads**.
- At each simulation timestep, the framework:
  - Evaluates occupancy, lighting, and appliance loads
  - Splits them into sensible and latent components (if modeled)
  - Injects them into the zone energy and moisture balance

This file is essential for capturing **daily and seasonal patterns** of internal gains.

---

## 2.6 Energy Sources & Emission Factors  
### `Input_City_scale_archetype_energy_source.txt`

<div align="center">
  <img src="../assets/input_energy_source.png" alt="Energy Source Archetype Example" width="80%">
</div>

### Purpose

This file describes how **energy carriers** (e.g., electricity, natural gas, district heating) are used in the model and assigns **GHG emission factors** to them.

### Typical Parameters

- Number of energy carriers defined
- For each energy carrier:
  - Name (e.g., `Electricity`, `NaturalGas`, `DistrictHeat`)
  - Emission factor (e.g., gCO₂e/kWh, kgCO₂e/kWh)
  - Possibly primary energy factor (if implemented)
  - Mapping to specific **end uses**, such as:
    - Space heating
    - Space cooling
    - Fans and pumps
    - Lighting
    - Appliances and plug loads

### How CityBEM Uses This File

- After computing energy demand and HVAC consumption, CityBEM assigns each end-use energy flow to an energy carrier.
- The corresponding **emission factors** are applied to compute:
  - Time-resolved GHG emissions
  - Aggregated annual GHG emission metrics

This enables comparison of retrofitting scenarios not only in terms of energy, but also in terms of **carbon footprint**.

---

## 2.7 Construction Materials (Embodied Emissions)  
### `Input_City_scale_archetype_construction_materials_general.txt`

<div align="center">
  <img src="../assets/input_construction_materials.png" alt="Construction Materials General Archetype" width="80%">
</div>

### Purpose

This file provides **general material properties** used to compute **embodied emissions** for the building stock.

Each row represents a **material type** commonly used in regional construction (e.g., Montreal).

### Typical Parameters

- Material name (e.g., `Concrete`, `Brick`, `MineralWool`, `GypsumBoard`)
- Density (kg/m³)
- Embodied emission factor for life cycle stages A1–A3 (e.g., kgCO₂e/m³ or kgCO₂e/kg)

### How CityBEM Uses This File

- Combined with envelope and material layer definitions (thickness, area), CityBEM can:
  - Estimate **material quantities** at the building scale
  - Compute total **embodied GHG emissions** (GHG-EE)
  - Compare embodied emissions across retrofitting scenarios

---

# 3. Building Geometry Input

## 3.1 ASCII STL Geometry  
### `Input_City_scale_geometry_CityBEM.stl`

<div align="center">
  <img src="../assets/input_geometry_stl.png" alt="CityBEM STL Geometry Example" width="80%">
</div>

### Purpose

This file provides the **3D geometry** of the building stock in **ASCII STL** format.  
Each building is represented as a separate `solid` block.

### Coordinate System

CityBEM uses a specific coordinate convention:

- **X-axis:** East  
- **Y-axis:** Vertical (upwards)  
- **Z-axis:** North  

This is important when:
- Importing geometry from other tools
- Aligning with external CFD or microclimate models
- Interpreting solar positions (azimuth, altitude)

### Structure

- Each building `solid` typically contains multiple triangular facets.
- Each facet is used to reconstruct planar surfaces (walls, roofs, slabs).
- A preprocessing step groups triangles that belong to the same physical surface.

### Preprocessing

A dedicated preprocessing tool is available to:
- Convert arbitrary STL/OBJ geometry into CityBEM’s **coordinate system**
- Assign `building_id` tags to each solid
- Optionally filter out non-building objects (terrain, furniture, etc.)

This geometry file is the basis for:
- Determining building envelope areas and orientation
- Computing solar radiation (direct, diffuse, reflected)
- Inter-building shading analysis

---

# 🌦️ 4. Weather Data

<div align="center">
  <img src="/assets/input_weatherdata.png" alt="Weather Data Example" width="120%">
</div>

---

## 📘 Overview

`Input_weatherdata.txt` provides the meteorological time series that CityBEM uses as boundary conditions for all buildings in the simulation domain.

This file typically contains **one full year** of data at a fixed time interval (e.g., hourly or sub-hourly), but **shorter custom periods** are also supported.

---

## 📄 File Structure

<div class="grid cards" markdown>

- **:material-file-table: Format**  
  Plain-text file containing a header row followed by **one row per timestep**.

- **:material-clock-outline: Time Resolution**  
  Must match the general simulation settings (e.g., `3600 s`, `900 s`).

- **:material-database: Columns**  
  Standard format includes **13 well-defined meteorological variables**.

</div>

---

## 📑 Weather Data Description

<div align="center" style="width: 95%; margin: 0 auto;">

<table style="width: 95%; margin: 0 auto;">
  <thead>
    <tr>
      <th style="width: 120px; text-align: left;">Column Name</th>
      <th style="text-align: left;">Description</th>
      <th style="width: 100px; text-align: left;">Unit</th>
    </tr>
  </thead>
  <tbody>

    <tr>
      <td><code>GHI</code></td>
      <td>Global Horizontal Irradiance</td>
      <td>W/m²</td>
    </tr>

    <tr>
      <td><code>DNI</code></td>
      <td>Direct Normal Irradiance</td>
      <td>W/m²</td>
    </tr>

    <tr>
      <td><code>DHI</code></td>
      <td>Diffuse Horizontal Irradiance</td>
      <td>W/m²</td>
    </tr>

    <tr>
      <td><code>CZ</code></td>
      <td>Cosine of solar zenith angle</td>
      <td>–</td>
    </tr>

    <tr>
      <td><code>Ta</code></td>
      <td>Dry-bulb outdoor air temperature</td>
      <td>°C</td>
    </tr>

    <tr>
      <td><code>TD</code></td>
      <td>Dew point temperature</td>
      <td>°C</td>
    </tr>

    <tr>
      <td><code>TG</code></td>
      <td>Ground (soil) temperature</td>
      <td>°C</td>
    </tr>

    <tr>
      <td><code>RH</code></td>
      <td>Relative humidity</td>
      <td>%</td>
    </tr>

    <tr>
      <td><code>WS</code></td>
      <td>Wind speed</td>
      <td>m/s</td>
    </tr>

    <tr>
      <td><code>WD</code></td>
      <td>Wind direction</td>
      <td>degrees</td>
    </tr>

    <tr>
      <td><code>P</code></td>
      <td>Atmospheric pressure</td>
      <td>Pa</td>
    </tr>

  </tbody>
</table>

</div>

---

## 🌐 Supported Weather Data Sources

CityBEM can operate with multiple types of meteorological datasets:

<div class="grid cards" markdown>

- **:material-weather-sunny: Single Weather Station**  
  Typical meteorological year (TMY), airport stations, or local met towers.

- **:material-server-network: Mesoscale Models (SPS / HRDPS)**  
  High-resolution gridded weather data interpolated to the study area.

- **:material-map-marker-distance: Custom / Research Datasets**  
  Any user-provided time series following the same column structure.

</div>

!!! tip "Download TMY Weather Data (Global)"
    You can access **TMY weather files for thousands of global locations** using the  
    **National Solar Radiation Database (NSRDB)**.

    🌐 **Access Database:**  
    <a href="https://nsrdb.nrel.gov/" target="_blank">National Solar Radiation Database</a>

    The NSRDB provides high-quality solar and meteorological datasets suitable for  
    CityBEM simulations, including **GHI, DNI, DHI, temperature, wind, and humidity**.

---

## ⚙️ How CityBEM Uses Weather Data

Weather inputs drive the physical models for every timestep in the simulation.

#### 1. Thermal Boundary Conditions
Used for:

- Surface heat balance  
- Indoor zone energy balance  
- Moisture balance (humidity, evaporation, infiltration)

#### 2. Solar Radiation Modeling
Provides:

- Horizontal and tilted plane irradiance  
- Direct + diffuse solar components  
- Inputs to shading models

#### 3. Wind-Dependent Models
Wind data supports:

- Infiltration + natural ventilation models  
- Microclimate coupling simulation (if enabled)

---

## 📝 Weather Data Summary

!!! note "🔧 Key Notes"
    - Must provide a **continuous time series** with a fixed time step  
    - Must include **sufficient weather variables**.  
      For example, **global horizontal irradiance (GHI)** is enough for solar modeling if other radiation terms are unavailable, and **air temperature + relative humidity (or dew point)** are sufficient since other psychrometric quantities can be derived.
    - The simulation time period must match the **valid period of the input weather data**  
    - The dataset must represent a **climate zone consistent with the study area**

---

# 5. Simulation Settings

## 5.1 General Settings  
### `Input_general_settings.txt`

<div align="center">
  <img src="../assets/input_general_settings.png" alt="General Settings Example" width="80%">
</div>

### Purpose

This file defines **global simulation flags and modes** that control how CityBEM behaves at runtime.

### Typical Parameters

- **Problem type**
  - Pure building energy simulation
  - Building–microclimate co-simulation (CityFFD)
  - GEM or other external couplings

- **Microclimate coupling**
  - Coupling interval (e.g., how often microclimate data is updated)
  - Flags to activate/deactivate microclimate feedback on surface boundary conditions

- **Debug and logging options**
  - Level of diagnostic printing
  - Options to produce additional intermediate files for debugging

- **Long simulation controls**
  - Rules for reseting internal state variables
  - Controls to avoid numerical drift in very long transient simulations

- **CityLayers integration**
  - Flags to enable linking with external **digital twin** or **city platform** tools (e.g., CityLayers)

### How CityBEM Uses This File

- Initializes the main simulation mode before any time-stepping loop is started.
- Determines which solvers, coupling schemes, and I/O routines are activated.
- Allows users to configure advanced workflows (e.g., coupled Urban Climate–Building Energy simulations) without changing the code.

---

## 5.2 City-Scale Simulation Settings  
### `Input_City_scale_settings.txt`

<div align="center">
  <img src="../assets/input_city_scale_settings.png" alt="CityBEM City-scale Settings Example" width="80%">
</div>

### Purpose

This is the **central configuration file** controlling city-scale simulation parameters.  
It defines **time controls**, **geographic parameters**, **building physics options**, and **optional add-ons**.

### Main Parameter Groups

#### 1. Time Controls
- Simulation start date and time
- Simulation end date or total duration
- Timestep size (seconds)
- Optionally, warm-up periods for stabilizing initial conditions

#### 2. Geographic Controls
- Latitude and longitude of the study area
- Time zone (e.g., UTC offset)
- Site elevation
- These parameters are used for:
  - Solar position calculation
  - Atmospheric corrections

#### 3. Building Physics Settings
- Number of wall and roof layers used in the conduction model
- Ventilation settings:
  - Infiltration model options
  - Mechanical ventilation rates (if globally applied)
- HVAC operation options:
  - Control strategy (e.g., thermostat-based, scheduled operation)
  - Operation mode flags (e.g., heating-season-only, cooling-season-only, or combined)

#### 4. Optional Add-ons
Flags and parameters to activate or configure:

- **Rooftop PV model**
  - Whether to simulate PV
  - Links to PV design inputs and shading models
- **Green roofs**
  - Whether green roof modules are activated
- **Shading model**
  - Whether detailed 3D ray-tracing shading calculations are used
- **Microclimate coupling**
  - Whether to use microclimate-adjusted boundary conditions from CityFFD or other tools

### How CityBEM Uses This File

- Read at initialization to:
  - Configure the internal solver structure
  - Set global constants and switches
- Every subsequent input file is interpreted **in the context** of these settings (e.g., time step, simulation period).

---

# 6. Result Selection

## 6.1 `Input_City_scale_result_selection.txt`

<div align="center">
  <img src="../assets/input_result_selection.png" alt="CityBEM Result Selection File" width="80%">
</div>

### Purpose

This file allows users to **control which variables** are written to the output files, reducing storage and focusing on relevant KPIs.

Each row typically represents one **output variable**.

### Typical Columns

- Variable index (internal ID used by CityBEM)
- Symbol or descriptive name (e.g., `Tzone`, `Q_heating`, `PV_power`)
- Print flag (Y/N) – whether to output this variable at all
- Transient output flag (Y/N) – whether to store time series for each timestep or only aggregate values

### Typical Variable Categories

- Indoor temperatures (air, mean radiant, surface)
- Thermal loads (heating, cooling)
- Humidity and moisture-related quantities
- Solar gains (per surface or per building)
- HVAC power and energy use by end use
- GHG emissions (operational and, if implemented, embodied)
- Rooftop PV outputs (power, energy, self-sufficiency indices, export/import flows)

### How CityBEM Uses This File

- During the simulation, only variables flagged for output are:
  - Computed at the detailed reporting level
  - Written to disk as time series or aggregated results
- This enables efficient handling of large-scale simulations (tens of thousands of buildings) without unnecessary I/O.

---

# 7. Solar Preprocessing

## 7.1 Cosine of Zenith Angle  
### `Input_cosine_zenith.txt`

<div align="center">
  <img src="../assets/input_cosine_zenith.png" alt="Cosine Zenith Example" width="80%">
</div>

### Purpose

This file stores **precomputed solar geometry quantities** that are reused across the simulation, especially for shading calculations and solar radiation decomposition.

### Typical Content

For each timestep:
- Cosine of the solar zenith angle
- Solar azimuth (north-based convention)
- Optionally additional derived angles if required by the solar model

### How CityBEM Uses This File

- Accelerates solar calculations by avoiding repeated solar-position computations.
- Used by the 3D ray-tracing shading module to:
  - Determine sun direction vectors
  - Compute incidence angles on surfaces
- Can be generated automatically as part of the pre-processing workflow.

---

# ✅ Summary

This page documents all **input files** required to run CityBEM V2, the role of each file, and the main parameters controlling city-scale building energy simulations.

Before running a simulation, users should ensure:

- All input text files are:
  - Properly formatted (tab-separated, consistent headers)
  - Free of missing values in critical columns
- IDs and codes are **consistent across files**:
  - `building_id` matches between building info, geometry, and outputs
  - `usage_type_code` and year classes correctly map to archetype definitions
- Weather data fully covers the desired simulation period with a timestep consistent with the settings.
- Geometry is provided in CityBEM-compliant STL format, respecting the coordinate system.
- Result selection is carefully configured to capture required outputs without generating unnecessary data.

For concrete examples, sample files, and case studies, refer to the **Test Cases** section of the documentation.