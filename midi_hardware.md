# PërKompanion — MIDI Hardware

> Schéma hardware des interfaces MIDI DIN du Teensy 4.1
> Document technique — **v0.9** — avril 2026

---

## Préambule

Ce document décrit le **circuit hardware** nécessaire pour interfacer le Teensy 4.1 avec du MIDI DIN 5 broches standard. Sans ce circuit, les pins UART du Teensy ne peuvent pas communiquer directement avec des périphériques MIDI DIN (niveaux de tension incompatibles, isolation galvanique requise).

Lecteur cible : monteur électronique, développeur firmware qui configure Serial1.

Ce document complète `teensy_pinout.md` qui définit seulement l'assignation des pins UART (pin 0 RX, pin 1 TX).

---

## 1. Spécification MIDI DIN

### Signaux

MIDI utilise un **courant boucle 5 mA** à 31250 baud (5 mA in = 1, no current = 0). Pas un signal de tension classique.

Référence normative : MIDI 1.0 Specification (MMA 1982, encore standard en 2026).

### Connecteur DIN 5 broches

Pinout vu de la face avant du connecteur femelle (côté câble) :

```
      2   4
    1       5
         3
```

| Pin | MIDI IN (réception) | MIDI OUT (émission) |
|-----|---------------------|---------------------|
| 1 | N/C | N/C |
| 2 | N/C ou shield | Shield (GND optionnel) |
| 3 | N/C | N/C |
| 4 | Current source (+5V via résistance) | Current source (+5V via résistance) |
| 5 | Current sink (Data) | Current sink (Data) |

**Important** : MIDI IN et MIDI OUT ont le même pinout mais des rôles opposés. Ne pas confondre.

---

## 2. Circuit MIDI IN (entrée — réception du clock)

### Principe : isolation optique obligatoire

La norme MIDI **exige** une isolation galvanique à l'entrée via opto-coupleur. Raison : éviter les boucles de masse entre équipements, qui provoquent du buzz et peuvent endommager le matériel.

### Opto-coupleur recommandé : 6N138

Le 6N138 est le standard MIDI depuis 40 ans. Alternative plus moderne : **H11L1** (plus rapide, meilleur timing) ou **6N137** (très rapide).

**Pour PërKompanion : H11L1 recommandé**. Plus rapide, moins de jitter sur le clock, pas plus cher (~1€).

### Schéma MIDI IN

```
DIN femelle        [Diode 1N4148
MIDI IN             protection]
                                             ┌─ 3.3V (Vcc Teensy)
 Pin 4 ──[R=220Ω]── (anode)                  │
                        │                   [R=10kΩ pullup]
                        │                    │
                    [opto H11L1]              │
                        │                    │
                     (cathode) ──┐      ┌────┴─── Output ────► Teensy RX (pin 0)
                                  │      │
 Pin 5 ──[R=220Ω]──── (anode)    │      │
                        │         │      │
                        │         ▼      │
                        └─────── (base) ─┘
                                  │
                                 GND
                                 
Pin 2 (shield) ──── GND ou N/C
Pin 1, 3 : N/C
```

**Valeurs de composants** :

- 2× résistances 220 Ω ± 1% (limitent le courant MIDI à ~5mA sous 5V nominal)
- 1× diode 1N4148 en parallèle inverse (protection contre polarité inversée)
- 1× opto-coupleur H11L1 (ou 6N138 + transistor driver si disponible à la place)
- 1× résistance 10 kΩ pullup sur l'output vers 3.3V
- 1× condensateur découplage 100 nF entre Vcc et GND du H11L1

### H11L1 vs 6N138

Le H11L1 intègre un Schmitt trigger output, donne direct un signal digital propre.

Le 6N138 nécessite un montage avec transistor driver + résistance additionnelle pour obtenir un signal digital. Plus de composants, plus de place.

**Recommandation : H11L1** pour simplicité et performance.

### Connexion au Teensy

- Pin output du H11L1 → Teensy pin 0 (RX Serial1)
- GND commun
- Vcc H11L1 = 3.3V Teensy (le H11L1 accepte 2.2V à 5.5V sur Vcc)

### Pourquoi pas direct sans opto

Sans opto :
- Risque boucle de masse (bruit, buzz)
- Pas de protection contre surtensions
- Non conforme à la norme MIDI
- Peut endommager le Teensy en cas de problème côté source MIDI

L'isolation optique est **obligatoire** par la norme et vivement recommandée en pratique.

---

## 3. Circuit MIDI OUT (sortie — émission des CCs au Perkons)

### Principe : current source via résistances

Le MIDI OUT doit fournir un courant de 5 mA quand la ligne de données est haute. Le Teensy sort du 3.3V, on calibre avec des résistances.

### Schéma MIDI OUT

```
Teensy pin 1 (TX Serial1) ────[buffer optionnel 74HC04]────┐
                                                            │
 Pin 4 DIN ──[R=33Ω]── 3.3V (Vcc Teensy)                    │
                                                            │
 Pin 5 DIN ──[R=10Ω]─────────────────────────────────┬──────┘
                                                     │
 Pin 2 DIN ──── GND (shield)                         │
                                                     │
 Pins 1, 3 : N/C                                     │
                                                     │
                                                   (ligne TX)
```

**Valeurs de composants** :

- **R1 = 33 Ω** entre Vcc et pin 4 (côté "current source" positif du DIN)
- **R2 = 10 Ω** entre Teensy TX et pin 5 (côté "data" du DIN)

### Dimensionnement des résistances

Si Teensy Vcc = 3.3V :
- Tension totale disponible : 3.3V
- Chute dans la LED MIDI source (côté récepteur) : ~1.5V
- Tension restante pour les résistances : 1.8V
- Courant cible : 5 mA
- R1 + R2 ≈ 1.8V / 5mA = 360 Ω... mais en pratique pour Vcc 3.3V, les valeurs recommandées par les DIY MIDI modernes sont :
  - R1 = 33 Ω (limit current en saturation)
  - R2 = 10 Ω (limit current côté data)

**Alternative plus robuste** : utiliser un **buffer 74HC04** (hex inverter) entre Teensy TX et R2. Permet de driver une ligne plus longue, protège le Teensy de court-circuits accidentels.

Avec buffer :
- Teensy TX (3.3V) → 74HC04 input → 74HC04 output (peut driver 5V si alimenté en 5V) → R2 → DIN pin 5
- Alim 74HC04 en 3.3V ou 5V selon besoins

**Recommandation v0.7 : sans buffer**, les résistances 33Ω + 10Ω directement. Plus simple et fonctionnel pour une connexion locale au Perkons (câble MIDI court, <2m).

Si issues de fiabilité observées → ajouter le 74HC04 en Phase 4.

### Attention polarité TX Teensy

Le Teensy en Serial1 émet en TX avec niveau logique "1 = Vcc (3.3V), 0 = GND".

Pour MIDI, la ligne doit être **inverted** à la convention : "1 MIDI = pas de courant, 0 MIDI = courant 5mA".

Heureusement, le UART standard MIDI a cette inversion implicite : quand Teensy TX est HIGH (idle), aucun courant ne circule dans le DIN = MIDI "1" = bon. Quand TX est LOW (data 0 bit), courant circule = MIDI "0" = bon.

Donc **pas d'inverseur nécessaire** pour MIDI OUT avec Teensy Serial1.

---

## 4. Circuit MIDI THRU (optionnel, v2+)

MIDI THRU = retransmission directe du MIDI IN vers un autre DIN, sans traitement logiciel.

**Implémentation simple** : câbler le signal à la sortie de l'opto H11L1 vers un second buffer DIN OUT, en parallèle du TX Teensy.

Pour v1, **pas de MIDI THRU nécessaire** (l'ESI M8U eX gère déjà le routage). À ajouter en v2 si besoin.

---

## 5. BOM hardware MIDI

### Composants minimum (v1)

| Composant | Quantité | Valeur / Ref | Prix unitaire |
|-----------|----------|--------------|----------------|
| Connecteur DIN 5 broches femelle PCB mount | 2 | Standard MIDI | 1-2€ |
| Opto-coupleur H11L1 | 1 | DIP-6, IC | 0.80€ |
| Résistance 220 Ω 1% | 2 | 1/4W | 0.10€ |
| Résistance 33 Ω 1% | 1 | 1/4W | 0.10€ |
| Résistance 10 Ω 1% | 1 | 1/4W | 0.10€ |
| Résistance 10 kΩ 1% | 1 | 1/4W (pullup IN) | 0.10€ |
| Diode 1N4148 | 1 | Protection | 0.10€ |
| Condensateur 100 nF céramique | 1 | Découplage H11L1 | 0.10€ |

**Total BOM MIDI hardware : ~3-5€**.

### Composants optionnels (v2+)

| Composant | Quantité | Usage |
|-----------|----------|-------|
| Buffer 74HC04 hex inverter | 1 | Renforce drive MIDI OUT |
| Second H11L1 | 1 | MIDI THRU |
| Connecteur DIN supplémentaire | 1 | MIDI THRU output |

---

## 6. Routing physique PërKompanion ↔ ESI M8U eX

### Câblage Phase 1-2 (software MVP, Teensy sur breadboard)

- Teensy MIDI OUT (pin 1 via circuit) → câble DIN → ESI IN 16 (input 16 de l'ESI)
- ESI OUT 15 → câble DIN → Teensy MIDI IN (pin 0 via circuit)

L'ESI est configuré pour :
- Recevoir clock de FL Studio (IN 1) et router vers OUT 15 (vers Teensy)
- Recevoir de Teensy (IN 16) et router vers OUT 8 (vers Perkons)

### Câblage Phase 4+ (panneau final)

Quand PërKompanion est dans son boîtier, les connecteurs DIN femelles sont accessibles en panneau arrière :
- MIDI IN (depuis ESI OUT 15)
- MIDI OUT (vers ESI IN 16)
- MIDI THRU (v2+, retransmit direct)

Câbles DIN courts (30-50 cm) entre le panneau arrière de PërKompanion et l'ESI M8U eX.

---

## 7. Validation Phase 1

Checklist de validation du circuit MIDI avant d'écrire du code applicatif :

### Test MIDI IN

1. Assembler le circuit IN sur breadboard (opto + résistances + DIN)
2. Connecter à une source MIDI connue (ESI, autre machine, interface USB-MIDI)
3. Envoyer un clock MIDI depuis la source
4. Flasher un sketch Teensy qui allume la LED built-in (pin 13) à chaque tick clock reçu
5. Vérifier :
   - LED clignote bien à chaque tick
   - Jitter visuel <1ms (à la vue, ou à l'oscilloscope pour mesure précise)
   - Pas de ticks manquants sur 1 minute de test

### Test MIDI OUT

1. Assembler le circuit OUT (résistances + DIN)
2. Connecter Teensy MIDI OUT à l'ESI (IN quelconque)
3. Router ESI de cet IN vers un PC avec logiciel MIDI monitor
4. Flasher un sketch Teensy qui envoie des CCs aléatoires
5. Vérifier que les CCs arrivent bien et sans corruption dans le monitor

### Test MIDI OUT → Perkons

1. Router ESI : Teensy OUT → Perkons IN
2. Envoyer un CC test : par exemple CC 70 (V1 Tune) valeur 64, canal 1
3. Vérifier que le Tune V1 du Perkons bouge (LED changement, son changement)

Si ces 3 tests passent, le circuit MIDI est fonctionnel. On peut passer au dev applicatif.

---

## 8. Dépannage — problèmes courants

### MIDI IN ne reçoit rien

- Vérifier polarité DIN (pin 4 vs pin 5 inversés est classique)
- Vérifier continuité du câble DIN (plusieurs câbles sont mal soudés en usine)
- Tester opto avec multimètre : quand un bit arrive, tension sur output pin doit osciller
- Vérifier orientation opto (sens du H11L1)

### MIDI OUT ne transmet pas

- Vérifier TX Teensy oscille quand on envoie des CCs (LED ou scope)
- Vérifier les 2 résistances (33Ω et 10Ω) soudées dans le bon ordre
- Vérifier que le Perkons est configuré pour recevoir sur le bon canal

### Jitter audible sur le clock

- Vérifier qualité des câbles DIN (blindage, soudures)
- Vérifier que le clock source n'est pas lui-même jittery (bug FL Studio parfois)
- Si persistant : passer d'un 6N138 à un H11L1 (plus rapide)

### Perkons ignore les CCs du Teensy

- Vérifier que le Perkons est en **Multi MIDI Channel** (SHIFT + MOD → Track 3 step 2)
- Vérifier que le Teensy envoie sur les bons canaux (1 pour V1, 2 pour V2, etc.)
- Vérifier les numéros de CC (70-80 pour V1, 81-91 pour V2, etc.)
- Activer `MIDI CC OUT` et `MIDI Seq OUT` dans la config Perkons si nécessaire

---

## 9. Notes pour le PCB final (Phase 4)

Quand on fabrique le PCB panneau, intégrer :

- **Ground plane** propre pour les circuits MIDI
- **Traces courtes** entre Teensy et opto/DIN
- **Condensateur de découplage 100 nF** près du H11L1
- **Condensateurs bulk 10 µF + 100 nF** près du Teensy
- **Test points** sur MIDI IN et OUT pour debug

Connecteurs DIN à monter en panneau arrière, câblés au PCB principal via câbles courts (<15cm).

---

## 10. Références

- MIDI 1.0 Specification : midi.org
- Notch's MIDI Specification (détails circuits) : www.notats.com/midi
- PJRC Teensy UART : pjrc.com/teensy/td_uart.html
- Tutoriel MIDI DIY : mitxela.com/projects/polyphonic_synth_cable (schémas de référence)

---

## 11. Historique des révisions

**v0.8 final (avril 2026)** :
- Pas de changement matériel MIDI par rapport à v0.7 (pins Serial1 pin 0/1 inchangés, opto H11L1 et résistances série inchangés)
- Source recommandée mise à jour : H11L1M **chez TME** (pas AliExpress) pour garantie authenticité
- Support DIP-6 ajouté à la BOM (pour montage/démontage facile du H11L1)
- Connecteurs DIN femelles PCB Lumberg/Cliff chez TME (qualité pro)

**v0.8 initial (avril 2026)** :
- Mise à jour header version pour cohérence documentaire

**v0.7 (précédente)** :
- Documentation initiale complète des circuits MIDI IN (opto H11L1) et MIDI OUT (résistances 33Ω + 10Ω)
- Ajout test points pour debug

---

*Fin de midi_hardware.md — v0.8 final, avril 2026*
