# PërKompanion — Teensy 4.1 Pinout

> Assignation GPIO complète du Teensy 4.1
> Document technique — **v0.9** — avril 2026

---

## Préambule

Ce document définit l'**assignation des pins du Teensy 4.1** pour tous les périphériques de PërKompanion v0.8.

**Principe directeur v0.8** : **séparation par fonction**, pas par disponibilité de pin. Chaque pin a une fonction dédiée et stable, pas de déplacement forcé par conflit. La matrice pad est déplacée sur MCP23S17 pour libérer entièrement les pins 2-9 (sensibles SAI1/SAI2).

Référence : pjrc.com/store/teensy41.html

---

## 1. Vue d'ensemble

Le Teensy 4.1 a **55 GPIO digital utilisables** et **2 bus I²S (SAI1 + SAI2)**.

### Architecture audio v0.8

**SAI1 (bus principal)** : 2 canaux in + 8 canaux out
- 4 canaux in (via 2× PCM1808) ← entrées Perkons V1-V4
- 8 canaux out (via 4× PCM5102A) → Master + V1 + V2 + V3

**SAI2 (bus secondaire)** : 4 canaux out
- 4 canaux out (via 2× PCM5102A) → V4 + V5-V8 mixées

**Total** : 4 in + 12 out = **16 canaux audio**

### Allocation fonctionnelle

| Fonction | Pins Teensy directs | Méthode |
|----------|---------------------|---------|
| **Matrice pad 8×4** | 0 | Sur MCP23S17 (chaîne dédiée) |
| **Encodeurs (82 total)** | 0 | Sur MCP23S17 (16 chips dédiés) |
| **Boutons (~68 total)** | 0 | Sur MCP23S17 (3 chips dédiés) |
| **LEDs status** | 0 | Sur MCP23S17 (1 chip dédié) |
| **OLEDs voix (8× SH1107 SPI)** | 10 | SPI partagé + 8 CS individuels |
| **OLED master SSD1306 SPI 128×64** | 1 CS | SPI (bus partagé avec voix) |
| **NeoPixel pad RGB (abandonné v0.8 final)** | 0 | Retiré — remplacé par LEDs 3mm dans switches MX |
| **MIDI IN + OUT (DIN)** | 2 | Serial1 |
| **Bus SPI MCP23S17 (3 chaînes)** | 6 | SCK + MOSI + MISO + 3 CS |
| **SAI1 audio** | 7 | MCLK + BCLK + LRCLK + 2 RX + 2 TX |
| **SAI2 audio** | 5 | MCLK2 + BCLK2 + LRCLK2 + 2 TX |
| **USB série Pi** | 0 | USB natif |
| **Réserve** | ~5 | Extensions futures |

**Total utilisé** : ~45 pins
**Réserve** : ~10 pins libres

### 20 MCP23S17 sur 3 chaînes SPI

Distribution des 20 MCP23S17 en DIP-28 :

**Chaîne 1 (CS = pin 24)** — 8 MCP adresses 0-7 — encodeurs voix V1-V4 :
- MCP 1-2 : encodeurs V1 + V2 (16 encodeurs × 3 pins = 48 GPIO)
- MCP 3-4 : encodeurs V3 + V4 (16 encodeurs × 3 pins = 48 GPIO)
- MCP 5-8 : encodeurs V5-V8 (32 encodeurs × 3 pins = 96 GPIO, étalés sur 4 MCP)

**Chaîne 2 (CS = pin 25)** — 8 MCP adresses 0-7 — encodeurs contextuels + master + boutons :
- MCP 9-10 : encodeurs écran E1-E8 + master (~18 encodeurs × 3 = 54 GPIO)
- MCP 11 : matrice pad 8 cols + 4 rows = 12 GPIO
- MCP 12 : boutons globaux (5) + mode pad (5) + voix pad (8) + boutons écran B1-B6 (6) + MODE GLOBAL (4) = 28 boutons
- MCP 13 : boutons voix tête (8) + LEDs voix tête (8) + spare = 16 GPIO
- MCP 14-15 : LEDs status (~16) + extensions

**Chaîne 3 (CS = pin 26)** — 4 MCP adresses 0-3 — réserve et extensions :
- MCP 16 : LED dédiées (activité, boutons globaux avec LED, 16 GPIO)
- MCP 17 : spare
- MCP 18-20 : réserve pour extensions v2+

**Total** : 20 MCP23S17 répartis sur 5 perfboards (4 MCP par perfboard).

---

## 2. Allocation complète des pins Teensy 4.1

### Tableau de référence complet

| Pin | Fonction | Catégorie |
|-----|----------|-----------|
| **0** | MIDI RX (Serial1) | MIDI |
| **1** | MIDI TX (Serial1) | MIDI |
| **2** | SAI2_TXD0 → PCM5102A #5 (V4 L+R) | Audio SAI2 |
| **3** | SAI2_LRCLK (L/R clock SAI2) | Audio SAI2 |
| **4** | SAI2_BCLK (Bit clock SAI2) | Audio SAI2 |
| **5** | SAI2_TXD1 → PCM5102A #6 (V5-V8 L+R) | Audio SAI2 |
| **6** | SAI1_RXD1 ← PCM1808 #2 (V3+V4) | Audio SAI1 RX |
| **7** | SAI1_TXD0 → PCM5102A #1 (Master L+R) | Audio SAI1 TX |
| **8** | SAI1_RXD0 ← PCM1808 #1 (V1+V2) | Audio SAI1 RX |
| **9** | SAI1_TXD2 → PCM5102A #3 (V2 L+R) | Audio SAI1 TX |
| **10** | SAI1_TXD3 → PCM5102A #4 (V3 L+R) | Audio SAI1 TX |
| **11** | SPI MOSI (OLEDs + MCP) | SPI |
| **12** | SPI MISO (MCP) | SPI |
| **13** | SPI SCK (OLEDs + MCP) | SPI |
| **14** | OLEDs DC partagé | OLEDs |
| **15** | OLEDs RES partagé | OLEDs |
| **16** | OLED V1 CS | OLEDs |
| **17** | OLED V2 CS | OLEDs |
| **18** | I²C SDA (**libéré v0.9**, réserve extensions futures) | - |
| **19** | I²C SCL (**libéré v0.9**, réserve extensions futures) | - |
| **20** | SAI1_LRCLK | Audio SAI1 |
| **21** | SAI1_BCLK | Audio SAI1 |
| **22** | OLED V3 CS | OLEDs |
| **23** | SAI1_MCLK | Audio SAI1 |
| **24** | MCP23S17 CS chaîne 1 | MCP |
| **25** | MCP23S17 CS chaîne 2 | MCP |
| **26** | MCP23S17 CS chaîne 3 | MCP |
| **27** | **OLED master SSD1306 SPI CS** (v0.9) | OLEDs |
| **28** | OLED V5 CS | OLEDs |
| **29** | OLED V6 CS | OLEDs |
| **30** | OLED V7 CS | OLEDs |
| **31** | OLED V8 CS | OLEDs |
| **32** | SAI1_TXD1 → PCM5102A #2 (V1 L+R) | Audio SAI1 TX |
| **33** | SAI2_MCLK | Audio SAI2 |
| **34** | **Réserve** (ancien NeoPixel DATA, retiré v0.8 final) | - |
| **35-36** | Réserve SAI2 RX (v3+ si besoin 4 in audio supplémentaires) | - |
| **37** | OLED V4 CS | OLEDs |
| **38-41** | Réserve (4 pins libres pour extensions) | - |

### Pins sensibles à ne pas toucher

- **Pins 2-10** : **entièrement dédiées à l'audio** (SAI1 + SAI2). Ne rien y mettre d'autre. C'est la clé de la stabilité audio.
- **Pins 11, 12, 13** : bus SPI partagé. OK pour partager entre OLEDs et MCP tant que les CS sont bien gérés.
- **Pins 18, 19** : I²C. **Libérés en v0.9** (OLED master passé en SPI). Disponibles pour extensions futures (autres capteurs, modules I²C supplémentaires). Le TCA9548A reste dans la BOM en réserve mais n'est pas utilisé en v0.9.

### Résolution des conflits d'origine

**Conflit Audio Shield vs matrice pad (v0.7/v0.8 initial)** :
- Résolu en déplaçant la matrice pad sur MCP23S17
- Libère pins 2-9 entièrement pour SAI1/SAI2
- Aucun pin "multifonction" dans le pinout v0.8 final

**Conflit pin 23 (MCLK vs OLED V4 CS)** :
- OLED V4 CS déplacé pin 37
- Pin 23 = SAI1_MCLK uniquement

---

## 3. Bus SPI — câblage

### Principe

**Un seul bus SPI physique** est partagé entre les OLEDs et les MCP23S17.

**Signaux partagés** :
- **SCK** : pin 13
- **MOSI** : pin 11
- **MISO** : pin 12

**CS individuels** :
- 8 CS pour les OLEDs voix (pins 16, 17, 22, 28, 29, 30, 31, 37)
- 3 CS pour les chaînes MCP23S17 (pins 24, 25, 26)

**Vitesse SPI** :
- OLEDs SH1107 : jusqu'à 20 MHz
- MCP23S17 : limite 10 MHz
- **Compromis** : bus à 10 MHz pour compatibilité universelle, ou jonglage de vitesse via `SPI.setClockDivider()` avant chaque transaction

**Librairies** :
- OLEDs : Adafruit_SH110X ou u8g2
- MCP23S17 : MCP23S17 library (par Pedro Rodrigues ou équivalent)

### Organisation des 20 MCP23S17

#### Perfboard 1 (4 MCP, CS = pin 24 Teensy, chaîne 1)

- MCP 1 (addr 0x00) : encodeurs V1 + partiel V2
- MCP 2 (addr 0x01) : partiel V2 + V3
- MCP 3 (addr 0x02) : V3 + partiel V4
- MCP 4 (addr 0x03) : partiel V4 + partiel V5

Câblage :
- **VCC** (3.3V) partagé entre 4 MCP
- **GND** partagé
- **SCK, MOSI, MISO** bus SPI partagé (1 fil chacun sur les 4 MCP)
- **CS** : 1 seul fil du Teensy pin 24 vers les 4 MCP (partagé)
- **A0, A1, A2** : strappé différemment sur chaque MCP (adresses 0-3)
- **RESET** : tiré à VCC via R10K (partagé, pull-up)
- **INT_A / INT_B** : optionnel, pour interruption de changement

#### Perfboard 2 (4 MCP, CS = pin 24, adresses 4-7)

Continue la chaîne 1 :
- MCP 5 (addr 0x04) à MCP 8 (addr 0x07)
- Encodeurs V5-V8

#### Perfboard 3 (4 MCP, CS = pin 25, adresses 0-3, chaîne 2)

- MCP 9-10 : encodeurs contextuels E1-E8 + master
- MCP 11 : matrice pad 8×4 (12 GPIO) + 4 GPIO spare
- MCP 12 : boutons globaux + mode pad + voix pad + boutons écran + MODE GLOBAL

#### Perfboard 4 (4 MCP, CS = pin 25, adresses 4-7)

- MCP 13 : boutons voix tête (8) + LEDs voix tête (8)
- MCP 14 : LEDs status activité (16)
- MCP 15 : LEDs boutons globaux avec LED (~8) + spare
- MCP 16 : spare / extensions

#### Perfboard 5 (4 MCP, CS = pin 26, adresses 0-3, chaîne 3 réserve)

- MCP 17-20 : **réserve** pour extensions futures (encodeurs supplémentaires, boutons, LEDs, capteurs, etc.)

---

## 4. Matrice pad 8×4 via MCP23S17

### Principe

La matrice pad est entièrement sur **MCP 11** (1 seul MCP) :
- 8 pins MCP = colonnes (GPIOA 0-7)
- 4 pins MCP = rangées (GPIOB 0-3)
- 4 pins MCP restants = spare

### Câblage

```
MCP 11 (perfboard 3, chaîne 2, addr 0x02)
├── GPIOA 0-7 : colonnes matrice pad (outputs)
├── GPIOB 0-3 : rangées matrice pad (inputs pull-up)
└── GPIOB 4-7 : spare

Matrice 8×4 :
  32 switches Cherry MX
  32 diodes 1N4148 (anti-ghosting)
```

**Scan rate** : ~1 kHz via polling du MCP depuis le Teensy (lecture de tous les ports en ~100 µs). Suffisant pour une détection réactive (<1ms de latence perçue).

**Avantage** : les 12 pins Teensy qui auraient été consommés par la matrice pad directe sont **libérés** pour l'audio SAI1/SAI2.

---

## 5. OLEDs SPI (8× SH1107 128×128)

### Signaux partagés

- **VCC** : 3.3V
- **GND** : masse commune
- **SCK** : pin 13 (bus SPI)
- **MOSI** : pin 11 (bus SPI)
- **DC** : pin 14 (Data/Command partagé)
- **RES** : pin 15 (Reset partagé, pulse au démarrage)

### CS individuels (8 pins Teensy)

| OLED | Voix | CS Teensy | Perfboard emplacement |
|------|------|-----------|-----------------------|
| OLED 0 | V1 | pin 16 | panneau, en tête V1 |
| OLED 1 | V2 | pin 17 | panneau, en tête V2 |
| OLED 2 | V3 | pin 22 | panneau, en tête V3 |
| OLED 3 | V4 | pin 37 | panneau, en tête V4 |
| OLED 4 | V5 | pin 28 | panneau, en tête V5 |
| OLED 5 | V6 | pin 29 | panneau, en tête V6 |
| OLED 6 | V7 | pin 30 | panneau, en tête V7 |
| OLED 7 | V8 | pin 31 | panneau, en tête V8 |

**Total pins pour 8 OLEDs** : 6 partagés (SCK, MOSI, DC, RES, VCC, GND) + 8 CS = **14 pins**.

### Performance

- Vitesse SPI : 10 MHz (limité par MCP partageant le bus, sinon 20 MHz possible)
- Temps de refresh complet d'un OLED 128×128 : ~15 ms à 10 MHz
- **8 OLEDs séquentiels** : 120 ms full refresh, soit **~8 fps full refresh**
- Avec dirty rect (ne rafraîchir que les pixels modifiés) : **50+ fps** facilement

---

## 6. OLED master SSD1306 128×64 SPI (v0.9)

### Choix v0.9 : tout en SPI

**v0.9** : l'OLED master passe du bus I²C (via TCA9548A) au **bus SPI partagé** avec les 8 OLEDs voix.

**Avantages** :
- **Architecture uniforme** : 1 seul type d'interface pour les 9 OLEDs, firmware simplifié
- **Bus I²C totalement libéré** : pins 18, 19 disponibles pour extensions futures
- **Actualisation plus rapide** (SPI 10-20 MHz vs I²C 400 kHz)
- **TCA9548A non nécessaire** (reste dans la BOM en réserve)

### Câblage OLED master

**Module OLED 1.3" 128×64 SSD1306 SPI 7-pin** :

| Broche module | Signal | Connecté à |
|---------------|--------|------------|
| GND | Masse | GND commun |
| VCC | Alimentation 3.3V | 3.3V rail |
| D0 (SCK) | Clock SPI | Pin 13 Teensy (partagé avec OLEDs voix) |
| D1 (MOSI) | Data SPI | Pin 11 Teensy (partagé avec OLEDs voix) |
| RES | Reset | Pin 15 Teensy (partagé avec OLEDs voix) |
| DC | Data/Command | Pin 14 Teensy (partagé avec OLEDs voix) |
| **CS** | Chip Select | **Pin 27 Teensy (dédié master)** |

**Câblage identique aux OLEDs voix**, seul le CS est dédié à l'OLED master.

### Firmware exemple

```cpp
#include <SPI.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define OLED_WIDTH    128
#define OLED_HEIGHT    64
#define OLED_RESET     15
#define OLED_DC        14
#define OLED_CS_MASTER 27

Adafruit_SSD1306 display_master(OLED_WIDTH, OLED_HEIGHT,
                                  &SPI, OLED_DC, OLED_RESET, OLED_CS_MASTER);

void setup() {
    SPI.begin();
    display_master.begin(SSD1306_SWITCHCAPVCC, 0);  // pas d'adresse I²C
    display_master.clearDisplay();
    display_master.setTextSize(2);
    display_master.setTextColor(WHITE);
    display_master.setCursor(0, 0);
    display_master.print("BPM: 140");
    display_master.display();
}
```

### TCA9548A — en réserve

Le module TCA9548A reste dans la BOM AliExpress comme **spare** pour extensions futures I²C éventuelles (capteurs, autres écrans, etc.), mais n'est pas utilisé en v0.9.

---

## 7. Audio — SAI1 + SAI2 câblage détaillé

### SAI1 (audio principal) — 4 modules audio

**Signaux horloges partagées** (connectés à chacun des 4 modules SAI1) :

| Signal | Pin Teensy | Vers modules |
|--------|-----------|--------------|
| SAI1_MCLK | 23 | PCM1808 #1, PCM1808 #2, PCM5102A #1-4 |
| SAI1_BCLK | 21 | idem |
| SAI1_LRCLK | 20 | idem |

**Signaux data** :

| Signal | Pin Teensy | Direction | Module |
|--------|-----------|-----------|--------|
| SAI1_RXD0 | 8 | Teensy ← | PCM1808 #1 (V1+V2) |
| SAI1_RXD1 | 6 | Teensy ← | PCM1808 #2 (V3+V4) |
| SAI1_TXD0 | 7 | Teensy → | PCM5102A #1 (Master) |
| SAI1_TXD1 | 32 | Teensy → | PCM5102A #2 (V1) |
| SAI1_TXD2 | 9 | Teensy → | PCM5102A #3 (V2) |
| SAI1_TXD3 | 10 | Teensy → | PCM5102A #4 (V3) |

### SAI2 (audio secondaire) — 2 modules audio

**Signaux horloges** (SAI2 a ses propres horloges indépendantes) :

| Signal | Pin Teensy | Vers modules |
|--------|-----------|--------------|
| SAI2_MCLK | 33 | PCM5102A #5, PCM5102A #6 |
| SAI2_BCLK | 4 | idem |
| SAI2_LRCLK | 3 | idem |

**Signaux data** :

| Signal | Pin Teensy | Direction | Module |
|--------|-----------|-----------|--------|
| SAI2_TXD0 | 2 | Teensy → | PCM5102A #5 (V4) |
| SAI2_TXD1 | 5 | Teensy → | PCM5102A #6 (V5-V8) |

### Alimentation modules audio

- **PCM1808** : 3.3V single supply (depuis rail Teensy 3.3V)
- **PCM5102A** : 3.3V ou 5V single supply (5V recommandé pour meilleure dynamique)
- **TPA6120 module MCU-612** : 5V single supply (doubleur interne génère ±V nécessaire)

**Découplage** : 100 nF + 10 µF/16V à proximité de chaque module.

### Firmware PJRC Audio Library

```cpp
#include <Audio.h>
#include <Wire.h>
#include <SPI.h>

// Entrées audio (SAI1 RX, 4 canaux via 2 PCM1808)
AudioInputI2SQuad       audio_in;

// DSP par voix (exemple V1)
AudioFilterStateVariable filter_v1;
AudioEffectWaveshaper   drive_v1;
AudioEffectFreeverb     reverb_v1;
AudioEffectDelayExternal delay_v1;
AudioAmplifier          out_v1;

// Mixer master
AudioMixer4             master_mixer;
AudioEffectFreeverb     master_reverb;
AudioEffectDynamics     master_comp;

// Sorties SAI1 (4 DAC = 8 canaux)
AudioOutputI2S          out_master;     // PCM5102A #1
AudioOutputI2S          out_v1_stereo;  // PCM5102A #2
AudioOutputI2S          out_v2_stereo;  // PCM5102A #3
AudioOutputI2S          out_v3_stereo;  // PCM5102A #4

// Sorties SAI2 (2 DAC = 4 canaux)
AudioOutputI2S2         out_v4_stereo;  // PCM5102A #5
AudioOutputI2S2         out_virt_stereo; // PCM5102A #6

// Connexions (exemple pour V1) :
AudioConnection patch_in_v1(audio_in, 0, filter_v1, 0);
AudioConnection patch_v1_1(filter_v1, 0, drive_v1, 0);
AudioConnection patch_v1_2(drive_v1, 0, reverb_v1, 0);
AudioConnection patch_v1_3(reverb_v1, 0, delay_v1, 0);
AudioConnection patch_v1_4(delay_v1, 0, out_v1, 0);
AudioConnection patch_v1_master(out_v1, 0, master_mixer, 0);
AudioConnection patch_v1_direct_l(out_v1, 0, out_v1_stereo, 0);
AudioConnection patch_v1_direct_r(out_v1, 0, out_v1_stereo, 1);
// Idem V2, V3, V4, V5-V8
// Master mixer → out_master
```

**Attention** : `AudioOutputI2S2` n'existe pas encore officiellement dans PJRC Audio Library pour Teensy 4.1. Il peut falloir customiser un objet ou utiliser `AudioOutputTDM2` / `AudioOutputI2SSlave2` selon versions. **À valider en Phase 2**.

**Alternative** : si SAI2 n'est pas exploitable aisément via PJRC Audio Library, **fallback à 4 PCM5102A sur SAI1 uniquement** (Master + V1 + V2 + V3, soit 8 canaux = 4 stéréos), et V4 + V5-V8 mixées avec Master dans le mix interne. À évaluer à l'implémentation.

---

## 8. LEDs 3mm dans switches MX (v0.9)

### Approche retenue

**Chaque switch Cherry MX2A Brown** dispose d'un **slot pour LED 3mm** intégré. La LED 3mm blanche s'insère dans ce logement et éclaire le keycap par dessous via le stem.

**Pas de NeoPixel** : le pin 34 Teensy est libéré. Le 74AHCT125N level shifter **n'est plus nécessaire** pour le pad (mais conservé dans la BOM comme spare logique général).

### Câblage LEDs 3mm

**54 LEDs 3mm blanches** réparties sur plusieurs MCP23S17 :

**LEDs du pad** (32 LEDs) :
- **2 MCP23S17 dédiés** : MCP "LEDs pad 1" (16 GPIO pour rangées 0-1) + MCP "LEDs pad 2" (16 GPIO pour rangées 2-3)
- Alimentation : 5V (rail Pi 5)
- Résistance 220Ω par LED (limite courant à ~8-10 mA)
- **Current sink mode** : cathode LED → GPIO MCP

**LEDs boutons de contrôle** (22 LEDs) :
- 5 M1-M5 + 5 globaux + 8 voix pad + 4 master = 22 LEDs
- Réparties sur **1-2 MCP23S17** supplémentaires (parmi les 20 MCP de l'architecture)

**Total MCP23S17 dédiés aux LEDs** : **3-4 MCP23S17** (sur les 20 disponibles).

### Schéma de câblage par LED

```
      VCC 5V
         │
         │
       [220Ω]
         │
         │
      ┌──┴──┐
      │  Anode │  ← patte longue
      │       │
      │ LED 3mm │
      │ blanche │
      │       │
      │ Cathode │  ← patte courte
      └──┬──┘
         │
         │
      GPIO MCP23S17 (mode OUTPUT)
      ├─ LOW  → LED allumée (current sink)
      └─ HIGH → LED éteinte (no current)
```

### Firmware exemple

```cpp
#include <MCP23S17.h>

MCP23S17 mcp_leds_pad1(SPI_CS_PIN, 0x00);  // MCP dédié LEDs rangées 0-1
MCP23S17 mcp_leds_pad2(SPI_CS_PIN, 0x01);  // MCP dédié LEDs rangées 2-3

void setup() {
    // Configurer tous les GPIO LEDs en output
    for (int i = 0; i < 16; i++) {
        mcp_leds_pad1.pinMode(i, OUTPUT);
        mcp_leds_pad1.digitalWrite(i, HIGH);  // éteintes au boot
        
        mcp_leds_pad2.pinMode(i, OUTPUT);
        mcp_leds_pad2.digitalWrite(i, HIGH);
    }
}

// Allumer la LED du pad[row, col]
void led_pad_on(int row, int col) {
    int led_id = row * 8 + col;  // 0 à 31
    if (led_id < 16) {
        mcp_leds_pad1.digitalWrite(led_id, LOW);
    } else {
        mcp_leds_pad2.digitalWrite(led_id - 16, LOW);
    }
}

// Éteindre la LED du pad[row, col]
void led_pad_off(int row, int col) {
    int led_id = row * 8 + col;
    if (led_id < 16) {
        mcp_leds_pad1.digitalWrite(led_id, HIGH);
    } else {
        mcp_leds_pad2.digitalWrite(led_id - 16, HIGH);
    }
}
```

**Simple et direct**, pas de protocole série temps-critique comme les NeoPixel.

### Avantages de cette approche

1. **LEDs alignées parfaitement** avec les keycaps (slot LED intégré au switch)
2. **Simplicité firmware** : ON/OFF booléen, pas de gestion de couleurs RGB
3. **Simplicité hardware** : pas de level shifter, pas de timing critique
4. **Libération pin 34** Teensy pour extensions futures
5. **Différenciation par keycap coloré** : la couleur perçue vient du keycap translucide, pas de la LED

### Consommation électrique

- **Par LED** : 8-10 mA à 5V
- **Total 54 LEDs allumées simultanément** : 54 × 10 = **540 mA max**
- **En usage live typique** : 10-20 LEDs allumées en moyenne = 100-200 mA
- **Compatible** avec l'alimentation 5V du Pi 5 (27W = 5.4 A disponibles)

---

## 9. MIDI DIN IN + OUT

### Pins

- **MIDI IN (RX)** : pin 0 (Serial1 RX)
- **MIDI OUT (TX)** : pin 1 (Serial1 TX)

### Hardware

Voir `midi_hardware.md` pour schéma détaillé :
- MIDI IN : opto H11L1 + DIN 5 broches femelle
- MIDI OUT : résistances 33Ω + 10Ω (ou 220Ω selon variante) + DIN 5 broches femelle

---

## 10. Validation hardware Phase 1

Avant de coder la logique applicative, valider les briques :

1. **Test MIDI clock** : clock FL → ESI → Teensy. Jitter <1ms mesuré au scope
2. **Test SPI OLEDs** : afficher "V1" à "V8" sur 8 écrans simultanément
3. **Test I²C TCA9548A + OLED master** : "BPM 140" affiché
4. **Test chaîne SPI MCP23S17** : lire/écrire sur les 20 MCP individuellement
5. **Test matrice pad via MCP** : 32 touches détectées sans ghosting
6. **Test encodeurs** : 82 encodeurs détectés avec acceleration
7. **Test USB série Pi ↔ Teensy** : 1000 msg/s bidirectionnel
8. **Test LEDs 3mm pad** : 32 LEDs pad contrôlées individuellement via MCP23S17 ON/OFF
9. **Test audio SAI1 RX** : boucler signal PCM1808 → Teensy, mesurer SNR
10. **Test audio SAI1 TX** : générer sinus Teensy → PCM5102A #1, analyser au scope
11. **Test audio SAI2 TX** : idem pour les 2 PCM5102A sur SAI2
12. **Test ampli casque TPA6120** : signal master → casque, mesurer au scope
13. **Test NE5532 spare** (si utilisation analog custom)

Chaque test est un "Hello World" matériel. Si un échoue, debugger avant de continuer.

---

## 11. Test rig minimal Phase 1

**Sketch Arduino minimaliste** pour valider MIDI clock :

```cpp
#include <MIDI.h>
MIDI_CREATE_INSTANCE(HardwareSerial, Serial1, MIDI);

void setup() {
    MIDI.begin(MIDI_CHANNEL_OMNI);
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
    if (MIDI.read()) {
        if (MIDI.getType() == midi::Clock) {
            digitalWrite(LED_BUILTIN, HIGH);
            delayMicroseconds(100);
            digitalWrite(LED_BUILTIN, LOW);
        }
    }
}
```

Tester à différents BPM (90, 120, 140, 180 BPM), mesurer la régularité au scope.

---

## 12. Historique des révisions

### v0.9 (avril 2026) — Uniformisation OLEDs SPI + mécanique 9×5

- **OLED master SSD1306 128×64 passe en SPI 7-pin** (CS pin 27)
  - Bus I²C totalement libéré (pins 18, 19 disponibles pour extensions futures)
  - TCA9548A conservé dans la BOM mais non utilisé en v0.9
  - 9 OLEDs au total partageant le bus SPI (8 SH1107 voix + 1 SSD1306 master)
- **Pinout Teensy final v0.9** : pas de changement majeur, uniquement pin 27 passe de "Réserve" à "OLED master SPI CS"
- **Mécanique** : module violet 9×5 intégrant pad + V1-V8 + M1-M5 (voir `panel_design.md`), pas d'impact firmware
- **Sourcing Teensy** : ProtoSupplies via MyUS.com (transitaire US → France), le Teensy 4.1 Fully Loaded 32MB PSRAM reste la cible

### v0.8 final (avril 2026) — Pivot architecture audio

- **Architecture audio modulaire** : 2× PCM1808 + 6× PCM5102A + 1× TPA6120 module
- **Matrice pad déplacée sur MCP23S17** (libère pins 2-9 pour audio)
- **20 MCP23S17 DIP-28** sur 5 perfboards (au lieu de 15 en v0.8 initial)
- **3 chaînes SPI MCP** (pins CS 24, 25, 26)
- **SAI1 + SAI2 utilisés** pour 12 canaux de sortie audio
- **Allocation pinout clean** : chaque pin a une fonction dédiée, aucun déplacement forcé
- **OLED V4 CS sur pin 37** (au lieu de pin 23 conflit MCLK)
- **LEDs 3mm blanches dans switches MX** (pas de NeoPixel en v0.8 final, pin 34 libéré)
- **PCM3168A différé v3+** (voir VISION et PHASE0 sections roadmap)

### v0.8 initial (avril 2026)

- 15 MCP23S17 (remplacé par 20)
- Teensy Audio Shield (remplacé par modules discrets)
- Matrice pad sur pins 2-9 directs (remplacé par MCP)
- Conflits pins 7/8 audio vs matrice pad (résolu)

### v0.7 (précédente)

- SH1107 128×128 (gardé)
- Pins 22/23 pour CS V3/V4 OLEDs (V4 déplacé en v0.8 final)

---

*Fin de teensy_pinout.md — v0.9, avril 2026*
