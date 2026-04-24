# PërKompanion — Décisions de conception

> Journal chronologique des décisions de design et d'implémentation.
> Complément aux docs techniques (VISION, PHASE0, panel_design, etc.).
> À consolider périodiquement dans les docs principaux.

---

## Session 2026-04-24 — Décisions v0.9 bis

### 1. Matériau du module pad

**Décision** : **Contreplaqué peuplier 5mm** pour le module pad surélevé (190 × 115 mm).

**Historique d'évolution** :
- v0.8 : acrylique violet translucide satiné 3mm
- v0.9 initial : contreplaqué okoumé mono-essence 3mm
- **v0.9 bis (décision actuelle)** : contreplaqué peuplier CP 5mm

**Raisons du choix final (peuplier local 5mm)** :
- **Éthique écologique** : essence européenne, pas de bois tropical, pas de déforestation, faible empreinte carbone
- **Soutien filière locale** (scieries européennes)
- **Rigidité suffisante** en 5mm, largement renforcée par le collage des switches (voir point 3)
- **Prix raisonnable**
- **Usinage laser CO2** standard (Sculpteo OK, FabLab OK)
- **Cohérence avec l'identité "artisan noble + éthique locale"** du projet Sauvignac

**Performance vs okoumé** : le peuplier est ~30% moins rigide que l'okoumé à épaisseur équivalente, mais l'épaisseur 5mm (vs 3mm) compense largement, et le collage des switches neutralise la différence en usage réel.

### 2. Fixation du module pad à la plate principale

**Décision** : **Entretoises filetées M3 femelle-femelle de 3mm** + vis courtes (M3×15-20mm).

**Historique d'évolution** :
- v0.8 : entretoises M3×10mm + vis M3×25mm
- v0.9 bis : **entretoises M3 F/F × 3mm + vis M3×15-20mm**

**Raisons** :
- Avec entretoise 10mm + module 3mm (= 13mm au-dessus plate), le stem du switch MX (qui dépasse de ~10.5mm) ne ressortait pas du module → impossible de clipper le keycap.
- Avec **entretoise 3mm + module 5mm (= 8mm au-dessus plate)**, le stem dépasse de 2.5mm au-dessus du module → clippage keycap OK.
- L'entretoise filetée F/F permet un assemblage propre sans vis qui dépasse trop.

**Nombre d'entretoises** : 4 aux coins du module (6 si rigidité souhaitée renforcée au centre).

### 3. Fixation des switches dans le module

**Décision** : **Collage des switches par la face arrière du module**, en complément du clippage mécanique.

**Raisons** :
- Module en peuplier 5mm plus épais que l'épaisseur standard plate Cherry MX (1.5mm) → clips des switches peuvent ne pas se fermer complètement.
- Les switches seront **définitivement soudés** au PCB/matrice une fois le projet en place → pas de démontage prévu → collage acceptable.
- **Rigidifie l'ensemble** : les 45 switches collés deviennent un assemblage quasi monolithique avec le module, compensant la flexibilité du peuplier.
- **Sécurité** : pas de risque qu'un switch se désengage sous frappe violente en live.

**Colles recommandées** :
- Époxy 2 composants (JB Weld, Araldite) — résistance ultime, permanent
- Super-glue cyanoacrylate pour plastique (Loctite 406) — rapide, solide

**Procédure** :
1. Insertion des switches dans les trous du module (clippés et alignés)
2. Application de colle sur le pourtour du corps inférieur de chaque switch (face arrière du module)
3. Séchage 24h pour époxy, 1h pour cyanoacrylate

### 4. Calcul dimensionnel final du module pad

**Empilage** :
- Plate principale : acrylique 3mm noir mat (inchangé)
- Entretoises M3 F/F : **3mm**
- Module pad : **peuplier CP 5mm**
- Total matière : **11 mm** au-dessus de la plate principale

**Géométrie du switch** :
- Bas du module à 3mm au-dessus plate (entretoise 3mm)
- Haut du module à 8mm au-dessus plate (3mm entretoise + 5mm module)
- Stem Cherry MX dépasse de la plate : 10.5mm
- **Stem dépasse au-dessus du top du module de : 2.5 mm** ✅
- Le keycap peut se clipper sur ces 2.5mm de stem exposés

**Hauteur keycap R4 posé** : haut du keycap à ~15.6mm au-dessus de la plate principale (cohérent avec standards MPC/Push 15-20mm).

### 5. Finition du module peuplier

**Décision en cours** : finition à valider ultérieurement. Options en lice :

- **Huile dure (Rubio Monocoat, Osmo Polyx-Oil)** : aspect mat/satin chaud, artisanal, facile à appliquer, réparable
- **Laque brillante (vernis polyuréthane acrylique)** : look piano laqué premium, plus technique à appliquer, sensible aux empreintes
- **Teinture foncée + huile satin** : look "ébène de luthier", cohérent avec kit SUIE

**Préférence actuelle** : huile Rubio Monocoat (simple, cohérent avec artisan noble).

### 6. Rétroéclairage des voix virtuelles V5-V8

**Décision** : **Ajouter une jauge volume horizontale** au bas des cartes V5-V8 dans le Home Dashboard UI.

**Raison** : les voix physiques V1-V4 sont contrôlées par les knobs hardware du Perkons (visuel assuré par le Perkons lui-même). Les voix virtuelles V5-V8 sont générées par le Teensy, donc sans retour visuel hardware → la jauge UI est nécessaire.

**Implémentation** :
- Barre horizontale 8px de haut, full width de la card
- Gradient vert → jaune → orange → rouge selon niveau
- Tick vertical 2px à la position 0 dB (référence de gain staging)
- Pas de label "VOL" ni de valeur dB (épuré)

### 7. Identité graphique : Logo Sauvignac

**Décision** : **Logo "S" fracturé pixelisé** avec carré orange (#FF3D00) en accent.

**Direction créative** : "Destructuré × Pixelisé × Élégant" — noblesse typographique altérée par interventions brutales.

**Caractéristiques** :
- S serif classique de base
- 5 coupures horizontales qui déplacent des sections du S (glitch contrôlé)
- Petit carré orange en haut à droite (accent Perkompanion, cohérence palette)
- Look "noblesse fracturée" cohérent avec positionnement Sauvignac

**Variantes à produire** :
- [ ] Mono noir pur (sans carré) pour gravure laser
- [ ] Mono blanc sur noir (inversion)
- [ ] Favicon 16×16 et 32×32
- [ ] Version SVG vectorisée (pour gravure laser et scalabilité)

### 8. UI Home Dashboard

**Décision** : **Home Dashboard validé après 6 itérations Claude Design**.

**Caractéristiques finales** :
- Résolution cible : 1024 × 600 landscape (écran Elecrow 7")
- Charte graphique : **Cyber Brutalist sans bleu**
- Palette : warm-black + orange incandescent + magenta + vert acide + violet profond
- Typographie : Archivo Black (display) + Inter (UI) + JetBrains Mono (data)
- Layout : top menu bar (B1-B6) + voice grid central + encoder pastilles latérales (E1-E8)
- Distinction physique/virtuelle : rouge pour V1-V4, fuchsia/magenta pour V5-V8
- Spectres de fréquences par voix (au lieu de waveforms)
- Bloc "Activité en direct" avec mini grille 4×2 des voix affectées par chaque activité

### 9. Branding et identité de marque

**Décision** : positionnement "**Sauvignac**" comme **label/atelier parent**, **Perkompanion** comme premier produit.

**Raisons** :
- Permet d'autres produits futurs sous même ombrelle Sauvignac (kits samples, outils software, futurs instruments)
- Cohérent avec stratégie long terme
- Crédite l'auteur Alexandre de Sauvignac sans enfermer le produit

**README** :
- Mention "by Sauvignac, amateur musicien passionné par la techno et les synthétiseurs hardware depuis plus de vingt ans"
- Transparence sur utilisation de Claude (Anthropic) comme "design partner" pour architecture, firmware, documentation

### 10. Infrastructure Git

**Décision** : **consolidation de tous les projets sur le compte GitHub "Sauvignac"**.

**Repos** :
- `Sauvignac/perkompanion` (public, licence MIT)
- Autres projets antérieurs transférés sur Sauvignac pour simplicité

**Connecteur Claude.ai GitHub** : limité à 1 compte, donc connecté à Sauvignac.

---

## À faire prochainement

### Court terme (semaines à venir, en attente composants)
- [ ] Variantes du logo Sauvignac (mono noir, blanc, favicon, SVG vectorisé)
- [ ] Intégration du logo définitif dans le Home Dashboard (itération 7 Claude Design)
- [ ] Setup Raspberry Pi 5 à sa réception (distribution Linux, Chromium kiosk, test écran)
- [ ] Préparer environnement de dev Teensy (PlatformIO/Arduino IDE avec Teensyduino)
- [ ] Finaliser dessin AutoCAD de la plate principale (attendre mesures précises des composants critiques)

### Moyen terme (après arrivée Teensy)
- [ ] Test circuit MIDI sur breadboard (MIDI IN + MIDI OUT)
- [ ] Commande Sculpteo : plate test puis plate principale + module peuplier
- [ ] Premier firmware Teensy (boot, ping série, test OLED)
- [ ] Écrans UI suivants (Voice Edit LFO, Settings, Libraries, etc.)

### Long terme
- [ ] Nouveau kit sample pack Perkons après SUIE
- [ ] Communication projet (build-in-public sur réseaux sociaux, dev log)

---

## Historique des sessions précédentes

Ce journal a été initié le 2026-04-24. Les décisions antérieures sont consolidées dans :
- `PERKOMPANION_VISION.md` v0.9
- `PERKOMPANION_PHASE0.md` v0.9
- `PERKOMPANION_PROTOCOL_TABLES.md` v0.9
- `panel_design.md` v0.9
- `teensy_pinout.md` v0.9
- `midi_hardware.md` v0.9
