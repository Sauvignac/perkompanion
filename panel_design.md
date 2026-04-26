# PërKompanion — Panel Design

> Layout détaillé du panneau hardware avec dimensions exactes
> Document technique d'implémentation — **v0.9** — avril 2026

---

## Préambule

Ce document est le **référentiel mécanique** du panneau PërKompanion. Il sert à :
- Dessiner la plate dans AutoCAD pour découpe laser chez **Sculpteo** (en ligne)
- Positionner précisément chaque composant
- Valider les encombrements avant usinage
- Documenter les dimensions critiques

**Fournisseur v0.9** : **Sculpteo** (découpe laser acrylique en ligne, France).

---

## Changements clés v0.8 final → v0.9

**Restructuration du module violet** :
- **Pad étendu 9×5 dans le module violet** (au lieu de 8×4 seul)
- **45 Cherry MX dans le module violet** (32 pad + 8 voix V1-V8 rangée haut + 5 modes M1-M5 colonne gauche)
- **Module violet 190×115 mm** (au lieu de 180×100 mm)
- **Ergonomie optimisée** : mains naturellement positionnées pour live (pouce gauche sur M, pouce droit sur V, trigger pad avec la main droite)

**Boutons Cherry MX partout** :
- **Uniformisation complète** : tous les boutons en Cherry MX2A Brown Hyperglide
- **LED 3mm blanche** dans chaque switch (slot LED intégré)
- **Keycaps R4 translucides colorés** selon la fonction (noir/rouge/bleu)
- **70 switches** au total utilisés

**OLEDs uniformes SPI** :
- **9 OLEDs au total** tous en SPI sur bus partagé :
  - 8 OLEDs voix SH1107 128×128 1.5" SPI
  - 1 OLED master SSD1306 128×64 1.3" SPI
- **Bus I²C totalement libéré** pour extensions futures
- **TCA9548A** conservé en réserve mais non utilisé en v0.9

**Bande centrale étendue** :
- **7 boutons** au lieu de 5 (5 globaux + 2 réservés pour évolution future)

**Architecture audio confirmée v0.8 final** :
- 2× PCM1808 breakout (4 canaux in Perkons)
- 6× PCM5102A breakout (12 canaux out)
- 1× TPA6120 module MCU-612 (ampli casque)
- Tranche arrière : 21 jacks (4 in + 12 out + 2 MIDI + 3 digital)

---

## Identité visuelle v0.9

Le panneau PërKompanion est conçu comme un **instrument lumineux tekno/industriel** à l'identité visuelle forte.

### Architecture 2 couches

**Couche 1 — Plate principale** :
- **Matériau** : acrylique 3 mm **noir mat** sérigraphié/gravé
- **Dimensions** : 450 × 370 mm
- **Fonction** : support de tous les composants, toutes les découpes, toute la signalétique gravée
- **Gravures** : logo PërKompanion, légendes des boutons (M1-M5, V1-V8, SHIFT, PANIC, etc.), numérotation des encodeurs, marques de repère

**Couche 2 — Module pad violet** :
- **Matériau** : acrylique 3 mm **violet translucide satiné**
- **Dimensions** : **190 × 115 mm** (révisé v0.9 pour accueillir le pad 9×5)
- **Position** : surélevé de 10 mm au-dessus de la plate principale, coin haut-gauche
- **Fonction** : identité visuelle tekno/industriel (décoratif pur en v0.9, pas de fonction lumineuse)
- **Découpe** : 45 trous carrés 18×18 mm alignés sur les switches Cherry MX (9 colonnes × 5 rangées, pitch 19.05 mm)
- **Fixation** : 4 vis M3×25mm + 4 entretoises M3×10mm aux 4 coins

### Composition du module violet 9×5

**Le module violet abrite 3 zones fonctionnelles intégrées** :

```
         C0   C1   C2   C3   C4   C5   C6   C7   C8
       ┌────┬────┬────┬────┬────┬────┬────┬────┬────┐
R0     │ M1 │ V1 │ V2 │ V3 │ V4 │ V5 │ V6 │ V7 │ V8 │ ← Rangée voix (bleu)
       ├────┼────┼────┼────┼────┼────┼────┼────┼────┤
R1     │ M2 │ P0 │ P1 │ P2 │ P3 │ P4 │ P5 │ P6 │ P7 │
R2     │ M3 │ P8 │ P9 │P10 │P11 │P12 │P13 │P14 │P15 │ ← Pad 8×4 (noir)
R3     │ M4 │P16 │P17 │P18 │P19 │P20 │P21 │P22 │P23 │
R4     │ M5 │P24 │P25 │P26 │P27 │P28 │P29 │P30 │P31 │
       └────┴────┴────┴────┴────┴────┴────┴────┴────┘
         ↑                                              
         └─── Colonne modes (rouge)                     
```

**Répartition** :
- **32 boutons pad** (rangées R1-R4, colonnes C1-C8) : keycaps noirs translucides
- **8 boutons voix** V1-V8 (rangée R0, colonnes C1-C8) : keycaps bleus translucides
- **5 boutons modes** M1-M5 (colonne C0, rangées R0-R4) : keycaps rouges translucides

**Total : 45 Cherry MX dans le module violet.**

### Ergonomie du layout 9×5

**Position naturelle des mains** :
- **Main gauche** : **pouce sur M1-M5** (changement de mode du pad en un clic)
- **Main droite** : **pouce sur V1-V8** (sélection de voix en un clic)
- **Main droite** : **autres doigts sur le pad 8×4** (trigger drums/hotcues)

**Avantages live** :
- Pas besoin de lever la main droite du pad pour changer de mode
- Pas besoin de regarder pour sélectionner une voix
- Workflow similaire Launchpad/Push/MPC mais optimisé pour Perkons

### Structure 3D du pad

```
   [keycap R4 translucide : noir/bleu/rouge]
        ║
        ║  (keycap dépasse du module violet)
        ║
   ═══╬═══  ← Module violet translucide 3mm (surélevé 10mm, décoratif)
        ║
   ═══╩═══  ← Plate principale noire 3mm (45 trous MX 14×14)
        ║
    [Cherry MX2A Brown avec LED 3mm blanche dans slot dédié]
        ║
        ║  2 pins switch (matrice ou GPIO direct) + 2 pins LED (MCP23S17 dédié)
```

### Rétroéclairage LED 3mm dans switch MX (v0.9)

**Approche retenue** : chaque Cherry MX2A Brown a un **logement interne pour LED 3mm** qui s'insère dessus du mécanisme switch. La LED éclaire le keycap par dessous via le stem (principe rétroéclairage traditionnel clavier gaming).

**Pour chaque switch** :
- **1 LED 3mm blanc diffusé** insérée dans le slot du switch MX
- **2 pins LED** (anode + cathode) qui sortent sous le switch avec les 2 pins contact
- **5 pins au total par switch** : 2 contact + 2 LED + 1 pin de stabilisation central

**Câblage LED** :
- Anode → résistance 220Ω → VCC 5V (rail Pi 5)
- Cathode → GPIO d'un MCP23S17 dédié (sink current : LOW = allumé, HIGH = éteint)

**Pilotage** : ON/OFF par keycap via MCP23S17.

**Pas de couleur variable par touche** : les LEDs sont **toutes blanches**. La couleur perçue est donnée par la **couleur du keycap R4 translucide** (noir, rouge, vert, jaune selon zone à définir à l'usage).

**Effet visuel** :
- LED éteinte : keycap translucide visible mais peu contrasté sur plate noire
- LED allumée : keycap **illuminé de l'intérieur** par la LED blanche qui traverse le keycap translucide coloré
  - Keycap noir translucide + LED blanche = **gris fumé lumineux**
  - Keycap rouge translucide + LED blanche = **rouge vif éclatant**
  - Keycap bleu translucide + LED blanche = **bleu électrique**

### Palette de keycaps par fonction (v0.9)

**Logique de couleurs** (inspirée standards Launchpad / Push / MPC) :

| Zone | Qté | Couleur keycap R4 translucide | Fonction |
|------|-----|-------------------------------|----------|
| **Pad 8×4** (dans module violet) | 32 | **Noir** translucide | Triggers (drums, hotcues, augmentations, patterns) |
| **Voix V1-V8** (rangée haut du module violet) | 8 | **Bleu** translucide | Sélection cible (voix pour le pad) |
| **Modes M1-M5** (colonne gauche du module violet) | 5 | **Rouge** translucide | Changement de mode du pad |
| **Voix tête colonne** (hors module violet) | 8 | **Bleu** translucide | Sélection voix pour édition |
| **MODE GLOBAL master** ([LFO][DSP][ARP][HOT]) | 4 | **Bleu** translucide | Sélection de cible (mode d'édition) |
| **Boutons globaux bande centrale** (SHIFT, PANIC, FREEZE, CLEAN SLATE, REC + 2 réservés) | 7 | **Rouge** translucide | Contrôle critique / modes futurs |
| **Boutons écran B1-B6** (au-dessus écran 7") | 6 | **Noir** translucide | Fonctions contextuelles écran |

**Total keycaps Cherry MX R4 translucides (v0.9)** :
- **Noirs** : 32 (pad) + 6 (B1-B6) = **38** → **40 achetés** (2 lots de 20)
- **Rouges** : 5 (M1-M5) + 7 (bande centrale) = **12** → **20 achetés** (1 lot de 20)
- **Bleus** : 8 (voix module) + 8 (voix tête colonne) + 4 (MODE GLOBAL) = **20** → **40 achetés** (2 lots de 20)
- **Total utilisés** : **70 keycaps**
- **Total commandés** : **100 keycaps** (30 spare)

**Total Cherry MX2A Brown Hyperglide (v0.9)** :
- **70 switches utilisés** au total
- **Achetés** : 70 MX2A Brown lubed (30€ pack économique)
- **Spare recommandé** : +10 switches supplémentaires (~5€) pour marge casse au montage

**Total LEDs 3mm blanches** :
- **70 utilisées**
- **Commandées** : 100 LEDs (~3€ pack économique)
- **Spare** : 30 LEDs

### Boutons sans Cherry MX (v0.9 : AUCUN)

**Note v0.9** : **tous les boutons sont en Cherry MX** pour cohérence visuelle et ergonomique totale. Pas de tact 6×6 ni SJMS 8×8 en v0.9 (sauf sur perfboards proto internes, pas sur la plate finale).

### Cohérence avec l'univers Perkons HD-01

Le PërKompanion reprend les **codes visuels du Perkons** :
- **Noir mat** de la face principale
- **Accents lumineux** (ici RGB adressable via NeoPixel, là LEDs multicolores Perkons)
- **Sobriété + caractère industriel** : suie, mazout, friction

Mais y ajoute son **identité propre** via :
- **Accent violet translucide** (unique au PërKompanion, non présent sur Perkons)
- **Matrice lumineuse 8×4** (logique contrôleur grid, complémentaire du Perkons linéaire)

---

## 1. Dimensions générales

### Panneau principal

- **Largeur** : 450 mm (45 cm)
- **Hauteur** : 370 mm (37 cm)
- **Épaisseur** : 3 mm (acrylique)
- **Matériau v0.9** : **acrylique 3 mm noir mat** (direct, pas de version transparent prototype intermédiaire)
- **Fabrication** : Sculpteo (découpe laser + gravures en ligne)

### Module pad violet (v0.9 — 9×5)

- **Largeur** : **190 mm** (au lieu de 180 mm en v0.8)
- **Hauteur** : **115 mm** (au lieu de 100 mm en v0.8)
- **Épaisseur** : 3 mm (acrylique)
- **Matériau** : **acrylique 3 mm violet translucide satiné**
- **Surélévation** : 10 mm au-dessus de la plate principale
- **Fixation** : 4 vis M3×25mm + 4 entretoises M3×10mm aux 4 coins
- **Position** : coin haut-gauche, X=10 mm à X=200 mm, Y=10 mm à Y=125 mm
- **Découpes** : 45 trous carrés 18×18 mm (9 colonnes × 5 rangées pitch 19.05 mm) + 4 trous M3 Ø3.2 mm aux coins

### Marges globales

- Marge extérieure : 5 mm (bord du panneau au premier élément)
- Marges de fixation : 10 mm des bords (trous M3 aux 4 coins)

### Alignement Perkons

Le PërKompanion se pose **au-dessus du Perkons HD-01** (dimensions Perkons : 45×35×10cm environ). Le panneau dépasse de 2 cm en hauteur, ce qui est acceptable.

---

## 2. Vue d'ensemble du layout

```
┌──────────────────────────────────────────────────────────────────────────┐
│ ○                                                                    ○  │  ← trous fixation M3
│  [M1]                                                                    │
│  [M2] ┌──────────────────────┐      [B1][B2][B3][B4][B5][B6]             │
│  [M3] │                      │    ┌────────────────────────┐             │
│  [M4] │    PAD 8×4           │ [E1│                        │[E5]         │
│  [M5] │    32 Cherry MX      │    │                        │             │
│       │    pitch 19.05mm     │ [E2│      ÉCRAN 7"          │[E6]         │
│       │    147×71 mm         │    │      Elecrow           │             │
│       │                      │ [E3│      1024×600          │[E7]   LEDs  │
│       │                      │    │      165×124 mm        │       x4-8  │
│       │                      │ [E4│                        │[E8]         │
│       └──────────────────────┘    └────────────────────────┘             │
│                                                                          │
│  [SHIFT][PANIC][FREEZE][CS][REC]                                         │
│                                                                          │
│  [V1] [V2] [V3] [V4]      [V5] [V6] [V7] [V8]                            │
│                                                                          │
│  ┌──┐┌──┐┌──┐┌──┐ ┌──┐┌──┐┌──┐┌──┐  ┌─────────────────────┐              │
│  │OL││OL││OL││OL│ │OL││OL││OL││OL│  │   MASTER ZONE        │             │
│  │MD││MD││MD││MD│ │MD││MD││MD││MD│  │                      │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉V││◉V││◉V││◉V│  │  ◉Vol   ◉Poly        │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │  ◉Rev1  ◉Rev2  ◉RevT │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │  ◉Del1  ◉Del2  ◉DelM │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │  ◉Cmp1  ◉Cmp2        │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │                      │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │ [LFO][DSP][ARP][HOT] │             │
│  │◉ ││◉ ││◉ ││◉ │ │◉ ││◉ ││◉ ││◉ │  │                      │             │
│  └──┘└──┘└──┘└──┘ └──┘└──┘└──┘└──┘  │    OLED BPM          │             │
│   V1  V2  V3  V4   V5  V6  V7  V8   │                      │             │
│  physiques           virtuelles     └──────────────────────┘             │
│                                                                          │
│                                          ◎ jack casque  [◉Vol casque]   │
│ ○                                                                    ○  │  ← trous fixation M3
└──────────────────────────────────────────────────────────────────────────┘
         ←─────────────────────── 450 mm ──────────────────────→
                                                                          
         ↕ 370 mm
```

---

## 3. Zone PAD étendue 9×5 (module violet)

### Composition

Le module violet contient **45 Cherry MX** organisés en grille 9 colonnes × 5 rangées :
- **Rangée R0** (haut) : 1 case spéciale (M1) + **8 boutons voix V1-V8** (keycaps bleus)
- **Colonne C0** (gauche) : **5 boutons modes M1-M5** (keycaps rouges)
- **Pad 8×4** (rangées R1-R4, colonnes C1-C8) : **32 boutons trigger** (keycaps noirs)

**Note** : la case coin haut-gauche **(C0, R0) = M1** (bouton mode). La rangée voix commence à la colonne C1.

### Position sur la plate principale

**Origine du premier trou (M1, coin haut-gauche du module)** : X=20mm, Y=20mm par rapport au coin haut-gauche du panneau.

### Dimensions switches MX

- **Trou switch MX** : **14.00 × 14.00 mm** exactement
- **Pitch centre-à-centre** : **19.05 mm** (standard keyboard)
- **Matériau** : acrylique 3mm (permet le clip mécanique MX correct)

### Keycaps R4 translucides

- **Keycap 1U R4** : 18 × 18 mm
- **Profil** : R4 uniforme (hauteur constante)
- **Matière** : PC ou PBT translucide
- **Espacement visuel entre caps** : 1.05 mm (19.05 - 18)

### Coordonnées précises des 45 trous

**Axe X (9 colonnes)** — centre X de chaque colonne :
- Colonne C0 (M1-M5) : 27.0 mm
- Colonne C1 (V1/P0/P8/P16/P24) : 46.05 mm
- Colonne C2 (V2/P1/P9/P17/P25) : 65.1 mm
- Colonne C3 (V3/P2/P10/P18/P26) : 84.15 mm
- Colonne C4 (V4/P3/P11/P19/P27) : 103.2 mm
- Colonne C5 (V5/P4/P12/P20/P28) : 122.25 mm
- Colonne C6 (V6/P5/P13/P21/P29) : 141.3 mm
- Colonne C7 (V7/P6/P14/P22/P30) : 160.35 mm
- Colonne C8 (V8/P7/P15/P23/P31) : 179.4 mm

**Axe Y (5 rangées)** — centre Y de chaque rangée :
- Rangée R0 (voix + M1) : 27.0 mm
- Rangée R1 (P0-P7 + M2) : 46.05 mm
- Rangée R2 (P8-P15 + M3) : 65.1 mm
- Rangée R3 (P16-P23 + M4) : 84.15 mm
- Rangée R4 (P24-P31 + M5) : 103.2 mm

**Chaque trou** : carré 14×14 mm centré sur (X, Y).

### Dimensions totales zone 9×5

- Largeur (centre C0 à centre C8) : 8 × 19.05 = 152.4 mm
- Avec bords des trous : 152.4 + 14 = **166.4 mm**
- Hauteur (centre R0 à centre R4) : 4 × 19.05 = 76.2 mm
- Avec bords des trous : 76.2 + 14 = **90.2 mm**
- **Zone 45 trous** : 166.4 × 90.2 mm
- **Zone module violet** (avec marges) : 190 × 115 mm

### Trous M3 de fixation module pad violet (v0.9)

**4 trous M3** aux 4 coins du périmètre du module violet (190×115 mm en coin haut-gauche plate) :

- Trou M3 haut-gauche : X = 10 mm, Y = 10 mm (Ø 3.2 mm)
- Trou M3 haut-droit : X = 200 mm, Y = 10 mm
- Trou M3 bas-gauche : X = 10 mm, Y = 125 mm
- Trou M3 bas-droit : X = 200 mm, Y = 125 mm

**Ces trous** accueillent les vis M3×25mm qui traversent le module violet, les entretoises M3×10mm, et la plate noire.

### NeoPixel — ABANDONNÉ (v0.8 final, confirmé v0.9)

**Décision** : **pas de NeoPixel WS2812B** sur le pad. Remplacé par **LEDs 3mm blanches intégrées dans chaque switch Cherry MX2A Brown** (slot LED natif).

**Pourquoi ce choix** :
- **Alignement parfait** LED-keycap via le switch (pas de bricolage de positionnement)
- **Simplicité de câblage** : 1 GPIO MCP23S17 par LED (vs chain adressable)
- **Budget réduit** : LEDs 3mm ~3€ (100pcs) vs matrice WS2812B ~15€
- **Feedback ON/OFF** suffisant pour les besoins du PërKompanion (pas de RGB adressable par touche nécessaire avec keycaps colorés par zone)

**Le module violet translucide** est **conservé** pour l'identité visuelle (rôle décoratif pur, pas de diffusion NeoPixel).

### Câblage matrice pad (8×4)

8 colonnes × 4 rangées = 32 switches en matrice.
- 8 GPIO pour colonnes (output, pull-down)
- 4 GPIO pour rangées (input, pull-up)
- 32 diodes 1N4148 (1 par switch, anode vers switch)

**Câblage via MCP23S17** : MCP 11 dédié à la matrice pad (voir `teensy_pinout.md` section 4).

**Note v0.9** : la matrice couvre uniquement les 32 boutons du pad trigger. Les 8 voix V1-V8 et 5 modes M1-M5 du module violet sont câblés **individuellement** sur GPIO dédiés (pas de matrice) pour éviter tout ghosting critique sur les boutons de sélection.

### Câblage LEDs module violet (v0.9)

**45 LEDs 3mm blanches** dans les switches MX du module violet 9×5, contrôlées en ON/OFF :

- **3 MCP23S17 dédiés** aux LEDs module violet :
  - MCP "LEDs pad 1" : 16 LEDs pad (rangées R1-R2) — 16 GPIO
  - MCP "LEDs pad 2" : 16 LEDs pad (rangées R3-R4) — 16 GPIO
  - MCP "LEDs V+M" : 13 LEDs voix + modes (8 V1-V8 + 5 M1-M5) — 13 GPIO (+3 spare)
- Alimentation : 5V (rail Pi 5) via résistance 220Ω par LED
- **Current sink** : cathode LED → GPIO MCP, GPIO LOW = allumé, GPIO HIGH = éteint
- Courant par LED : ~8-10 mA
- **Current total max** : 45 × 10 mA = 450 mA (typique : 100-200 mA en usage live)

**LEDs hors module violet** (Cherry MX autres zones) :
- 7 LEDs bande centrale (globaux) + 6 LEDs B1-B6 écran + 8 LEDs voix tête colonne + 4 LEDs MODE GLOBAL master = **25 LEDs**
- Réparties sur **2 MCP23S17 supplémentaires** (25 GPIO sur 32 disponibles)

**Total LEDs 3mm blanches** : 45 + 25 = **70 LEDs**, réparties sur **5 MCP23S17 dédiés LED** (sur les 20 MCP de l'architecture).

**Pas de câblage direct Teensy** : les pins 2-9 du Teensy sont réservés au SAI1/SAI2 audio.

---

## 4. Boutons M1-M5 (intégrés dans module violet v0.9)

**v0.9 : les 5 boutons M1-M5 sont intégrés dans la colonne C0 du module violet 9×5** (voir section 3).

Leur positionnement est donc :
- M1 : (C0, R0) → X = 27.0 mm, Y = 27.0 mm
- M2 : (C0, R1) → X = 27.0 mm, Y = 46.05 mm
- M3 : (C0, R2) → X = 27.0 mm, Y = 65.1 mm
- M4 : (C0, R3) → X = 27.0 mm, Y = 84.15 mm
- M5 : (C0, R4) → X = 27.0 mm, Y = 103.2 mm

### Spécifications v0.9

- **Switches** : Cherry MX2A Brown Hyperglide (intégrés dans le module violet comme le reste du pad)
- **Keycaps** : R4 translucides **rouges** (indique "changement de mode structurel")
- **Trous plate** : 14 × 14 mm (dans les 45 trous du module violet)
- **LED** : 3mm blanche dans le slot du switch MX
- **Câblage** : 5 GPIO directs sur MCP23S17 (pas dans la matrice pad, boutons isolés pour réactivité immédiate)
- **Feedback LED** : LED 3mm contrôlée ON/OFF via MCP23S17 dédié, indique mode actif

**Ergonomie** : M1-M5 accessible avec le **pouce gauche** sans bouger la main droite qui reste sur le pad.

---

## 5. Écran 7" Elecrow

### Position

À droite du pad, avec marge.

**Centre de l'écran** :
- X ≈ 280 mm (à droite du pad)
- Y = 80 mm (au milieu de la section haute)

### Dimensions module complet

- **Zone visible** : ~155 × 90 mm
- **Module PCB complet** : ~165 × 124 mm
- **Trous de fixation** : 4 trous M3 aux 4 coins du PCB (à mesurer précisément à réception)

### Découpe

**Découpe rectangulaire** dans la plate :
- Largeur : 155 mm (zone visible)
- Hauteur : 90 mm
- Position : X = 200 mm à 355 mm, Y = 35 mm à 125 mm

**Trous de fixation** autour de la découpe :
- 4 trous Ø 3.2 mm (passage vis M3)
- Position à mesurer sur le PCB de l'écran
- Typiquement : décalage de ~5 mm par rapport à la découpe

### Connectique

L'écran utilise :
- 1 câble **HDMI** (du côté arrière)
- 1 câble **USB tactile** (du côté arrière)

Ces câbles passent par **l'arrière du panneau** vers le Pi 5.

---

## 6. Boutons B1-B6 au-dessus de l'écran

### Position

Au-dessus de la découpe de l'écran.

- **6 boutons** en ligne horizontale
- **Espacement** : ~22 mm centre-à-centre (répartis sur la largeur de l'écran)
- **Centre Y** : 20 mm (au-dessus de la découpe écran)

### Coordonnées

Centres X des 6 boutons (espacés ~22 mm) :
- B1 : 210 mm
- B2 : 232 mm
- B3 : 254 mm
- B4 : 276 mm
- B5 : 298 mm
- B6 : 320 mm

Centre Y : 20 mm

### Spécifications

- Proto : tact 6×6mm pas 2.54mm avec LED
- Final : tact 6×6 ou SJMS 8×8
- Trous : 6.5 × 6.5 mm ou 8.5 × 8.5 mm

---

## 7. Encodeurs E1-E8 autour de l'écran

### Position

- **4 encodeurs à gauche** de l'écran (E1-E4)
- **4 encodeurs à droite** de l'écran (E5-E8)

### Coordonnées E1-E4 (gauche écran)

Centre X : 188 mm (à gauche de la découpe écran, laissant 10mm d'espace)
Centres Y :
- E1 : 45 mm
- E2 : 70 mm
- E3 : 95 mm
- E4 : 120 mm

**Pitch vertical** : ~25 mm

### Coordonnées E5-E8 (droite écran)

Centre X : 370 mm
Centres Y : idem E1-E4

### Spécifications

- Encodeur KY-040 (avec switch intégré)
- **Trou passage axe** : Ø 7 mm
- **Knob** : 15 mm de diamètre

---

## 8. LEDs d'activité (haut droite)

### Position

En haut à droite du panneau, à droite de B6.

### Coordonnées

Centres :
- LED1 : X=380 mm, Y=20 mm (MIDI IN)
- LED2 : X=395 mm, Y=20 mm (MIDI OUT)
- LED3 : X=410 mm, Y=20 mm (Clock sync)
- LED4 : X=425 mm, Y=20 mm (Power)
- Espace réservé pour 4 LEDs supplémentaires (Pi activity, Teensy activity, Overload, Record active)

### Spécifications

- LED 3mm travers-trou
- **Trou** : Ø 3.2 mm
- **Pitch** : 15 mm centre-à-centre (espace confortable)

---

## 9. Boutons globaux (bande centrale)

### Position

Entre le pad et les colonnes voix, bande horizontale à Y ≈ 145-155 mm.

### 5 boutons globaux

Ordre proposé (gauche vers droite) :
1. **SHIFT**
2. **PANIC**
3. **FREEZE**
4. **CLEAN SLATE**
5. **REC**
6. **Réserve futur #1** (ex : UNDO, RANDOMIZE)
7. **Réserve futur #2** (ex : COPY, LOCK)

### Coordonnées (v0.9 — 7 boutons)

Centres X (7 boutons, pitch 30 mm pour optimiser l'espace) :
- SHIFT : 40 mm
- PANIC : 70 mm
- FREEZE : 100 mm
- CLEAN SLATE : 130 mm
- REC : 160 mm
- Réserve #1 : 190 mm
- Réserve #2 : 220 mm

Centre Y : 150 mm

**Pitch** : 30 mm (espacé suffisamment pour éviter appui accidentel, réduit de 35 à 30 pour placer 7 boutons dans la bande centrale sans empiéter).

**Note sur le risque PANIC accidentel** : avec pitch 30mm vs 35mm, le risque augmente légèrement. Si tu veux rester à pitch 35mm, la bande centrale occupera 245 mm de large, à valider sur le layout AutoCAD.

### Spécifications v0.9

- **Switches** : Cherry MX2A Brown Hyperglide (même que le reste du panneau)
- **Keycaps** : **R4 translucides rouges** (indique le "mode critique" visuellement)
- **Trous plate** : 14 × 14 mm
- **LED 3mm blanche** dans slot du switch MX
- **Câblage** : chaque bouton connecté à 1 GPIO direct sur MCP23S17 (pas dans la matrice du pad, pour éviter déclenchement accidentel en cas de défaut matrice)

### Différenciation visuelle

**Couleur keycap rouge translucide** unifiée pour les 7 boutons (signale leur fonction critique/mode).

**Feedback LED** via ON/OFF MCP23S17 :
- SHIFT : LED allumée quand modifier actif
- PANIC : LED flash rouge au déclenchement (500 ms)
- FREEZE : LED allumée quand freeze actif
- CLEAN SLATE : LED clignotante avant confirmation
- REC : LED clignotante pendant enregistrement
- Réserve #1 et #2 : LEDs contrôlées par firmware selon fonction attribuée

### Justification de la non-intégration dans le pad

Les 7 boutons globaux sont **physiquement séparés** du module violet pour plusieurs raisons :

1. **PANIC et CLEAN SLATE sont destructifs** : un appui accidentel lors d'un trigger live serait catastrophique
2. **Espacement ergonomique** : pitch 30 mm entre ces boutons vs 19.05 mm dans le pad
3. **Hiérarchie visuelle** : le rouge des keycaps signale le niveau "contrôle global" distinct du noir des triggers
4. **Câblage indépendant** : pas de dépendance à la matrice anti-ghosting du pad

---

## 10. Boutons voix V1-V8 (intégrés dans module violet v0.9)

**v0.9 : les 8 boutons V1-V8 sont intégrés dans la rangée R0 du module violet 9×5** (voir section 3).

Leur positionnement est donc :
- V1 : (C1, R0) → X = 46.05 mm, Y = 27.0 mm
- V2 : (C2, R0) → X = 65.1 mm, Y = 27.0 mm
- V3 : (C3, R0) → X = 84.15 mm, Y = 27.0 mm
- V4 : (C4, R0) → X = 103.2 mm, Y = 27.0 mm
- V5 : (C5, R0) → X = 122.25 mm, Y = 27.0 mm
- V6 : (C6, R0) → X = 141.3 mm, Y = 27.0 mm
- V7 : (C7, R0) → X = 160.35 mm, Y = 27.0 mm
- V8 : (C8, R0) → X = 179.4 mm, Y = 27.0 mm

### Spécifications v0.9

- **Switches** : Cherry MX2A Brown (intégrés dans le module violet)
- **Keycaps** : R4 translucides **bleus** (indique "sélection de cible")
- **Trous plate** : 14 × 14 mm (dans les 45 trous du module violet)
- **LED** : 3mm blanche dans slot MX
- **Câblage** : 8 GPIO directs sur MCP23S17 dédié
- **Feedback LED** : allumé pour la voix active sur le pad

**Ergonomie** : V1-V8 accessible avec le **pouce droit** sans bouger la main droite qui reste sur le pad trigger.

### Cohérence avec MODE GLOBAL master

Les 4 boutons MODE GLOBAL de la zone master utilisent aussi des **keycaps bleus translucides** (même catégorie fonctionnelle "sélection de cible").

### Ancienne rangée V1-V8 hors module — v0.8 final

**Note v0.9** : dans la v0.8 final, les boutons V1-V8 étaient positionnés **dans une bande séparée entre la bande centrale et les colonnes voix** (Y ≈ 185 mm). Cette configuration est **abandonnée en v0.9** au profit de l'intégration dans la rangée R0 du module violet, plus ergonomique.

---

## 11. Colonnes voix (8 colonnes)

### Position

Sous la bande centrale, de Y ≈ 175 mm à Y ≈ 370 mm.

### Dimensions par colonne

- **Largeur** : 40 mm (dictée par l'OLED SH1107 de 34 mm + marges)
- **Hauteur totale** : ~195 mm
- **8 colonnes** : V1-V4 physiques + V5-V8 virtuelles

### Gap entre V4 et V5

**Décision v0.8** : pas de gap physique (économie de place). Juste une **ligne gravée verticale** au laser entre V4 et V5 pour marquer la distinction physique/virtuelle.

### Composition d'une colonne (de haut en bas)

**1. Bouton voix tête colonne** (Y = 185 mm)
- **v0.9** : **Cherry MX2A Brown + keycap R4 bleu translucide** (cohérence avec V1-V8 du module violet)
- Trou plate : **14×14 mm**
- LED 3mm blanche dans slot MX
- Câblage : GPIO direct sur MCP23S17 dédié

**2. OLED SH1107 128×128 SPI 1.5"** (Y = 200-240 mm)
- Découpe rectangulaire : ~30 × 30 mm (zone visible)
- Trous de fixation : 2 à 4 trous M2 (à mesurer à réception)

**3. Encodeur MODE** (Y = 255 mm, knob 18mm)
- Trou Ø 7 mm

**4. Encodeur USER** (Y = 280 mm, knob 18mm)
- Trou Ø 7 mm

**5-10. 6 encodeurs contextuels E1-E6** (knob 12mm chacun)
- Centres Y : 300, 315, 330, 345, 360, ... (pitch 15mm)
- 6 encodeurs × 15mm = 90mm de hauteur
- Trous Ø 7 mm chacun

### Coordonnées X des 8 colonnes (centres)

- Colonne V1 : X = 40 mm
- Colonne V2 : X = 80 mm
- Colonne V3 : X = 120 mm
- Colonne V4 : X = 160 mm
- Colonne V5 : X = 220 mm (saut visuel marqué par gravure)
- Colonne V6 : X = 260 mm
- Colonne V7 : X = 300 mm
- Colonne V8 : X = 340 mm

**Pitch entre colonnes** : 40 mm (sauf V4→V5 = 60 mm pour marquer la distinction).

### Coordonnées précises des éléments par colonne

Pour **chaque colonne voix** (ex. V1 à X=40 mm) :

| Élément | Centre Y | Trou |
|---------|----------|------|
| **Bouton voix tête (Cherry MX v0.9)** | 185 mm | **14×14 mm** |
| Découpe OLED | 200-240 mm (centre 220) | 30×30 mm + 4 trous M2 |
| Encodeur MODE | 255 mm | Ø 7 mm |
| Encodeur USER | 280 mm | Ø 7 mm |
| Encodeur E1 | 300 mm | Ø 7 mm |
| Encodeur E2 | 315 mm | Ø 7 mm |
| Encodeur E3 | 330 mm | Ø 7 mm |
| Encodeur E4 | 345 mm | Ø 7 mm |
| Encodeur E5 | 360 mm | Ø 7 mm |
| Encodeur E6 | 375 mm → **NON, dépasse panneau** — à ajuster |

**Attention v0.9** : avec un bouton Cherry MX 14×14 mm à la place d'un tact 6.5×6.5 mm en tête de colonne, la hauteur disponible se réduit. Si le dernier encodeur E6 dépasse de la plate, il faudra :
- Soit réduire le pitch des encodeurs contextuels à 14 mm (au lieu de 15 mm)
- Soit réduire l'espace entre les encodeurs MODE/USER et les contextuels
- Soit remonter la zone colonne voix plus haut (Y début à 175 mm au lieu de 185 mm)

**À valider au dessin AutoCAD**.

**Note** : les 6 encodeurs contextuels ont un pitch de 15 mm (ou 14 mm si ajustement), plus serré que les 2 permanents (pitch 25 mm).

---

## 12. Zone Master

### Position

**Sous l'écran**, à droite des colonnes voix.

- X : de 380 mm à 445 mm (65 mm de large)
- Y : de 165 mm à 355 mm (190 mm de haut)

### Composition

**OLED BPM** (SSD1306 128×64 I2C)
- Position : X=400 mm, Y=170-190 mm
- Découpe : 30 × 15 mm
- Trous fixation : 4 trous M2 aux 4 coins

**Encodeurs master** (~10 encodeurs, knobs 15mm) :

| Encodeur | X | Y |
|----------|---|---|
| Volume Master | 390 mm | 200 mm |
| Polyphony Hotcue | 420 mm | 200 mm |
| Reverb Time | 390 mm | 220 mm |
| Reverb Mix | 420 mm | 220 mm |
| Reverb Type | 405 mm | 240 mm |
| Delay Time | 390 mm | 260 mm |
| Delay Feedback | 420 mm | 260 mm |
| Delay Mix | 405 mm | 280 mm |
| Comp Threshold | 390 mm | 300 mm |
| Comp Ratio | 420 mm | 300 mm |

**Trou par encodeur** : Ø 7 mm

**Boutons MODE GLOBAL** (4 boutons) :
- [LFO] : X=390 mm, Y=325 mm
- [DSP] : X=410 mm, Y=325 mm
- [ARP] : X=430 mm, Y=325 mm
- [HOT] : X=410 mm, Y=345 mm

**Trou par bouton** : 6.5 × 6.5 mm (ou 8.5 × 8.5 avec cap)

---

## 13. Jack casque et encodeur volume casque

### Position

**Façade bas-droite** du panneau, en dessous de la zone master.

### Composants

**Jack TRS 6.35mm casque**
- Position : X = 400 mm, Y = 365 mm (au-dessus du bord)
- Trou Ø 10-12 mm (selon modèle jack)

**Encodeur volume casque** (knob 15mm)
- Position : X = 425 mm, Y = 365 mm
- Trou Ø 7 mm

---

## 14. Tranche arrière — jacks audio et MIDI (anticipation finale v0.8)

### Position

Sur la **tranche arrière** du panneau (côté opposé à l'utilisateur).

### Layout final

```
Tranche arrière (450 mm)
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│  [USB][HDMI][ALIM]   [DIN IN][DIN OUT]   ◎◎◎◎   ◎◎◎◎◎◎◎◎◎◎◎◎             │
│   3 digital          MIDI                 4 TS IN  12 TS OUT              │
│                      2 connecteurs         Perkons  Master+V1-V4+V5-V8   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Détail complet des emplacements

**Connecteurs digitaux** (3 connecteurs, montés dès v1) :
- USB-C (Pi 5 data/debug) : X = 20 mm
- Micro-HDMI (Pi 5 → écran externe si désiré) : X = 40 mm
- Alim barrel jack 5V (backup, si pas alim directe Pi 5) : X = 60 mm

**Connecteurs MIDI DIN** (2 connecteurs, montés dès v1) :
- DIN 5 broches IN (MIDI entrée) : X = 85 mm
- DIN 5 broches OUT (MIDI sortie) : X = 110 mm

**4 entrées audio Perkons** (percés dès v1, jacks montés en Phase 2) :
- TS 6.35mm × 4 (Neutrik NRJ6HF-ADAM ou équiv.)
- Espacement : 20 mm centre-à-centre
- Positions : X = 140, 160, 180, 200 mm
- Câblage vers 2× PCM1808 breakout

**Sorties audio** (13 jacks TS + casque en façade, percés dès v1) :

Master et V1-V4 sur SAI1 (4 PCM5102A) :
- **Master L** : TS 6.35mm à X = 230 mm
- **Master R** : TS 6.35mm à X = 250 mm
- **V1 L** : TS à X = 280 mm
- **V1 R** : TS à X = 300 mm
- **V2 L** : TS à X = 320 mm
- **V2 R** : TS à X = 340 mm
- **V3 L** : TS à X = 360 mm
- **V3 R** : TS à X = 380 mm

V4 et V5-V8 sur SAI2 (2 PCM5102A) :
- **V4 L** : TS à X = 400 mm
- **V4 R** : TS à X = 420 mm
- **V5-V8 L** : TS à X = 440 mm (limite panneau)
- **V5-V8 R** : TS à X... → reporter en deuxième rangée ou sur tranche latérale

**Attention** : 12 sorties × 20mm d'espacement = 240 mm + 4 entrées = 80 mm + MIDI = 25mm + digital = 60 mm = **~405 mm**. Juste dans les clous pour 450 mm de largeur panneau.

**Solution compacte** : utiliser un **espacement de 18 mm** entre jacks TS (au lieu de 20 mm). Les jacks Neutrik NRJ6HF font 15 mm de diamètre, 18 mm laisse 3 mm entre corps. Tout tient.

### Stratégie v1 vs v2+ vs v3+

**v1 démarrage (Phase 1-2)** :
- 3 digital : montés (alim Pi, USB, HDMI optionnel)
- 2 MIDI DIN : montés
- 2 Master L+R : montés avec 1× PCM5102A #1
- Autres jacks TS : **percés et bouchés avec caps noirs esthétiques**

**v1 complet (Phase 3+)** :
- Ajout progressif des PCM5102A #2-6 et PCM1808 #1-2
- Montage progressif des jacks correspondants (débouchage des caps)
- Architecture full 4 in + 12 out atteinte

**v3+ (PCM3168A future)** :
- Remplacement du système PCM1808/PCM5102A discret par PCM3168A sur PCB custom
- Les jacks de la plate **restent inchangés** (même routage, même nombre)
- Seule la carte audio change (transition non destructive)

### Casque en façade (bas-droite, monté dès v1)

Voir section 13 :
- Jack TRS 6.35mm (Neutrik NRJ6HF) : X = 400 mm, Y = 365 mm (façade)
- Potentiomètre 10K volume casque : X = 425 mm, Y = 365 mm
- Connecté au module TPA6120 breakout MCU-612

### Total connectique tranche arrière + façade

**Tranche arrière** : 3 digital + 2 MIDI + 4 in audio + 12 out audio = **21 jacks**
**Façade** : 1 casque + 1 potentiomètre volume = **2 éléments**

**Total général** : **23 éléments connectique** à prévoir sur la plate.

---


## 15. Trous de fixation du panneau

### 4 trous M3 aux coins

- Trou 1 : X=5 mm, Y=5 mm
- Trou 2 : X=445 mm, Y=5 mm
- Trou 3 : X=5 mm, Y=365 mm
- Trou 4 : X=445 mm, Y=365 mm

**Diamètre** : Ø 3.2 mm (passage vis M3)

### Fixation interne (à définir en Phase 4)

Pour fixer la plate à un boîtier ou châssis :
- Standoffs M3 aux 4 coins
- Ou rails latéraux (à décider selon boîtier)

---

## 16. Gravures laser

### Recommandations

- **Logo PërKompanion** : en haut à droite, à côté des LEDs (10×20 mm)
- **Séparation V4/V5** : ligne verticale gravée entre colonnes V4 et V5, de Y=180 à Y=360 mm
- **Numéros voix** : V1, V2, ... V8 sous le bouton voix tête de chaque colonne
- **Légendes modes** : LFO, DSP, ARP, HOT à côté des boutons MODE GLOBAL
- **Légendes boutons globaux** : SHIFT, PANIC, FREEZE, CLEAN SLATE, REC
- **Légendes master** : Vol, Poly, Rev, Del, Cmp, etc.

### Profondeur de gravure

- Profondeur : 0.1-0.2 mm dans l'acrylique noir mat
- Donne un rendu blanc-satiné sur fond noir, très lisible

---

## 17. Vérifications avant envoi FabLab

### Check-list

- [ ] Tous les trous switches MX sont exactement **14.00 × 14.00 mm**
- [ ] Le pitch pad est **19.05 mm** centre-à-centre (horizontal ET vertical)
- [ ] Les trous encodeurs KY-040 sont **Ø 7 mm**
- [ ] Les trous tact switches sont **6.5 × 6.5 mm** (corps) ou **8.5 × 8.5 mm** (cap)
- [ ] La découpe écran fait **~155 × 90 mm** (à valider à réception de l'écran)
- [ ] Les 4 trous de fixation écran sont positionnés correctement
- [ ] Les découpes OLED font **~30 × 30 mm** (à valider à réception des OLEDs)
- [ ] Les trous de fixation panneau sont **Ø 3.2 mm** aux 4 coins
- [ ] Les dimensions générales sont **450 × 370 mm**
- [ ] Le fichier est à **l'échelle 1:1** (unités mm)
- [ ] Format d'export : **DXF** ou **SVG** selon exigences FabLab
- [ ] Épaisseur des lignes de découpe : selon convention FabLab (souvent rouge fin)
- [ ] Gravures distinctes des découpes (souvent couleur différente)

### Test avant usinage complet

Avant de lancer la découpe du panneau complet :

**1. Faire une plaque de test** (~5×5 cm) avec :
- 4 trous switches MX (2×2) à pitch 19.05 mm
- 2 trous encodeurs KY-040

**Valider** :
- Les switches MX s'enclenchent correctement
- Les encodeurs passent bien dans les trous
- L'épaisseur de l'acrylique est compatible

**2. Faire une plaque partielle** (~10×10 cm) avec :
- Une colonne voix complète (OLED + 8 encodeurs + bouton tête)
- Pour valider l'ergonomie et l'espacement

**3. Ensuite** : plate complète 45×37cm.

---

## 18. Matériaux recommandés

### Acrylique

- **Épaisseur** : 3 mm (standard, compatible clip MX)
- **Teinte v1** : transparent (économique, peu esthétique)
- **Teinte v2** : noir mat (look pro, gravures visibles)
- **Fournisseur** : FabLab Poitiers ou commande en ligne (~15-25€ la plaque 50×50 cm)

### Alternative : MDF 3mm

Plus bon marché mais moins durable et plus difficile à graver proprement.

### Alternative : aluminium 1.5mm

Pour la version v3+ ultra-pro, demande **fraiseuse CNC** (pas de découpe laser CO2).

---

## 19. Document de référence pour le dessin AutoCAD

Ce document sert de **référence** pour le dessin. En pratique dans AutoCAD :

1. **Créer un nouveau fichier** en mm, échelle 1:1
2. **Tracer le rectangle 450×370 mm**
3. **Placer les 4 trous de fixation** Ø 3.2 mm aux coins
4. **Tracer les 32 trous du pad** avec ARRAY RECTANGULAR (voir doc précédente)
5. **Tracer les trous de chaque zone** une par une, en se référant aux coordonnées ci-dessus
6. **Vérifier toutes les dimensions** avec la commande DIST
7. **Exporter en DXF** pour le FabLab

### Blocs réutilisables à créer

Créer des **blocs AutoCAD** pour :
- **Trou switch MX** (14×14 mm)
- **Trou encodeur KY-040** (Ø 7 mm)
- **Trou tact 6×6** (6.5×6.5 mm)
- **Trou tact 12×12** (12.5×12.5 mm)
- **Trou LED 3mm** (Ø 3.2 mm)
- **Trou M3 fixation** (Ø 3.2 mm)
- **Découpe OLED SH1107** (30×30 mm + 4 trous M2)
- **Colonne voix complète** (bouton tête + OLED + 8 encodeurs)

Cela permet de **réutiliser** les éléments et d'éviter les erreurs de duplication.

---

## Conclusion

Ce document fournit **toutes les dimensions nécessaires** pour dessiner le panneau PërKompanion dans AutoCAD et le faire usiner au FabLab Poitiers.

**Dimensions globales** : 450 × 370 mm (acrylique 3mm)
**Éléments principaux** : 32 switches MX + 64 encodeurs + 9 OLEDs + écran 7" + 80+ boutons
**Jacks audio anticipés** : 19 emplacements sur tranche arrière

**Prochaines étapes** :
1. Dessiner dans AutoCAD selon ces coordonnées
2. Valider visuellement
3. Exporter DXF
4. Visite FabLab Poitiers pour première découpe plate v1 (acrylique transparent)
5. Test mécanique avec composants à réception
6. Ajustements si nécessaire
7. Découpe plate v2 finale (acrylique noir mat + gravures)
