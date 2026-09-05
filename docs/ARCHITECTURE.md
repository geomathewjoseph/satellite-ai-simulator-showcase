# System Architecture & Technical Specifications

The **Satellite AI Simulator** models the autonomous operation of Low Earth Orbit (LEO) satellites equipped with onboard edge neural inference payloads, dynamic solar/battery power systems, thermal equilibrium modeling, and a multi-station ground communication mesh.

---

## 1. Subsystem Architecture Overview

```
                      +---------------------------------------+
                      |         SPACE SEGMENT (LEO)           |
                      |                                       |
                      |   +-------------------------------+   |
                      |   |   Orbital Dynamics (SGP4/J2)  |   |
                      |   +---------------+---------------+   |
                      |                   |                   |
                      |   +---------------v---------------+   |
                      |   |   Thermal & Power Subsystem   |   |
                      |   |  - Solar Flux (1361 W/m^2)    |   |
                      |   |  - Battery DOD (LiFePO4)      |   |
                      |   |  - Radiative Thermal Balance  |   |
                      |   +---------------+---------------+   |
                      |                   |                   |
                      |   +---------------v---------------+   |
                      |   |   Edge AI Vision Payload      |   |
                      |   |  - YOLOv8 Optical Detection   |   |
                      |   |  - INT8 Tensor Quantization   |   |
                      |   |  - Anomaly Target Isolation   |   |
                      |   +---------------+---------------+   |
                      |                   |                   |
                      |   +---------------v---------------+   |
                      |   |  RF Downlink & Link Budget    |   |
                      |   |  - Friis Transmission Model   |   |
                      |   |  - Dynamic Doppler Tracking   |   |
                      |   +---------------+---------------+   |
                      +-------------------|-------------------+
                                          | Space-to-Ground RF
                                          v
                      +---------------------------------------+
                      |        GROUND SEGMENT (MESH)          |
                      |                                       |
                      |   [Svalbard]  [White Sands]   [Perth] |
                      |       |             |            |    |
                      |       +-------------+------------+    |
                      |                     |                 |
                      |          +----------v----------+      |
                      |          | Station Contact Mgr |      |
                      |          +----------+----------+      |
                      +---------------------|-----------------+
                                            | Telemetry WebSockets
                                            v
                      +---------------------------------------+
                      |       MISSION CONTROL (FRONTEND)      |
                      |                                       |
                      |  - Three.js WebGL 3D Celestial Earth  |
                      |  - Real-time Subsystem Gauge Deck     |
                      |  - Edge AI Inference Feed Monitor     |
                      |  - Orbital Pass Historical Archive    |
                      +---------------------------------------+
```

---

## 2. Orbital Mechanics & Perturbation Dynamics

### SGP4 Analytical Propagation
The satellite position and velocity vectors are propagated using Keplerian elements perturbed by non-spherical Earth gravity (J2, J3, J4 zonal harmonics), atmospheric drag (exponential atmospheric density model), and solar/lunar third-body gravitational perturbations.

$$\mathbf{r}(t) = \mathbf{r}_{\text{Kepler}}(t) + \delta\mathbf{r}_{J_2} + \delta\mathbf{r}_{\text{drag}}$$

- **True Equator, Mean Equinox (TEME)** to **Earth-Centered, Earth-Fixed (ECEF)** coordinates
- **Geodetic Coordinate Conversion**: Sub-satellite latitude $\phi$, longitude $\lambda$, and altitude $h$ above WGS-84 ellipsoid:
  $$N(\phi) = \frac{a}{\sqrt{1 - e^2 \sin^2\phi}}$$

---

## 3. Power, Thermal & Energy Subsystems

### Solar Array Power Generation
$$P_{\text{solar}} = G_{\text{solar}} \cdot A_{\text{array}} \cdot \eta_{\text{cell}} \cdot \cos(\theta_{\text{inc}}) \cdot \sigma_{\text{eclipse}}$$
Where:
- $G_{\text{solar}} = 1361 \, \text{W/m}^2$ (Solar irradiance constant)
- $\sigma_{\text{eclipse}} \in \{0, 1\}$ calculated via cylindrical Earth shadow geometry.

### Radiative Equilibrium & Thermal Dynamics
$$Q_{\text{in}} = \alpha \cdot A \cdot G_{\text{solar}} + P_{\text{internal}}$$
$$Q_{\text{out}} = \epsilon \cdot \sigma_{\text{SB}} \cdot A_{\text{rad}} \cdot (T_{\text{sat}}^4 - T_{\text{space}}^4)$$
$$\frac{dT}{dt} = \frac{Q_{\text{in}} - Q_{\text{out}}}{C_{\text{thermal}}}$$

---

## 4. Edge AI Vision Inference Payload

- **Architecture**: Optimized lightweight CNN/Transformer architecture (YOLOv8 nano/small quantized).
- **Target Classes**: Maritime vessels, storm formations, terrestrial wildfires, aircraft, infrastructure anomalies.
- **Inference Budget**: Sub-45ms execution cycle under constrained 15W compute envelope.
- **Selective Downlink**: High-confidence detections trigger telemetry metadata packets, while uncompressed raw imagery is retained on onboard solid-state storage until dedicated high-bandwidth X-band ground contact.

---

## 5. RF Link Budget & Doppler Tracking

### Friis Downlink Equation
$$\frac{P_r}{P_t} = G_t \cdot G_r \cdot \left(\frac{c}{4\pi d f}\right)^2 \cdot L_{\text{atm}} \cdot L_{\text{rain}}$$
- **Frequency**: S-band (2.2 GHz) for telemetry, X-band (8.4 GHz) for high-speed AI payload downlink.
- **Doppler Shift Compensation**:
  $$\Delta f = f_0 \left(\frac{\mathbf{v}_{\text{rel}} \cdot \hat{\mathbf{r}}}{c}\right)$$
