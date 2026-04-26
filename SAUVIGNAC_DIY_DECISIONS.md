# Sauvignac DIY — Journal des décisions

> Journal de conception des projets DIY Sauvignac.
> Le projet PërKompanion (#44) possède son propre journal : `PERKOMPANION_DECISIONS.md`.
> La roadmap complète des 45 projets est documentée dans `sauvignac_projets.md`.
>
> **Ce document est public.** Notes privées et stratégie chaîne dans des fichiers séparés non commités.

---

## Sommaire

- [Roadmap 45 projets — vue d'ensemble](#roadmap)
- [Direction visuelle — Cartoon Custom Paint](#direction)
- [Pédales d'effets — Décisions par projet](#pedales)
  - [Épisode 1 — Test physique du langage Cartoon Custom Paint](#pedales-ep1)
- [À faire prochainement](#a-faire)

---

<a id="roadmap"></a>
## Roadmap 45 projets — vue d'ensemble

La roadmap complète, détaillée par projet et par tier, est dans **`sauvignac_projets.md`** (document maître).

### Logique des tiers

| Tier | Description | Complexité |
|---|---|---|
| **TIER 1 — PURE** | Analogique pur, through-hole, zéro numérique. Kit ultra accessible. | ⭐ |
| **TIER 2 — HYBRID** | Même circuit + microcontrôleur. Preset saving, MIDI control, LEDs. | ⭐⭐ |
| **TIER 3 — MACHINE** | Interface complète. OLED, encodeurs, banque presets, MIDI in/out, CV. | ⭐⭐⭐ |

### Vue par phase

| Phase | Thème | Projets |
|---|---|---|
| 1 | Fondations | #1 Noise Box CD40106 · #2 Bazz Fuss · #3 PT2399 Lo-fi Delay ⭐ |
| 2 | Identité sonore | #4 Filtre Steiner-Parker ⭐ · #5 Ring Modulator · #6 Wavefolder · #7 Spring Reverb · #8 Drone Box VCO |
| 3 | Modulation | #9 LFO standalone · #10 Phaser BBD ⭐ · #11 Flanger BBD · #12 Chorus BBD · #13 Tremolo optique · #14 Vibrato optique |
| 4 | Delay / Réverb étendus | #15 Reverb Spin FV-1 · #16 Tape Delay PT2399 multi |
| 5 | Dynamique | #17 Compresseur optique · #18 Noise gate · #19 VCA |
| 6 | Synthèse sonore | #20 Atari Punk Console · #21 Drum kick · #22 Drum snare · #23 VCO AS3340 |
| 7 | Utilitaires CV/MIDI | #24 MIDI to CV · #25 Clock divider · #26 Random gate · #27 Séquenceur 8 pas · #28 Slew limiter · #29 Sample & Hold · #30 Quantizer |
| 8 | Utilitaires audio | #31 DI Box · #32 Mixer passif · #33 Buffer/splitter · #34 Envelope follower · #35 Octaver |
| 9 | Expérimental | #36 Piezo preamp · #37 Feedback mixer · #38 Theremin optique · #39 Touch controller · #40 Feedback osc · #41 Circuit Bending kit |
| 10 | Numérique / DSP | #42 Daisy Bitcrusher · #43 Daisy Granulaire ⭐ |
| 11 | Fil rouge / Long terme | **#44 PërKompanion** (journal séparé) · **#45 SAUVIGNAC MULTIEFFET** ⭐ |

### Calendrier éditorial suggéré

```
PHASE 1  Noise Box + Bazz Fuss + PT2399 T1     → Épisodes Sauvignac DIY 1-3
PHASE 2  PT2399 T2 + Filtre Steiner T1         → Épisodes 4-5
PHASE 3  PT2399 T3 + Filtre Steiner T2/T3      → Épisodes 6-8
...      En parallèle : PërKompanion fil rouge (chaîne séparée)
FINAL    Daisy Granulaire + Multieffet         → Épisodes 20+
```

> Les **deux chaînes YouTube** (Sauvignac DIY pour les projets analogiques, et la série PërKompanion pour le fil rouge) tournent en parallèle. Chacune avec sa playlist dédiée.

---

<a id="direction"></a>
## Direction visuelle — Cartoon Custom Paint

**Date** : 2026-04-26
**Statut** : ✅ Validée pour toute la roadmap pédales (et applicable aux autres familles DIY).

**Décision** : direction visuelle Sauvignac officielle = **Cartoon Custom Paint** — boîtier peint à la bombe en couleur saturée flat + contours et détails redessinés à la main au Posca PC-7M par-dessus + knobs métal moletés réels + vernis acrylique satin.

### TL;DR

Style culturellement ancré dans le custom auto européen (Cherry Lighter, voitures BD jaunes), la BD européenne (Métal Hurlant, ligne claire), et l'esthétique flyer free party / hardtek 90s-2000s. Signature Sauvignac systémique : **carré orange #FF4500 constant + numéro d'édition manuscrit**.

### Pivot stratégique

Cette direction est l'aboutissement d'une exploration qui a traversé plusieurs pistes :

| Étape | Direction explorée | Issue |
|---|---|---|
| 0 | Plaque peuplier 3mm collée + frappe laser + acrylique orange | Direction initiale — abandonnée |
| 1 | Cuir vachette TV gainage 5 faces, coutures sellier, fil orange contrasté | Architecture conçue mais abandonnée (presse 5T trop petite pour le boîtier entier) |
| 2 | Étiquette cuir orange rivetée laiton sur face avant | Abandonnée pour la même raison |
| 3 | Posca + vernis intégral, langage maroquinier abandonné | Risque "look cheap" identifié |
| 4 | Patchwork UV (cuir simulé en impression UV) | Abandonné car nécessite R&D graphique trop lourde |
| 5 | **Cartoon Custom Paint — bombe + Posca + vernis** | ✅ **Direction validée** |

### Rationale

- **Cohérence ADN free party** : le langage hand-painted custom paint vient directement de la culture du flyer free party / customisation auto européenne — c'est natif à l'univers Sauvignac
- **Différenciation marché** : aucun autre builder de pédales n'utilise ce langage (les pédales boutique américaines type Walrus, Chase Bliss, Strymon sont en UV propre standard)
- **Faisabilité technique** : matériel léger (~50 € de Posca + bombes + vernis), aucun investissement lourd, exécutable seul à l'atelier
- **Scalabilité validée** : le langage tient sur 4 pédales différentes (Noise Box / Bazz Fuss / Echo Tribe / Grain Mill) avec 4 couleurs et 4 niveaux de complexité différents
- **Signature artistique** : geste main du fabricant, chaque pédale unique tout en étant cohérente avec la famille
- **Réparable et itérable** : un boîtier peut être repeint et redessiné si raté ou à réinventer dans 5 ans

### Style identifié

**Cartoon Custom Paint** (terme du custom auto européen) — technique consistant à peindre un objet en aplat de couleur saturée puis à redessiner par-dessus tous ses détails et arêtes au marqueur épais, comme si l'objet devenait une illustration BD de lui-même qui aurait pris du volume.

### Références visuelles

- **Cherry Lighter** et ateliers similaires de custom paint cartoon (Instagram #cartooncarpaint, #sketchcarpaint, #handpaintedcar)
- Voitures customisées en cel-shaded paint
- Métal Hurlant magazine (BD européenne, contours épais, hachures)
- Frank Miller / Sin City (hachures gestuelles d'ombre)
- Flyers free party France/Allemagne 1995-2005 (Heretik, Spiral Tribe, Neurotixon)

### Système graphique — règles invariantes

#### 1. Base couleur (étape bombe)

- Couleur saturée flat unique par pédale
- Bombe acrylique pulvérisée à plat sur le boîtier complet (Hammond 1590B, 1590BB ou 1590DD selon le modèle)
- **Imperfections naturelles conservées** : grain de spray visible, légères micro-variations de saturation, éventuelles très légères coulures discrètes
- **Aucun dégradé**, aucun aérographe, aucune texture imitant un autre matériau
- Séchage 24h avant Posca

#### 2. Contours noirs au Posca PC-7M (étape main)

- Marqueur **Posca PC-7M** (épaisseur 4.5-5.5mm), encre noire
- Tracé de l'**ensemble du contour extérieur** du boîtier
- Tracé de **chaque arête** de la boîte
- **Geste main assumé** : épaisseur variable 4-8mm, légers dérapages, occasionnels dépassements aux coins en pointes effilées, parfois un trait doublé en correction, parfois une rupture
- **Jamais vectoriel**, jamais à la règle, jamais au compas

#### 3. Redoublement de chaque composant fonctionnel

Principe Cherry Lighter : **chaque détail mécanique est retracé sur la peinture**.

- Cercle hand-painted noir épais autour de la base de **chaque knob**
- Cercle hand-painted noir autour du **footswitch** + quelques traits courbes lâchés en sunburst sketch
- Petit cercle autour de la **LED** + 4-6 petits traits "rayonnement" courts
- Hexagone hand-painted autour de chaque **écrou de jack** sur les flancs
- Petit cercle autour du **jack DC** + symbole **DC ⊖–⊕** main
- Rectangle hand-painted autour du port **USB-C** si applicable
- Cadre rectangulaire hand-painted autour de l'**OLED** si applicable

#### 4. Hachures sketch lâchées

**Convention critique** : les hachures doivent être **gestuelles, rapides, irrégulières**, pas des trames régulières.

Caractéristiques obligatoires :
- Traits lâchés en gestes courts (~1/2 seconde par trait)
- **Alignés avec l'orientation de la surface** (verticaux sur faces verticales, horizontaux sur faces horizontales, suivent la perspective)
- Longueurs variables (5-25mm), épaisseurs variables, espacements irréguliers
- Pas tous parallèles
- Référence : storyboard sketches, brouillons de graphic novels, illustrations de fanzines punk

**Interdits** :
- ❌ Trames Ben-Day (points réguliers)
- ❌ Cross-hatching mécanique régulier
- ❌ Demi-tons type pop-art américain
- ❌ Lignes parallèles tirées à la règle
- ❌ Patterns imprimés ou répétitifs

#### 5. Highlights blancs flat (convention ligne claire)

- 2-3 bandes blanches courtes (8-15mm × 2-3mm) sur la face supérieure, alignées avec la perspective
- 1-2 petits triangles blancs aux arêtes arrondies
- Éventuellement un fin trait blanc le long de l'arête avant
- Chaque highlight est cerné d'un fin contour noir
- **Petits et discrets**

#### 6. Vernis acrylique satin (étape finition)

- 2-3 couches fines de vernis acrylique satin en aérosol
- Mat ou satin selon préférence finale (à valider en test physique) — pas brillant
- Protection mécanique + UV + fixation Posca
- Séchage 24h entre couches
- Recouvre tout : peinture, Posca, signature, logo

### Palette couleurs par modèle (validée mockups)

| Pédale | Couleur fond | Format Hammond |
|---|---|---|
| NOISE BOX | Orange #FF4500 | 1590BB |
| BAZZ FUSS | Violet #6B2D8C | 1590B |
| ECHO TRIBE (PT2399 T1) | Vert acide #A8FF00 | 1590B |
| GRAIN MILL (Daisy Granulaire) | Magenta #FF00AA | 1590DD |
| SAUVIGNAC MULTIEFFET (#45) | Noir mat + accents fluo (à valider) | 1590DD ou custom |

### Workflow de production — 5 étapes

1. **Préparation** : perçage trous fonctionnels (knobs, jacks, footswitch, LED, USB), masquage des trous, ponçage léger 240
2. **Bombe** : pulvérisation couleur saturée flat sur boîtier complet, séchage 24h
3. **Posca** : tracé contours + composants + hachures + highlights blancs, séchage 4h
4. **Vernis** : 2-3 couches fines acrylique satin, séchage 24h entre couches
5. **Montage final** : démasquage, montage composants, câblage, soudures, test, sticker mentions techniques sous le boîtier, signature manuscrite, pieds antidérapants

**Temps estimé par pédale** (finition seule, hors électronique) : ~3h actif / ~4 jours calendaires (avec séchages). En parallélisant des lots de 5-10 pédales, **8-12 pédales / mois** réalistement.

### Signature systémique Sauvignac

Chaque pédale porte deux marques constantes :
- **Carré orange #FF4500** (clin d'œil au logo Sauvignac, présent en fond ou en accent)
- **Numéro d'édition manuscrit** sous le boîtier (ex : "Édition n°007")

Numérotation à arbitrer ultérieurement : globale toutes pédales (Noise Box N°001 → Bazz Fuss N°002 → ...) ou par modèle. Recommandation provisoire : **globale** au moins pour la phase 1 (10-30 premières pédales), pour donner un sens "collection" à la roadmap.

### Différenciation finition PërKompanion vs pédales Sauvignac

| | PërKompanion | Pédales Sauvignac |
|---|---|---|
| Support principal | Plate acrylique noir mat + module peuplier 5mm | Boîtier Hammond peint à la bombe |
| Finition | Huile Rubio Monocoat (à confirmer) | Bombe + Posca + vernis acrylique satin |
| Identité visuelle | Cyber Brutalist sans bleu (interface logicielle) | Cartoon Custom Paint (objet physique) |
| Logique | Instrument de scène, toucher chaleureux, gravures naturelles | Objet compact au sol, lisibilité immédiate, geste main |

Les deux univers visuels sont **distincts mais cohérents** sous l'identité Sauvignac : le carré orange #FF4500 du logo se retrouve à la fois en accent dans l'UI Cyber Brutalist et en signature systémique sur les pédales Cartoon.

---

<a id="pedales"></a>
## Pédales d'effets — Décisions par projet

---

<a id="pedales-ep1"></a>
### Épisode 1 — Test physique du langage Cartoon Custom Paint

**Statut** : à tourner — c'est le **premier épisode YouTube de la chaîne Sauvignac DIY**.

**Pourquoi cet épisode en premier** : avant de lancer la production série de la première pédale (Noise Box), il faut **valider physiquement le geste main** sur un boîtier de récup. C'est rapide à filmer, très visuel (timelapse de la bombe + Posca), et ça pose immédiatement l'identité visuelle Sauvignac à la caméra. Bon candidat pour roder le workflow vidéo avant des épisodes plus complexes (électronique).

**Arc narratif pressenti** :
1. Introduction du projet Sauvignac DIY et de sa philosophie ("on part de zéro, on construit")
2. Présentation des références visuelles (Cherry Lighter, custom paint cartoon, flyers free party)
3. Démonstration de la technique sur un boîtier de récup
4. Auto-évaluation honnête à la caméra : « ça correspond au mockup ? je suis content ou pas ? »
5. Ouverture sur le prochain épisode : la première vraie pédale, la **Noise Box**

#### §1. Boîtier de test

**Décision** : utiliser un **boîtier Hammond 1590B premier prix** (~10 €) ou un vieux boîtier alu de récup. Pas de composants montés — c'est un test purement esthétique.

#### §2. Matériel à acheter

| Matériel | Quantité | Usage |
|---|---|---|
| Bombe acrylique flat couleur (orange #FF4500 par exemple) | 1 | Base couleur |
| Posca PC-7M noir | 1 | Contours et hachures épais |
| Posca PC-3M noir | 1 | Détails fins |
| Posca PC-1MR noir | 1 | Très fins (signature, numéros) |
| Vernis acrylique satin en aérosol (Montana, MTN94, Liquitex) | 1 | Finition |
| Papier de verre 240 et 400 | 1 de chaque | Préparation |

Budget total estimé : **~50 €**.

#### §3. Critères de validation à la caméra

À l'issue du test physique, auto-évaluation honnête :
- Le rendu correspond-il aux mockups validés ?
- Le geste Posca est-il agréable à exécuter ? (production série de 30+ pédales prévue, donc plaisir nécessaire)
- Le rendu paraît-il « professionnel artisanal » ou « DIY débutant » ?

#### §4. Plan B si le test échoue

Alternative pré-validée : **sous-traiter l'impression UV directe sur Hammond** chez un fournisseur (Tayda, Sycamore, fabricant français), avec un fichier vectoriel reproduisant le langage Sauvignac. Le rendu sera moins "vivant" (pas de geste main) mais cohérent et reproductible.

---

<a id="a-faire"></a>
## À faire prochainement

### Court terme — pédales
- [ ] Acheter le matériel test (~50 €)
- [ ] Récupérer un boîtier de test (1590B premier prix ou récup)
- [ ] Tournage de l'**Épisode 1 — Test physique du Cartoon Custom Paint**
- [ ] Décision sur la première vraie pédale à produire (Noise Box CD40106 / projet #1 par défaut)
- [ ] Définir BOM complète première pédale (Noise Box T1)
- [ ] PCB custom KiCad ou perfboard pour la première pédale ?

### Moyen terme — pédales
- [ ] Production série Noise Box → Bazz Fuss → Echo Tribe (épisodes 2-3-4)
- [ ] Test pochoir laser pour le logo Sauvignac (cohérence inter-pédales sur le brand mark)
- [ ] Arbitrage de la numérotation des éditions (globale ou par modèle)

### Long terme — autres familles DIY
- [ ] Définir si Eurorack se rajoute à la roadmap (les projets #4, #5, #6, #9, #10, #28-#30 sont compatibles)
- [ ] Définir support build pour les modules (panneau alu standard 3U vs custom)

### Long terme — chaîne YouTube
- [ ] Tournage du Trailer Sauvignac DIY (similaire au Trailer PërKompanion)
- [ ] Coordination des deux playlists (DIY et PërKompanion)

---

*Sauvignac — On part de zéro. On construit.*
