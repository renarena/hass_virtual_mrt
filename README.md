# 🌡️ Virtual Thermal Comfort (MRT & Operative Temperature)

[![Open your Home Assistant instance and open a repository inside the Home Assistant Community Store.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=baudneo&repository=hass_virtual_mrt&category=integration)

## 🖼️ Example Config Flow
See [docs/setup.md](./docs/setup.md) for a full example with screenshots.

---

## 🙏 Thanks
This logic was ported from a blueprint shared by [@Ecronika](https://community.home-assistant.io/t/blueprint-virtual-mrt-v0-1-10-mean-radiant-temperature-operative-temperature/945267/3).

---

## 📝 Note
This integration differs from the existing [Thermal Comfort integration](https://github.com/dolezsa/thermal_comfort) by exposing **Mean Radiant Temperature (MRT)** and **Operative Temperature (\(T_{op}\))**, accounting for both surfaces and air movement. Optional relative humidity enables psychrometric sensors.

---

## 🏡 Introduction
Instead of reporting just air temperature \(T_\text{air}\), this integration provides:

- **Mean Radiant Temperature (MRT)**: Temperature of walls, windows, and ceiling.  
- **Operative Temperature \(T_{op}\)**: Weighted average of air and radiant temperatures — best indicator of human comfort.

---

## 💡 Why MRT & \(T_{op}\) Matter

| Sensor | Definition | Usage |
|--------|-----------|-------|
| **MRT** | Effective temperature of room surfaces | Tracks thermal lag, critical for radiant systems. |
| **Operative Temp (\(T_{op}\))** | Weighted average of air and MRT | Trigger HVAC automations based on *perceived* comfort. |

---

## 🏠 Room Configuration: Inputs & Profiles

### Core Inputs

| Name | Meaning | Range | Example |
|------|--------|-------|---------|
| **Exterior Envelope Ratio (\(f_\text{out}\))** | Fraction of room surfaces touching outside | 0–1 | 0.15 apartment, 0.95 attic |
| **Window Share (\(f_\text{win}\))** | % of exterior wall that is glass | 0–1 | 0.1 small basement, 0.5 large window |
| **Insulation Loss (\(k_\text{loss}\))** | Heat leakage / U-value | 0.05–0.3 | 0.10 modern home, 0.25 old walls |
| **Solar Gain (\(k_\text{solar}\))** | Solar heat gain coefficient | 0–2 | 0.8 standard glass, 1.5 skylight |

### Thermal Smoothing Factor (\(\alpha\))
Controls response speed of MRT to changing conditions.

| Name | Purpose | Range | Example |
|------|--------|-------|---------|
| **Thermal Smoothing (\(\alpha\))** | Controls MRT inertia | 0.05–0.95 | 0.05 thick masonry, 0.6 wood-frame |

**Behavior:** Changes apply immediately but the MRT update is smoothed.

### Profile Management
- Save profiles for rooms.  
- Selecting a default and editing creates **"Unsaved Custom Profile"**.  
- Max **100 custom profiles per device**.

---

## 🧠 Model Calibration (Optional)
Estimate insulation (\(k_\text{loss}\)) using a **wall surface sensor**:

\[
k_\text{loss} \approx \frac{T_\text{air} - T_\text{wall}}{T_\text{air} - T_\text{out}}
\]

- Place on interior surface of exterior wall.  
- Wait for stable night conditions.  
- Update **Insulation Loss Factor** in room configuration.

---

## 🌬️ Dynamic Comfort Factors

### Air Speed (\(v_\text{air}\)) Determination
Maximum of:

| Source | Air Speed Contribution |
|--------|----------------------|
| Door open | 0.8 m/s |
| Window open | 0.5 m/s |
| HVAC active | Default 0.4 m/s |
| Fan | low:0.3, med:0.5, high:0.8 |
| Manual input | >0.1 m/s |
| Default | 0.1 m/s |

### Radiant Heating
- Checkbox **"Is Radiant Heating?"**: boosts MRT instead of air temp.  
- Smoothing simulates thermal mass (slab/radiator heat-up/cool-down).

### Shading Factor
- 0% = closed, 1 = open.  
- Reduces solar gain in MRT calculation.

---

## 📡 External Data

| Data | Source | Usage |
|------|--------|------|
| Outdoor Temp (\(T_\text{out}\)) | Weather entity | Heat loss |
| Apparent Temp | Weather | Adjust for wind chill |
| Wind Speed | Weather | Convective loss |
| Cloud / UV | Weather/sensor | Solar gain estimate |
| Rain | Weather | Solar gain penalty |
| Sun Elevation/Azimuth | `sun.sun` | Solar angle calculation |
| Global Solar Radiation | Optional sensor | Direct input for solar gain |

**Fallback:** Virtual solar sensor via `forecast.solar` if physical sensor not available. Cap at 1000 W/m².

---

## 🧠 Calculation Flow

1. **Convective Weighting \(A_\text{Radiant}\)**

\[
A_\text{Radiant} = \frac{h_r}{h_c + h_r}
\]

2. **Instant MRT**

\[
\text{MRT}_\text{calc} = T_\text{air} - \text{Loss term} + \text{Solar term} \cdot \text{Shading factor} + \text{Radiant Boost}
\]

3. **Clamping & Smoothing**

\[
\text{MRT}_\text{final} = (1-\alpha)\text{MRT}_\text{prev} + \alpha \text{MRT}_\text{calc}
\]

4. **Operative Temperature**

\[
T_{op} = A_\text{Radiant} \cdot \text{MRT}_\text{final} + (1 - A_\text{Radiant}) \cdot T_\text{air}
\]

---

## 💧 Optional Psychrometrics

| Sensor | Unit | Use |
|--------|------|----|
| Dew Point (\(T_{dp}\)) | °C | Mold prevention |
| Frost Point (\(T_{fp}\)) | °C | Winter frost risk |
| Absolute Humidity | g/m³ | Moisture load comparison |
| Air Enthalpy (\(h\)) | kJ/kg | ERV/Economizer control |
| Humidex | °C | Summer comfort |
| Thermal Perception | Text | Dashboard indicator |

**Mold Risk:**  
Calculates **surface RH** from air temp, humidity, outdoor temp, and insulation.

| Surface RH | Status | Meaning |
|------------|-------|--------|
| <65% | Low | Safe |
| 65–80% | Warning | Mold pre-conditions |
| >80% | Critical | Immediate action required |

---

## 🔌 KNX Integration

Expose MRT and \(T_{op}\) via DPT 9.001:

```yaml
knx:
  expose:
    - type: temperature
      address: "1/0/10"
      entity_id: sensor.living_room_mean_radiant_temperature
    - type: temperature
      address: "1/0/11"
      entity_id: sensor.living_room_operative_temperature
