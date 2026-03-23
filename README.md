# open-meteo-api-skill

This repository contains an AI Agent Skill for interacting with the **Open-Meteo APIs**. It provides comprehensive instructions for an AI assistant to correctly format requests, handle timezones, and parse complex JSON responses across the diverse Open-Meteo services.

## Installation

This repository is structured for the `skills` CLI so it can be installed with:

```bash
npx skills add Enter-tainer/open-meteo-api-skill --skill open-meteo-api
```

Or from a local checkout:

```bash
npx skills add ./open-meteo-api-skill --skill open-meteo-api
```

## Features

The `open-meteo-api` skill comprehensively covers 4 core Open-Meteo APIs:

1. **Geocoding API**: Resolving location names, postal codes, and partial matches to precise WGS84 coordinates.
2. **Weather Forecast API**: Fetching current conditions, hourly forecasts, 15-minutely granularity data, and daily aggregations.
3. **Ensemble Models API**: Exploring forecast uncertainty and probabilities across 30+ member models over up to 35-day horizons.
4. **Air Quality API**: Getting regional and global pollutant data, composite AQI (US & European), and European pollen forecasts.

It also thoroughly covers Open-Meteo's specific behaviors such as:
- Timezone handling (`timezone=auto`, UTC unix timestamps).
- Processing multi-location JSON arrays.
- Automatic model blending (`auto`) vs explicit ensemble member iteration (`models=...`).

## Repository Layout

```text
skills/
  open-meteo-api/
    SKILL.md          # Head skill file detailing endpoints, parameters, and workflows
    references/       # (Optional) Full dictionary of granular parameters for each API
```
