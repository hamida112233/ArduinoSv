# GPIO

## Wat is een GPIO?

**General Purpose Input/Output** — maar niet altijd echt "general purpose".

* Kan digitaal of analoog werken, als input of output
* Functionaliteit bepaald door de **GPIO Multiplexing Table** van de MCU
* Module of dev-board kan pins verder vastleggen (bv. USB-pins)

**Vaste pins op FireBeetle ESP32 (niet muxen):**

* IO0/D5, IO1/TXD, IO3/RX → bezet door USB
* Strapping pins: GPIO0 (pull-up), GPIO2 (pull-down), GPIO5 (pull-up)

## Digitale output

```cpp
pinMode(pin, OUTPUT);
digitalWrite(pin, HIGH);  // 3.3V
digitalWrite(pin, LOW);   // 0V
```

**Use cases:** LED aansturen, MOSFET schakelen voor DC-motor, trigger-signalen sturen, blokgolf genereren.

## Digitale input

```cpp
pinMode(pin, INPUT);
int state = digitalRead(pin);  // geeft 0 of 1
```

**Use cases:** drukknop uitlezen (pull-up of pull-down), rotary encoder, echo meten van ultrasoon.

## Spanningsniveaus (3.3V GPIO)

| Symbool | Betekenis | Waarde |
|---|---|---|
| VOL | Max. uitgangsspan. voor LAAG | 0.4V |
| VIL | Max. ingangsspan. nog als LAAG | 0.8V |
| Vt | Drempelspanning HIGH/LOW | 1.5V |
| VIH | Min. ingangsspan. nog als HOOG | 2.0V |
| VOH | Min. uitgangsspan. voor HOOG | 2.4V |

{% hint style="info" %}
**Noise margin:** de ruimte tussen VOL en VIL (LOW kant) en tussen VIH en VOH (HIGH kant) beschermt tegen valse detecties door ruis.
{% endhint %}

## Button debounce

Mechanische knoppen oscilleren bij contact → meerdere pulsen worden geregistreerd.

**Hardware oplossing:** RC-filter + Schmitt-trigger inverter

**Software oplossing (met millis):**

```cpp
const int btnPin = 2;
const int ledPin = 13;
unsigned long lastDebounce = 0;
const unsigned long debounceDelay = 50;
int lastBtnState = LOW;
int btnState, ledState = HIGH;

void setup() {
  pinMode(btnPin, INPUT);
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, ledState);
}

void loop() {
  int reading = digitalRead(btnPin);
  if (reading != lastBtnState)
    lastDebounce = millis();
  if ((millis() - lastDebounce) > debounceDelay) {
    if (reading != btnState) {
      btnState = reading;
      if (btnState == HIGH)
        ledState = !ledState;
    }
  }
  digitalWrite(ledPin, ledState);
  lastBtnState = reading;
}
```

{% hint style="warning" %}
Gebruik nooit `delay()` als debounce in een echte applicatie — dit blokkeert de volledige loop.
{% endhint %}

## Analoge input (ADC)

```cpp
int waarde = analogRead(pin);  // 0 tot 2^N - 1
```

| Board | Resolutie | VREF | Stapgrootte |
|---|---|---|---|
| Arduino Nano | 10-bit (1024 stappen) | 5V | ≈ 4.88 mV |
| ESP32 | 12-bit (4096 stappen) | 3.3V | ≈ 0.81 mV |

{% hint style="warning" %}
Resolutie ≠ nauwkeurigheid. De ADC van de ESP32 is gevoelig voor ruis en niet-lineariteit, zeker boven 3.1V. Gebruik een externe referentie voor nauwkeurige metingen.
{% endhint %}
