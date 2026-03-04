# Időkép weather integration for HA

Az időkép oldaláról scrape-pel lehet adatokat szenzorokba kinyerni. Az általam használt konfigok ebben a mappában találhatóak.

This package contains a scrape config to get weather data from Időkép, a popular Hungarian weather service. This repository contains my configs.

FONTOS: Az időjárás típusa (napos, felhős, esik, stb.) az ikonokból került visszafejtésre, és a lista nem biztos, hogy teljes.

FONTOS: Az Időképről nem minden adatot nyertem eddig ki (főleg a típus, hőmérséklet és a csapadék volt fókuszban), így 1-2 helyen egy másik időjárás integrációból veszek dolgokat (pl. szélerősség, légnyomás, stb.), a példámban ez a `weather.otthon` időjárás szenzorból jön.

Példa egyaránt felhasználva a rövid (órás) és a hosszú (napi) előrejelzéseket:

![Minta](/assets/sample.png) ![Még egy minta](/assets/sample2.png)

## Változások

- **v1.0.1** / 2026.03.04: Riasztás szelektor is frissítve
- v1.0.0 / 2026.03.03: Breaking - Városnév kivezetése secret változóba, a könnyebb frissítések miatt (+ egy új példa)
- 2026.03.02: Frissítés az idokep.hu új designja/elnevezései miatt
- 2025.12.15: Weather template syntax frissítése az új HA verzióhoz
- 2024.08.27: Csapadék támogatás
- 2024.02.09: Breaking - A jobb konfig elkülönítés miatt áttértem arra, hogy package-be rakom az időképes dolgokat. Emiatt frissült a felhasználás szekció.

## Felhasználás

- ha még nincs, telepítsd a [HACS-t](https://hacs.xyz/)
- telepítsd a "Multiscrape" integrációt HACS-ben
- másold fel a packages mappát a config-jaid mellé (vagyis tedd az idokep\_\*.yaml fájlokat egy packages/idokep mappába a HA-ban)
- győződj meg, hogy van egy másik időjárás integráció, amire néhány érték fallback-el (az `idokep_weather.yaml`-ben a `weather.otthon` lecserélendő a sajátodra, vagy azokat a sorokat törölni kell)
- a packages mappa-struktúrát a `homeassistant.yaml`-ban található `packages` szekcióval lehet aktiválni
- a saját városod URL-jét add meg a `secrets.yaml`-ben (lásd alább)
- a good_morning script egy példa, hogyan használjuk fel az eredményt egy media player-rel, spotify-jal összekötve, és google cloud TTS-t használva (ez a fizetős TTS, de ez ad jobb magyar hangot, és normál kereteken belül még az ingyenes limiten belül lehet maradni)

### Package struktúra

A yaml fájlokat egy package-be csoportosítva:

- jobban egymás mellé rakhatók és elkülöníthetők az összetartozó fájlok, illetve
- nem kell külön include-olni megfelelő root elembe, hanem a yaml-ben lehet a gyökér elem típus is (pl. `multiscrape`)

Aktiváld a package-eket ha még nem tetted meg korábban a `configuration.yaml`-ben:

```yaml
homeassistant:
  packages: !include_dir_named packages/
```

### URL beállítás secret-tel

Mindenki más településre kíváncsi, ha ezt átírod az idokep_multiscrape.yaml file-ban, akkor mindig újra át kell írni amikor letöltesz egy friss verziót.

Egyszerűbb, ha az URL-t felveszed a secret-ek közé. Írd be az alábbit a `secrets.yaml` file-odba (vagy hozd létre ha még nincs):

```yaml
idokep_idojaras_url: https://www.idokep.hu/idojaras/T%C3%B6r%C3%B6kb%C3%A1lint
idokep_elorejelzes_url: https://www.idokep.hu/elorejelzes/T%C3%B6r%C3%B6kb%C3%A1lint
```

Ezután a `!secret idokep_idojaras_url` be fogja helyettesíteni az URL-t.

A település nevében az ékezeteket le kell fordítani, bármely online tool-ban, pl. [urlencoder.org](https://www.urlencoder.org/).

## Minta kártya konfigok

Alább néhány HACS-es kártya konfigja.

### Sima aktuális időjárás kártya

![Sima aktuális időjárás kártya](/assets/idokep1.png)

<details>
<summary>Konfig</summary>

```yaml
- type: custom:weather-card
  entity: weather.idokep
  number_of_forecasts: '5'
  current: true
  details: true
  forecast: false
  hourly_forecast: false
```

</details>

### Órás előzmény + előrejelzés kártya

![Órás előzmény + előrejelzés kártya](/assets/idokep2.png)

<details>
<summary>Konfig</summary>

```yaml
- type: custom:apexcharts-card
  graph_span: 48h
  now:
    show: true
  span:
    start: hour
    offset: '-1d'
  all_series_config:
    stroke_width: 3
    show:
      datalabels: true
  apex_config:
    legend:
      show: false
    dataLabels:
      formatter: |
        EVAL:function(value, { seriesIndex, dataPointIndex, w }) {
          if (seriesIndex === 0) return value;
          const icon = w.config.series[seriesIndex].data[dataPointIndex][2];
          return icon? [value, icon] : value;
        }
  series:
    - entity: sensor.idokep_temperature
      name: Hőmérséklet
      extend_to: false
    - entity: sensor.idokep_hourly_data
      name: Előrejelzés
      data_generator: |
        const conditionToIcon = {
          "clear-night": "🌙",
          "cloudy": "☁️",
          "exceptional": "⚠️",
          "fog": "🌫️",
          "hail": "🌨️",
          "lightning": "🌩️",
          "lightning-rainy": "⛈️",
          "partlycloudy": "⛅",
          "pouring": "🌧️",
          "rainy": "🌧️",
          "snowy": "🌨️",
          "snowy-rainy": "🌨️",
          "sunny": "☀️",
          "windy": "",
          "windy-variant": "",
        };
        const data = [];
        for (let i = 1; i <= 24; i++) {
          const condition = entity.attributes[`hour${i}_condition`];
          data.push([new Date(entity.attributes[`hour${i}_date`]).getTime(), entity.attributes[`hour${i}_temperature`], conditionToIcon[condition] || ''])
        }
        return data;
```

</details>

### Napi előrejelzés

![Napi előrejelzés](/assets/idokep3.png)

<details>
<summary>Konfig</summary>
```yaml
- type: custom:clock-weather-card
  entity: weather.idokep_2
  hide_clock: true
  locale: hu
  weather_icon_type: fill
  hide_today_section: true
```
</details>

### Kombinált órás/napi előrejelzés

![Még egy minta](/assets/sample2.png)

<details>
<summary>Konfig</summary>

```yaml
type: custom:vertical-stack-in-card
cards:
  - type: custom:simple-weather-card
    card_mod:
      style: |
        ha-card > .weather__icon {
          height: 50px !important;
          width: 50px !important;
          flex: 0 0 50px !important;
        }
        .weather__info__title {
          font-size: 32px;
          line-height: 32px;
        }
        .weather__info__item {
          font-size: 16px;
        }
        .weather__icon--small {
          margin: 0 0.4em !important;
        }
    entity: weather.idokep_2
    name: ' '
    custom:
      - state: sensor.idokep_condition_hu
    primary_info:
      - humidity
    secondary_info:
      - precipitation
    tap_action:
      action: navigate
      navigation_path: /lovelace/idojaras
  - type: custom:apexcharts-card
    graph_span: 28h
    experimental:
      color_threshold: true
    now:
      show: true
    span:
      start: hour
      offset: '-4h'
    all_series_config:
      stroke_width: 3
      show:
        datalabels: true
    apex_config:
      chart:
        height: 150px
      legend:
        show: false
      dataLabels:
        textAnchor: middle
        offsetY: -15
        background:
          opacity: 0.5
        formatter: |
          EVAL:function(value, { seriesIndex, dataPointIndex, w }) {
            if (seriesIndex === 0) return '';
            if (seriesIndex === 1) return value.toFixed(0);
            const icon = w.config.series[seriesIndex].data[dataPointIndex][2];
            return icon? [value, icon] : value;
          }
    series:
      - entity: sensor.idokep_temperature
        name: Hőmérséklet
        extend_to: false
        color_threshold:
          - value: -20
            color: gray
          - value: 35
            color: gray
        fill_raw: last
      - entity: sensor.kinti_homerseklet
        name: Hőmérséklet
        extend_to: false
        color_threshold:
          - value: -20
            color: white
          - value: 5
            color: blue
          - value: 15
            color: green
          - value: 26
            color: yellow
          - value: 30
            color: orange
          - value: 35
            color: red
        fill_raw: last
      - entity: sensor.idokep_hourly_data
        name: Előrejelzés
        color_threshold:
          - value: -20
            color: white
          - value: 5
            color: blue
          - value: 15
            color: green
          - value: 26
            color: yellow
          - value: 30
            color: orange
          - value: 35
            color: red
        fill_raw: last
        data_generator: |
          const conditionToIcon = {
            "clear-night": "🌙",
            "cloudy": "☁️",
            "exceptional": "⚠️",
            "fog": "🌫️",
            "hail": "🌨️",
            "lightning": "🌩️",
            "lightning-rainy": "⛈️",
            "partlycloudy": "⛅",
            "pouring": "🌧️",
            "rainy": "🌧️",
            "snowy": "🌨️",
            "snowy-rainy": "🌨️",
            "sunny": "☀️",
            "windy": "",
            "windy-variant": "",
          };
          const data = [];
          for (let i = 1; i <= 24; i++) {
            const condition = entity.attributes[`hour${i}_condition`];
            data.push([new Date(entity.attributes[`hour${i}_date`]).getTime(), entity.attributes[`hour${i}_temperature`], conditionToIcon[condition] || ''])
          }
          return data;
  - type: custom:clock-weather-card
    entity: weather.idokep_2
    hide_clock: true
    locale: hu
    weather_icon_type: fill
    hide_today_section: true
    forecast_rows: 4
    date_pattern: d D
    tap_action:
      action: navigate
      navigation_path: /lovelace/idojaras
  - type: custom:stack-in-card
    mode: horizontal
    cards:
      - type: conditional
        conditions:
          - entity: sensor.uv_b_sugarzas
            state_not: unavailable
          - entity: sensor.uv_b_sugarzas
            state_not: '0.0'
          - entity: sensor.uv_b_sugarzas
            state_not: '1.0'
          - entity: sensor.uv_b_sugarzas
            state_not: '2.0'
          - entity: sensor.uv_b_sugarzas
            state_not: '3.0'
          - entity: sensor.uv_b_sugarzas
            state_not: '4.0'
        card:
          type: custom:button-card
          entity: sensor.uv_b_sugarzas
          size: 36px
          name: '[[[ return "UV-B: " + entity.state ]]]'
          styles:
            icon:
              - display: inline-block
              - color: |
                  [[[
                    if ( entity.state >= "8") { return "red"}
                    if ( entity.state >= "7") { return "orange"}
                    if ( entity.state >= "5") { return "yellow"}
                    if ( entity.state >= "3") { return "green"}
                    return "var(--paper-item-icon-color)";
                  ]]]
      - type: conditional
        conditions:
          - entity: sensor.pollen_adatok
            state_not: '0'
          - entity: sensor.pollen_adatok
            state_not: '1'
        card:
          type: custom:button-card
          icon: mdi:blur
          size: 36px
          name: '[[[ return "Parlagfű (" + entity?.state + ")" ]]]'
          label: Parlagfű
          show_name: true
          entity: sensor.pollen_adatok
          color_type: icon
          styles:
            label:
              - display: inline
            icon:
              - display: inline-block
              - color: |
                  [[[
                    if ( entity.state == "4") { return "red"}
                    if ( entity.state == "3") { return "orange"}
                    if ( entity.state == "2") { return "yellow"}
                    return "var(--paper-item-icon-color)";
                  ]]]
      - type: conditional
        conditions:
          - entity: sensor.met_alerts
            state_not: '0'
        card:
          type: custom:button-card
          entity: sensor.met_alerts
          size: 30px
          label: |
            [[[
              var label = ""
              var icolor = "black"
              var met_alerts = states['sensor.met_alerts'].attributes.alerts;
              for (var k=0; k < states['sensor.met_alerts'].attributes.nr_of_alerts; k++) {
                if ( met_alerts[k].level == 1 ) {
                  icolor = "yellow";
                } else if ( met_alerts[k].level == 2 ) {
                  icolor = "orange";
                } else if ( met_alerts[k].level == 3 ) {
                  icolor = "red";
                }
                label += `<ha-icon icon="` + met_alerts[k].icon +
                         `" style="width: 28px; height: 28px; color:` + icolor +
                          (states['sensor.met_alerts'].attributes.nr_of_alerts == 1 ? `; margin-bottom: 10px;">` : `;">`) +
                          `</ha-icon>&nbsp;` +
                          (states['sensor.met_alerts'].attributes.nr_of_alerts == 1 ? `<br>` : ``) +
                         `<span>` + met_alerts[k].type + ` (` + met_alerts[k].level + `)</span><br>`;
              }
              return label;
            ]]]
          show_label: true
          show_name: false
          show_icon: false
          color_type: icon
          styles:
            label:
              - font-size: 90%
            card:
              - padding-top: 9px
              - padding-bottom: 8px
```

</details>

### Radar

Bónusz, nem időképes forrás, de az időkép app-ból elmaradhatatlan a radar, íme egy példa

![Radar](/assets/idokep4.png)

<details>
<summary>Konfig</summary>

```yaml
- type: custom:weather-radar-card
  data_source: RainViewer-UniversalBlue
  map_style: Voyager
  zoom_level: 8
  marker_latitude: 47.4394497
  marker_longitude: 18.8695576
  show_marker: true
  static_map: false
  show_scale: false
  show_zoom: true
  show_playback: true
  show_range: false
  square_map: false
  show_recenter: true
  extra_labels: false
```

</details>

Némely kártyát én kinyithatóra csináltam, mert nem nézegetem mindig, és így gyorsabban is tölt be a
felület, de mégis ott van szükség esetén.

<details>
<summary>Konfig</summary>

```yaml
- type: custom:expander-card
  title: Radar
  cards:
    - type: ...
      ...
  clear: false
  child-padding: '0'
  padding: '0'
```

</details>

## Chat-GPT

A repó-ban lévő minta felolvasást én azóta egy ChatGPT-sre cseréltem, amit Node-RED-ből hívok meg. A szenzorok értékeit összegyűjtöm, majd összeállítok hozzá valami ilyesmit:

<details>
<summary>Kód</summary>

```javascript
const preText = `Vicces és cuki vagy, Cilinek hívnak, [X városban] vagy.`;
const text = `Mondd el a reggeli időjárásjelentést rádiós műsor formátumban legfeljebb 6 mondatban.
[X városban] ${msg.day1Min.replace(/\.[0-9]/, '')} és ${msg.day1Max.replace(/\.[0-9]/, '')}  fok között lesz a hőmérséklet és az idő ${msg.day1CondHu}.
Most ${msg.tempOut.replace(/\.[0-9]/, '')} fok van és az idő ${msg.condition}.
Ma ${msg.nevnap.attributes.message} névnap van.
A végén mondj egy érdekes kapcsolódó ismeretterjesztő tényt.
Ebből is hozzátehetsz valamit, ha fontosnak tűnik: ${msg.forecastDetailsEntity.attributes.reszletek}`;

msg.payload = {
  model: 'gpt-4',
  temperature: 0.7,
  max_tokens: 300,
  messages: [
    { role: 'system', content: preText },
    { role: 'user', content: text },
  ],
};
return msg;
```

</details>

Az üzenetet egy sima POST kéréssel küldöm el a `https://api.openai.com/v1/chat/completions` címre, `Authorization: Bearer [token]` és `OpenAI-Organization: [org-id]` header-ekkel.

A választ egyszerű kinyerni, majd felolvastatni egy TTS-sel:

<details>
<summary>Kód</summary>
```javascript
msg.payload = msg.payload.choices[0].message.content;

return msg;

```
</details>

<a href="https://www.buymeacoffee.com/tolnai" target="_blank"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-blue.png" alt="Buy Me A Coffee" style="height: 60px !important;width: 217px !important;" ></a>
```
