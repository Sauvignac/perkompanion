# PërKompanion — Vision — v1.0.0

## Changelog v0.9 → v1.0.0

### En-tête modifiée
- Avant : "v0.9 — avril 2026" + section "Changements clés v0.7 → v0.8".
- Après : "v1.0.0 — avril 2026". La section v0.7 → v0.8 est conservée
  comme historique. Une nouvelle section "Changements clés v0.9 → v1.0.0"
  est ajoutée juste après.

### Pitch officiel §1 (`### Pitch officiel`) — INCHANGÉ
- Décision chef d'orchestre : le pitch reste **Perkons-centric** pour la
  phase 1. Le pivot framework deviendra un événement narratif au moment
  du lancement de la 2e machine supportée. Pas de modification publique.
- Rationale : préserver la lisibilité côté nouveaux viewers YouTube et
  contributeurs externes en phase 1. Le framework est un détail
  d'architecture, pas un produit séparé.

### Philosophie §1 — UNE puce ajoutée
- Ajout du bullet : "**Architecture device-agnostic en interne** : le
  firmware ne connaît que la grammaire des profiles, pas le hardware
  audio externe — ouverture progressive vers d'autres machines à partir
  de v2/v3."
- Rationale : un contributeur MIT qui lit la philosophie comprend
  immédiatement que le projet est conçu pour scaler. Pas de marketing,
  juste une vérité architecturale exposée.

### §10 "Mode dégradé et résilience" — paragraphe ajouté
- Ajout d'un paragraphe sur le mode dégradé multi-machine (Option A +
  fallback B), conforme à la décision provisoire des notes privées du
  28 avril 2026.
- Rationale : la VISION expose déjà le mode dégradé en v0.9 ; il faut
  refléter le fait que le mode dégradé est désormais "par profile actif"
  et non "Perkons hardcodé".

### §13 "Inventaire matériel v1 complet" — UNE phrase ajoutée
- Ajout : "À noter que les jacks d'entrée audio physiques (4 in PCM1808)
  doivent être étiquetés génériquement IN 1 / IN 2 / IN 3 / IN 4 et non
  V1 IN / V2 IN / V3 IN / V4 IN, pour ne pas figer conceptuellement les
  entrées sur les voix Perkons."
- Rationale : conforme à la sous-section "Étiquetage de la plate" des
  notes privées. Conséquence pratique de la roadmap multi-machine v2-v4
  qui doit être prise en compte avant l'envoi en gravure laser.

### Roadmap §8 — UN bullet ajouté à v3
- v3 — ajout du bullet : "**Premier profile externe additionnel**
  (Digitakt II, Hydrasynth, ou autre selon disponibilité et besoins de la
  communauté) — première itération multi-machine de PërKompanion."
- Rationale : le pivot framework apparaîtra de fait à ce moment-là (par
  l'arrivée d'une 2e machine supportée), sans qu'on l'annonce préalablement
  comme un événement narratif dans la roadmap. Cohérent avec l'arbitrage
  "phase 1 invisible" + discrétion narrative.

### Sections inchangées
- §1 (Identité) : pitch et cible inchangés.
- §2 (Architecture deux axes) : inchangée — les V5-V8 restent décrites
  comme "voix virtuelles internes" sans mention du pivot framework.
- §3 (Voix virtuelles), §4 (Modulation Engine), §5 (Architecture matérielle),
  §6 (Audio), §7 (Fabrication), §9 (Sample packs), §11 (Budget), §12
  (Contraintes) : inchangées.

---

# PërKompanion — Vision

> Extension modulaire hardware + software pour Erica Synths Perkons HD-01
> Document produit et fonctionnel — **v1.0.0** — avril 2026

---

## Préambule

Ce document décrit la **vision produit** de PërKompanion : ce que c'est, pour qui, comment ça se vit, quelles features, quelle roadmap.

Pour les fondations techniques d'implémentation, voir `PERKOMPANION_PHASE0.md`.
Pour les tables de référence exhaustives, voir `PERKOMPANION_PROTOCOL_TABLES.md`.
Pour le layout détaillé du panneau hardware, voir `panel_design.md`.

### Changements clés v0.7 → v0.8

**Architecture contrôle et UI** :
- **Architecture étendue à 8 voix physiques** (4 Perkons + 4 voix virtuelles explicitement représentées sur le panneau)
- **Bibliothèque hotcues globale** de 128 hotcues, paginée par boutons V1-V8 en mode pad Hotcue
- **Lock hotcues** et **polyphonie ajustable en direct** (encodeur dédié en zone master)
- **Layout panneau 45×37cm** (au lieu de 45×32cm)
- **Écran 7" Elecrow raw panel** (au lieu de 10.1")
- **OLEDs SH1107 128×128** partout sur voix (au lieu de SH1106 128×64)
- **8 encodeurs par colonne voix** (2 permanents + 6 contextuels) avec appui+rotation pour fonctions secondaires
- **REC de motions live** (style Perkons parameter lock live recording)
- **5 boutons globaux** : SHIFT, PANIC, FREEZE, CLEAN SLATE, REC
- **Casque en façade** avec encodeur volume dédié
- **Mode pad M5 Step Edit** ajouté (édition step sequencer hotcue)

**Architecture audio (pivot majeur)** :
- **Architecture audio modulaire PCM1808 + PCM5102A** (rollback du PCM3168A initialement prévu)
- **PCM3168A différé en v3+** via PCB custom + JLCPCB PCBA (quand maturité KiCad et besoin réel démontré)
- **Anticipation finale complète** sur la plate : 4 in Perkons + 12 out (Master L+R, V1-V4 stéréo, V5-V8 mixées stéréo)
- **6× PCM5102A** pour 12 canaux de sortie (SAI1 × 4 DAC + SAI2 × 2 DAC)
- **2× PCM1808** pour 4 canaux d'entrée Perkons
- **Ampli casque externe TPA6120 module MCU-612** (façade avec potentiomètre volume dédié)
- **DSP par voix modulable** : 4 slots reconfigurables parmi 13 types d'effets (112 params DSP)
- **Enregistrement via Tascam uniquement** (pas de Pi 5 USB audio gadget)

**Hardware et fabrication** :
- **20 MCP23S17 DIP-28** répartis sur 5 perfboards (4 MCP chacune, chaînage adresse A0/A1/A2)
- **Matrice pad sur MCP23S17** (pas sur pins Teensy directs) → libération complète des pins 2-9 pour SAI1/SAI2
- **Pinout Teensy clean** : chaque pin a une fonction dédiée, aucun déplacement "forcé par conflit"
- **Fabrication** : plate FabLab dès le début du projet
- **Prototypage TOUT en tact 6×6mm pas 2.54mm** sur breadboard
- **Nouveau fournisseur TME** pour composants critiques (ICs authentiques, jacks Neutrik, condensateurs qualité)

### Changements clés v0.9 → v1.0.0

**Pivot framework (architecture interne)**. PërKompanion est désormais conçu
en interne comme un framework générique de companion MIDI, avec le Perkons
HD-01 comme premier profile de référence. Cette évolution est **architecturale
et invisible côté pitch public** en phase 1. Conséquences techniques :

- Tables `param_id` du protocole : bloc `0x0000-0x0FFF` réservé aux profiles
  externes (4 slots × 1024 params). Voix virtuelles V5-V8 internes
  relocalisées vers `0x6000-0x6FFF`. Voir `PERKOMPANION_PROTOCOL_TABLES.md`
  §2 et §14.
- Format Device Profile YAML défini : `device_profile_schema.md` +
  `profile_template_with_docs.yaml`.
- 4 profiles de référence rédigés et stress-testés : Perkons HD-01,
  Digitakt II, Electribe 2 Sampler, TD-3-MO. Voir `profiles/*.yaml`.
- Mode dégradé Teensy étendu pour fonctionner avec n'importe quel profile
  actif (cf. §10 ci-dessous), avec fallback Perkons hardcodé en sécurité
  ultime si EEPROM corrompue.
- Étiquetage des jacks d'entrée audio neutralisé (IN 1/2/3/4 plutôt que
  V1 IN / V2 IN / V3 IN / V4 IN) pour ne pas figer conceptuellement les
  entrées sur les voix Perkons (cf. §13 BOM).

**Aucun changement** sur le pitch officiel, l'architecture deux axes, le
Modulation Engine, l'architecture matérielle, l'audio, la fabrication, le
budget v1. Les évolutions v0.9 ergonomiques restent valides.

---

## 1. Identité

### Nom
**PërKompanion** (affichage) / **perkompanion** (ASCII pour URLs, repo)

### Repository
github.com/sauvignac/perkompanion — public, MIT license

### Pitch officiel

PërKompanion est une extension modulaire pour l'Erica Synths Perkons HD-01. Son architecture repose sur **deux axes de contrôle indépendants et simultanés** : un pad polymorphe pour l'exploration et la performance ponctuelle, et **huit colonnes voix** en contrôle permanent (4 physiques pour les voix Perkons + 4 virtuelles pour des hotcues assignables).

Au cœur du système, un **moteur de modulation unifié** permet de piloter n'importe quel paramètre par des LFOs, augmentations, séquenceurs, enveloppes ou motions capturées en live.

Commençant en v1 comme overlay MIDI avec arpégiateur et multi-LFO, PërKompanion évolue vers un véritable super-séquenceur : DSP par voix (v2+), sampleur intégré avec 128 hotcues (v3+), séquenceur et modulation complète par hotcue (v4+).

Le Perkons fournit la source sonore analogique. PërKompanion lui donne la puissance créative d'un synthé modulaire complet, avec en plus 4 voix virtuelles internes jouables simultanément.

### Cible

**Primaire** : possesseurs de Perkons HD-01 frustrés par les limites natives (LFO global, pas d'écran, pas de DSP post-voix).

**Secondaire** : musiciens free tekno, hardcore, hardtek, ambient et expérimentaux qui veulent pousser le Perkons au-delà.

### Philosophie

- **Overlay non-destructive** : le Perkons reste un Perkons
- **Le Perkons reste autonome** : fonctionne sans PërKompanion
- **Musicien virtuel** : ajoute des knobs, ne remplace rien
- **Philosophie Perkons préservée** : touché direct par voix, pas de menu en live
- **Instrument, pas gadget** : "en jam concrètement, ça fait quoi ?"
- **Pas de saisie de secrétariat** en live
- **Mode dégradé** : reste jouable même si Pi ou Teensy tombe
- **Règle du "un pour un"** : pour chaque ajout, une justification ou un retrait équivalent
- **Open source MIT** : aligné free tekno
- **Esthétique suie, mazout, friction** : industrielle, organique, dirty
- **Architecture device-agnostic en interne** : le firmware ne connaît que la grammaire des profiles, pas le hardware audio externe — ouverture progressive vers d'autres machines à partir de v2/v3

---

## 2. Architecture de contrôle à deux axes

**Le point central de PërKompanion.**

### Axe 1 — Le pad (exploration, configuration, performance ponctuelle)

- **5 boutons mode pad (M1-M5)** en colonne à gauche du pad
  - M1 : Drum Trigger
  - M2 : Augmentation Launcher
  - M3 : Hotcue (v3+)
  - M4 : Pattern/Kit/Scene Selector
  - M5 : Step Edit (v4+)
- **8 boutons voix pad (V1-V8)** au-dessus du pad
  - V1-V4 : voix Perkons physiques
  - V5-V8 : voix virtuelles (hotcues assignés)
- **En mode M3 (Hotcue)** : boutons V1-V8 = pagination (8 pages × 16 hotcues = 128 hotcues globaux)

### Axe 2 — Les 8 colonnes voix

- Chaque colonne a son propre mode encodeurs indépendant
- **1 encodeur MODE par voix** (rotation cycle les modes, tap valide, long press = mode défaut)
- Les encodeurs contextuels prennent leur fonction selon le mode local

**Cycle des modes** : LFO → DSP → Arp → Hotcue → LFO

Modes non disponibles dans la version actuelle sont **skippés automatiquement**. En v1 : LFO → Arp → LFO.

### Distinction V1-V4 vs V5-V8

**V1-V4 (physiques)** :
- Pilotent les 4 voix du Perkons via MIDI CC (CC 70-113)
- 44 paramètres modulables au total (11 × 4 voix)
- Limités à ce qui est exposé en MIDI CC Perkons

**V5-V8 (virtuelles)** :
- Pilotent des hotcues assignés (samples en PSRAM Teensy)
- Paramètres du hotcue + DSP PërKompanion
- Plus riche en fonctionnalités
- OLED affiche le hotcue assigné (numéro + nom)

### Feedback visuel par mode

- **LFO** : bleu
- **DSP** : violet
- **Arp** : vert
- **Hotcue** : orange

### Exemple concret live

- V1 en mode LFO (tweak Rate + Destination sur Cutoff Perkons)
- V2 en mode DSP (Cutoff + Drive DSP PërKompanion sur voix Perkons V2)
- V5 en mode Hotcue (Start + Length du hotcue assigné)
- V6 en mode Arp sur hotcue assigné
- V7-V8 en mode LFO
- Pad en Drum Trigger V2 pour kicks
- Switch instantané vers Hotcue page 3 pour déclencher d'autres hotcues

**Les deux axes sont parallèles. Polyphonie de contrôle maximum.**

---

## 3. Voix virtuelles et bibliothèque hotcues

### Concept

Bibliothèque globale de **128 hotcues** stockés en PSRAM Teensy (32 MB). Chaque hotcue = sample audio + paramètres + séquenceur + arpégiateur + modulations + DSP (v4 complet).

Organisés en **bibliothèque linéaire 0-127**, accessibles par pagination.

### Les 4 voix virtuelles V5-V8

Représentées physiquement sur le panneau. Chaque peut avoir un hotcue assigné à un instant T.

**Assignation** :
- Maintenir bouton V5/V6/V7/V8 + tap sur hotcue dans pad (mode Hotcue M3)
- OLED V5 affiche numéro et nom du hotcue
- Encodeurs de V5 pilotent les paramètres du hotcue

**Dynamique** :
- Les modulations attachées à V5 restent en place même si le hotcue change
- Permet setup "constant" (LFO + DSP + FX sur V5) avec source variable

### Déclenchement

Trois modes :
1. **Via pad en mode M3** : tap = déclenchement avec paramètres pré-configurés
2. **Via voix virtuelle assignée** : encodeurs pilotent en temps réel
3. **Via Augmentation** : slot déclenche hotcue avec courbe pré-programmée

### Polyphonie

**Limite configurable en direct** via encodeur dédié zone master (default 16, range 1-32).

**Coupure automatique** :
- Si limite atteinte → coupe le hotcue **non-locké** le plus ancien
- Si tous lockés → nouveau hotcue ne démarre pas (WARNING)

### Lock des hotcues

Protège les hotcues essentiels (nappe ambient, drone) de la coupure auto.

**Workflow** :
- Long press hotcue pad → menu écran avec "Lock"
- LED pad devient violette
- Toggle off pour délocker

**Commande protocole** : `SET_HOTCUE_LOCK` (0x29).

### Édition

**Via écran tactile** (page d'édition dédiée) :
- Waveform
- Sliders Start/Length/Pitch/Feedback/Volume
- Options : Loop/Reverse/Quantize/Fade
- Step sequencer (16 ou 32 steps) avec probability, ratchet, tie
- Boutons contextuels B1-B6 + encodeurs E1-E8 autour de l'écran

**Via pad en mode Step Edit (M5)** :
- Pad affiche les 16/32 steps
- Tap pour toggle
- Long press pour sous-paramètres

---

## 4. Modulation Engine unifié

### Sources

1. **LFOs** : sinus, carré, triangle, dent de scie, aléatoire, formes custom
2. **Envelopes** : ADSR déclenchés
3. **Augmentations** : courbes pré-programmées 16/32/64/128 steps
4. **Séquenceurs** : patterns avec valeurs par step
5. **Random** : avec contraintes
6. **Motions** (v2+) : mouvements capturés en live via REC

### Cibles

Toute valeur paramétrique :
1. **Paramètres Perkons V1-V4** via MIDI CC (44 params)
2. **Paramètres voix virtuelles V5-V8** (Start, Length, Pitch, Feedback, Volume, DSP)
3. **Paramètres DSP par voix** (v2+)
4. **Paramètres Master** (Volume, Reverb, Delay, Comp, EQ)
5. **Paramètres Arp**
6. **Paramètres Hotcue** (v3+)

### Nombre simultané

**v1** : 32 modulations actives simultanées max.
**v2+** : augmenté selon capacités CPU.

### Motions (v2+)

**Concept** : capturer des mouvements d'encodeurs en live.

**Workflow** :
1. Maintenir **REC** (main gauche)
2. Tourner encodeurs (main droite)
3. Motions enregistrées avec timing
4. Relâcher REC

**Philosophie "live recording"** (style Perkons parameter lock) :
- Destiné principalement au live, pas au save
- Tourne en loop pendant session
- Remplaçable, effaçable

**Sauvegarde individuelle** :
- Une motion sur 1 encodeur spécifique peut être extraite et sauvegardée
- Assignable ensuite à un slot Augmentation

**Longueur** :
- Par défaut : 16 steps (aligné Perkons)
- Modifiable : 32 ou 64 steps
- Quantize 1/16 par défaut

### Augmentations

Distinctes du REC live :
- Slots du pad déclenchent courbes pré-programmées
- Configurées via écran tactile ou importées de motions sauvegardées
- Types : courbe math, motion sauvegardée, preset
- Durée : 16/32/64/128 steps
- Loop ou one-shot

---

## 5. Architecture matérielle

### Topologie générale

```
[Perkons HD-01] ←──────────────────────────────┐
       ▲                                       │
       │ MIDI DIN                              │
[ESI M8U eX] ←→ [Laptop FL Studio]             │
   ▲                                           │
   │ MIDI DIN (clock + CCs)                    │
   ▼                                           │
[Teensy 4.1 FL 32MB]  ────── Panneau hardware  │
    (cœur temps-réel)                          │
       │                                       │
       ├── I2S ─→ [Teensy Audio Shield]        │ (v1)
       │                                       │
       ├── I2S ─→ [Codec PCM3168A multi-canal] │ (v2+)
       │          │                            │
       │          └── 17 jacks audio ──────────┘
       │
       │ USB série 2 Mbps
       ▼
[Raspberry Pi 5 4GB]
       │
       │ HDMI + USB tactile
       ▼
[Écran 7" Elecrow 1024×600]

(Wi-Fi pour laptop/téléphone)
```

**Règle** : MIDI time-critical (clock, CCs, notes arp) ne transite jamais par le Pi.

### Teensy 4.1 (cœur temps-réel)

- MIDI clock + modulations (LFO, Arp, Augmentations, Motions)
- CC MIDI vers Perkons
- DSP audio (V5-V8 en v1, V1-V8 en v2+)
- Pad 32 touches (matrice avec diodes)
- 64+ encodeurs via MCP23S17 SPI
- 80+ boutons via MCP23S17
- 9 OLEDs
- USB série avec Pi
- 32 MB PSRAM pour hotcues

### Raspberry Pi 5 4GB

- UI écran tactile 7" (Flask + Chromium kiosk)
- Gestion projets (save/load)
- Édition hotcues
- Bibliothèque motions
- Wi-Fi (laptop, téléphone)
- Téléchargement sample packs (v3+)
- Configuration pad layouts

### Écran Elecrow 7" 1024×600 raw panel

**Pourquoi ce choix** :
- **Raw panel** = trous de fixation intégrés → intégration mécanique propre
- **1024×600** IPS 170° → lisible live
- **Tactile capacitif multi-touch 5 points**
- **Plug & play** : HDMI + USB tactile, pas de driver
- **Alim 5V via USB Pi**
- **16.5×12.4 cm**
- **Prix** : 52€ (ASIN Amazon B07H79XMLT)

### Panneau 45×37cm

- **Largeur** : 45 cm (aligné Perkons)
- **Hauteur** : 37 cm (2cm de plus que Perkons 35cm)
- **Épaisseur** : acrylique 3mm noir mat (final) ou transparent (proto)

Fabrication : Sculpteo

### Composition par zones

**Section haute** (y = 0 à 14 cm) :
- Colonne M1-M5 à gauche (5 boutons mode pad)
- **Module violet 9×5 Cherry MX** (pad 8×4 + 8 voix V1-V8 + 5 modes M1-M5, total 45 touches dans 19×11.5 cm)
- Écran 7" à droite avec 6 boutons B1-B6 au-dessus + 8 encodeurs E1-E8 sur les côtés
- LED activité en haut-droite

**Bande centrale** (y 14-17 cm) :
- 5 boutons globaux : SHIFT, PANIC, FREEZE, CLEAN SLATE, REC
- Boutons voix pad V1-V8 au-dessus des colonnes voix

**Section basse** (y 17-37 cm) :
- 8 colonnes voix monocolonnes (40mm de large chacune)
- Zone master sous écran, à droite des voix (~10 cm)
- Encodeur volume casque + jack casque façade bas-droite

**Tranche arrière** :
- Connecteurs digitaux (USB, HDMI, alim)
- 16 emplacements jacks audio anticipés :
  - 4 TS in Perkons (v2+)
  - 8 TS out voix Perkons stéréo L+R par voix (v2+)
  - 2 TS out bus virtuelles stéréo (v2+)
  - 2 TS out master stéréo (v1 via Audio Shield)

### Colonne voix — composition détaillée

**Largeur** : 40 mm
**Hauteur** : ~17 cm

**De haut en bas** :

1. **Bouton voix tête colonne** (~10 mm)
   - Tact 6×6 (proto) / SJMS 8×8 LED (final)
   - Combos : assignation hotcue, panic/freeze voix
   - LED indique voix sélectionnée

2. **OLED SH1107 128×128 SPI 1.5"** (~36×34 mm)
   - Mode actif, valeurs encodeurs, hotcue assigné (V5-V8)

3. **Encodeur MODE** (knob 18mm)
   - Rotation : cycle modes
   - Tap : valide
   - Long press : mode défaut

4. **Encodeur USER** (knob 18mm)
   - Permanent :
     - **V1-V4** : non utilisé v1, réservé (assignable futur)
     - **V5-V8** : Volume du hotcue assigné
   - Long press : reset valeur défaut

5. **6 encodeurs contextuels E1-E6** (knobs 12mm)
   - Fonction selon mode actif
   - Pitch 22 mm centre-à-centre (10 mm entre knobs)
   - **Appui+rotation** pour fonction secondaire

### Mapping encodeurs contextuels par mode

**Mode LFO** :
- E1 : Rate | (appui+) Rate fin
- E2 : Depth | (appui+) Min value
- E3 : Destination | (appui+) Polarity
- E4 : Shape | (appui+) Phase
- E5 : Smoothing | (appui+) Output curve
- E6 : Sync on/off | (appui+) Divider

**Mode DSP** (v2+) :
- E1 : Filter Cutoff | (appui+) Filter Type
- E2 : Resonance | (appui+) Filter Slope
- E3 : Drive | (appui+) Drive Type
- E4 : FX Send Reverb | (appui+) Reverb Type
- E5 : FX Send Delay | (appui+) Delay Mode
- E6 : Wet/Dry | (appui+) Pre/Post Filter

**Mode Arp** :
- E1 : Mode | (appui+) Direction reset
- E2 : Rate | (appui+) Resolution
- E3 : Octave Range | (appui+) Octave shift
- E4 : Gate Time | (appui+) Accent
- E5 : Swing | (appui+) Groove preset
- E6 : Variation | (appui+) Randomness

**Mode Hotcue** (v3+) :
- E1 : Start | (appui+) Start fine
- E2 : Length | (appui+) Length fine
- E3 : Pitch | (appui+) Pitch fine
- E4 : Feedback | (appui+) Feedback shape
- E5 : Reverse/speed | (appui+) Pitch tracking
- E6 : Filter | (appui+) Filter mode

### Zone Master

**Position** : sous l'écran, à droite des 8 colonnes voix (10cm × 20cm)

**Encodeurs** (~10 encodeurs) :
- Volume Master
- **Polyphony Hotcue** (encodeur dédié, range 1-32)
- Reverb Time / Reverb Mix / Reverb Type
- Delay Time / Delay Feedback / Delay Mix
- Comp Threshold / Comp Ratio

**Boutons MODE GLOBAL** (4 boutons) :
- [LFO] [DSP] [ARP] [HOT]
- Bascule toutes les voix simultanément
- Combo "voix tête + MODE GLOBAL" = basculer une seule voix

**OLED master** : SSD1306 128×64 **SPI 7-pin** (v0.9 : uniformisation SPI pour cohérence avec les 8 OLEDs voix)
- BPM, polyphonie actifs, état clock
- Partage du bus SPI avec les 8 OLEDs voix
- CS dédié sur pin 27 Teensy

### Casque façade

**Position** : bas-droite
**Composants** :
- Jack TRS 6.35mm stéréo
- Encodeur volume casque (knob 15mm)
- Bouton casque on/off optionnel (mute master)

**Câblage v1** : sortie headphone Teensy Audio Shield.

### 7 boutons globaux (v0.9)

**Position** : bande centrale, entre pad et voix (accès live rapide)

1. **SHIFT** : modifier pour combos
2. **PANIC** : arrêt d'urgence MIDI
3. **FREEZE** : gel des modulations actives
4. **CLEAN SLATE** : reset global (avec confirmation écran)
5. **REC** : enregistrement motions live
6. **Réserve futur #1** (ex : UNDO, RANDOMIZE, COPY)
7. **Réserve futur #2** (ex : PASTE, LOCK, DUPLICATE)

**v0.9** : passage de 5 à 7 boutons pour anticiper l'évolution fonctionnelle. Les 2 boutons supplémentaires gardent les mêmes keycaps rouges translucides que les 5 globaux existants et sont câblés sur GPIO MCP23S17 dédiés.

### LED d'activité

**Position** : haut-droite du panneau

**LEDs** (4-8 LEDs 3mm) :
- MIDI IN (clignote)
- MIDI OUT (clignote)
- Clock sync (fixe/clignotant)
- Power
- Pi activity
- Teensy activity
- Overload (rouge fixe)
- Record active (rouge fixe pendant REC)

---

## 6. Audio

### Philosophie

**Architecture modulaire à base de modules breakout PCM1808/PCM5102A sur bus I²S du Teensy 4.1.** Constructible solo, sans CMS, évolutive sur le même bus I²S au fur et à mesure des paliers.

**Le PCM3168A initialement prévu en v0.8 est différé en v3+** via PCB custom JLCPCB PCBA, quand le projet aura démontré sa maturité et qu'Alex aura pris la main sur KiCad. Voir roadmap pour les conditions de réintroduction.

**Enregistrement** : via Tascam Model 24 uniquement (chaîne existante, SanDisk Extreme 256GB). Pas de Pi 5 USB audio gadget.

### Architecture audio complète v0.8

**Entrées audio (4 canaux mono Perkons)** :
- 4 × jacks TS 6.35mm sur tranche arrière → **2× PCM1808** breakout (stéréo ADC 24-bit/96 kHz)
- Bus SAI1 RX du Teensy (2 pins data DATA_IN_1 + DATA_IN_2)
- Objet firmware : `AudioInputI2SQuad` (PJRC Audio Library, 4 canaux natifs)

**Traitement DSP par voix** :
- 4 chaînes DSP indépendantes pour V1-V4 (voix Perkons traitées)
- Bus DSP pour V5-V8 (voix virtuelles, mixées en bus stéréo commun)
- Voir section DSP ci-dessous

**Sorties audio (12 canaux = 6 stéréos)** :
- **SAI1 TX (4 DAC = 8 canaux)** :
  - PCM5102A #1 → **Master L+R** (2 jacks TS)
  - PCM5102A #2 → **V1 Perkons traité L+R** (2 jacks TS)
  - PCM5102A #3 → **V2 Perkons traité L+R** (2 jacks TS)
  - PCM5102A #4 → **V3 Perkons traité L+R** (2 jacks TS)
- **SAI2 TX (2 DAC = 4 canaux)** :
  - PCM5102A #5 → **V4 Perkons traité L+R** (2 jacks TS)
  - PCM5102A #6 → **V5-V8 mixées L+R** (2 jacks TS)

**Total sorties** : **12 jacks TS** (6 stéréos)

**Casque façade** :
- Ampli casque dédié **module TPA6120 breakout MCU-612** alimenté par le signal analog master du PCM5102A #1
- Jack TRS 6.35mm stéréo en façade bas-droite
- Potentiomètre 10K volume casque dédié

**MIDI** :
- 2 × jacks DIN 5 broches (IN + OUT) sur tranche arrière

**Total connectique I/O** :
- 4 in audio TS (Perkons)
- 12 out audio TS (voix + master)
- 1 jack casque TRS
- 2 DIN MIDI
- 3 digital (USB Pi debug, HDMI ext optionnel, alim)

**= 22 connecteurs au total sur tranche arrière + façade**

### Modules breakout utilisés

**PCM1808 breakout (× 2)** :
- Stéréo ADC Texas Instruments 24-bit / jusqu'à 96 kHz
- SSOP-14 pré-soudé sur PCB breakout avec headers 2.54 mm
- Single-ended 2.1 Vrms (niveau ligne standard)
- Prix : ~12€/module (AliExpress, Amazon)
- Source alimentation : 3.3V ou 5V single supply

**PCM5102A breakout (× 6)** :
- Stéréo DAC Texas Instruments 24-bit / jusqu'à 384 kHz
- SSOP-20 pré-soudé sur PCB breakout
- Single-ended 2.1 Vrms sortie ligne
- Prix : ~10€/module
- Source alimentation : 3.3V ou 5V single supply
- Pas besoin de MCLK externe (PLL interne)

**TPA6120 breakout module MCU-612** :
- Ampli casque stéréo haute performance TI
- SNR ~120 dB, 700 mW / 32Ω
- Compatible casques 16-600Ω
- Prix : ~8€
- Source alimentation : 5V single supply (doubleur interne)

### Topologie bus I²S

**Un seul oscillateur master clock** partagé entre tous les modules I²S (le Teensy génère MCLK).

**SAI1 (bus principal)** :
- MCLK, BCLK, LRCLK partagés
- DATA_IN_1 ← PCM1808 #1 (V1 + V2)
- DATA_IN_2 ← PCM1808 #2 (V3 + V4)
- DATA_OUT_0 → PCM5102A #1 (Master)
- DATA_OUT_1 → PCM5102A #2 (V1)
- DATA_OUT_2 → PCM5102A #3 (V2)
- DATA_OUT_3 → PCM5102A #4 (V3)

**SAI2 (bus secondaire)** :
- MCLK_2, BCLK_2, LRCLK_2 (horloges séparées)
- DATA_OUT_0 → PCM5102A #5 (V4)
- DATA_OUT_1 → PCM5102A #6 (V5-V8 mixées)

**Total pins Teensy pour audio** : ~12 pins (voir `teensy_pinout.md`)

### DSP par voix — 4 slots reconfigurables

**Principe** : chaque voix V1-V4 dispose d'une **chaîne DSP de 4 slots** que l'utilisateur configure librement via l'UI écran. Les voix V5-V8 ont également leurs propres chaînes DSP (mais sortent sur le bus V5-V8 mixé).

**Types d'effets disponibles** (enum Slot.Type) :

| Value | Effet | Params 1-4 |
|-------|-------|------------|
| 0x00 | None (slot vide, bypass) | - |
| 0x01 | Filter (State Variable) | Cutoff, Resonance, Type (LP/HP/BP/Notch), Drive |
| 0x02 | Distortion (Waveshaper) | Drive, Type (Soft/Hard/Fold), Tone, PreGain |
| 0x03 | Bitcrusher | SampleRate, BitDepth, Mix, — |
| 0x04 | Chorus | Rate, Depth, Feedback, Spread |
| 0x05 | Phaser | Rate, Depth, Feedback, Stages |
| 0x06 | Reverb (Freeverb) | RoomSize, Damping, Width, — |
| 0x07 | Delay | Time, Feedback, Filter, PingPong |
| 0x08 | Compressor | Threshold, Ratio, Attack, Release |
| 0x09 | EQ 3-band | Low, Mid, High, MidFreq |
| 0x0A | Ring Mod | Freq, Depth, Shape, — |
| 0x0B | AutoPan | Rate, Depth, Shape, Phase |
| 0x0C | Tremolo | Rate, Depth, Shape, — |
| 0x0D | LoFi | Saturation, SRReduce, BitReduce, Noise |

**Structure d'une voix** :
```
[Entrée voix] → [Slot 0] → [Slot 1] → [Slot 2] → [Slot 3] → [Sends Master] → [Sortie voix]
                                                            ├── Reverb bus master
                                                            └── Delay bus master
```

**UI écran** : page dédiée "DSP Chain V1-V4" :
- Vue colonne par voix
- Tap slot → menu enum type d'effet
- Ajustement params via encodeurs contextuels écran
- Presets sauvegardables (Dub Chain, Distorted Filter, Ambient Wash, etc.)
- Drag & drop pour réorganiser l'ordre des slots

**Total paramètres DSP modulables** :
- Par voix : 4 slots × 6 params (Type, P1, P2, P3, P4, Mix) + 4 sends (ReverbBus, DelayBus, OutputLevel, Pan) = **28 params**
- 4 voix Perkons + 4 voix virtuelles : 8 × 28 = **224 params DSP total**

Bloc param_id : `0x2000-0x2FFF` (voir PROTOCOL_TABLES).

### Firmware PJRC Audio Library

Construction des chaînes DSP avec **objets natifs PJRC** :

```cpp
// Exemple chaîne DSP pour V1 :
AudioInputI2SQuad      audio_in;
AudioFilterStateVariable filter_v1;
AudioEffectWaveshaper    drive_v1;
AudioEffectFreeverb      reverb_v1;
AudioEffectDelayExternal delay_v1;
AudioAmplifier           out_v1;

AudioConnection patch_v1_1(audio_in, 0, filter_v1, 0);
AudioConnection patch_v1_2(filter_v1, 0, drive_v1, 0);
AudioConnection patch_v1_3(drive_v1, 0, reverb_v1, 0);
AudioConnection patch_v1_4(reverb_v1, 0, delay_v1, 0);
AudioConnection patch_v1_5(delay_v1, 0, out_v1, 0);
AudioConnection patch_v1_out(out_v1, 0, master_mixer, 0);  // mix vers master

// Bus mix master :
AudioMixer4            master_mixer;       // 4 voix → stéréo
AudioEffectFreeverb    master_reverb;
AudioEffectDynamics    master_comp;
AudioOutputI2S         master_out;          // PCM5102A #1

// Idem pour V2, V3, V4, V5-V8
```

**Avantages** :
- Objets éprouvés, documentation PJRC complète
- Exemples en nombre
- Pas de code driver custom à écrire
- Performance optimisée pour Cortex-M7 du Teensy 4.1

### Paliers d'évolution

**Palier 1 — Phase 2 démarrage (v0.8+)** :
- 2× PCM1808 + 1× PCM5102A = **4 in / 2 out**
- Seul Master stéréo actif au début
- Budget : ~34€ (2× 12€ + 10€)

**Palier 2 — Phase 2 étendue (v0.9)** :
- +2 PCM5102A = **4 in / 6 out**
- Ajout de 4 sorties mono = 1 insert send par voix (pédales analog externes)
- Ou début des sorties V1-V4 stéréo par paires

**Palier 3 — Version complète (v1.0)** :
- 2× PCM1808 + 6× PCM5102A = **4 in / 12 out**
- Architecture finale : Master + V1-V4 stéréo + V5-V8 mixées stéréo
- Utilise SAI1 et SAI2 du Teensy

**Palier 4 — v3+ (si besoin démontré)** :
- Réintroduction PCM3168A sur PCB custom JLCPCB PCBA
- Capture simultanée master + 4 voix individuelles en USB audio gadget
- Sorties différentielles analog
- Condition : besoin réel en jam + maturité KiCad d'Alex

### Anticipation sur la plate

**Tous les 13 jacks audio + 1 casque + 2 MIDI sont dessinés et percés sur la plate dès la v1**, même si seulement Master L+R et casque sont câblés au début.

Les autres emplacements accueillent des **caps noirs esthétiques** (existent pour jacks 6.35mm, ~0.20€/pièce) jusqu'au montage du composant correspondant.

**Avantage** : aucune refonte de la plate quand on passe de Palier 1 à Palier 3. Tout est déjà là.

---

## 7. Architecture fabrication et prototypage

### Approche

**Prototypage sur breadboard TOUT en pas 2.54mm** jusqu'à validation firmware. Sauf Cherry MX.

**Avantages** :
- Breadboard/perfboard friendly
- Modifications rapides sans soudure
- Validation firmware avant investissement matériel
- Budget maîtrisé (tact 6×6 ~0.05€ vs MX ~1.20€)

**En Phase 4** :
- Migration Cherry MX Brown RGB pour pad 32 + 5 boutons critiques
- SJMS 8×8 avec LED pour boutons techniques (optionnel)
- Plate acrylique noir mat 3mm découpée Sculpteo

### Plate Sculpteo

**v0.8** : usiner **dès le début du projet**, pas en fin.

**Raisons** :
- Nécessaire de toute façon pour tests composants
- Apprentissage AutoCAD/Inkscape en début (période motivée)
- Itérations possibles (v1 proto, v2 corrigée, v3 finale)
- Validation ergonomie en conditions réelles

**Deux plates** :
- **v1 prototype** : acrylique 3mm transparent (~15€)
- **v2 finale** : acrylique 3mm noir mat + gravures (~25€)








### Dimensions critiques

- Trous switches MX : **14.00 × 14.00 mm**
- Pitch pad MX : **19.05 mm** (standard keyboard)
- Keycaps 1U : **18 × 18 mm**
- Trous encodeurs KY-040 : **Ø 7 mm**
- Pitch encodeurs voix : **10 mm entre knobs** (pitch 22 mm)
- Knobs : **18 mm** (MODE+USER) / **12 mm** (contextuels) / **15 mm** (master+casque)
- Trous tact 6×6 : 6.5 × 6.5 mm (corps) ou 8.5 × 8.5 (cap)
- Trous SJMS 8×8 : 8.5 × 8.5 mm
- Découpe OLED SH1107 : ~30 × 30 mm + trous M2
- Découpe écran 7" : ~155 × 90 mm + trous fixation
- Jacks audio 6.35mm : Ø 10-12 mm
- Vis fixation plate : M3 aux 4 coins

### Kerf du laser

Laser CO2 acrylique 3mm : kerf ~0.1-0.2 mm
Dessiner à 14.00 mm, résultat 14.05-14.15 mm (switches clippent bien).



---

## 8. Roadmap v1 → v5

### Approche multi-front

Développement parallèle (hardware, Pi, Teensy, UI, doc). Timeline ~3 ans pour v1-v4.

### v1 — Overlay MIDI complet (12-15 mois)

**Hardware** :
- Panneau 45×37cm FabLab
- Teensy 4.1 + Pi 5 + écran 7"
- Teensy Audio Shield pour master + casque
- Pad 32 MX (Phase 4) ou tact 6×6 (Phase 1-3)
- 9 OLEDs (8 SH1107 voix + 1 SSD1306 master)
- 64+ encodeurs KY-040
- 80+ boutons
- 15 MCP23S17 en chaîne SPI
- Jacks audio v2+ dessinés non percés (sauf master + casque)

**Software** :
- Hot-swap voix/kits
- Multi-LFO (32 modulations simultanées)
- Arpégiateur par voix
- Augmentations 16/32/64/128 steps
- Modulation Engine unifié
- Monitoring CPU
- UI Flask + écran tactile 7"
- Save/Load projets
- MIDI clock sync (esclave Perkons/DAW)
- Probability/ratchet sur éléments PërKompanion
- Architecture contrôle 2 axes
- Voix virtuelles V5-V8 placeholders
- Autosave trois niveaux
- Mode dégradé

**Pas en v1** :
- DSP par voix → v2
- Hotcues complets → v3
- Séquenceur par hotcue → v4
- Intégration audio Perkons → v2
- REC motions complet → v2

**Publishable** comme projet complet.

### v2 — DSP par voix + audio intégré (+6-8 mois)

**Hardware** :
- Codec PCM3168A sur PCB custom (KiCad + JLCPCB)
- 17 jacks audio montés
- Perkons entre dans PërKompanion

**Software** :
- FX chains par voix
- Master FX
- Modulation Engine étendu
- Augmentations DSP
- Monitoring CPU par effet
- **REC motions complet**
- I/O audio 4 in + 10 out

**Gros saut qualitatif** — processeur audio pro.

### v3 — Hotcue complet (+3-6 mois)

**Software** :
- 128 hotcues globaux bibliothèque
- Paramètres Start/Length/Pitch/Feedback
- Quantize configurable
- Loop/One-shot/N-repeat
- Vues Overview + Detail
- Jauge PSRAM
- Persistence (WAV + JSON)
- **Assignation hotcue → V5-V8**
- **Lock hotcues**
- **Polyphonie ajustable**
- Modulation basique par hotcue
- FX chain simple par hotcue
- **Premier profile externe additionnel** (Digitakt II, Hydrasynth, ou
  autre selon disponibilité et besoins de la communauté) — première
  itération multi-machine de PërKompanion.

**PërKompanion devient sampleur complet** et **multi-machine**.

### v4 — Séquenceur et modulation par hotcue (+6-9 mois)

**Software** :
- Séquenceur par hotcue (16/32 steps, probability, ratchet, tie) via pad M5
- Arpégiateur par hotcue
- DSP chain complète par hotcue (3-4 effets)
- Step Editor au pad
- Presets traitement par hotcue
- Pattern chain hotcues
- **Variations automatiques** (pitch-shift, time-stretch, reverse, glitch)
- **Sample packs communautaires** (téléchargement Wi-Fi, format .pkmp)

**Super-séquenceur** avec 128 voix virtuelles.

### v5 — Extensions et écosystème (+6-12 mois)

- Extensions audio optionnelles (MIDI THRU x4, CV/Gate, inserts par voix)
- Marketplace presets et sample packs
- Intégration DAW
- Jams réseau multi-PërKompanion

---

## 9. Sample packs communautaires (v4+)

### Concept

Plateforme de partage :
- Sample packs de hotcues
- Voice Kits Perkons (CC presets)
- Patterns modulations/augmentations
- DSP chains presets (v2+)

### Format .pkmp (ZIP)

- `manifest.json` : métadonnées
- `voice_kits/*.json`
- `hotcues/*.wav + .json`
- `modulations/*.json`
- `pad_layouts/*.json`
- `cover.png`

### Distribution

**Site officiel** : perkompanion.com/sounds (hypothétique)
- Catalogue free/libre
- Upload utilisateurs
- Tags par genre
- Téléchargement Wi-Fi

**Philosophie** : 100% gratuit/open au début. Marketplace pro plus tard si décolle.

---

## 10. Mode dégradé et résilience

### Trois niveaux autosave

1. Par action critique (change de kit, load)
2. Périodique (60s en jam)
3. Arrêt propre

### Fallback hardware

- **Pi tombe** : Teensy continue (LFO, Arp, Augmentations, Motions). Pas d'UI mais jouable.
- **Teensy tombe** : Perkons reste autonome. Pi affiche état.
- **Tous tombent** : Perkons = Perkons. Autosave préservé.

### Principe free party

**L'instrument reste jouable en mode dégradé. C'est une règle.**

---


### Mode dégradé multi-profile (v1.0.0+)

Avec le pivot framework, le mode dégradé du Teensy est étendu pour piloter
**le profile actuellement actif**, quel qu'il soit, et plus seulement le
Perkons. Mécanisme :

- Le Teensy garde en EEPROM (4 KB sur Teensy 4.1) un snapshot binaire
  minimal du profile actif : CCs, channels, ranges, courbes par param.
  Ni l'UI ni les noms — uniquement ce qui est nécessaire au pilotage MIDI.
- Mise à jour automatique de l'EEPROM à chaque changement de page côté Pi
  (commande `SET_ACTIVE_PROFILE_FOR_DEGRADED_MODE` 0xC5).
- Validation au boot Teensy via CRC + version.
- **Si EEPROM corrompue ou vide** : fallback automatique sur Perkons
  hardcodé (la machine fondatrice est toujours présente dans le setup
  Sauvignac et reste un filet de sécurité ultime).
- Indicateur visuel sur OLED master pour signaler quel mode dégradé est
  actif (selon profile X / fallback Perkons hardcodé).

Cette extension préserve la philosophie initiale du mode dégradé (rester
jouable même si le Pi tombe) tout en accommodant la pluralité de devices
gérés à partir de v3.
## 11. Budget v1 global

### Résumé par fournisseur

| Source | Montant |
|--------|---------|
| Amazon FR | ~180€ |
| Kubii.fr | ~105€ |
| AliExpress | ~340€ |
| TME (nouveau — ICs authentiques + jacks Neutrik) | ~210€ |
| ProtoSupplies USA | ~110-130€ |
| FabLab Poitiers (Phase 4) | ~50€ |
| Frais divers (ports + boîtier final) | ~70-120€ |
| **TOTAL projet complet v0.8** | **~1065-1135€** |

Le projet complet avec **anticipation finale v2+/v3+** (tous les jacks audio, tous les modules PCM1808/PCM5102A, ampli casque TPA6120, qualité ICs authentiques Mouser/TME, jacks Neutrik pro) sort à environ **~1100€**.

### Si budget plus serré (Phase 1 minimum)

On peut reporter à plus tard les 5× PCM5102A supplémentaires (ne garder que 1 pour Master), les 2× PCM1808 (entrées audio uniquement en Phase 2), et les jacks Neutrik (passer sur no-name AliExpress). Économie ~150€ → projet démarré à **~950€**.

**Recommandation** : commander tout d'un coup pour éviter les rounds AliExpress successifs (10-15 jours de livraison à chaque fois).

---

## 12. Contraintes et limites connues

### Perkons
- Séquenceur hardware 16 steps max
- Probability/Ratchet non exposés en MIDI (hardware only)
- Clock esclave en v1
- Pot Catch problématique (évité via paramètres non-dupliqués)

### Teensy 4.1
- 2 bus I²S natifs (SAI1 + SAI2) → 8 canaux out sur SAI1 + 4 canaux out sur SAI2 = **12 canaux out max**
- 55 GPIO utilisables → **20 MCP23S17** en chaîne SPI pour encodeurs/boutons/LEDs
- 32 MB PSRAM → ~100-200 hotcues selon qualité/durée

### SH1107 128×128
- Module 34 mm → colonnes voix 40 mm
- SPI partagé OK à 20 MHz (50+ fps possible sur 8 OLEDs)

### Écran 7" Elecrow
- Raw panel : trous de fixation à mesurer à réception
- 2 câbles (HDMI + USB tactile)
- Alim USB 500mA depuis Pi

### Audio
- **v1 minimum** : Master stéréo uniquement (1 PCM5102A + Teensy Audio Shield de secours pour casque)
- **v1 complet** : 12 canaux out (6 PCM5102A, anticipation finale)
- **v3+ futur** : réintroduction PCM3168A pour capture multicanal simultanée (nécessite PCB custom)

---

## 13. Inventaire matériel v1 complet (BOM finale)

### Amazon FR (~180€)

**Interface utilisateur et hardware base** :
- Elecrow 7" tactile 1024×600 (ASIN B07H79XMLT) — 52€
- SD SanDisk Extreme PRO 64 Go A2 — 20€
- Lecteur SD USB — 10€
- 1× OLED SH1107 128×128 SPI 1.5" (test) — 8€
- 1× OLED SSD1306 0.96" I²C (master) — 5€
- 4× Breadboards grandes 830 points — 12€
- Pack câbles Dupont mixtes (F/F, M/M, F/M, 120 pcs) — 12€
- Pack résistances E24 assorties 1/4W — 10€

**Outillage (si non possédé)** :
- Multimètre digital — 20€
- Fer à souder + étain + flux — 30€

**Volume casque** :
- Potentiomètre linéaire 10K (volume casque) — 2€

**Total Amazon FR** : **~181€** (ou ~131€ si outillage déjà possédé)

### Kubii.fr (~105€)

- Raspberry Pi 5 4GB — 70€
- Alim officielle Pi 5 27W USB-C — 15€
- Boîtier officiel Pi 5 avec ventilo — 12€
- Câble micro-HDMI → HDMI — 8€

**Total Kubii** : **~105€**

**Note** : Teensy Audio Shield retiré (remplacé par architecture modulaire PCM1808/PCM5102A).

### TME (~210€) — NOUVEAU fournisseur pour composants critiques

**ICs authentiques (ne pas prendre sur AliExpress)** :
- 22× MCP23S17-E/SP DIP-28 (20 utiles + 2 spare) — 40€
- 2× H11L1M opto-coupleur DIP-6 (MIDI IN) — 3€

- 3× NE5532P DIP-8 (op-amp audio spare) — 1.50€

**Supports DIP (montage/démontage facile)** :
- 22× support DIP-28 tulip — 9€
- 2× support DIP-6 — 0.30€
- 2× support DIP-14 — 0.50€
- 3× support DIP-8 — 0.45€

**Composants passifs qualité** :
- Pack résistances E24 1/4W 1% métallique (100 pcs assorties) — 5€
- 100× résistances 10 KΩ 1% (pull-ups) — 1.50€
- 100× résistances 220Ω 1% (LEDs) — 1.50€
- 10× résistances 33Ω 1% (MIDI OUT) — 0.30€

- 50× condensateurs électrolytiques 10 µF / 16V radial (Panasonic/Rubycon) — 5€
- 100× condensateurs céramiques 100 nF X7R (découplage) — 3€
- 20× condensateurs électrolytiques 47 µF / 16V (bulk audio) — 3€
- 10× condensateurs électrolytiques 100 µF / 16V — 2€

**Jacks Neutrik qualité pro** :
- 14× Neutrik NRJ6HF-ADAM jack 6.35mm PCB (entrées + sorties audio) — 42€
- 1× Neutrik NRJ6HF casque TRS — 3€

**Connecteur MIDI** :
- 2× connecteur DIN 5 broches femelle PCB mount (Lumberg/Cliff) — 5€

**Connecteurs modulaires** :
- 20× bornier vis 2 pôles pas 5.08mm — 4€
- 10× bornier vis 3 pôles — 2.50€
- 10× Molex KK 2.54mm 4 pins — 5€
- 10× Molex KK 2.54mm 6 pins — 6€
- 10× Molex KK 2.54mm 10 pins — 8€

**Câblage propre** :
- Kit fil solid core 22 AWG 5 couleurs × 10m — 15€
- Kit fil multibrin 24 AWG 5 couleurs × 10m — 12€
- Straps précoupés breadboard/perfboard (pack 140 pcs) — 5€

**LEDs qualité** :
- 20× LED 3mm rouge Kingbright — 2€
- 20× LED 3mm verte — 2€
- 10× LED 3mm jaune — 1€
- 10× LED 3mm bleue — 2€
- 10× LED 3mm blanche — 2€

**Port** : ~10-15€

**Total TME** : **~210€**

### AliExpress (~340€, livraison 10-15j)

**OLEDs et écrans** :
- **9× OLED SH1107 128×128 SPI 1.5"** (8 voix + 1 spare, 3 packs de 3) — 50€
- **1× OLED SSD1306 128×64 SPI 7-pin** (master, v0.9 SPI au lieu de I²C) — 3€

**Encodeurs et potentiomètres** :
- 4× packs 20× encodeurs KY-040 avec switch (80 unités) — 24€
- 20× knobs 15mm (master + casque + divers) — 10€
- 16× knobs 18mm (MODE + USER des 8 colonnes voix) — 16€
- 48× knobs 12mm (contextuels E1-E6 des 8 colonnes voix) — 29€

**Boutons tactiles et finaux** :
- Pack 200× tact 6×6mm avec LED pas 2.54mm (proto + finaux non critiques) — 10€
- Pack 20× tact 12×12mm pas 2.54mm (gros boutons proto) — 5€
- 20× SJMS 8×8mm avec LED (boutons voix tête + LEDs activité) — 15€
- _Cherry MX + keycaps : voir section "Switches et keycaps v0.9" ci-dessous_

**Expanders et multiplexers** :
- 2× module TCA9548A breakout (I²C multiplexer) — 5€

**Matrice pad et divers passifs** :
- 200× diodes 1N4148 (matrice pad + divers) — 4€
- 100× condensateurs céramiques 100 nF (supplémentaires) — 2€

**Modules audio (anticipation complète v2+/v3+)** :
- 2× module PCM1808 breakout (ADC stéréo 24-bit/96kHz) — 24€
- 6× module PCM5102A breakout (DAC stéréo 24-bit/384kHz) — 60€
- 1× module TPA6120 breakout MCU-612 (ampli casque 700mW) — 8€

**LEDs et éclairage pad (v0.9)** :
- **100× LED 3mm blanc diffusé** (dans slots Cherry MX2A) — 3€
- Level shifter 74AHCT125N conservé comme spare logique (plus nécessaire pour NeoPixel)
- ~~1× NeoPixel WS2812B strip/ring 32 LEDs~~ **RETIRÉ v0.8 final, confirmé v0.9**

**Switches et keycaps (v0.9 — pack économique)** :
- **70× Cherry MX2A Brown Hyperglide lubed 5-pin avec slot LED** — 30€ (pack économique)
- **100× keycaps R4 translucides mix couleurs** (2 lots noirs + 1 lot rouge + **2 lots bleus** pour 9×5) — 30€ (pack économique)

**Connectique et câblage** :
- 10× headers mâle droits 2.54mm (40 pins × 10) — 5€
- 10× headers femelle droits 2.54mm — 8€
- 5× headers mâle coudés 2.54mm — 4€
- 20× connecteurs JST XH 2.54mm — 6€
- 2m câble plat 40 conducteurs — 6€
- 1 pack gaine thermorétractable assortie — 5€
- 5m gaine tressée noire (finition pro) — 5€

**Consommables** :
- 5× PCB perfboard 9×15cm (pour 5 cartes MCP) — 10€
- 50× vis M2×8mm (fixation OLEDs) — 3€
- 20× vis M3×10mm (fixation plate + Pi + Teensy) — 2€
- 20× entretoises M3 hexagonales 10mm — 3€
- 20× entretoises M2 hexagonales 6mm — 2€

**Total AliExpress** : **~340€**

### ProtoSupplies USA → MyUS → France (~135-155€ v0.9)

**v0.9** : ProtoSupplies ne livre pas en France. On passe par **MyUS.com** (transitaire US → France).

- Teensy 4.1 Fully Loaded 32MB PSRAM chez ProtoSupplies : **64€** (port US local inclus vers MyUS) ✅ commandé
- Réexpédition MyUS → France via **FedEx** : ~40-50€ (Priority 3-5j) ou ~30€ (Economy 5-8j)
- TVA France 20% sur (Teensy + port) : ~20€
- Frais dossier FedEx France : ~15-20€

**Total Teensy rendu France** : **~135-155€**

**Alternative évaluée et rejetée** : Teensy standard Gotronic + soudure PSRAM maison (Mouser) = ~90€ + investissement matos CMS ~90€. Pragmatique mais plus long. Choix ProtoSupplies via MyUS pour produit testé, soudé pro, opérationnel d'emblée.

### Sculpteo (~110€, découpe laser en ligne)

- Plate test 15×15 cm noir mat (validation dimensions avant plate finale) — 20€
- Plate principale 450×370 mm acrylique 3mm noir mat avec gravures — 70€
- Module pad 190×115 mm peuplier 5mm (v0.9) — 20€

**Total Sculpteo** : **~110€**

Sculpteo remplace l'option FabLab Poitiers initialement envisagée (distance 30 km, contrainte de déplacement).

### Frais divers à provisionner

- Port Amazon (si pas Prime) — 10€
- Port Kubii — 10€
- Port AliExpress — inclus généralement
- Port TME — inclus dans les 210€
- Port ProtoSupplies US → MyUS — inclus dans les 64€
- Frais FedEx MyUS → France + TVA + dossier — ~70-90€ (inclus dans ligne ProtoSupplies ci-dessus)
- Boîtier/châssis final (Phase 4, bois/métal) — 50-100€

**Total frais** : **~70-120€**

---

### GRAND TOTAL v0.9 complet avec anticipation v2+/v3+

| Fournisseur | Total |
|-------------|-------|
| Amazon FR (corrigé sans Pi 5) | ~150€ |
| Kubii.fr | ~90€ |
| AliExpress (avec ajustements v0.9) | ~350€ |
| TME (avec Neutrik) | ~200€ |
| ProtoSupplies + MyUS + FedEx | ~135-155€ |
| Sculpteo | ~110€ |
| Frais divers | ~70-120€ |
| **TOTAL v0.9** | **~1105-1175€** |

**~1100-1170€** pour un PërKompanion v0.9 complet avec :
- Architecture audio anticipée (4 in + 12 out + casque)
- 20 MCP23S17 authentiques
- 14 jacks Neutrik pro
- **Module violet 9×5 Cherry MX** (45 touches) + boutons partout en Cherry MX uniforme
- **Teensy 4.1 Fully Loaded 32MB PSRAM** testé et monté pro
- **9 OLEDs SPI** (8 voix SH1107 + 1 master SSD1306)
- Plate test + plate finale + module violet Sculpteo
- Tous les consommables, vis, entretoises, câbles, headers
- Outils de base si pas déjà possédés

---


### Note v1.0.0 — étiquetage des jacks d'entrée audio

Avec le pivot framework et la roadmap multi-machine v2-v4, les 4 jacks
d'entrée audio (PCM1808 v1, multi-canal v3+) sur la tranche arrière de la
plate doivent être **étiquetés génériquement** :

> ✅ **IN 1 / IN 2 / IN 3 / IN 4** (gravure laser FabLab)
> 
> ❌ ~~V1 IN / V2 IN / V3 IN / V4 IN~~ (figerait conceptuellement les
> entrées sur les voix Perkons)

À propager dans `panel_design.md` avant l'envoi de la plate en gravure
FabLab. Pas urgent (la plate ne part pas avant validation prototype),
mais à ne pas oublier.
## Conclusion

PërKompanion v0.8 consolide la vision d'un instrument live complet pour le Perkons HD-01 :
- **Architecture 8 voix physiques** (4 Perkons + 4 virtuelles)
- **Pad polymorphe** avec 5 modes
- **Bibliothèque hotcues scalable** à 128 globaux
- **Architecture audio modulaire** (PCM1808 + PCM5102A + TPA6120) constructible solo
- **DSP par voix modulable** (4 slots parmi 13 types d'effets, 224 params total)
- **Roadmap claire** vers processeur audio multicanal v3+ (PCM3168A sur PCB custom)

**Prototypage breadboard** valide la logique firmware avant investissement final.

**Plate Sculpteo dessinée dès le début** anticipe v2+ et v3+ (tous les jacks audio, MIDI, digital) sans refonte future.

**Faisable par un non-électronicien** grâce à :
- Prototypage tact 6×6 (pas de CMS en Phase 1-3)
- Modules breakout pour audio (PCM1808, PCM5102A, TPA6120)
- MCP23S17 en DIP-28 (soudable à la main sans difficulté)
- Plate acrylique Sculpteo (pas de PCB mécanique complexe)
- Anticipation v2+/v3+ (pas de refonte plate)

**Budget v1 complet : ~1065-1135€**

**Philosophie** : l'instrument reste jouable en mode dégradé. Le Perkons reste un Perkons. PërKompanion enrichit sans jamais remplacer. Constructibilité solo prime sur perfection technique.
