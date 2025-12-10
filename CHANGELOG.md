# 0.2.1
- Added psychrometric data via an optional relative humidity input sensor
  - frost and dew points, actual humidity, humidex, thermal comfort, mold risk, air enthalpy
- Added an auto modeling calibration output via an optional 'wall surface temperature' sensor
  - sensor that shows a calibrated K_loss value, no more guessing your insulation loss value!
- Expanded dedicated sensor inputs for users with a local weather station or sensors
  - outdoor temp/rel. humidity, wind speed, atmospheric pressure
- remove forgotten 2.5 multiplier in mold risk sensor logic
# 0.2.0
- Added using sun azimuth to calculate the **Solar Angle of Incidence Factor**
  - impacts how the sun hits the window
- Convective weighting using ASHRAE simplified coefficients (hr, hc)
  - adjust the rooms estimated air speed in m/s.
  - adds logic for monitoring windows, doors, fans and also a climate entity if the system is forced air
- Config/option bool to tell the integration if your optional climate entity controls a forced air system or a radiant heat system.
- Radiant boost based on radiant climate entity and the radiant system target temperature
- Shade/Cover/Blind solar factor
  - Monitor a sensor that tells us how much of the window is covered 
# 0.1.0
Initial release with basic functionality.