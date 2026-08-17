# PërKompanion — Protocol Tables — v1.0.0

## Changelog v0.9 → v1.0.0

### Section "Note v0.9" remplacée par "Note v1.0.0"
- Avant : note ergonomique sur module violet 9×5, OLED master SPI, etc.
- Après : note sur le pivot framework + Device Profiles. Les modifications v0.9
  restent valides ; seules les tables param_id et le bloc commandes évoluent.
- Rationale : la v1.0.0 marque le passage de "PërKompanion = Perkons-only" à
  "framework générique avec Perkons comme profile de référence". Le pivot est
  conceptuel et impacte directement l'allocation des param_id.

### §2 — Bloc 0x0000-0x0FFF restructuré
- Avant : bloc Perkons hardcodé (0x0000-0x0FFF avec sous-bloc 0x0500-0x08FF
  pour V5-V8 internes).
- Après : bloc partitionné en **4 slots × 1024 paramètres** réservés aux
  profiles externes chargés dynamiquement (perkons_hd01.yaml, etc.). Le
  Perkons devient un profile parmi d'autres, chargé en slot 0 par défaut.
- Rationale : le firmware ne connaît plus le hardware audio par hardcoding ;
  il connaît une grammaire de profile et alloue les param_id par slot.
  Cohérent avec le principe §13 ("le firmware ne connaît pas le hardware
  audio, seulement les param_id"). Capacité par profile : 1024 params, marge
  confortable pour Digitakt II (~414 params) et tout device prévisible.

### §2 — Sous-bloc V5-V8 relocalisé
- Avant : V5-V8 occupent 0x0500-0x08FF (collision avec le partitionnement
  framework du bloc 0x0000-0x0FFF).
- Après : V5-V8 relocalisés vers **0x6000-0x6FFF** (qui était réservé pour
  modulations complexes v4+).
- Rationale : les voix virtuelles internes V5-V8 sont du code embarqué
  PërKompanion, pas un device externe. Elles méritent leur bloc dédié,
  séparé des profiles externes.

### §2 — Blocs réservés réajustés
- 0x6000-0x6FFF : ancienne réserve "modulations complexes v4+" → désormais
  V5-V8 internes.
- 0x7000-0x7FFF : ancienne réserve "matrix modulation v5+" → désormais
  réserve double : modulations complexes ET extension profiles si plus de
  4 profiles simultanés en v5+.
- 0x8000-0xFFFF : inchangé (réservé utilisateur / plugins custom).
- Rationale : le futur a moins besoin de "matrix modulation" dédiée que de
  marge profile/modulation combinée. Concentration des réserves.

### §6 — Bloc 0xC0-0xCF Device Profile ajouté (Pi → Teensy)
- Contenu : nouvelles commandes pour transférer un profile binaire sérialisé
  Pi → Teensy avec chunking, validation CRC, gestion multi-slots. Une note
  documentaire clarifie le partage de plage 0xC0-0xCF entre les deux
  directions du protocole (Pi → Teensy intégralement Device Profiles ;
  Teensy → Pi partagé entre hotcues 0xC0-0xC4 et ACK profile 0xCA-0xCC).
- Rationale : le Pi parse le YAML, le Teensy reçoit du binaire compact qu'il
  stocke en RAM (mode normal) et en EEPROM (mode dégradé). Cette commande
  était l'inconnue mentionnée à l'arbitrage chef d'orchestre §4 des
  décisions transverses. Bloc choisi parmi les libres Pi → Teensy, adjacent
  thématiquement aux blocs 0x80 (DSP) et 0x90 (Voice Kits).

### §14 — Section "Device Profile Schema" ajoutée
- Contenu : pointeur vers `device_profile_schema.md` et
  `profile_template_with_docs.yaml`. Décrit brièvement le rôle du profile
  YAML et la chaîne YAML → binaire → Teensy.
- Rationale : sans cette section, le lecteur de PROTOCOL_TABLES découvre
  le bloc 0xC0-0xCF sans comprendre d'où viennent les profiles. Pointeur
  documentaire vers la spec dédiée.

### Sections inchangées
- §1 (CC MIDI Perkons), §3 (numérotation physique), §4 (encoding wire),
  §5 (endianness), §7 (codes WARNING/ERROR), §8 (snapshot), §9 (priorisation),
  §10 (position-in-bar), §11 (combinaison modulations), §12 (mode dégradé pad),
  §13 (versionnage protocole) : aucune modification.
- Tous les autres blocs commandes (0x10-0xBF, 0xD0-0xFF) Pi→Teensy et
  Teensy→Pi : inchangés.

---

# PërKompanion — Protocol Tables

> Tables de référence exhaustives pour l'implémentation
> Document technique annexe — **v1.0.0** — avril 2026

---

## Note v1.0.0

**Pivot framework** : PërKompanion devient un framework générique de companion
MIDI. Le Perkons HD-01 est désormais le **premier profile de référence**, plus
le projet lui-même. Cette évolution conceptuelle se traduit dans les tables
par deux modifications majeures :

1. Le bloc `0x0000-0x0FFF` (anciennement Perkons hardcodé) est restructuré en
   **4 slots × 1024 paramètres** réservés aux profiles externes chargés
   dynamiquement.
2. Les voix virtuelles internes V5-V8 sont **relocalisées** depuis
   `0x0500-0x08FF` vers `0x6000-0x6FFF`.

Un nouveau bloc commandes `0xC0-0xCF` (Pi → Teensy) est ajouté pour la
gestion des Device Profiles (chargement, déchargement, listing).

Une nouvelle section §14 "Device Profile Schema" pointe vers les documents
dédiés (`device_profile_schema.md`, `profile_template_with_docs.yaml`,
quatre profiles de référence dans `profiles/`).

**Pas de changement** sur l'encoding des valeurs sur le fil, les numéros de
CC MIDI Perkons (qui restent dans le profile `perkons_hd01.yaml`), la
numérotation physique du matériel, les commandes hors bloc 0xC0-0xCF, le
format snapshot, ou le versionnage du protocole.


## Préambule

Ce document contient les **tables de référence** nécessaires à l'implémentation. Il est référencé par `PERKOMPANION_PHASE0.md` et complète ses spécifications.

Lecteur cible : développeur firmware Teensy et développeur software Pi qui écrivent le code.

**Ce document est la source de vérité unique** pour :
- L'encoding des valeurs sur le fil (section 4)
- Les numéros de CC MIDI du Perkons HD-01
- L'allocation des IDs de paramètres (`param_id`)
- La numérotation physique (encoders, pads, buttons, LEDs)
- Les commandes du protocole USB série (complètes, avec chunking)
- Les formats de données bulk
- Les codes d'erreurs et warnings

Toute modification de ces tables doit être versionnée explicitement.

---

## 1. Numéros de CC MIDI du Perkons HD-01 (firmware v1.2)

Source : `/mnt/project/perkons_reference.md` — Erica Synths User Manual December 2024.

### Voix 1 — CC 70-80

| CC | Paramètre | Type |
|----|-----------|------|
| 70 | Tune | Continu 0-127 |
| 71 | Param1 | Continu 0-127 |
| 72 | Cutoff | Continu 0-127 |
| 73 | FX Send | Continu 0-127 |
| 74 | Decay | Continu 0-127 |
| 75 | Param2 | Continu 0-127 |
| 76 | Drive | Continu 0-127 |
| 77 | Level | Continu 0-127 |
| 78 | Algo switch | Enum 0-2 (3 algos) |
| 79 | Mode switch | Enum 0-2 (3 modes) |
| 80 | Filter switch | Enum 0-2 (LP/BP/HP) |

### Voix 2 — CC 81-91
### Voix 3 — CC 92-102
### Voix 4 — CC 103-113

Même ordre que voix 1 (Tune, Param1, Cutoff, FX Send, Decay, Param2, Drive, Level, Algo, Mode, Filter).

### Canal MIDI par voix

**Mode Multi MIDI Channel** (choisi pour PërKompanion) :
- Voix 1 : canal MIDI 1
- Voix 2 : canal MIDI 2
- Voix 3 : canal MIDI 3
- Voix 4 : canal MIDI 4

Configuration Perkons : `SHIFT + MOD` → Track 3 Step 2.

### Triggering de notes

En Multi MIDI Channel, n'importe quelle note sur le canal d'une voix déclenche la voix (vélocité mappée au volume).

### Paramètres non-accessibles via MIDI CC

**Important** : les paramètres suivants du Perkons ne sont PAS contrôlables en MIDI :
- Probability per step (réglage hardware via `Hold step + PROB + tap`)
- Ratchet per step (hardware : `Hold step + RATCHET + tap`)
- Odds per step (hardware : `Hold step + ODDS1/ODDS2 + tap`)
- Parameter locks (hardware : `Hold step + ajuster paramètre`)
- Section Master (BBD, compresseur, volume master) — pas de CC documenté

**Conséquence pour PërKompanion v1** : la feature "Probabilité/ratchet via MIDI CC sur le Perkons" annoncée initialement est **impossible**. La feature est repensée en "Probability/ratchet sur les augmentations, hotcues et arpégiateurs générés par PërKompanion" (voir VISION §12).

---

## 2. Table param_id — Allocation complète

`param_id` est un entier 16-bit (2 bytes, little endian) utilisé dans les commandes `SET_PARAM`, `SET_MOD`, etc.

### Bloc 0x0000-0x0FFF — Profiles externes (4 slots × 1024 paramètres)

Le bloc `0x0000-0x0FFF` est entièrement réservé aux **profiles de devices
externes** chargés dynamiquement (cf. `device_profile_schema.md`). Il est
partitionné en 4 slots de 1024 paramètres chacun :

| Slot | Plage param_id | Capacité |
|------|---------------|----------|
| 0 | `0x0000 - 0x03FF` | 1024 params |
| 1 | `0x0400 - 0x07FF` | 1024 params |
| 2 | `0x0800 - 0x0BFF` | 1024 params |
| 3 | `0x0C00 - 0x0FFF` | 1024 params |

**Allocation à l'intérieur d'un slot** : le runtime alloue les `param_id`
séquentiellement selon l'ordre de déclaration des params dans le profile
(parcours `tracks` puis `global_param_groups`, dans l'ordre du fichier YAML).

**Hint utilisateur** : un profile YAML peut déclarer un champ
`preferred_slot: N` (0-3) pour suggérer un slot d'accueil au runtime. Le
runtime résout les conflits si deux profiles préfèrent le même slot.

**Capacité observée** sur les 4 profiles de référence :

| Profile | Params | Saturation slot |
|---------|--------|-----------------|
| perkons_hd01 | 44 | < 5% |
| elektron_digitakt_ii | ~414 | ~40% |
| korg_electribe_2_sampler | ~234 | ~23% |
| behringer_td3_mo | 13 | < 2% |

Aucun profile observé ne sature un slot.

**Profile de référence Perkons** : le mapping concret CC MIDI ↔ param_id
pour le Perkons HD-01 vit désormais dans `profiles/perkons_hd01.yaml`. Les
44 paramètres Perkons (4 voix × 11 params) sont chargés en slot 0 par défaut
au boot du système.

Les CCs MIDI du Perkons (70-113) restent documentés dans la §1 ci-dessus,
qui sert maintenant de référence-source pour la validation du profile YAML.

### Bloc 0x1000-0x1FFF — Paramètres Master (DSP, v2+)

| param_id | Paramètre | Range | Courbe | Type |
|----------|-----------|-------|--------|------|
| 0x1000 | Master.Volume | 0.0-1.0 | exp | continuous |
| 0x1001 | Master.Reverb.Time | 0.0-1.0 | exp | continuous |
| 0x1002 | Master.Reverb.Mix | 0.0-1.0 | exp | continuous |
| 0x1003 | Master.Reverb.Type | 0-5 | none | **enum** (Plate/Spring/Hall/Room/Shimmer/Freeze) |
| 0x1010 | Master.Delay.Time | 0.0-1.0 | exp | continuous |
| 0x1011 | Master.Delay.Feedback | 0.0-1.0 | exp | continuous |
| 0x1012 | Master.Delay.Mix | 0.0-1.0 | exp | continuous |
| 0x1020 | Master.Comp.Threshold | 0.0-1.0 | linear | continuous |
| 0x1021 | Master.Comp.Ratio | 0.0-1.0 | log | continuous |
| 0x1022 | Master.Comp.Attack | 0.0-1.0 | exp | continuous |
| 0x1023 | Master.Comp.Release | 0.0-1.0 | exp | continuous |
| 0x1030 | Master.Limiter.Threshold | 0.0-1.0 | linear | continuous |
| 0x1040 | Master.EQ.Low | -1.0 à +1.0 | linear | continuous bipolar |
| 0x1041 | Master.EQ.Mid | -1.0 à +1.0 | linear | continuous bipolar |
| 0x1042 | Master.EQ.High | -1.0 à +1.0 | linear | continuous bipolar |
| 0x1050 | System.Tempo.BPM | 600-2400 (centimals) | linear | integer |

### Bloc 0x2000-0x2FFF — Paramètres DSP par voix (v2+)

**Architecture modulaire v0.8 final** : chaque voix (V1-V4 et V5-V8) dispose d'une **chaîne DSP de 4 slots reconfigurables** parmi 13 types d'effets.

**Encodage** : `0x2VSP` où :
- V = voix (1-8, voix Perkons V1-V4 = 0x21-0x24, voix virtuelles V5-V8 = 0x25-0x28)
- S = slot DSP (0-3, 4 slots par voix)
- P = paramètre (0 = Type, 1-4 = Params contextuels, 5 = Mix)

**Structure complète pour V1** (0x2100-0x2143) :

| param_id | Paramètre | Type |
|----------|-----------|------|
| 0x2100 | V1.FX_Slot0.Type | **enum** (0x00-0x0D, voir table effets) |
| 0x2101 | V1.FX_Slot0.Param1 | continuous 0.0-1.0 |
| 0x2102 | V1.FX_Slot0.Param2 | continuous |
| 0x2103 | V1.FX_Slot0.Param3 | continuous |
| 0x2104 | V1.FX_Slot0.Param4 | continuous |
| 0x2105 | V1.FX_Slot0.Mix | continuous (wet/dry) |
| 0x2110 | V1.FX_Slot1.Type | enum |
| 0x2111-0x2115 | V1.FX_Slot1 params | continuous |
| 0x2120 | V1.FX_Slot2.Type | enum |
| 0x2121-0x2125 | V1.FX_Slot2 params | continuous |
| 0x2130 | V1.FX_Slot3.Type | enum |
| 0x2131-0x2135 | V1.FX_Slot3 params | continuous |
| 0x2140 | V1.FX_Send.ReverbBus | continuous |
| 0x2141 | V1.FX_Send.DelayBus | continuous |
| 0x2142 | V1.OutputLevel | continuous |
| 0x2143 | V1.Pan | continuous bipolar |

**Total par voix** : 4 slots × 6 params + 4 sends = **28 params**

**Voix 2-8** : structure symétrique
- V2 : 0x2200-0x2243
- V3 : 0x2300-0x2343
- V4 : 0x2400-0x2443
- V5 : 0x2500-0x2543
- V6 : 0x2600-0x2643
- V7 : 0x2700-0x2743
- V8 : 0x2800-0x2843

**Total params DSP** : 8 voix × 28 = **224 paramètres DSP modulables**.

### Types d'effets DSP (enum FX_Slot.Type)

Table des 14 valeurs possibles pour chaque slot :

| Value | Effet | Param1 | Param2 | Param3 | Param4 |
|-------|-------|--------|--------|--------|--------|
| 0x00 | **None** (slot vide, bypass) | - | - | - | - |
| 0x01 | **Filter** (State Variable) | Cutoff | Resonance | Type (LP/HP/BP/Notch) | Drive |
| 0x02 | **Distortion** (Waveshaper) | Drive | Type (Soft/Hard/Fold) | Tone | PreGain |
| 0x03 | **Bitcrusher** | SampleRate | BitDepth | - | - |
| 0x04 | **Chorus** | Rate | Depth | Feedback | Spread |
| 0x05 | **Phaser** | Rate | Depth | Feedback | Stages |
| 0x06 | **Reverb** (Freeverb) | RoomSize | Damping | Width | - |
| 0x07 | **Delay** | Time | Feedback | Filter | PingPong |
| 0x08 | **Compressor** | Threshold | Ratio | Attack | Release |
| 0x09 | **EQ 3-band** | Low gain | Mid gain | High gain | Mid freq |
| 0x0A | **Ring Mod** | Freq | Depth | Shape | - |
| 0x0B | **AutoPan** | Rate | Depth | Shape | Phase |
| 0x0C | **Tremolo** | Rate | Depth | Shape | - |
| 0x0D | **LoFi** | Saturation | SRReduce | BitReduce | Noise |

### Correspondance avec PJRC Audio Library

| Effet PërKompanion | Objet PJRC |
|--------------------|-----------|
| Filter | `AudioFilterStateVariable` |
| Distortion | `AudioEffectWaveshaper` |
| Bitcrusher | `AudioEffectBitcrusher` |
| Chorus | `AudioEffectChorus` |
| Phaser | custom ou `AudioEffectPhaser` |
| Reverb | `AudioEffectFreeverb` |
| Delay | `AudioEffectDelayExternal` |
| Compressor | `AudioEffectDynamics` |
| EQ | `AudioFilterBiquad` (×3) |
| Ring Mod | custom (mult oscillateur) |
| AutoPan | `AudioAmplifier` modulé |
| Tremolo | `AudioAmplifier` modulé |
| LoFi | combo de bitcrusher + waveshaper |

### Commandes protocole pour DSP

Voir bloc 0x80-0x8F dans la section commandes :

- **SET_FX_SLOT** (0x80) : configure type d'effet dans un slot (voice + slot + fx_type + preset_id)
- **SET_FX_PARAM** (0x81) : règle un paramètre (voice + slot + param + value)
- **CLEAR_FX_CHAIN** (0x82) : efface tous les slots d'une voix
- **LOAD_FX_CHAIN_PRESET** (nouveau 0x83) : charge un preset complet de chaîne (Dub Chain, Distorted Filter, Ambient Wash, etc.)

### Bloc 0x3000-0x3FFF — Paramètres Arpégiateur (v1+)

**Voix 1** :
| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x3100 | V1.Arp.Enabled | 0-1 | boolean |
| 0x3101 | V1.Arp.Mode | 0-9 | **enum** (Up/Down/UpDown/DownUp/Random/AsPlayed/Chord/MultiOct/Inversion/Custom) |
| 0x3102 | V1.Arp.Rate | 0-8 | **enum** (1/1, 1/2, 1/4, 1/8, 1/16, 1/32, 1/4T, 1/8T, 1/16T) |
| 0x3103 | V1.Arp.OctaveRange | 0-4 | integer |
| 0x3104 | V1.Arp.GateTime | 10-100 (%) | continuous |
| 0x3105 | V1.Arp.Swing | -50 à +50 (%) | continuous bipolar |
| 0x3106 | V1.Arp.Hold | 0-1 | boolean |
| 0x3107 | V1.Arp.NoteSource | 0-2 | **enum** (Manual/Pad/MIDI_External) |

Pour l'arp dirigé 16-step, utiliser `SET_ARP_STEP` (voir section 5).

**Voix 2-4** : 0x32XX, 0x33XX, 0x34XX.

### Bloc 0x4000-0x4FFF — Paramètres Hotcue (v3+)

Encodage : `0x4VSS` où V = voix (1-4) et SS = slot hotcue × 0x20 + param_index.

**Voix 1, slot 5 exemple** :
| param_id | Paramètre | Type |
|----------|-----------|------|
| 0x4105 | V1.Hotcue05.Start (steps) | integer 0-31 |
| 0x4125 | V1.Hotcue05.Length (steps) | integer 1-32 |
| 0x4145 | V1.Hotcue05.Pitch (semitones) | integer -12 à +12 (bipolar) |
| 0x4165 | V1.Hotcue05.Feedback | continuous 0.0-1.0 |
| 0x4185 | V1.Hotcue05.LoopMode | **enum** (continuous/oneshot/n_repeat) |
| 0x41A5 | V1.Hotcue05.Quantize | **enum** (1/16, 1/4, 1/2, 1_bar, free) |
| 0x41C5 | V1.Hotcue05.Reverse | boolean |
| 0x41E5 | V1.Hotcue05.FadeMs | integer 0-100 |

Formule :
```
param_id = 0x4000 + (V * 0x100) + (sub_param_index * 0x20) + slot
```

### Bloc 0x5000-0x5FFF — Paramètres système

| param_id | Paramètre | Type |
|----------|-----------|------|
| 0x5000 | System.ClockSource | **enum** (0=External, 1=Internal) |
| 0x5001 | System.Standalone | boolean |
| 0x5002 | System.WiFi.Enabled | boolean |
| 0x5003 | System.OLED.Brightness | continuous 0.0-1.0 |
| 0x5010 | Pad.ActiveLayout | integer (id layout) |
| 0x5020 | Display.ActivePage | **enum** |
| 0x5030 | System.MaxSimultaneousHotcues | integer 1-32 (default 16) |
| 0x5031 | System.HotcueAutoStealOldest | boolean (default true) |
| 0x5040 | Headphone.Volume | continuous 0.0-1.0 |
| 0x5041 | Headphone.MuteMaster | boolean |

### Bloc 0x6000-0x6FFF — Voix virtuelles internes V5-V8 (v3+)

> **Relocalisation v1.0.0** : ce bloc occupait précédemment `0x0500-0x08FF`,
> en collision avec le partitionnement framework du bloc `0x0000-0x0FFF`.
> Les voix virtuelles internes (hotcues PërKompanion) sont du code embarqué,
> pas un device externe — elles méritent leur bloc dédié, séparé des profiles.

Les 4 voix virtuelles V5-V8 pilotent des hotcues assignés. Elles ont une
structure de param_id similaire à V1-V4 mais adaptée aux paramètres de
hotcue + DSP PërKompanion.

**Structure** : `0x6VNN` où V = numéro de voix (5-8) et NN = sous-paramètre.

**V5 — Paramètres de base (0x6500-0x650F)**

| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x6500 | V5.AssignedHotcueId | 0-127 | integer (hotcue assigné) |
| 0x6501 | V5.Volume | 0.0-1.0 | continuous |
| 0x6502 | V5.Pitch | -24 à +24 (semitones) | integer bipolar |
| 0x6503 | V5.Pan | -1.0 à +1.0 | continuous bipolar |
| 0x6504 | V5.Start | 0.0-1.0 | continuous |
| 0x6505 | V5.Length | 0.0-1.0 | continuous |
| 0x6506 | V5.Feedback | 0.0-1.0 | continuous |
| 0x6507 | V5.Reverse | -1.0 à +1.0 | continuous bipolar (vitesse) |
| 0x6508 | V5.LoopMode | 0-2 | **enum** (Loop/OneShot/NRepeat) |
| 0x6509 | V5.Quantize | 0-5 | **enum** (Off/1/16/1/4/1/2/1bar/Free) |
| 0x650A | V5.FadeIn | 0.0-1.0 | continuous |
| 0x650B | V5.FadeOut | 0.0-1.0 | continuous |

**V5 — Paramètres DSP (0x6510-0x651F)**

| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x6510 | V5.Filter.Cutoff | 0.0-1.0 | continuous |
| 0x6511 | V5.Filter.Resonance | 0.0-1.0 | continuous |
| 0x6512 | V5.Filter.Type | 0-3 | **enum** (LP/HP/BP/Notch) |
| 0x6513 | V5.Filter.Slope | 0-2 | **enum** (12/24/48 dB) |
| 0x6514 | V5.Drive | 0.0-1.0 | continuous |
| 0x6515 | V5.Drive.Type | 0-2 | **enum** (Tube/Tape/Digital) |
| 0x6516 | V5.FX.ReverbSend | 0.0-1.0 | continuous |
| 0x6517 | V5.FX.DelaySend | 0.0-1.0 | continuous |
| 0x6518 | V5.FX.WetDry | 0.0-1.0 | continuous |

**V5 — Paramètres Arp (0x6520-0x652F)**

| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x6520 | V5.Arp.Enabled | 0-1 | boolean |
| 0x6521 | V5.Arp.Mode | 0-9 | **enum** |
| 0x6522 | V5.Arp.Rate | 0-8 | **enum** |
| 0x6523 | V5.Arp.OctaveRange | 0-4 | integer |
| 0x6524 | V5.Arp.GateTime | 10-100 | continuous (%) |
| 0x6525 | V5.Arp.Swing | -50 à +50 | continuous bipolar (%) |

**V5 — Paramètres LFO (0x6530-0x653F)**

| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x6530 | V5.LFO.Rate | 0.0-1.0 | continuous |
| 0x6531 | V5.LFO.Depth | 0.0-1.0 | continuous |
| 0x6532 | V5.LFO.Destination | param_id cible | integer |
| 0x6533 | V5.LFO.Shape | 0-4 | **enum** (Sine/Tri/Saw/Square/Random) |
| 0x6534 | V5.LFO.Phase | 0.0-1.0 | continuous |
| 0x6535 | V5.LFO.Smoothing | 0.0-1.0 | continuous |
| 0x6536 | V5.LFO.SyncMode | 0-1 | **enum** (Free/Sync) |
| 0x6537 | V5.LFO.Polarity | 0-1 | **enum** (Unipolar/Bipolar) |

**V6, V7, V8** : structure identique mais préfixe 0x66XX, 0x67XX, 0x68XX.

**Total params V5-V8** : ~30 paramètres × 4 voix = **120 paramètres modulables
pour voix virtuelles**.

### Bloc 0x9000-0x9FFF — Motions (v2+)

Les motions sont les mouvements d'encodeurs capturés en live via REC. Elles peuvent être sauvegardées individuellement et assignées à des slots Augmentation.

| param_id | Paramètre | Range | Type |
|----------|-----------|-------|------|
| 0x9000 | Motion.Length | 16/32/64/128 | integer (steps) |
| 0x9001 | Motion.Quantize | 0-5 | **enum** (Off/1/16/1/8/1/4/1/2/1bar) |
| 0x9002 | Motion.Interpolation | 0-3 | **enum** (Linear/Step/Smooth/Exponential) |
| 0x9003 | Motion.Loop | 0-1 | boolean |
| 0x9004 | Motion.Intensity | 0.0-1.0 | continuous (scale du mouvement) |
| 0x9005 | Motion.Reverse | 0-1 | boolean |

### Blocs réservés

- 0x7000-0x7FFF : réserve double — modulations complexes / séquenceurs hotcue
  (v4+) **et** extension profiles si plus de 4 profiles externes simultanés
  (v5+).
- 0x8000-0xFFFF : réservé utilisateur / plugins custom.

> Note v1.0.0 : le bloc `0x6000-0x6FFF` est désormais occupé par les voix
> virtuelles internes V5-V8 (cf. ci-dessus). L'ancienne réserve "matrix
> modulation v5+" est consolidée dans `0x7000-0x7FFF` pour libérer du
> design space.

---

## 3. Numérotation physique du matériel

### Encoders (encoder_id : 1 byte, 0-255)

**Encodeurs voix (16)** :

| encoder_id | Position |
|------------|----------|
| 0x01 | V1 position 1 (haut-gauche) |
| 0x02 | V1 position 2 (haut-droite) |
| 0x03 | V1 position 3 (bas-gauche) |
| 0x04 | V1 position 4 (bas-droite) |
| 0x05 | V2 position 1 |
| ... | ... |
| 0x10 | V4 position 4 |

**Encodeurs contextuels écran (8)** :

| encoder_id | Position |
|------------|----------|
| 0x11 | Haut-gauche 1 |
| 0x12 | Haut-gauche 2 |
| 0x13 | Haut-droite 1 |
| 0x14 | Haut-droite 2 |
| 0x15 | Bas-gauche 1 |
| 0x16 | Bas-gauche 2 |
| 0x17 | Bas-droite 1 |
| 0x18 | Bas-droite 2 |

**Encodeurs master (7)** :

| encoder_id | Position |
|------------|----------|
| 0x19 | Master Volume (gros) |
| 0x1A | Reverb Time |
| 0x1B | Reverb Mix |
| 0x1C | Delay Time |
| 0x1D | Delay Feedback |
| 0x1E | Comp Threshold |
| 0x1F | Comp Ratio |

**Encodeur optionnel Tempo** : encoder_id 0x20. Réservé mais pas monté en v1 (Tap Tempo suffit). Si absent physiquement, le Teensy ne l'émet jamais.

Total encoder_id réservés : 0x01 à 0x20. IDs 0x21-0xFF libres pour extensions.

### Pads (pad_id : 1 byte, 0-31)

Numérotation gauche-à-droite, haut-en-bas.

```
Rangée 1 : 0x00 0x01 0x02 0x03 0x04 0x05 0x06 0x07
Rangée 2 : 0x08 0x09 0x0A 0x0B 0x0C 0x0D 0x0E 0x0F
Rangée 3 : 0x10 0x11 0x12 0x13 0x14 0x15 0x16 0x17
Rangée 4 : 0x18 0x19 0x1A 0x1B 0x1C 0x1D 0x1E 0x1F
```

### Boutons (button_id : 2 bytes)

**Bloc 0x0100-0x01FF — Pad** :

| button_id | Nom |
|-----------|-----|
| 0x0100 | V1 (voix pad) |
| 0x0101 | V2 |
| 0x0102 | V3 |
| 0x0103 | V4 |
| 0x0110 | M1 (mode pad) |
| 0x0111 | M2 |
| 0x0112 | M3 |
| 0x0113 | M4 |

**Bloc 0x0200-0x02FF — Voix** (Panic + Freeze + MODE par voix) :

| button_id | Nom |
|-----------|-----|
| 0x0200 | V1 Panic |
| 0x0201 | V1 Freeze |
| 0x0202 | V1 MODE (cycle encodeurs) |
| 0x0210 | V2 Panic |
| 0x0211 | V2 Freeze |
| 0x0212 | V2 MODE |
| 0x0220 | V3 Panic |
| 0x0221 | V3 Freeze |
| 0x0222 | V3 MODE |
| 0x0230 | V4 Panic |
| 0x0231 | V4 Freeze |
| 0x0232 | V4 MODE |

**Bloc 0x0300-0x03FF — Écran contextuels** :

| button_id | Nom |
|-----------|-----|
| 0x0300 | F1 (gauche écran, haut) |
| 0x0301 | F2 |
| 0x0302 | F3 |
| 0x0303 | F4 (gauche écran, bas) |
| 0x0304 | F5 (droite écran, haut) |
| 0x0305 | F6 |
| 0x0306 | F7 |
| 0x0307 | F8 (droite écran, bas) |

**Bloc 0x0400-0x04FF — Globaux** :

| button_id | Nom |
|-----------|-----|
| 0x0400 | Panic global |
| 0x0401 | Freeze global |
| 0x0402 | Shift |
| 0x0403 | Load |
| 0x0404 | Clean Slate |
| 0x0410 | REC |
| 0x0411 | PLAY |
| 0x0420 | Tap Tempo |
| 0x0421 | Metronome (optionnel) |

### LEDs (led_id : 2 bytes)

ALORS LA IL FAUT VRAIMENT VERIFIER CAR ON A DESORMAIS DES LEDS A CHAQUE TOUCHE CHERRY MX OU PRESQUE

**LEDs indicateurs voix (0x0100-0x01FF)** : 4 LEDs par voix (Panic, Freeze, MODE, Indicateur mode).

**LEDs boutons pad (0x0200-0x02FF)** : 4 voix pad + 4 mode pad.

**LEDs master (0x0300-0x03FF)** : BPM clock, MIDI sync IN/OUT, REC, PLAY.

### OLEDs (oled_id : 1 byte)

| oled_id | Type | Interface |
|---------|------|-----------|
| 0x00 | SH1106 128×64 V1 | SPI (CS Teensy pin 16) |
| 0x01 | SH1106 128×64 V2 | SPI (CS Teensy pin 17) |
| 0x02 | SH1106 128×64 V3 | SPI (CS Teensy pin 22) |
| 0x03 | SH1106 128×64 V4 | SPI (CS Teensy pin 23) |
| 0x10 | SSD1306 128×32 Master BPM | I2C (via TCA9548A canal 0) |<=I2C remplacé par un nouveau SPI <=
=>I2C remplacé par un nouveau SPI <=
---

## 4. **Encoding des valeurs sur le fil — SECTION CRITIQUE**

### Principe universel

Toutes les valeurs de type "continuous" et "bipolar" sont transmises en **int16 signed, little endian**, représentant une valeur normalisée.

### Convention

```
value_wire : int16 signed, range [-32767, +32767]
value_float : float normalisé, range [-1.0, +1.0]

Conversion :
  value_wire = round(clamp(value_float, -1.0, 1.0) * 32767)
  value_float = value_wire / 32767.0
```

### Exemples

| value_float | value_wire (hex) | value_wire (decimal) |
|-------------|------------------|----------------------|
| -1.0 | 0x8001 | -32767 |
| -0.5 | 0xC001 | -16383 |
| 0.0 | 0x0000 | 0 |
| +0.5 | 0x3FFF | 16383 |
| +1.0 | 0x7FFF | 32767 |

**Note** : on n'utilise jamais -32768 (0x8000) pour garder la symétrie autour de zéro.

### Paramètres unipolaires (range [0.0, 1.0])

La plupart des paramètres Perkons et DSP sont unipolaires. Convention :

```
value_float : [0.0, 1.0]
value_wire : [0, 32767] (la plage négative est interdite)

Conversion :
  value_wire = round(clamp(value_float, 0.0, 1.0) * 32767)
```

Le Teensy et le Pi assument `assert value_wire >= 0` pour ces paramètres.

### Mapping vers CC MIDI 7-bit

Pour les paramètres Perkons qui sortent vers CC MIDI 0-127 :

```
cc_value = (value_wire >> 8) & 0x7F  // Prend les 7 bits de poids fort
```

**Exemple** :
- value_float = 0.5
- value_wire = 16383 (0x3FFF)
- cc_value = 0x3F = 63 (milieu du range CC)

La résolution 15-bit du wire est donc réduite à 7-bit pour le CC, mais c'est attendu (Perkons n'accepte que 7-bit de toute façon).

**Pour les modulations qui dépassent 7-bit** (modulation multiple d'un paramètre), le calcul se fait en int16 full precision côté Teensy, et seulement à la sortie MIDI on décime vers 7-bit.

### Paramètres enum (range discret)

Les paramètres `enum` (Algo, Mode, Filter, Reverb Type, Arp Mode, etc.) n'utilisent pas le mapping normalisé.

**Convention enum** :

```
value_wire : int16, range [0, N-1] où N = nombre de valeurs de l'enum
curve : 'none'
```

Exemple : V1.Algo (0x0108), enum 0-2 (3 algos) :
- value_wire = 0 → Algo 1
- value_wire = 1 → Algo 2
- value_wire = 2 → Algo 3

Le firmware fait un mapping direct vers la valeur CC si nécessaire (CC 78 pour V1.Algo).

### Paramètres integer (range entier)

Certains paramètres sont intrinsèquement entiers (Pitch en semitones, OctaveRange, BPM en centimals).

**Convention integer** :

```
value_wire : int16, valeur entière directe (pas de normalisation)
curve : 'none' (ou 'linear' si conversion future souhaitée)
```

Exemple : Hotcue.Pitch (-12 à +12 semitones) :
- value_wire = -12 → -12 semitones
- value_wire = 7 → +7 semitones

Exemple : System.Tempo.BPM en centimals (600-2400 = 60.0 à 240.0 BPM) :
- value_wire = 1400 → 140.0 BPM

### Récapitulation

| Type | value_wire range | Notes |
|------|------------------|-------|
| `continuous` | 0 à 32767 | Unipolar, mapping normalisé |
| `continuous bipolar` | -32767 à +32767 | Bipolar, mapping normalisé |
| `enum` | 0 à N-1 | Index direct, pas de mapping |
| `integer` | valeur entière | Direct, range dépend du paramètre |
| `boolean` | 0 ou 1 | Direct |

Le type est indiqué dans la table param_id (section 2). Le code doit vérifier le type avant de décoder.

### Amount de modulation (SET_MOD)

Le champ `amount` de SET_MOD est **toujours bipolaire** même si la source ou la cible sont unipolaires.

```
amount_wire : int16 signed, range [-32767, +32767]
amount_float : -1.0 à +1.0
```

Un amount négatif inverse le sens de la modulation.

---

## 5. Endianness

**Little endian** sur tout le protocole USB série.

Exemple : `value = 0x1234` → octets envoyés : `0x34, 0x12`.

Python :
```python
import struct
bytes_out = struct.pack('<h', value_wire)  # '<' = LE, 'h' = int16 signed
```

C++ Teensy :
```cpp
int16_t value = 0x1234;
uint8_t bytes_out[2];
memcpy(bytes_out, &value, 2);  // little endian natif
```

---

## 6. Table complète des commandes du protocole

### Structure de trame

```
[SYNC 0xF0][DIR 1 byte][CMD 1 byte][LEN 1 byte][PAYLOAD N bytes][CHECKSUM 1 byte][END 0xF7]
```

- DIR : `0x01` (Pi → Teensy), `0x02` (Teensy → Pi)
- CMD : identifiant de commande (1 byte)
- LEN : longueur du PAYLOAD (0-240, marge de sécurité)
- CHECKSUM : XOR de DIR + CMD + LEN + PAYLOAD
- Total : 6 bytes d'overhead + payload

### Commandes Pi → Teensy (DIR = 0x01)

**Bloc 0x00-0x0F : Connection**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x01 | HELLO | proto_major (1) + proto_minor (1) + proto_patch (1) + reserved (1) |
| 0x02 | HEARTBEAT | timestamp_ms (4) |
| 0x03 | GOODBYE | reason (1) |
| 0x04 | REQUEST_SNAPSHOT | scope (1) |

scope values :
- 0x00 : full snapshot
- 0x01 : params only
- 0x02 : modulations only
- 0x03 : hotcues only
- 0x04 : pad bindings only
- 0x05 : arp states only
- 0x10-0x13 : snapshot for voice 1-4

**Bloc 0x10-0x1F : Paramètres et modulations**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x10 | SET_PARAM | param_id (2) + value (2) |
| 0x11 | SET_MOD | mod_id (2) + source_type (1) + source_id (2) + target (2) + amount (2) + curve (1) + trigger_mode (1) + enabled (1) |
| 0x12 | DELETE_MOD | mod_id (2) |
| 0x13 | TRIGGER_AUG | aug_id (2) |
| 0x14 | STOP_AUG | aug_id (2) |
| 0x15 | LOAD_AUG_CURVE | aug_id (2) + length (1) + is_loop (1) + values (N × 1 bytes, N = length) |
| 0x16 | DELETE_AUG | aug_id (2) |
| 0x17 | SET_PARAM_CURVE | param_id (2) + curve_type (1) + custom_table (256 bytes si curve_type=custom, 0 bytes sinon) |
| 0x18 | RESET_PARAM | param_id (2) |

source_type values :
- 0x01 : LFO
- 0x02 : Envelope
- 0x03 : Augmentation
- 0x04 : Sequencer
- 0x05 : Random

curve values :
- 0x00 : none (pour enum)
- 0x01 : linear
- 0x02 : exp
- 0x03 : exp_strong
- 0x04 : log
- 0x05 : log_strong
- 0x06 : scurve
- 0x07 : inverse
- 0x08 : custom (nécessite SET_PARAM_CURVE avec table)

trigger_mode values :
- 0x01 : free (LFO perpétuel)
- 0x02 : one_shot (une seule passe)
- 0x03 : gated (active tant que bouton appuyé)
- 0x04 : looped (aug/env loopée)

**Bloc 0x20-0x2F : Hotcues (v3+)**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x20 | LOAD_HOTCUE_START | hotcue_id (2) + total_size_bytes (4) + sample_rate (2) + bit_depth (1) + channels (1) |
| 0x21 | LOAD_HOTCUE_CHUNK | hotcue_id (2) + offset (4) + chunk_size (1) + data (N bytes) |
| 0x22 | LOAD_HOTCUE_END | hotcue_id (2) + total_checksum (4) |
| 0x23 | UNLOAD_HOTCUE | hotcue_id (2) |
| 0x24 | PLAY_HOTCUE | hotcue_id (2) + quantize_override (1) |
| 0x25 | STOP_HOTCUE | hotcue_id (2) |
| 0x26 | REQUEST_HOTCUE_DATA | hotcue_id (2) |
| 0x27 | SET_HOTCUE_STEP | hotcue_id (2) + step_index (1) + active (1) + velocity (1) + probability (1) + ratchet (1) + pitch_mod (1 signed) + tie (1) |
| 0x28 | DELETE_HOTCUE | hotcue_id (2) |
| **0x29** | **SET_HOTCUE_LOCK** | hotcue_id (2) + locked (1) |
| **0x2A** | **ASSIGN_HOTCUE_TO_VIRTUAL_VOICE** | hotcue_id (2) + virtual_voice_id (1) [5-8] |
| **0x2B** | **UNASSIGN_VIRTUAL_VOICE** | virtual_voice_id (1) [5-8] |
| **0x2C** | **REQUEST_HOTCUE_LIBRARY** | — (retourne la liste des 128 hotcues et leur état) |
| **0x2D** | **RENAME_HOTCUE** | hotcue_id (2) + name_length (1) + name (N bytes, max 32) |
| **0x2E** | **SET_POLYPHONY_LIMIT** | limit (1) [1-32] |
| **0x2F** | **HOTCUE_STEAL_NOTIFICATION** | hotcue_id_stolen (2) + hotcue_id_new (2) (notification Teensy → Pi) |

**UNLOAD_HOTCUE vs DELETE_HOTCUE** :
- UNLOAD : libère PSRAM côté Teensy, garde sample sur SD. Peut être rechargé.
- DELETE : efface sample côté SD **et** libère PSRAM. Définitif.

**Bloc 0x30-0x3F : LEDs**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x30 | LED_UPDATE | led_id (2) + r (1) + g (1) + b (1) |
| 0x31 | LED_BULK | count (1) + [led_id (2) + r (1) + g (1) + b (1)] × N |
| 0x32 | LED_ANIMATION | led_id (2) + animation_type (1) + speed (1) + color1 (3) + color2 (3) |
| 0x33 | LED_ALL_OFF | - |

**Bloc 0x40-0x4F : OLEDs**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x40 | OLED_TEXT | oled_id (1) + row (1) + col (1) + text_length (1) + text (N bytes ASCII) |
| 0x41 | OLED_CLEAR | oled_id (1) |
| 0x42 | OLED_BITMAP | oled_id (1) + x (1) + y (1) + w (1) + h (1) + data (N bytes packed 1bpp) |
| 0x43 | OLED_DIRTY_RECT | oled_id (1) + x (1) + y (1) + w (1) + h (1) + data (N bytes) |

**Bloc 0x50-0x5F : Système et reset**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x50 | CLEAN_SLATE | flags (1) |
| 0x51 | FULL_RESET | confirm_token (4) |
| 0x52 | SET_CLOCK_SOURCE | source (1) (0=external, 1=internal) |
| 0x53 | SET_BPM | bpm_centimals (2) |
| 0x54 | START_TRANSPORT | - |
| 0x55 | STOP_TRANSPORT | - |

**Bloc 0x60-0x6F : Arpégiateur**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x60 | SET_ARP_PARAM | voice (1) + arp_param_id (1) + value (2) |
| 0x61 | SET_ARP_STEP | voice (1) + step_index (1) + active (1) + note_offset (1 signed) + velocity (1) + probability (1) + ratchet (1) + tie (1) |
| 0x62 | LOAD_ARP_CUSTOM | voice (1) + length (1) + pattern (N bytes) |
| 0x63 | RESET_ARP | voice (1) |

**Bloc 0x70-0x7F : Pad mapping**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x70 | SET_PAD_BINDING | context_mode (1) + context_voice (1) + pad_id (1) + action_type (1) + action_data (4) + color (3) |
| 0x71 | LOAD_PAD_LAYOUT | layout_id (1) + bindings_count (1) + [binding × N] — pour layouts courts ≤21 bindings |
| 0x72 | CLEAR_PAD_LAYOUT | - |
| 0x73 | LOAD_PAD_LAYOUT_START | layout_id (1) + total_bindings (2) |
| 0x74 | LOAD_PAD_LAYOUT_CHUNK | chunk_index (2) + binding_count (1) + [binding × N] |
| 0x75 | LOAD_PAD_LAYOUT_END | layout_id (1) + checksum (2) |
| 0x76 | ENTER_STEP_EDIT_MODE | hotcue_id (2) |
| 0x77 | EXIT_STEP_EDIT_MODE | - |
| **0x78** | **SET_PAD_HOTCUE_PAGE** | page (1) [0-7 pour pages 1-8 de 16 hotcues] |

**Bloc 0xA0-0xAF : Motions (v2+)**

Les motions sont les mouvements d'encodeurs capturés en live via REC. Ce bloc gère la capture, le playback et l'édition des motions.

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xA0 | MOTION_START_RECORD | — (commence la capture, armé jusqu'au premier mouvement) |
| 0xA1 | MOTION_STOP_RECORD | — (termine la capture) |
| 0xA2 | MOTION_EVENT | timestamp (2) + voice_id (1) + param_id (2) + value (2) [envoyé Teensy → Pi pendant capture] |
| 0xA3 | MOTION_SAVE | voice_id (1) + param_id (2) + motion_name (max 32 bytes) — extrait et sauvegarde une motion individuelle |
| 0xA4 | MOTION_DELETE | motion_id (2) |
| 0xA5 | MOTION_LOAD | motion_id (2) — charge une motion sauvegardée |
| 0xA6 | MOTION_PLAYBACK_START | motion_id (2) + target_voice (1) + target_param (2) + loop (1) |
| 0xA7 | MOTION_PLAYBACK_STOP | motion_id (2) |
| 0xA8 | MOTION_ASSIGN_TO_AUG_SLOT | motion_id (2) + pad_id (1) + context_mode (1) + context_voice (1) |
| 0xA9 | MOTION_REQUEST_LIBRARY | — (retourne la liste des motions sauvegardées) |
| 0xAA | MOTION_SET_LENGTH | motion_id (2) + length_steps (1) [16/32/64] |
| 0xAB | MOTION_SET_QUANTIZE | motion_id (2) + quantize (1) [enum] |
| 0xAC | MOTION_LIBRARY_RESPONSE | count (2) + [motion_id (2) + name_length (1) + name (N)] × count |

**action_type 0x0E** (ajouté pour SET_PAD_BINDING) :
- **TriggerMotion** : action_data = motion_id (2) + target_voice (1) + target_param (2, si override)

**Bloc 0x70-0x7F : Pad mapping (mise à jour action_types)**

**action_type values** pour SET_PAD_BINDING :

| value | action_type | action_data (4 bytes interpretation) |
|-------|-------------|--------------------------------------|
| 0x00 | None (slot vide) | - |
| 0x01 | TriggerVoice | voice (1) + velocity (1) + pitch (1, signed) + reserved (1) |
| 0x02 | TriggerAugmentation | aug_id (2) + reserved (2) |
| 0x03 | TriggerAugmentationDSP | aug_id (2) + target_voice (1) + reserved (1) |
| 0x04 | PlayHotcue | hotcue_id (2) + quantize (1) + reserved (1) |
| 0x05 | StopHotcue | hotcue_id (2) + reserved (2) |
| 0x06 | LoadPattern | pattern_id (2) + quantize (1) + reserved (1) |
| 0x07 | LoadVoiceKit | kit_id (2) + voice (1) + quantize (1) |
| 0x08 | LoadScene | scene_id (2) + reserved (2) |
| 0x09 | LoadPerkonsSnapshot | snapshot_id (2) + quantize (1) + reserved (1) |
| 0x0A | SavePerkonsSnapshot | snapshot_id (2) + reserved (2) |
| 0x0B | CleanSlate | flags (1) + reserved (3) |
| 0x0C | VoiceMute | voice (1) + reserved (3) |
| 0x0D | VoiceSolo | voice (1) + reserved (3) |

**Bloc 0x80-0x8F : DSP chains (v2+)**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x80 | SET_FX_SLOT | voice (1) + slot (1) + fx_type (1) + preset_id (1) |
| 0x81 | SET_FX_PARAM | voice (1) + slot (1) + param (1) + value (2) |
| 0x82 | CLEAR_FX_CHAIN | voice (1) |
| 0x83 | LOAD_FX_CHAIN_PRESET | voice (1) + preset_id (2) |

**Bloc 0x90-0x9F : Voice Kits et Perkons Snapshots (v1+)**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x90 | SAVE_VOICE_KIT | voice (1) + kit_id (2) + name_length (1) + name (N ASCII ≤32) |
| 0x91 | LOAD_VOICE_KIT | voice (1) + kit_id (2) + quantize (1) |
| 0x92 | DELETE_VOICE_KIT | kit_id (2) |
| 0x93 | LIST_VOICE_KITS | - (retour via VOICE_KIT_LIST) |
| 0x94 | SAVE_PERKONS_SNAPSHOT | snapshot_id (2) + name_length (1) + name (N ASCII ≤32) |
| 0x95 | LOAD_PERKONS_SNAPSHOT | snapshot_id (2) + quantize (1) |
| 0x96 | DELETE_PERKONS_SNAPSHOT | snapshot_id (2) |

**Bloc 0xC0-0xCF : Device Profiles (v1.0.0+)**

Charge, décharge et liste les Device Profiles. Le profile YAML est parsé
et sérialisé en binaire compact côté Pi avant transfert au Teensy.

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xC0 | LOAD_DEVICE_PROFILE_BEGIN | profile_slot (1) + total_size_bytes (4) + crc32 (4) + name_length (1) + name (N ASCII ≤ 32) |
| 0xC1 | LOAD_DEVICE_PROFILE_CHUNK | profile_slot (1) + chunk_index (2) + chunk_size (1) + data (N bytes) |
| 0xC2 | LOAD_DEVICE_PROFILE_END | profile_slot (1) + checksum (4) |
| 0xC3 | UNLOAD_DEVICE_PROFILE | profile_slot (1) |
| 0xC4 | LIST_LOADED_PROFILES | — (retour via DEVICE_PROFILE_LIST_ENTRY) |
| 0xC5 | SET_ACTIVE_PROFILE_FOR_DEGRADED_MODE | profile_slot (1) |

**LOAD_DEVICE_PROFILE_BEGIN** :
- `profile_slot` (0-3) : slot cible.
- `total_size_bytes` : taille totale du blob binaire à transférer.
- `crc32` : checksum du blob complet, pour validation à la fin.
- `name` : nom du profile (e.g. "perkons_hd01"), max 32 ASCII.

**LOAD_DEVICE_PROFILE_CHUNK** :
- Chunks de 240 bytes max (cohérent avec LEN max protocole).
- Indexés en ordre croissant à partir de 0.
- Le Teensy n'ACK pas chaque chunk (gain bande passante).

**LOAD_DEVICE_PROFILE_END** :
- `checksum` recalculé par le Teensy et comparé au `crc32` initial.
- ACK via PROFILE_LOAD_ACK (0xCB Teensy → Pi, voir ci-dessous).

**UNLOAD_DEVICE_PROFILE** : libère le slot. Si le slot était l'actif du
mode dégradé, le runtime applique la politique de fallback (cf. notes
privées "Mode dégradé en multi-machine").

**SET_ACTIVE_PROFILE_FOR_DEGRADED_MODE** : indique au Teensy quel slot
sérialiser en EEPROM pour le mode dégradé. Mise à jour automatique à
chaque changement de page côté Pi (cf. anticipations v2-v4 "Mode
dégradé en multi-machine").

**Note sur le partage de plage 0xC0-0xCF** : le bloc 0xC0-0xCF Pi → Teensy
est intégralement dédié aux Device Profiles (CMD 0xC0-0xC5). Le bloc
0xC0-0xCF Teensy → Pi est en revanche partagé : 0xC0-0xC4 sont les états
hotcues existants (cf. ci-dessous), 0xCA-0xCC sont les ACK profile ajoutés
en v1.0.0. Les deux directions du protocole étant indépendantes (DIR=0x01
vs DIR=0x02 dans la trame), ce partage de plage hexa n'introduit aucun
conflit fonctionnel.

**Bloc 0xF0-0xFF : Maintenance et debug**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xF0 | ENTER_BOOTLOADER | - |
| 0xF1 | DEBUG_DUMP | section (1) |
| 0xF2 | CONFIG_PUSH_DONE | - (signale fin de push config initiale) |

### Commandes Teensy → Pi (DIR = 0x02)

**Bloc 0x80-0x8F : Connection**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x81 | HELLO_ACK | fw_version (4) + proto_major (1) + proto_minor (1) + proto_patch (1) + snapshot_size (4) |
| 0x82 | HEARTBEAT_ACK | timestamp_ms (4) |
| 0x83 | GOODBYE_ACK | - |
| 0x84 | CONFIG_PUSH_ACK | status (1) (0=OK, autre=error) |

**Bloc 0x90-0x9F : Valeurs de paramètres**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0x90 | PARAM_VALUE | param_id (2) + value (2) |
| 0x91 | PARAM_BULK | count (1) + [param_id (2) + value (2)] × N |
| 0x92 | PARAM_MODULATED | param_id (2) + final_value (2) + modulated_part (2) + override_active (1) |

**Bloc 0xA0-0xAF : Événements utilisateur**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xA0 | ENCODER | encoder_id (1) + delta (1 signed) + acceleration_mult (1) |
| 0xA1 | BUTTON_DOWN | button_id (2) + timestamp_ms (4) |
| 0xA2 | BUTTON_UP | button_id (2) + duration_ms (2) |
| 0xA3 | PAD_DOWN | pad_id (1) + velocity (1) + timestamp_ms (4) |
| 0xA4 | PAD_UP | pad_id (1) + duration_ms (2) |
| 0xA5 | PAD_STEP_EDIT | pad_id (1) + current_state (1) |

**Bloc 0xB0-0xBF : États de modulation**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xB0 | MOD_STATE | mod_id (2) + state (1) + progress_percent (1) |
| 0xB1 | MOD_COMPLETE | mod_id (2) |
| 0xB2 | AUG_TRIGGERED | aug_id (2) + started_at_step (2) |
| 0xB3 | AUG_COMPLETED | aug_id (2) |

**Bloc 0xC0-0xCF : États hotcues (v3+)**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xC0 | HOTCUE_STATE | hotcue_id (2) + state (1) + progress_percent (1) |
| 0xC1 | HOTCUE_CAPTURED | hotcue_id (2) + size_bytes (4) + sample_rate (2) + bit_depth (1) + channels (1) |
| 0xC2 | HOTCUE_CHUNK | hotcue_id (2) + offset (4) + chunk_size (1) + data (N bytes) |
| 0xC3 | HOTCUE_CHUNK_END | hotcue_id (2) + total_checksum (4) |
| 0xC4 | HOTCUE_LOADED_ACK | hotcue_id (2) + status (1) |

status values :
- 0x00 : OK
- 0x01 : checksum_fail
- 0x02 : memory_full
- 0x03 : ready (pour LOAD_HOTCUE_START)

**Bloc 0xCA-0xCF : Device Profiles (Teensy → Pi, v1.0.0+)**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xCA | DEVICE_PROFILE_LIST_ENTRY | profile_slot (1) + name_length (1) + name (N ASCII) + active_for_degraded (1) |
| 0xCB | PROFILE_LOAD_ACK | profile_slot (1) + status (1) + crc_received (4) |
| 0xCC | PROFILE_UNLOAD_ACK | profile_slot (1) + status (1) |

**status** values pour PROFILE_LOAD_ACK / PROFILE_UNLOAD_ACK :
- 0x00 : OK
- 0x01 : checksum_fail
- 0x02 : slot_already_occupied (LOAD vers un slot non vide)
- 0x03 : slot_empty (UNLOAD d'un slot vide)
- 0x04 : eeprom_write_fail (échec persistance mode dégradé)

**Bloc 0xD0-0xDF : Monitoring**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xD0 | CPU_USAGE | total_percent (1) + peak_percent (1) + effect_count (1) + [effect_id (2) + usage (1)] × N |
| 0xD1 | MEM_USAGE | psram_used_kb (4) + psram_total_kb (4) + ram_used_kb (2) |
| 0xD2 | AUDIO_LEVELS | v1_peak (1) + v2_peak (1) + v3_peak (1) + v4_peak (1) + master_peak (1) + master_rms (1) |

**Fréquences d'envoi recommandées** :
- PARAM_BULK : 30 Hz
- AUDIO_LEVELS : 50 Hz
- CPU_USAGE + MEM_USAGE : 5 Hz
- MOD_STATE : 30 Hz par mod active
- BAR_POSITION : à chaque mesure (1-3 Hz)
- BPM_DETECTED : sur changement >0.5 BPM, ou max 10 Hz

**Bloc 0xE0-0xEF : Clock et transport**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xE0 | BPM_DETECTED | bpm_centimals (2) |
| 0xE1 | CLOCK_STATUS | state (1) |
| 0xE2 | BAR_POSITION | bar (2) + beat (1) + step (1) |
| 0xE3 | TAP_TEMPO_RESULT | bpm_centimals (2) + tap_count (1) |

**Bloc 0xF0-0xFF : Erreurs et warnings**

| CMD | Nom | Payload |
|-----|-----|---------|
| 0xF0 | XRUN | count (2) + since_ms (4) |
| 0xF1 | WARNING | code (1) + data (N bytes, dépend du code) |
| 0xF2 | ERROR | code (1) + data (N bytes, dépend du code) |
| 0xF3 | SNAPSHOT_BEGIN | total_size_bytes (4) + chunk_count (2) |
| 0xF4 | SNAPSHOT_CHUNK | chunk_index (2) + data (N bytes) |
| 0xF5 | SNAPSHOT_END | checksum (4) |
| 0xF6 | VOICE_KIT_LIST_ENTRY | kit_id (2) + voice (1) + name_length (1) + name (N ASCII) |
| 0xF7 | PERKONS_SNAPSHOT_LIST_ENTRY | snapshot_id (2) + name_length (1) + name (N ASCII) |

---

## 7. Codes WARNING et ERROR

### WARNING codes (0xF1)

| code | Nom | data |
|------|-----|------|
| 0x01 | MIDI_LOOP_DETECTED | channel (1) + cc (1) |
| 0x02 | SET_PARAM_IGNORED_OVERRIDE_ACTIVE | param_id (2) |
| 0x03 | CPU_DEGRADATION | effect_id (2) + action_taken (1) |
| 0x04 | XRUN_THRESHOLD_EXCEEDED | count (2) |
| 0x05 | HOTCUE_TRANSFER_TIMEOUT | hotcue_id (2) |
| 0x06 | PAD_BINDING_NOT_FOUND | mode (1) + voice (1) + pad_id (1) |
| 0x07 | MOD_CLAMPED | param_id (2) (modulation dépasse 1.0) |
| 0x08 | AUG_CURVE_TOO_LONG | aug_id (2) (curve tronquée) |

### ERROR codes (0xF2)

| code | Nom | data |
|------|-----|------|
| 0x01 | PSRAM_FULL | free_kb (4) |
| 0x02 | SD_WRITE_FAIL | error_code (1) |
| 0x03 | CHECKSUM_FAIL | context (1) |
| 0x04 | PROTOCOL_VERSION_MISMATCH | required_major (1) + required_minor (1) |
| 0x05 | HARDWARE_INIT_FAIL | subsystem_id (1) |
| 0x06 | MIDI_OUT_FAIL | channel (1) |
| 0x07 | HOTCUE_LOAD_FAIL | hotcue_id (2) + reason (1) |
| 0x08 | UNKNOWN_COMMAND | cmd_id (1) |

### Subsystem IDs pour HARDWARE_INIT_FAIL

| value | Subsystem |
|-------|-----------|
| 0x01 | SPI |
| 0x02 | I2C |
| 0x03 | MIDI UART |
| 0x04 | OLED V1 |
| 0x05 | OLED V2 |
| 0x06 | OLED V3 |
| 0x07 | OLED V4 |
| 0x08 | OLED Master |
| 0x09 | MCP23S17 (expander) |
| 0x0A | NeoPixel | <=NeoPixel est SUPPRIMé>
| 0x0B | I2S Audio |

---

## 8. Format du SNAPSHOT

Le snapshot est envoyé par le Teensy au Pi après HELLO ou via `REQUEST_SNAPSHOT`. Chunké via SNAPSHOT_BEGIN / SNAPSHOT_CHUNK / SNAPSHOT_END.

### Structure du snapshot complet

```
[SNAPSHOT_HEADER]
  fw_version: 4 bytes
  proto_version: 3 bytes (major, minor, patch)
  timestamp_ms: 4 bytes
  section_count: 1 byte

[SECTION_PARAMS]
  section_id: 1 byte (0x01)
  section_size: 4 bytes
  param_count: 2 bytes
  [param_id (2) + value (2)] × param_count

[SECTION_MODULATIONS]
  section_id: 1 byte (0x02)
  section_size: 4 bytes
  mod_count: 2 bytes
  [mod_id (2) + source_type (1) + source_id (2) + target (2) + amount (2) + curve (1) + trigger_mode (1) + enabled (1) + state (1)] × mod_count

[SECTION_HOTCUES] (v3+)
  section_id: 1 byte (0x03)
  section_size: 4 bytes
  hotcue_count: 2 bytes
  [hotcue_id (2) + size_bytes (4) + loaded (1) + playing (1) + params] × hotcue_count

[SECTION_PAD_BINDINGS]
  section_id: 1 byte (0x04)
  section_size: 4 bytes
  binding_count: 2 bytes
  [context_mode (1) + context_voice (1) + pad_id (1) + action_type (1) + action_data (4) + color (3)] × binding_count

[SECTION_ARP]
  section_id: 1 byte (0x05)
  section_size: 4 bytes
  [params + steps × 4 voix]

[SECTION_SYSTEM]
  section_id: 1 byte (0x06)
  section_size: 4 bytes
  clock_source: 1 byte
  bpm_centimals: 2 bytes
  transport_state: 1 byte
  active_layout_id: 1 byte
```

### Taille typique

- v1 sans hotcues : ~2 KB
- v3 avec 50 hotcues chargés (métadonnées seules) : ~5 KB
- v4 complet : ~10-20 KB

Note : le snapshot contient seulement les **métadonnées** des hotcues, pas l'audio. L'audio est transféré séparément via `REQUEST_HOTCUE_DATA` si nécessaire.

---

## 9. Priorisation des messages dans le thread série

**3 niveaux de priorité** :

**Niveau 1 — Critique** :
- HEARTBEAT / HEARTBEAT_ACK
- ENCODER, BUTTON_DOWN/UP, PAD_DOWN/UP
- ERROR, XRUN
- SET_PARAM (utilisateur)

**Niveau 2 — Standard** :
- PARAM_VALUE, PARAM_BULK
- MOD_STATE
- LED_UPDATE, LED_BULK
- CPU_USAGE, MEM_USAGE, AUDIO_LEVELS
- OLED_TEXT, OLED_CLEAR

**Niveau 3 — Bulk** :
- LOAD_HOTCUE_CHUNK, HOTCUE_CHUNK
- SNAPSHOT_CHUNK
- OLED_BITMAP pleine taille

Entre chaque chunk niveau 3, vider les queues niveaux 1 et 2.

---

## 10. Position-in-bar et quantification

Le Teensy maintient un compteur depuis la MIDI Clock (24 PPQN).

### État

```cpp
struct BarPosition {
  uint16_t bar;       // mesures depuis start
  uint8_t beat;       // 0-3
  uint8_t step;       // 0-15
  uint32_t tick_us;   // depuis dernier pulse
};
```

### Synchronisation

- MIDI Start : reset à bar=0, beat=0, step=0
- MIDI Continue : reprend position précédente
- MIDI SPP : setter position absolue
- Comptage : 6 pulses/step, 24/beat, 96/bar

### Quantization options

| Valeur | Nom |
|--------|-----|
| 0x00 | 1/16 step |
| 0x01 | 1/4 beat |
| 0x02 | 1/2 half-bar |
| 0x03 | 1 bar |
| 0x04 | free (no quantization) |

---

## 11. Règles de combinaison des modulations

Pour un paramètre P ayant N modulations actives :

```
contribution_i = source_value_i * amount_i
combined = sum(contribution_i)
final_value = clamp(base_value + combined, min_range, max_range)
```

où :
- `base_value` : valeur encodeur physique
- `source_value_i` : valeur de la source à l'instant t, range [-1, +1]
- `amount_i` : range [-1, +1] (signed)
- `min_range, max_range` : selon type du paramètre (0..1 unipolar, -1..+1 bipolar)

### Priorité encodeur vs modulation

- Mode B (défaut) : override gagne pendant sa fenêtre, puis fade vers `base + combined`
- Mode C (SHIFT + encodeur) : `final = base + combined` en continu
- Mode A (override permanent) : modulations désactivées

### SET_PARAM pendant override encodeur

Si SET_PARAM arrive alors que l'encodeur correspondant est en override actif :
- SET_PARAM est ignoré
- WARNING envoyé au Pi (code 0x02)

---

## 12. Mapping par défaut du pad en mode dégradé

Mode pad automatique : **Drum Trigger**
- 4 rangées = 4 voix (rangée 0 = V1, rangée 3 = V4)
- 8 colonnes = 8 niveaux de vélocité (16 à 127)

| pad_col | Vélocité MIDI |
|---------|---------------|
| 0 | 16 |
| 1 | 32 |
| 2 | 48 |
| 3 | 64 |
| 4 | 80 |
| 5 | 96 |
| 6 | 112 |
| 7 | 127 |

Note MIDI : C3 (60) par convention, sur le canal de la voix (canal 1 pour V1, etc.).

### Mapping encodeurs voix en mode dégradé

En mode dégradé, les encodeurs voix envoient des CCs directement au Perkons selon leur position :

| Position | Paramètre | CC V1 | CC V2 | CC V3 | CC V4 |
|----------|-----------|-------|-------|-------|-------|
| Haut-gauche | Tune | 70 | 81 | 92 | 103 |
| Haut-droite | Decay | 74 | 85 | 96 | 107 |
| Bas-gauche | Cutoff | 72 | 83 | 94 | 105 |
| Bas-droite | Drive | 76 | 87 | 98 | 109 |

Canal MIDI : 1 pour V1, 2 pour V2, etc.

### Feedback LED mode dégradé

- Rangée V1 : rouge
- Rangée V2 : orange
- Rangée V3 : jaune
- Rangée V4 : bleu
- Intensité croissante avec la colonne (marque la vélocité)

---

## 13. Versionnage du protocole

### SemVer : major.minor.patch

- **Major** : breaking changes
- **Minor** : nouvelles commandes compatibles
- **Patch** : corrections

### Version actuelle : **1.0.0** (v0.7 de la spec PërKompanion)

### Négociation au handshake

Via `HELLO` (0x01) et `HELLO_ACK` (0x81). Si majors différents → ERROR PROTOCOL_VERSION_MISMATCH (0x04) + GOODBYE.

### Évolution

- Ajout d'une commande → incrémenter minor
- Modification d'une commande existante → incrémenter major
- Les IDs ne sont **jamais réassignés**

---

## 14. Device Profile Schema

> Section ajoutée en v1.0.0 dans le sillage du pivot framework.

PërKompanion charge dynamiquement des **Device Profiles** YAML qui décrivent
la grammaire MIDI des drum machines, synthés ou samplers hardware pilotés.
Chaque profile occupe un des 4 slots du bloc `0x0000-0x0FFF` (cf. §2).

### Documents associés

- **`device_profile_schema.md`** : spécification narrative complète du format
  YAML (sections obligatoires, optionnelles, grammaire des params, mécanismes
  supportés, conventions YAML).
- **`profile_template_with_docs.yaml`** : template auto-documenté pour
  contributeurs qui rédigent un nouveau profile.
- **`profiles/perkons_hd01.yaml`** : profile de référence Perkons HD-01.
- **`profiles/elektron_digitakt_ii.yaml`** : profile Digitakt II (paradigme
  Multi Channel, tracks hétérogènes).
- **`profiles/korg_electribe_2_sampler.yaml`** : profile Electribe 2 Sampler
  (paradigme Single Channel + Part Select, NRPN).
- **`profiles/behringer_td3_mo.yaml`** : profile TD-3-MO (minimaliste).

### Chaîne YAML → Binaire → Teensy

1. Le Pi parse le YAML (`js-yaml`), valide contre le schéma.
2. Le Pi sérialise le profile en binaire compact (table de params + offsets
   CC + courbes + preface MIDI). Format binaire : à spécifier en pré-v1
   du firmware Teensy.
3. Le Pi transfère le blob via `LOAD_DEVICE_PROFILE_BEGIN/CHUNK/END`
   (bloc 0xC0-0xCF, cf. §6).
4. Le Teensy stocke le blob en RAM (mode normal) et l'écrit en EEPROM si
   désigné comme actif pour le mode dégradé.
5. Le Teensy alloue les `param_id` du slot ciblé selon l'ordre de
   déclaration et expose ces paramètres au moteur de modulation.

### Capacité

- 4 profiles externes simultanés en v1 (slots 0-3 du bloc 0x0000-0x0FFF).
- 1024 paramètres par slot — capacité largement supérieure aux profiles
  observés (~414 params pour Digitakt II, le plus dense des 4 testés).
- Extension v5+ envisagée vers le bloc `0x7000-0x7FFF` si plus de 4 profiles
  simultanés sont nécessaires.

### Mode dégradé

Le Teensy stocke en EEPROM (4 KB sur Teensy 4.1) un snapshot binaire du
profile désigné actif via `SET_ACTIVE_PROFILE_FOR_DEGRADED_MODE` (CMD 0xC5).
Si le Pi tombe, le Teensy continue à piloter le device avec ce profile.
En cas d'EEPROM corrompu, fallback automatique vers Perkons hardcodé
(cf. notes privées "Mode dégradé en multi-machine").

---

*Fin de PERKOMPANION_PROTOCOL_TABLES.md — v1.0.0, avril 2026*
