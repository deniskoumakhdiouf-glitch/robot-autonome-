# robot-autonome-# Smart Navigation System
> Système de navigation embarqué — Arduino Uno + HC-SR04 + OLED SSD1306  
> Projet alternance ingénieur embarqué

---

##  Présentation

Ce projet implémente un système de mesure de distance en temps réel avec affichage local et transmission des données via UART. Il démontre la maîtrise des protocoles de communication embarqués (I²C, UART, GPIO) ainsi que l'architecture logicielle d'un système embarqué propre.

---

##  Objectifs du projet

- Mesurer une distance via capteur ultrason HC-SR04
- Afficher la distance en temps réel sur écran OLED SSD1306 (I²C)
- Transmettre les données de debug via UART (Serial Monitor / PC)
- Structurer le code en couches logicielles indépendantes
- Réaliser un schéma PCB professionnel sous KiCad

---

##  Matériel utilisé

| Composant | Référence | Rôle |
|---|---|---|
| Microcontrôleur | Arduino Uno R3 (ATmega328P) | MCU principal |
| Capteur distance | HC-SR04 | Mesure ultrason 2cm–400cm |
| Écran | OLED SSD1306 128×64 | Affichage I²C (0x3C) |
| Résistances | R1, R2 — 4.7kΩ | Pull-up I²C SDA/SCL |
| Condensateurs | C1 100nF, C2 10µF | Découplage alimentation |

---

## ⚡ Architecture système

```
            ┌──────────────────┐
            │   Arduino Uno    │
            │   ATmega328P     │
            └──┬───────────┬───┘
       I²C     │           │  GPIO
  ┌────────┐   │           │   ┌────────────┐
  │  OLED  │   │           │   │  HC-SR04   │
  │SSD1306 │   │           │   │  Ultrason  │
  └────────┘   │           │   └────────────┘
               │ UART
          ┌────┴────┐
          │  PC log │
          │ 115200  │
          └─────────┘
```

---

##  Schéma de câblage

### Alimentation
| Arduino | Composant | Pin |
|---|---|---|
| +5V | OLED VCC | Pin 1 |
| +5V | HC-SR04 VCC | Pin 1 |
| GND | OLED GND | Pin 2 |
| GND | HC-SR04 GND | Pin 4 |

### I²C — OLED SSD1306
| Arduino | Signal | OLED | Rôle |
|---|---|---|---|
| A4 (pin 31) | SDA | Pin 3 | Données I²C |
| A5 (pin 32) | SCL | Pin 4 | Horloge I²C |

> Résistances pull-up : R1 4.7kΩ sur SDA, R2 4.7kΩ sur SCL (entre +5V et le fil)

### GPIO — HC-SR04
| Arduino | Signal | HC-SR04 | Rôle |
|---|---|---|---|
| D9 (pin 24) | TRIG | Pin 2 | Déclenche la mesure |
| D10 (pin 25) | ECHO | Pin 3 | Retourne la durée du signal |

### UART — Debug PC
| Signal | Connexion | Baud rate |
|---|---|---|
| TX/RX | USB natif ATmega16U2 | 115 200 |

---

##  Architecture logicielle

```
smart-nav/
├── firmware/
│   ├── smart_nav.ino       ← Boucle principale
│   ├── sensor.h/.cpp       ← Couche capteur HC-SR04
│   ├── display.h/.cpp      ← Couche affichage OLED
│   └── uart_log.h          ← Macros de debug UART
├── hardware/
│   ├── smart_nav.kicad_sch ← Schéma KiCad
│   ├── smart_nav.kicad_pcb ← PCB KiCad
│   └── gerbers/            ← Fichiers fabrication
├── docs/
│   └── connexions.txt      ← Tableau de câblage
└── README.md
```

### Couches logicielles

```
┌─────────────────────────────┐
│        smart_nav.ino        │  ← Orchestration
├──────────────┬──────────────┤
│  sensor.cpp  │ display.cpp  │  ← Drivers
├──────────────┴──────────────┤
│          uart_log.h         │  ← Debug
├─────────────────────────────┤
│    Arduino HAL (Wire, Serial│  ← Hardware
└─────────────────────────────┘
```

---

## 💻 Code principal

```cpp
// smart_nav.ino
#include "sensor.h"
#include "display.h"
#include "uart_log.h"

void setup() {
  Serial.begin(115200);
  display_init();
  sensor_init();
  LOG_INFO("Smart Navigation System démarré");
}

void loop() {
  float distance = sensor_get_distance_cm();

  display_show_distance(distance);
  LOG_INFO("Distance: %.1f cm", distance);

  if (distance < 10.0f) {
    LOG_WARN("Obstacle proche !");
  }

  delay(100);
}
```

```cpp
// sensor.cpp — mesure HC-SR04
float sensor_get_distance_cm() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH);
  return (duration * 0.034f) / 2.0f;
}
```

```cpp
// uart_log.h — macros de debug
#define LOG_INFO(msg, ...) \
  Serial.printf("[INFO] " msg "\n", ##__VA_ARGS__)

#define LOG_WARN(msg, ...) \
  Serial.printf("[WARN] " msg "\n", ##__VA_ARGS__)
```

---

##  Bibliothèques Arduino

| Bibliothèque | Usage |
|---|---|
| `Wire.h` | I²C (intégrée Arduino) |
| `Adafruit_SSD1306` | Driver OLED |
| `Adafruit_GFX` | Graphiques OLED |

Installation via Arduino IDE :
`Outils → Gérer les bibliothèques → Adafruit SSD1306`

---

##  Outils utilisés

| Outil | Usage |
|---|---|
| KiCad 8 | Schéma + PCB |
| Arduino IDE | Développement firmware |
| Git | Gestion de version |
| Serial Monitor | Debug UART 115200 |

---

## Compétences démontrées

- Protocole I²C (maître/esclave, adressage 0x3C)
- Protocole UART (debug série 115200 baud)
- GPIO (sortie TRIG, entrée ECHO avec pulseIn)
- Architecture firmware en couches
- Schéma électrique KiCad professionnel
- Documentation technique complète

