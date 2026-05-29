# InfluxDB

## Wat is InfluxDB?

InfluxDB is een gespecialiseerde **time-series database (TSDB)**.

* Snelle opslag en queries op tijdreeksen
* Ingebouwde data-retentie en aggregatie
* Ideaal voor IoT, server-monitoring en sensordaten

**Voorbeelden van tijdreeksen:** temperatuurmetingen, hartslagen, aandelenkoersen, neerslagdata, server-metrics.

## Structuur

```
Organization
└── Bucket
    └── Measurement
        ├── Tags      (geïndexeerde metadata)
        ├── Fields    (meetwaarden)
        └── Timestamp
```

| Niveau | Vergelijkbaar met |
|---|---|
| Organization | Database-server |
| Bucket | Database |
| Measurement | Tabel |
| Tags + Fields | Kolommen |

## Line protocol

Zo schrijf je data naar InfluxDB:

```
measurement,tag1=val1,tag2=val2 field1=23.5,field2=100i 1714382400000000000
```

| Onderdeel | Rol | Type | Verplicht? |
|---|---|---|---|
| `measurement` | Naam van de meting | string | Ja |
| `tags` | Metadata, worden geïndexeerd (WHERE / GROUP BY) | altijd string | Nee |
| `fields` | Werkelijke meetwaarden, niet geïndexeerd | float / int / string / bool | Min. 1 |
| `timestamp` | Unix-tijd in nanoseconden | int64 | Nee* |

{% hint style="warning" %}
\* Timestamp weglaten → InfluxDB gebruikt de servertijd. Dit kan misleidend zijn door netwerklatentie — gebruik liever een timestamp van het apparaat zelf.
{% endhint %}

**Syntaxregels:**

* Spatie scheidt de tag-set van de field-set
* Komma scheidt tags/fields onderling (geen spatie!)
* Integer field: voeg `i` toe → `field=100i`

## Timestamping — best practices

| Methode | Nauwkeurigheid | Opmerking |
|---|---|---|
| Servertijd | Variabel | Misleidend bij hoge latency of batches |
| NTP | ±1–50 ms | Afhankelijk van netwerkkwaliteit |
| GPS | Nanoseconde | Werkt zonder internet |
| 4G/LTE | Voldoende voor logging | Minder stabiel |

## Token aanmaken

1. Ga naar **Load Data → API Tokens → Generate API Token**
2. Kies **Custom API Token** voor beperkte rechten
3. Selecteer Read en/of Write per bucket
4. Kopieer het token onmiddellijk

{% hint style="danger" %}
Na het aanmaken is het token **niet meer zichtbaar**. Kopieer het direct en sla het op een veilige plek op.
{% endhint %}

## InfluxDB koppelen aan Node-RED

1. Installeer `node-red-contrib-influxdb` via de palette manager
2. Voeg een **influxdb out**-node toe aan de flow
3. Maak een nieuwe server aan: versie 2.0, URL, Token
4. Vul Organization, Bucket en Measurement in
5. Stel Time Precision in (ms of s aanbevolen)

## Voorbeelden — Line protocol

**Smart home klimaat:**
```
klimaat,kamer=woonkamer,sensor=DHT22 temperatuur=21.5,luchtvochtigheid=45.2 1714382400000000000
```

**Weerstation:**
```
wind,stad=Antwerpen,richting=NW snelheid=15.6,rukwind=22.1 1714382500000000000
```

**Systeemmonitoring:**
```
cpu,host=server01,core=0 usage_user=12.4,usage_system=2.1,usage_idle=85.5 1714382600000000000
```
