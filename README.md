# PërKompanion

> Un compagnon hardware + software qui transforme l'Erica Synths Perkons HD-01 en instrument augmenté, sans jamais modifier son fonctionnement natif.

**Status** : Phase 0 — Caractérisation MIDI et préparation  
**Première release prévue** : fin 2026 / début 2027

---

## Le projet

PërKompanion ajoute au Perkons HD-01 ce que son firmware ne propose pas :

- **Multi-LFOs indépendants** par voix et par paramètre (le Perkons n'en a qu'un seul, global, non-resettable)
- **Transitions synchronisées au step 1** pour tout changement de kit, pattern ou modulation
- **Hybrid Kit Mode** — hot-swap d'une voix individuelle vers le timbre d'un autre kit, sans toucher aux 3 autres voix
- **Monitoring temps réel** de tous les paramètres, avec comparaison live vs stored
- **Captures de jam** automatiques, nommées selon le contexte kit/pattern actif

Le Perkons reste la source sonore unique. PërKompanion pilote ses paramètres en MIDI, en overlay non-destructive. Débranche PërKompanion, le Perkons reste exactement comme avant.

## Architecture

- **Cerveau** : Raspberry Pi 5 4GB — moteur Python (Flask + mido)
- **Interface** : écran tactile 10.1" HDMI intégré au module + accès Wi-Fi depuis laptop/téléphone
- **Panneau hardware** : 16 encodeurs (4 par voix en 2×2), 4 OLEDs voix, pad mécanique 8×4 avec LEDs RGB, 8 boutons voix, 5 boutons globaux
- **Bridge I/O** : Arduino Mega 2560 (ou RP2040 en alt build)
- **MIDI** : dongle USB-MIDI en Phase 1-2, sortie DIN via GPIO en Phase 4+

Le module se pose ou se clipse au-dessus du Perkons, aligné sur ses 45cm de large.

## Philosophie

- **Overlay non-destructive** : le Perkons reste un Perkons
- **Musicien virtuel** : PërKompanion joue des knobs à côté de l'humain, ne remplace rien
- **Pas de saisie de secrétariat** : toute feature qui demande de remplir des champs pendant un jam est suspecte
- **Open source MIT** : partage, reproductibilité, contributions bienvenues

## État actuel

- Reverse engineering du format `.KIT` du Perkons (firmware v1.2) : structure Protocol Buffers décodée, 90% des paramètres mappés
- Mapping complet des CC MIDI (single + multi mode) documenté
- Budget MIDI Perkons caractérisé : encaisse 800+ messages/sec sans artefact, validation du scénario multi-LFO ambitieux
- Spec v0.3 complète disponible dans `docs/`

Le code n'existe pas encore. Le projet démarre publiquement maintenant.

## Roadmap

| Phase | Objectif | Statut |
|-------|----------|--------|
| 0 | Caractérisation MIDI + préparation | ✅ en cours |
| 1 | MVP moniteur basique | 🔜 |
| 2 | Moniteur complet avec snapshots | 🔜 |
| 2.5 | Premier LFO musical | 🔜 |
| 3 | Premier module hardware breadboard | 🔜 |
| 4 | Panneau hardware complet | 🔜 |
| 5 | Moteur de LFOs externes | 🔜 |
| 6 | Hybrid Kit Mode + Pad fonctionnel | 🔜 |
| 7 | Scenes/Setlist + Kit Editor | 🔜 |
| 8 | Documentation et lancement public | 🔜 |

Estimation globale : 14-18 mois de développement solo en parallèle d'un job.

## Matériel de référence

- Erica Synths Perkons HD-01 (firmware v1.2)
- Documentation officielle : [erica-synths.lv](https://www.erica-synths.lv/shop/desktop-synthesizers-and-accessories/perkons-hd-01/)

## Licence

[MIT](LICENSE) — libre d'utilisation, modification, redistribution.

## Auteur

Alexandre de Sauvignac — producteur hardtek / free tekno, basé en France.

---

## English summary

PërKompanion is a hardware + software companion that turns the Erica Synths Perkons HD-01 into an augmented instrument, without modifying its native behavior.

Key features in development :
- Multi-LFO modulation (independent per voice and per parameter)
- Step-1 synchronized transitions for all changes
- Voice-level kit hot-swapping (Hybrid Kit Mode)
- Real-time parameter monitoring with live vs stored comparison

Built around Raspberry Pi 5 + touch screen + custom hardware panel. Communicates with the Perkons via MIDI DIN.

Open source, MIT licensed. Contributions welcome once the project reaches Phase 2.

Project started April 2026. First usable release expected late 2026 / early 2027.
