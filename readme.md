# Ostrom Price Monitoring and Energy History

This integration monitors electricity spot prices and energy consumption via the [Ostrom Energy API](https://docs.ostrom-api.io/reference/api-access). It provides real-time price sensors, a historical energy usage chart in the HA Energy Dashboard, and a historical spot price chart in the Statistics panel.

The API provides prices for the current day. After ~14:00 it also provides prices for the following day (until 23:00).

The integration polls every **10 minutes** and aligns the first poll to the next full hour after startup.

## ⚙️ Sensors

![Ostrom Sensors](https://github.com/oliverwehrens/homeassistant_ostrom_integration/blob/main/images/ostrom_sensors.png?raw=true)

| Sensor | Description |
|--------|-------------|
| Spot Price | Current hour's gross price in €/kWh (includes tax & levies). Attributes contain all upcoming prices for use in charts and automations. |
| Average Price | Average across all prices currently available from the API |
| Lowest Price | Lowest price of the available forecast window |
| Highest Price | Highest price of the available forecast window |
| Next Hour Price | Price for the next full hour |
| Lowest Price Time | Timestamp of the cheapest hour |
| Highest Price Time | Timestamp of the most expensive hour |
| **Price Category** | `Günstig` / `Normal` / `Teuer` — based on current price vs. daily average (±15 %) |

### Price Category attributes

The **Price Category** sensor exposes these extra attributes useful for automations:

| Attribute | Description |
|-----------|-------------|
| `current_price` | Current price in €/kWh |
| `average_price` | Average price of the forecast window |
| `ratio` | current / average (e.g. 0.78 = 22 % below average) |
| `cheapest_upcoming_hours` | List of the 3 cheapest upcoming hour timestamps (ISO 8601, local time) |

**Example automation — charge car during cheapest 3 hours:**
```yaml
trigger:
  - platform: template
    value_template: >
      {{ now().isoformat() in state_attr('sensor.ostrom_energy_preiskategorie', 'cheapest_upcoming_hours') }}
action:
  - service: switch.turn_on
    target:
      entity_id: switch.wallbox
```

## 📈 Charts

### ApexCharts — upcoming prices

[thomsbe](https://github.com/thomsbe) added a nice apexcharts card. Thanks for the '[Issue](https://github.com/oliverwehrens/homeassistant_ostrom_integration/issues/1)'.

```yaml
type: custom:apexcharts-card
graph_span: 23h
span:
  start: hour
  offset: "-1h"
header:
  title: Strompreise Zukunft (€/kWh)
  show: true
apex_config:
  xaxis:
    type: datetime
    labels:
      datetimeFormatter:
        hour: HH:mm
        day: dd MMM
  plotOptions:
    bar:
      colors:
        ranges:
          - from: 0
            to: 0.15
            color: "#2ecc71"
          - from: 0.15
            to: 0.2
            color: "#a6d96a"
          - from: 0.2
            to: 0.25
            color: "#ffff99"
          - from: 0.25
            to: 0.3
            color: "#fdae61"
          - from: 0.3
            to: 0.35
            color: "#f46d43"
          - from: 0.35
            to: 1
            color: "#d73027"
series:
  - entity: sensor.ostrom_energy_spotpreis
    attribute: prices
    float_precision: 3
    type: column
    name: Preis
    data_generator: |
      const prices = entity.attributes.prices;
      return Object.entries(prices).map(([timestamp, value]) => {
        const date = new Date(timestamp);
        return [date, value];
      });
    show:
      datalabels: false
      in_header: true
yaxis:
  - min: 0
    max: 0.5
```

### Historical spot price chart (Statistics panel)

Since version 1.1.2 the integration writes spot prices as external statistics (`ostrom:ostrom_hourly_spot_price`). You can add them via **Settings → Devices & Services → Ostrom → Statistics** or directly in the HA Statistics graph card.

![Home Assistant Energy Ostrom](images/ostrom-usage-history.png?raw=true)

## 🔐 Credentials

You need a **Client ID** and **Client Secret** from the [Ostrom Developer Portal](https://developer.ostrom-api.io/).

![Ostrom Developer Portal](https://github.com/oliverwehrens/homeassistant_ostrom_integration/blob/main/images/ostrom_client.png?raw=true)

## 👨🏻‍🔧 Installation

### Via HACS

[Add to Home Assistant](https://my.home-assistant.io/redirect/hacs_repository/?owner=oliverwehrens&repository=homeassistant_ostrom_integration&category=integration)

Or add manually:

- Home Assistant → HACS → Integrations
- Top-right ⋮ → Custom repositories
- URL: `https://github.com/oliverwehrens/homeassistant_ostrom_integration`
- Category: Integration

### Manually

- Copy the `ostrom` folder to your `config/custom_components/` directory
- Restart Home Assistant
- Add the integration under **Settings → Devices & Services → Add Integration → Ostrom**
- Enter your Client ID, Client Secret, ZIP code, and select the environment

## 🐛 Debugging

To see detailed logs, add this to your `configuration.yaml`:

```yaml
logger:
  default: warning
  logs:
    custom_components.ostrom: debug
```

## 📋 Changelog

### 1.2.0
- Fix intermittent energy usage data caused by silent crash when statistics query returned empty results
- Fix negative initial update interval on HA restart within the first minutes of an hour
- Limit historical data fetch to once per hour to avoid blocking the coordinator
- Add **Price Category** sensor (`günstig` / `normal` / `teuer`) with cheapest upcoming hours attribute
- Store spot prices as external statistics for historical charts in the Statistics panel
- Clean up log levels (normal operations no longer logged as WARNING)

### 1.1.1
- Add unit class to hourly energy consumption statistics

### 1.1.0
- Add historical energy usage fetch and Energy Dashboard integration

## ❤️ Pull Requests

Are welcome!

## 🪪 License

MIT License — see the LICENSE file for details.

## Other Integrations

Not tested:

- https://github.com/ChrisCarde/homeassistant-ostrom
- https://github.com/melmager/ha_ostrom

## Questions?

Contact on [🦋 Bluesky](https://bsky.app/profile/owehrens.com).
