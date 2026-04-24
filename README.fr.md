# Perkompanion

> Un compagnon hardware + software pour l'Erica Synths Perkons HD-01.

🇬🇧 [Read in English](README.md)

---

## Qu'est-ce que Perkompanion ?

Perkompanion est une extension modulaire, matérielle et logicielle, conçue pour étendre les capacités de l'[Erica Synths Perkons HD-01](https://www.ericasynths.lv/shop/standalone-instruments/perkons-hd-01/). Il ajoute un pad de déclenchement drum, des voix virtuelles, l'enregistrement de motions, une bibliothèque de hotcues, une chaîne d'effets DSP, et un routing MIDI complet — tout en conservant le Perkons comme horloge maître et moteur percussif principal.

L'objectif est de transformer le Perkons en un véritable instrument de live performance sans remplacer ce qui fait son caractère.

---

## Status

**Version 0.9** — design finalisé, composants en commande, démarrage de la phase d'assemblage.

Le projet est en développement actif. L'architecture matérielle et les spécifications logicielles sont documentées (voir [Documentation](#documentation) ci-dessous), mais aucun firmware ni logiciel n'a encore été publié. Le dépôt contient pour l'instant la documentation de conception.

---

## Fonctionnalités prévues

- **Pad trigger 8×4 Cherry MX** avec 8 sélecteurs de voix et 5 boutons de mode intégrés dans un module en contreplaqué okoumé
- **8 colonnes de voix** (V1–V4 physiques via le Perkons, V5–V8 virtuelles) avec OLED et encodeurs dédiés
- **Bibliothèque de 128 hotcues** stockés en PSRAM pour déclenchement instantané
- **Enregistrement de motions** pour capturer les automations de paramètres en live
- **Chaîne DSP multi-slots** par voix avec 13 types d'effets
- **Routing MIDI avancé** via une interface tactile 7 pouces
- **Mode dégradé** : l'instrument reste jouable même si le Pi ou le Teensy cesse de répondre — le Perkons reste autonome

---

## Architecture matérielle

- **Teensy 4.1 Fully Loaded (32 MB PSRAM)** : cœur temps-réel pour MIDI, DSP audio, séquençage, hotcues
- **Raspberry Pi 5** : interface utilisateur (Chromium en mode kiosk sur écran tactile 7"), gestion de la bibliothèque de hotcues
- **Chaîne audio** : 2× PCM1808 (4 entrées depuis le Perkons) + 6× PCM5102A (12 sorties) + 1× TPA6120 ampli casque
- **Extension IO** : 20× MCP23S17 sur 3 chaînes SPI pour boutons, LEDs, encodeurs
- **Panneau** : plate principale 450×370 mm en acrylique noir mat + module 190×115 mm en contreplaqué okoumé pour la grille 9×5
- **MIDI** : 1 IN + 1 OUT DIN, connecté au Perkons via le routeur ESI M8U eX

Détails complets dans [`panel_design.md`](panel_design.md) et [`teensy_pinout.md`](teensy_pinout.md).

---

## Roadmap

- **Phase 0** (en cours) : documentation, figement de l'architecture, approvisionnement
- **Phase 1** : mise en route Teensy + Pi, routing MIDI basique, affichage écran
- **Phase 2** : voix virtuelles, moteur de hotcues, DSP basique
- **Phase 3** : chaîne DSP complète, enregistrement de motions, routing avancé
- **Phase 4** : boîtier final, finitions performance, publication

Voir [`PERKOMPANION_VISION.md`](PERKOMPANION_VISION.md) pour la roadmap détaillée.

---

## Documentation

Le design et les spécifications techniques vivent dans ce dépôt :

- [`PERKOMPANION_VISION.md`](PERKOMPANION_VISION.md) — Vision produit, fonctionnalités, roadmap, BOM
- [`PERKOMPANION_PHASE0.md`](PERKOMPANION_PHASE0.md) — Décisions techniques fondatrices
- [`PERKOMPANION_PROTOCOL_TABLES.md`](PERKOMPANION_PROTOCOL_TABLES.md) — Tables de référence (MIDI, protocole, IDs)
- [`panel_design.md`](panel_design.md) — Layout mécanique du panneau et dimensions
- [`teensy_pinout.md`](teensy_pinout.md) — Assignation GPIO du Teensy 4.1
- [`midi_hardware.md`](midi_hardware.md) — Circuit de l'interface MIDI DIN

---

## Construire le tien

Un guide de construction complet sera publié une fois la Phase 1 validée. En attendant, la documentation ci-dessus contient assez d'informations pour commencer à approvisionner les composants et planifier un build. N'hésite pas à ouvrir une issue si tu veux suivre le projet ou l'adapter à ton propre setup.

---

## À propos du développement

Perkompanion est conçu et construit par Sauvignac, musicien amateur passionné par la techno et les synthétiseurs hardware depuis plus de vingt ans, sans formation en électronique ni en développement embarqué.

L'architecture, le firmware et la documentation sont développés avec l'assistance de [Claude](https://www.anthropic.com/claude) (Anthropic), utilisé comme partenaire de conception pour naviguer les décisions techniques, explorer les compromis, et structurer les détails d'implémentation. Chaque choix de design et direction artistique reste celui de Sauvignac — Claude est un outil, pas un co-auteur.

Ce projet est aussi, modestement, une petite démonstration que les constructions hardware ambitieuses deviennent accessibles aux non-ingénieurs lorsque l'assistance IA est utilisée de manière réfléchie.

---

## Licence

MIT License. Voir [`LICENSE`](LICENSE) pour les détails.

---

## Contact

Issues et discussions bienvenues sur ce dépôt.
