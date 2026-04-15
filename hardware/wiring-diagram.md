# GPIO Pinout and Wiring Diagram

## Raspberry Pi 4 GPIO Header

```
                    3V3  (1)  (2)  5V
                  GPIO2  (3)  (4)  5V      ← Power for breadboard
                  GPIO3  (5)  (6)  GND     ← GND for breadboard
    DHT22 DATA →  GPIO4  (7)  (8)  GPIO14  ← LED
                    GND  (9)  (10) GPIO15  ← Relay IN
  HC-SR04 TRIG → GPIO17 (11)  (12) GPIO18
  HC-SR04 ECHO → GPIO27 (13)  (14) GND
                 GPIO22 (15)  (16) GPIO23
                    3V3 (17)  (18) GPIO24
                 GPIO10 (19)  (20) GND
                  GPIO9 (21)  (22) GPIO25
                 GPIO11 (23)  (24) GPIO8
                    GND (25)  (26) GPIO7
                  GPIO0 (27)  (28) GPIO1
                  GPIO5 (29)  (30) GND
                  GPIO6 (31)  (32) GPIO12
                 GPIO13 (33)  (34) GND
                 GPIO19 (35)  (36) GPIO16
                 GPIO26 (37)  (38) GPIO20
                    GND (39)  (40) GPIO21
```

---

## Components

| Component | Description | Pins |
|-----------|-------------|------|
| DHT22 | Temperature & Humidity Sensor | VCC, DOUT, GND |
| HC-SR04 | Ultrasonic Distance Sensor | VCC, TRIG, ECHO, GND |
| LED | Status Indicator | Anode (+), Cathode (-) |
| Relay | 5V Relay Module | VCC, IN, GND |

---

## Wiring Connections

### 1. DHT22 (Direct connection, no breadboard needed)

```
DHT22 Module         Raspberry Pi
─────────────        ─────────────
VCC          ───────► Pin 1 (3.3V)
DOUT         ───────► Pin 7 (GPIO4)
GND          ───────► Pin 9 (GND)
```

### 2. Breadboard Power Rails

```
Raspberry Pi         Breadboard
─────────────        ─────────────
Pin 2 (5V)   ───────► Red rail (+)
Pin 6 (GND)  ───────► Blue rail (-)
```

### 3. LED with 220Ω Resistor

```
        220Ω
Pin 8 ──/\/\/──┬── LED (+) Anode (long leg)
               │
               └── LED (-) Cathode (short leg) ── GND
```

**On breadboard:**
```
Row   a   b   c   d   e       f   g   h   i   j
─────────────────────────────────────────────────
 5   [=== 220Ω resistor ===]
     ↑
     │ jumper to Pin 8 (GPIO14)
     
10        [LED anode]                 
11        [LED cathode]────────────► jumper to GND rail
```

### 4. HC-SR04 with Voltage Divider

⚠️ **IMPORTANT:** ECHO returns 5V! Voltage divider required for 3.3V GPIO!

```
HC-SR04              Breadboard/Pi
─────────────        ─────────────
VCC          ───────► Red rail (5V)
GND          ───────► Blue rail (GND)
TRIG         ───────► Pin 11 (GPIO17)
ECHO         ───┬───► 1kΩ resistor
                │
                ├───► to Pin 13 (GPIO27)
                │
               2.2kΩ
                │
                └───► GND
```

**Voltage Divider Calculation:**
```
ECHO (5V) ────┬──── 1kΩ ────┬──── GPIO27 (Pin 13)
              │             │
              │            2.2kΩ
              │             │
              └─────────────┴──── GND

Calculation: Vout = 5V × 2.2kΩ / (1kΩ + 2.2kΩ) = 3.44V ✓
```

**On breadboard:**
```
Row   a   b   c   d   e       f   g   h   i   j
─────────────────────────────────────────────────
20   [===== 1kΩ resistor =====]
     ↑                       ↑
     │                       └── jumper from HC-SR04 ECHO
     │
25   [=2.2kΩ=]───────────────────► to GND rail
     ↑
     └── jumper to Pin 13 (GPIO27)
```

### 5. Relay Module

```
Relay Module         Raspberry Pi / Breadboard
─────────────        ──────────────────────────
VCC          ───────► Red rail (5V)
GND          ───────► Blue rail (GND)
IN           ───────► Pin 10 (GPIO15)
```

---

## Complete Breadboard Layout

```
                    BREADBOARD
    ─────────────────────────────────────────
    (+) ○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○ (+)  ← 5V from Pin 2
    (-) ○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○ (-)  ← GND from Pin 6
    
        a b c d e     f g h i j
     1  ○ ○ ○ ○ ○     ○ ○ ○ ○ ○
     2  ○ ○ ○ ○ ○     ○ ○ ○ ○ ○
     3  ○ ○ ○ ○ ○     ○ ○ ○ ○ ○
     4  ○ ○ ○ ○ ○     ○ ○ ○ ○ ○
     5  [R 220Ω ]     ○ ○ ○ ○ ○  ← LED resistor
     6  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
     7  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
     8  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
     9  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
    10  └─[+LED-]     ○ ○ ○ ○ ○  ← LED
    11    ○ │ ○ ○     ○ ○ ○ ○ ○
    12    ○ └─────────────────────► to GND rail
    ...
    20  [=1kΩ==]─────────────────► from HC-SR04 ECHO
    21  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
    22  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
    23  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
    24  │ ○ ○ ○ ○     ○ ○ ○ ○ ○
    25  ├─[2.2kΩ]─────────────────► to GND rail
        │
        └─────────────────────────► to Pin 13 (GPIO27)

    (-) ○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○ (-)
    (+) ○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○○ (+)
```

---

## OpenPLC I/O Mapping

| Type | Address | GPIO | Component |
|------|---------|------|-----------|
| Digital Input | %IX0.2 | GPIO4 | DHT22 (via Python) |
| Digital Output | %QX0.0 | GPIO14 | LED |
| Digital Output | %QX0.1 | GPIO15 | Relay |
| Digital Input | %IX0.3 | GPIO17 | HC-SR04 TRIG |
| Digital Input | %IX0.4 | GPIO27 | HC-SR04 ECHO |

---

## Test Scripts

### Test DHT22
```bash
python3 << 'EOF'
import adafruit_dht
import board
import time

dht = adafruit_dht.DHT22(board.D4)
time.sleep(2)

try:
    print(f"Temperature: {dht.temperature:.1f}°C")
    print(f"Humidity: {dht.humidity:.1f}%")
except Exception as e:
    print(f"Error: {e}")
finally:
    dht.exit()
EOF
```

### Test LED
```bash
python3 << 'EOF'
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setup(14, GPIO.OUT)

for i in range(5):
    GPIO.output(14, GPIO.HIGH)
    time.sleep(0.5)
    GPIO.output(14, GPIO.LOW)
    time.sleep(0.5)

GPIO.cleanup()
print("LED test complete!")
EOF
```

### Test HC-SR04
```bash
python3 << 'EOF'
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
TRIG = 17
ECHO = 27

GPIO.setup(TRIG, GPIO.OUT)
GPIO.setup(ECHO, GPIO.IN)

GPIO.output(TRIG, False)
time.sleep(0.5)

GPIO.output(TRIG, True)
time.sleep(0.00001)
GPIO.output(TRIG, False)

while GPIO.input(ECHO) == 0:
    pulse_start = time.time()

while GPIO.input(ECHO) == 1:
    pulse_end = time.time()

distance = (pulse_end - pulse_start) * 17150
print(f"Distance: {distance:.2f} cm")

GPIO.cleanup()
EOF
```

### Test Relay
```bash
python3 << 'EOF'
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)
GPIO.setup(15, GPIO.OUT)

print("Relay ON")
GPIO.output(15, GPIO.HIGH)
time.sleep(2)

print("Relay OFF")
GPIO.output(15, GPIO.LOW)

GPIO.cleanup()
print("Relay test complete!")
EOF
```

---

## Safety Guidelines

⚠️ **WARNING:**

1. **Always power off Pi before connecting/disconnecting wires**
2. **Never connect 5V directly to GPIO** — max 3.3V!
3. **HC-SR04 ECHO requires voltage divider** — or you'll damage GPIO!
4. **Double-check connections before powering on**

## Simulation

Before physical wiring, test in [Wokwi](https://wokwi.com/) or [Diode](https://www.withdiode.com/).
