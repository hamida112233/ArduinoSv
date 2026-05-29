# Node-RED

## Wat is Node-RED?

Node-RED is een visuele editor om **dataflows** te bouwen door nodes te slepen en te verbinden.

* NPM-pakket op basis van Node.js / JavaScript
* Draait op Windows, Mac, Linux, Raspberry Pi, Docker, Cloud
* Laagdrempelig: ideaal voor complete IoT-prototypes

## Wat kan Node-RED?

* Luisteren naar MQTT / API berichten
* HTTP GET/POST verwerken
* Data parsen (JSON, XML, YAML)
* Data opslaan in database of filesystem
* Dashboard bouwen

## Interface

| Onderdeel | Functie |
|---|---|
| **Palette** | Lijst van beschikbare nodes |
| **Workspace** | Hier bouw je flows |
| **Sidebar** | Debug-output en node-info |
| **Deploy** | Flow activeren / herstart |

## Core nodes

### Inject

* Triggert de volgende node handmatig of periodiek
* Kan het msg-object opbouwen (payload, topic, …)
* Handig voor testen van flows
* Ondersteunt JSONata-expressies voor dynamische data

### Debug

* Toont het msg-object in de sidebar
* Kan aan/uit gezet worden zonder de flow aan te passen

### Function

* JavaScript om het msg-object dynamisch aan te passen
* Toegang tot node-, flow- en global context
* Geschikt voor berekeningen en stringmanipulatie

### Change

* "Eenvoudige" functie-node via configuratie (geen code nodig)
* Set, move of delete velden in het msg-object

### Switch

* Routeert berichten naar verschillende outputs op basis van regels
* Ondersteunt: Value / Sequence / Expression / Otherwise

## Message object

Het msg-object is een JavaScript-object dat tussen verbonden nodes wordt doorgegeven.

| Veld | Beschrijving |
|---|---|
| `msg._msgid` | Unieke trace-ID door de flow |
| `msg.payload` | De eigenlijke data |
| `msg.topic` | MQTT-topic of ander label |

## Context

| Type | Scope |
|---|---|
| **Node** | Alleen zichtbaar binnen de node zelf |
| **Flow** | Gedeeld door alle nodes op dezelfde flow-tab |
| **Global** | Gedeeld over de hele Node-RED server |

{% hint style="info" %}
Context is handig om de huidige meting te vergelijken met de vorige zonder een database te gebruiken.
{% endhint %}

## Plugins installeren

Via **Hamburger → Manage palette → Install**

Nuttige plugins:

* `node-red-contrib-influxdb` — schrijven naar InfluxDB
* `@flowfuse/node-red-dashboard` — dashboard (gebruik **niet** de verouderde `node-red-dashboard`)

## Function node → InfluxDB (voorbeeld)

```javascript
// Zet MQTT JSON om naar InfluxDB-formaat
const payload = msg.payload;

msg.payload = [
  {
    measurement: "temp-extended",
    tags: {},
    fields: {
      temperature: payload.temperature,
      humidity:    payload.humidity,
      latitude:    payload.latitude  || 0,
      longitude:   payload.longitude || 0
    },
    timestamp: new Date(payload.timestamp)
  }
];

return msg;
```

## Inject node — JSONata voor InfluxDB

```javascript
(
  $minTemp := 18.5; $maxTemp := 24.0;
  $minHum  := 35;   $maxHum  := 60;
  $temp := $round($minTemp + $random() * ($maxTemp - $minTemp), 1);
  $hum  := $floor($minHum  + $random() * ($maxHum  - $minHum));
  [
    { "temperatuur": $temp, "vochtigheid": $hum },
    { "kamer": "woonkamer", "sensor": "DHT22" }
  ]
)
```
