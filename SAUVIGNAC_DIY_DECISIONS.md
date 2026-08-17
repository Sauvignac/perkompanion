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
- [Convention layout pédales — décision méta-gamme](#convention-layout)
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

<a id="convention-layout"></a>
## Convention layout pédales — décision méta-gamme

**Date** : 2026-04-28
**Statut** : ✅ Validée pour toute la roadmap pédales (formats 1590B / 1590BB / 1590DD).

**Décision** : adoption d'une **convention de layout physique commune à toutes les pédales Sauvignac**, formalisée dans le document `pedales_panel_layout.md` à la racine du repo.

### TL;DR

Six conventions invariantes posées au démarrage du projet #1 Mazout, avant que la moindre pédale soit construite, pour garantir la cohérence de gamme physique sur les 45 projets de la roadmap :

1. **Connecteurs externes** sur la tranche arrière longue uniquement (jamais sur les flancs courts ni sur le top)
2. **Pas de footswitch true bypass** — bypass continu via potar DRY/WET
3. **Fixation PCB par standoffs nylon snap-in M3** + vis acier inox brut tête fraisée Allen affleurées
4. **Disposition PCB intérieur** centrée, collée au flanc avant, avec convention "composants > 8mm couchés"
5. **Format PCB EPLZON noir mat breadboard-style**, soudure traversante systématique
6. **Identité visuelle** Cartoon Custom Paint extérieur + PCB noir mat intérieur + bend points laiton optionnels selon pertinence projet

### Pivot stratégique

Cette convention est l'aboutissement d'un arbitrage ergonomique-design réalisé en début de phase 1, avant le premier build :

| Question explorée | Issue |
|---|---|
| Jacks top-mounted (au-dessus des knobs) | ❌ Câbles obstruent le tweak en jam, bruit mécanique sur prise, OLED masqués |
| Jacks tranche arrière courte (flancs latéraux) | ❌ Empêche pédales jointives sur table |
| **Jacks tranche arrière longue** | ✅ **Convention validée** — top 100% libre, pédales jointives possibles, câbles vers fond de table |
| Footswitch 3PDT classique | ❌ Inadapté usage tabletop debout, demande de baisser pour bypass |
| **Potar DRY/WET continu** | ✅ **Convention validée** — main reste sur le top, mix continu plus expressif |
| Perçage couvercle pour fixation PCB | ✅ Validé après débat collage VHB vs perçage — perçage gagne en démontabilité |
| Têtes vis dépassantes | ❌ Pédale instable sur table, raye les surfaces |
| **Vis tête fraisée + perçage chanfreiné** | ✅ **Convention validée** — affleurement, esthétique pro, raccord visuel inox/alu |
| Perfboard simple face vert FR-4 | ❌ Capacité routage limitée |
| Stripboard pistes longues | 🟡 Bon pour publication tutoriel, mais limité pour build perso |
| Tayda double face A-1194 | 🟡 Pas de plated through-holes, esthétique correcte mais pas signature |
| **EPLZON noir mat breadboard-style** | ✅ **Convention validée** — coordonnées sérigraphiées, breadboard 1:1, cohérence interne |

### Rationale

- **Cohérence de gamme** : 45 pédales prévues sur 5+ ans → poser les conventions une fois évite des ajustements rétroactifs cauchemardesques
- **Ergonomie tabletop assumée** : positionne explicitement la gamme Sauvignac comme **instruments de jam debout devant le setup**, pas pédales pedalboard guitariste — décision identitaire forte
- **Précédent industriel reconnu** : Strymon, Eventide, Chase Bliss tabletop adoptent les mêmes conventions (jacks arrière + dry/wet sans footswitch + têtes fraisées affleurées)
- **Faisabilité économique** : chaque convention est réalisable au prix unitaire d'une pédale T1 (~30-50€) sans investissement spécial
- **Reproductibilité par viewers** : EPLZON breadboard-style avec coordonnées sérigraphiées = layout publiable trivialement, viewers reproduisent à l'identique

### Document de référence

L'intégralité des conventions, avec spécifications dimensionnelles, justifications et procédures, est documentée dans :

> **`pedales_panel_layout.md`** — Convention layout pédales Sauvignac

Ce document fait autorité pour toutes les décisions de conception physique sur les pédales de la roadmap. Il est référencé depuis chaque journal de décision projet (Mazout, Bazz Fuss, etc.) plutôt que dupliqué.

### Outillage commun à acquérir

Achat unique amorti sur toute la roadmap :

- Foret HSS métal Ø3.5mm (perçage couvercle)
- Foret à fraiser 90° HSS (chanfrein vis fraisées)
- Pointeau automatique
- Gabarit de perçage Sauvignac (à créer en MDF FabLab ou carton épais — 4 trous M3 standardisés sur 1590BB)

### Conséquences pour le projet #1 Mazout

Mazout est le **premier projet d'application** de cette convention. Toutes ses décisions individuelles (positions des knobs, jacks, toggle, bend points, layout PCB EPLZON, fixation à 2 standoffs centrés) découlent directement des 6 conventions ci-dessus.

L'épisode 1 Sauvignac DIY (test physique du Cartoon Custom Paint sur boîtier vide) reste la première étape de mise en pratique. L'épisode 2 (premier build complet Mazout) sera la **première démonstration publique** de la convention en action.

### Évolution — ajout des Conventions 7 et 8 (28 avril 2026, même session)

Au cours de la même session de travail sur les conventions de gamme, deux décisions méta-gamme additionnelles ont été figées :

**Convention 7 — Modules d'infrastructure et grille chromatique**

La gamme Sauvignac est désormais structurée en **2 types d'objets** (pédales effets + modules d'infrastructure), avec **4 catégories chromatiques** distinctes lisibles au coup d'œil :

| Catégorie | Boîtier | Posca |
|---|---|---|
| Pédales effets | 1590BB paysage couleur saturée flat (par projet) | Noir + accent blanc |
| Modules audio (Routing) | 1590B portrait noir mat | Orange + blanc |
| Modules contrôle (Modulation, Conversion, CV/MIDI) | 1590B portrait violet pastel/lavande | Noir + orange |
| Modules alim (PSU, plomberie) | 1590B portrait alu brut | Noir + orange |

L'**orange Sauvignac #FF4500** reste la couleur signature unique présente sur tous les modules infra en label Posca.

Les modules d'infrastructure adoptent le **format 1590B portrait** (60×112mm) pour glisser entre les pédales 1590BB sans bouffer de largeur de table, tout en respectant l'alignement de la tranche arrière (Convention 1).

Cette grille pose un système chromatique fonctionnel : **l'utilisateur lit la fonction d'un objet à son boîtier** sans avoir besoin de lire les labels.

**Convention 8 — Finition sur boîtier Tayda prépeint**

Découverte du sourcing : Tayda vend des boîtiers **prépeints en poudre époxy industrielle** dans une gamme de ~70 couleurs incluant l'orange Sauvignac (validé visuellement comme matchant #FF4500), un noir mat satisfaisant, du violet pastel, et l'alu brut standard.

Décision : **basculement de la peinture maison à la bombe vers les boîtiers Tayda prépeints** pour toutes les catégories où une couleur Tayda matche, ce qui couvre l'intégralité de la grille chromatique de Convention 7.

**Gain estimé** : ~3-4h de production par pédale (peinture + sous-couche + couches + séchage 24h initial éliminés). À l'échelle de la roadmap 30+ pédales, gain de ~100h de production.

**Workflow de finition Sauvignac sur boîtier Tayda prépeint** documenté dans Convention 8 : dégraissage alcool isopropylique → ponçage léger optionnel → application Posca → cuisson Posca 60-70°C → vernis acrylique satin à base d'eau → cuisson finale optionnelle.

Cette décision **ne modifie pas** la convention 6 (Cartoon Custom Paint) : le langage hand-painted Posca + vernis reste intact, ce qui change c'est uniquement la **base peinte** qui passe de "bombe maison" à "Tayda prépeint". L'effet visuel final reste 100% Sauvignac.

### Test physique combiné

L'épisode 1 YouTube prévu (test physique du Cartoon Custom Paint sur boîtier vide) devient également le **test du workflow Convention 8** : dégraissage + Posca + vernis sur boîtier Tayda prépeint. Si le test valide la durabilité, l'ensemble de la phase 1 Sauvignac peut être commandée en boîtiers prépeints.

### Évolution — ajout de la Convention 9 (29 avril 2026, durant montage Mazout)

**Convention 9 — Connectique alimentation DC**

Décision figée durant le montage du projet #1 Mazout, suite à un échange diagnostique sur le bon choix de jack DC chassis-mount (le Cliff FC68148 initialement en stock s'est révélé être un PCB-mount horizontal, incompatible avec la convention EPLZON + fils volants de Mazout).

**Référence retenue** : **Tayda DC-025M** (~0.24 USD), chassis-mount 5.5/2.1, filetage M8 métal nickelé, écrou hexagonal métal, corps interne typiquement isolé.

**Polarité** : **centre négatif** (tip = GND, sleeve = +9V) — convention Boss historique, standard industrie pédale, compatible avec toutes les alims pedalboard du marché (One Spot, Cioks, Strymon Zuma, etc.).

**Protection inversion polarité** : **diode Schottky 1N5817 en montage shunt** (cathode/+9V, anode/GND), montage Boss historique le plus répandu en DIY pédale.

**Perçage** : trou Ø 8.2 mm dans la tranche arrière, à reporter dans le gabarit de perçage Sauvignac (en plus des Ø 9.5 mm jacks audio et Ø 3.5 mm vis M3 Convention 3).

**Pivot d'arbitrage**

| Question explorée | Issue |
|---|---|
| Cliff FC68148 PCB-mount horizontal (stock existant 20 pièces) | ❌ Incompatible avec convention EPLZON + fils volants. Conservé en stock pour phase KiCad future (T3, multieffet #45). |
| Switchcraft 712A (premier réflexe — standard industrie pédale) | 🟡 Standard reconnu, mais impose une commande séparée Mouser/Reverb qui casse la cohérence one-stop-shop Tayda. ~3 USD pièce. |
| Lumberg NEB/J 21 chez TME | 🟡 Qualité solide, mais second fournisseur pour une seule pièce. |
| **Tayda DC-025M** | ✅ **Convention validée** — panier Tayda unifié, ~0.24 USD pièce, écrou métal nickelé cohérent visuellement avec écrous jacks audio Neutrik. |
| Cliff DC-10S panel mount | 🟡 Bonne option de **backup** en cas de rupture stock DC-025M chez Tayda. |

**Rationale**

Le DC-025M Tayda tient la promesse "standard reproductible par les viewers" *mieux* que le 712A en pratique : toutes les autres pièces Sauvignac (boîtier prépeint Convention 8, knobs, LEDs, footswitches) sont déjà sourcées chez Tayda. Le DC-025M s'inscrit dans un panier Tayda unifié plutôt que d'imposer une commande séparée. Friction zéro pour qui reproduit Mazout depuis une publication YouTube.

Gain secondaire : ~10x moins cher (0.24 vs ~3 USD), soit ~125 USD cumulés sur la roadmap 45 pédales — pas un driver à lui seul, mais cohérent avec la logique de mutualisation Tayda.

**Test multimètre obligatoire** sur la première unité de chaque batch Tayda (référence générique chinoise, construction théoriquement variable d'un sous-traitant à l'autre — vérifier que l'écrou métal n'a aucune continuité avec les cosses de soudure). Si une continuité existe, prévoir rondelle d'isolation nylon ou changer de fournisseur.

Workflow de câblage standard, schéma diode shunt, alternatives écartées et procédure de test multimètre documentés dans **Convention 9 de `pedales_panel_layout.md`**.

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


## Nouvelles entrées datées 2026-04-29 :

Convention naming saga = T1/T2/T3 en com publique (Option A actée, retire la question ouverte)
Règle méta-gamme : projet rythmique = T2 natif par construction (delay, LFO, tremolo, vibrato, modulations, etc.)
Format saga = 3 modules distincts (pas un PCB évolutif)
Standard MIDI in modules Sauvignac = TRS 3.5mm Type A (norme MMA 2018)
Premier T2 natif de la roadmap = Echo Tribe T2 (Phase 1, projet #3)


---

*Sauvignac — On part de zéro. On construit.*
