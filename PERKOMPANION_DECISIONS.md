# PërKompanion — Journal des décisions

> Journal de conception du projet PërKompanion, structuré par épisode YouTube.
> Chaque épisode regroupe les décisions associées à un arc narratif vidéo.
> La numérotation des décisions est locale à chaque épisode.
>
> **Ce document est public.** Notes privées et stratégie chaîne dans des fichiers séparés non commités.

---

## Sommaire

- [Vidéo Trailer — Carte de visite du projet](#video-trailer)
- [Épisode 0 — Vision et fondations](#episode-0)
- [Épisode 1 — Pi 5, le cerveau du PërKompanion](#episode-1)
- [Épisode 2 — Premier signal hardware](#episode-2)
- [À faire prochainement](#a-faire)
- [Documents techniques de référence](#references)

---

<a id="video-trailer"></a>
## Vidéo Trailer — Carte de visite du projet

**Statut** : à tourner.

**Format cible** : 1 à 2 minutes.

**Objectif narratif** : poser un drapeau. Dire **quoi**, sans expliquer le **pourquoi** — l'épisode 0 (manifeste complet) fera le **pourquoi** plus tard.

**Place dans la chaîne** : première position de la playlist YouTube, sert d'ancrage à tous les épisodes ultérieurs.

---

<a id="episode-0"></a>
## Épisode 0 — Vision et fondations

**Statut** : à tourner.

**Objectif narratif** : poser le **pourquoi** du PërKompanion.
- Présentation de Sauvignac (artiste, scène free party 2000-2010, retour après 15 ans)
- Le Perkons HD-01 comme machine fondatrice et ses limites assumées
- Vision PërKompanion : compagnon, pas remplacement — overlay, pas refonte
- Philosophie projet : open source, build-in-public, documentation
- Architecture cible (sans rentrer dans le code) — Pi 5, Teensy, Perkons en cœur de système
- Roadmap par phases — Phase 0 (caractérisation MIDI) déjà faite, Phase 1+ à venir

Décisions à mobiliser dans ce script : §1 à §14 ci-dessous (les Fondations).

---

### Fondations — Décisions de pré-production

Les décisions §1 à §14 ci-dessous ont été prises avant le démarrage du tournage. Elles forment la matière première de l'Épisode 0.

---

### §1. Matériau du module pad

**Décision** : **Contreplaqué peuplier 5mm** pour le module pad surélevé (190 × 115 mm).

**Historique** :
- v0.8 : acrylique violet translucide satiné 3mm
- v0.9 : contreplaqué okoumé mono-essence 3mm
- **v0.9 bis (actuelle)** : contreplaqué peuplier CP 5mm

**Raisons** :
- **Éthique écologique** : essence européenne (France/Italie), pas de bois tropical, pas de déforestation, faible empreinte carbone
- **Soutien filière locale** (scieries européennes)
- **Rigidité suffisante** en 5mm, largement renforcée par le collage des switches (voir §3)
- **Prix raisonnable**
- **Usinage laser CO2** standard
- **Cohérence avec l'identité « artisan noble + éthique locale »** Sauvignac

**Performance vs okoumé** : peuplier ~30% moins rigide à épaisseur équivalente, mais l'épaisseur 5mm (vs 3mm) compense largement, et le collage des switches neutralise la différence en usage réel.

---

### §2. Fixation du module pad à la plate principale

**Décision** : **Entretoises filetées M3 femelle-femelle de 3mm** + vis courtes M3×15-20mm.

**Raisons** :
- Avec entretoise 10mm + module 3mm, le stem du switch MX (10.5mm) ne ressortait pas du module → impossible de clipper le keycap
- Avec entretoise 3mm + module 5mm, le stem dépasse au-dessus du module et le clippage devient possible (voir §4 pour le détail)
- Entretoise filetée F/F : assemblage propre sans vis qui dépasse trop

**Nombre d'entretoises** : 4 aux coins du module (6 si rigidité renforcée souhaitée au centre).

---

### §3. Fixation des switches dans le module — pas de PCB

**Décision** : **Collage des switches par la face arrière du module + câblage en fils soudés directement** sur les broches des switches. **Pas de PCB.**

**Raisons** :
- Module en peuplier 5mm plus épais que l'épaisseur standard plate Cherry MX (1.5mm) → clips switches peuvent ne pas se fermer complètement → collage requis
- Câblage par fils soudés directement sur les broches : choix conscient, cohérent avec l'esprit DIY artisan du projet, et adapté à une matrice de switches non-PCB
- Switches **non démontables** une fois soudés et collés → cohérent avec un instrument live définitif
- **Rigidifie l'ensemble** : les 45 switches collés deviennent un assemblage quasi monolithique avec le module, compensant la flexibilité du peuplier
- **Sécurité** : pas de risque qu'un switch se désengage sous frappe violente en live

**Colles recommandées** :
- Époxy 2 composants (JB Weld, Araldite) — résistance ultime, permanent
- Super-glue cyanoacrylate pour plastique (Loctite 406) — rapide, solide

---

### §4. Calcul dimensionnel du module pad

L'épaisseur de la plate principale n'est pas comptée — on raisonne **en hauteur au-dessus de la plate**.

**Empilage au-dessus de la plate principale** :

| Élément | Hauteur cumulée au-dessus de la plate |
|---|---|
| Bas du module pad (sur entretoise 3mm) | 3 mm |
| Haut du module pad (épaisseur 5mm) | **8 mm de matière au-dessus de la plate** |

**Switch et keycap** :

Le switch MX est **enfilé dans le module pad**. Le keycap au repos dépasse de **15.6 mm au-dessus du dessus du module pad**.

**Hauteur totale du keycap au repos au-dessus de la plate principale** :

```
8 mm (matière au-dessus de la plate) + 15.6 mm (keycap au-dessus du module) = 23.6 mm
```

Cohérent avec les standards MPC/Push (15-25 mm).

---

### §5. Finition du module peuplier

**Décision en cours** : finition à valider lors du test physique.

**Options en lice** :
- **Huile dure (Rubio Monocoat, Osmo Polyx-Oil)** — préférence actuelle : aspect mat/satin chaud, artisanal, facile à appliquer, réparable
- Laque brillante (vernis polyuréthane acrylique) : look piano laqué premium, sensible aux empreintes
- Teinture foncée + huile satin : look "ébène de luthier"

---

### §6. Rétroéclairage des voix virtuelles V5-V8

**Décision** : **Jauge volume horizontale** au bas des cartes V5-V8 dans le Home Dashboard UI.

**Raison** : les voix physiques V1-V4 sont contrôlées par les knobs hardware du Perkons (visuel assuré par le Perkons lui-même). Les voix virtuelles V5-V8 sont générées par le Teensy, donc sans retour visuel hardware → jauge UI nécessaire.

**Implémentation** :
- Barre horizontale 8px de haut, full width de la card
- Gradient vert → jaune → orange → rouge selon niveau
- Tick vertical 2px à la position 0 dB (référence de gain staging)
- Pas de label "VOL" ni de valeur dB (épuré)

---

### §7. Identité graphique : Logo Sauvignac

**Décision** : **Logo « S » fracturé pixelisé** avec carré orange (#FF3D00) en accent.

**Direction créative** : « Destructuré × Pixelisé × Élégant » — noblesse typographique altérée par interventions brutales.

**Caractéristiques** :
- S serif classique de base
- 5 coupures horizontales qui déplacent des sections du S (glitch contrôlé)
- Petit carré orange en haut à droite (accent palette)
- Look « noblesse fracturée » cohérent avec positionnement Sauvignac

**Variantes à produire** :
- [ ] Mono noir pur (sans carré) pour gravure laser
- [ ] Mono blanc sur noir (inversion)
- [ ] Favicon 16×16 et 32×32
- [ ] Version SVG vectorisée

---

### §8. Branding et identité de marque

**Décision** : **Sauvignac** = label/atelier parent. **PërKompanion** = premier produit.

**Raisons** :
- Permet d'autres produits futurs sous l'ombrelle Sauvignac (kits samples, outils software, futurs instruments, pédales d'effets)
- Cohérent avec stratégie long terme
- Crédite l'auteur Sauvignac sans enfermer le produit

**README** :
- Mention « by Sauvignac, amateur musicien passionné par la techno et les synthétiseurs hardware depuis plus de vingt ans »
- Transparence sur utilisation de Claude (Anthropic) comme « design partner » pour architecture, firmware, documentation

---

### §9. Budget projet — prévisionnel

**Note** : ces chiffres sont **prévisionnels**. Ils seront mis à jour avec les tarifs réels au fur et à mesure des achats. L'objectif est aussi d'**optimiser** ces coûts à mesure que le projet avance.

| Poste | Estimation prévisionnelle | Statut |
|---|---|---|
| Teensy 4.1 Fully Loaded 32MB PSRAM (US, frais inclus) | ~135-155 € | À commander, délai 3-4 semaines |
| Raspberry Pi 5 + écran Elecrow 7" + accessoires | ~200 € | ✅ Reçu |
| Composants (OLEDs, encodeurs, keycaps, switches, LEDs, modules audio) | ~447 € | À commander |
| Composants spécialisés (ICs qualité, jacks Neutrik, passifs premium) | ~200 € | À commander |
| Découpe laser (plates acrylique + module peuplier) | ~100-110 € | À commander |
| **Total prévisionnel projet v0.9 bis** | **~1100-1180 €** | |

À jour : 2026-04 (sera révisé).

---

### §10. Infrastructure Git

**Décision** : consolidation de tous les projets sur le compte GitHub **Sauvignac**.

**Repos** :
- `Sauvignac/perkompanion` (public, licence MIT, identité artistique impérative)

**Workflow local** :
- Dossier de travail synchronisé via Nextcloud
- Éditeur : Notepad++ (migration VSCode + Claude Code envisagée plus tard)

---

### §11. Encodeurs — KY-040 pour proto, Bourns PEC11R pour panel final

**Décision** : **KY-040 pour le prototypage uniquement. Bourns PEC11R (ou Alps EC11) pour le panel définitif.**

**Limitations KY-040** :
- Rebonds importants → faux pulses en polling → debouncing logiciel obligatoire
- Résolution 20 détentes/tour (correct mais pas optimal)
- Durabilité ~15 000 cycles → insuffisant pour usage live intensif sur 64 encodeurs

**Bourns PEC11R / Alps EC11** :
- Même empreinte mécanique, même shaft 6mm, même trou panel Ø7mm → **swap transparent**
- 30 détentes/tour, meilleur anti-rebond mécanique, rated 30 000+ cycles
- Surcoût acceptable pour un instrument live

**Impact firmware** :

**Logique firmware identique.** Les deux encodeurs utilisent la quadrature standard (signaux A et B). Le firmware lit les transitions et compte les pas de la même manière dans les deux cas.

**Seul ajustement** : la **constante de pas par tour** en variable de config (20 pour KY-040, 24 ou 30 pour PEC11R) — pas une refonte. Le **debouncing logiciel** reste implémenté quel que soit l'encodeur (bonne pratique, plus critique sur KY-040 mais utile sur PEC11R aussi pour absorber l'usure).

**Impact panel_design.md** : aucun — trou Ø7mm identique pour les deux.

---

### §12. Modules audio breakout — politique V1, roadmap V3+

**Décision** : architecture **breakout modules** validée pour V1. Migration **PCB custom** envisagée en V3+.

**Politique de remplacement V1** :
- **PCM1808 breakout (×2)** — ADC entrées Perkons : remplaçable à l'unité (headers 2.54mm). ⚠️ Vérifier configuration pins (FMT, XSMT) lors d'un remplacement
- **PCM5102A breakout (×6)** — DAC sorties : remplaçable à l'unité. Même précaution
- **TPA6120 breakout MCU-612** — ampli casque : remplaçable librement, module standard
- **TCA9548A (×2)** — multiplexeur I²C : non activement utilisé en V1, réserve pour extensions

**Roadmap upgrade PCB custom V3+** :

Cible : **PCM3168A** (ADC + DAC multi-canal intégré, qualité pro).

Conditions de déclenchement :
- Besoin réel démontré en jam (capture simultanée master + 4 voix individuelles)
- Maturité KiCad acquise (6-12 mois de pratique Sauvignac DIY)

Ce qui change : PCB custom KiCad remplace les perfboards + breakouts. JLCPCB PCBA pour le PCM3168A (HTQFP-64, CMS).

Ce qui ne change pas : firmware Teensy, protocole, param_id, panel, jacks, interface utilisateur.

**Conclusion** : l'architecture V1 est un premier palier solide, pas un prototype à jeter.

---

### §13. Crédibilité du projet PërKompanion

L'architecture breakout modules n'est pas un compromis — c'est une décision d'ingénierie mature et documentée. Elle est cohérente avec :
- La PJRC Audio Library qui documente les PCM5102A breakouts comme référence pour projets Teensy audio
- Des projets open source hardware reconnus (Music Thing Modular, Mutable Instruments early designs)
- La philosophie build-in-public : chaque palier est fonctionnel, documenté, partageable

Ce qui fait la crédibilité du PërKompanion :
- Architecture documentée au niveau protocole (param_id, moteur de modulation, courbes)
- Décisions justifiées et tracées
- Abstraction bien conçue : le firmware ne connaît pas le hardware audio, seulement les param_id
- Anticipation V3+ intégrée dès la conception V1

La migration V1 → V3+ devient elle-même un contenu YouTube : « On migre le PërKompanion vers un PCB custom » — KiCad, JLCPCB PCBA, assemblage SMT.

---

### §14. UI Home Dashboard — version pré-production

**Décision** : **Home Dashboard validé après 6 itérations Claude Design**. Détails du processus de prototypage UX et de la charte graphique : voir [Épisode 1 §1](#ep1-claude-design).

**Caractéristiques finales (résumé)** :
- Résolution cible : 1024 × 600 landscape (écran Elecrow 7")
- Charte graphique : **Cyber Brutalist sans bleu**
- Distinction physique/virtuelle : rouge V1-V4, fuchsia/magenta V5-V8
- Spectres de fréquences par voix (vs waveforms)
- Bloc « Activité en direct » avec mini grille 4×2

---

<a id="episode-1"></a>
## Épisode 1 — Pi 5, le cerveau du PërKompanion

**Statut** : décisions techniques validées et en production. À filmer.

**Arc narratif** : du dessin UX au cerveau qui parle. Le spectateur voit naître l'interface dans Claude Design (phase design), puis assiste à sa matérialisation sur l'écran Elecrow 7" connecté au Pi 5 (phase hardware). Le cerveau s'anime ensuite : on installe le serveur, on établit le canal de communication temps réel, et on observe le système vivant des deux côtés. L'épisode boucle visuellement sur l'interface qui s'allume sur l'écran physique.

**Intro courte récurrente (30-60 sec)** à placer en ouverture :

> *« Mon Perkons HD-01 est génial, mais il y a des choses qu'il ne fait pas — modulations complexes, mémoire de moments, voix synthétiques additionnelles. Le PërKompanion, c'est l'ordinateur dédié que je construis pour ajouter ces capacités sans toucher au Perkons. Aujourd'hui, on commence par le cerveau. »*

(Cette intro pourra être remplacée par une référence à l'Épisode 0 quand celui-ci sera publié.)

**Pour la reproduction step-by-step** : voir [`PERKOMPANION_EPISODE1_GUIDE.md`](./PERKOMPANION_EPISODE1_GUIDE.md) qui rassemble toutes les commandes, fichiers de configuration, et codes utilisés.

---

<a id="ep1-claude-design"></a>
### §1. Prototypage UX avec Claude Design

**Date** : 2026-04-25
**Statut** : ✅ Validé après 6 itérations.

Avant tout setup hardware ou code, l'interface du PërKompanion a été prototypée visuellement avec **Claude Design** (Anthropic), l'outil de design génératif intégré à Claude.ai.

**Démarche stratégique** :
- **Approche modulaire** : valider d'abord le Home Dashboard, puis dériver les autres écrans (Settings, Voice Edit, Libraries) à partir de la charte établie
- **Contraintes architecturales injectées dès le brief initial** :
  - Système **i18n** : strings textuelles considérées comme remplaçables, aucune largeur fixe basée sur le texte (utilisation de `min-width`), boutons et labels adaptables à des chaînes 20-40% plus longues. Langues v1 : FR + EN. Langues v2+ : DE, JA, ES.
  - Système de **thèmes (skins)** : variables CSS sémantiques (`--bg-primary`, `--accent-primary`, etc.), aucune couleur hardcodée. Tous les composants theme-agnostic dans leur structure.
  - Thèmes prévus à terme : Cyber Brutalist (défaut), Warm Artisan, Monochrome, Hardcore, Liquid, Matrix.

**Itérations** : 6 versions successives ont permis de converger vers une charte finale validée.

**Charte graphique finale — Cyber Brutalist sans bleu** :

| Élément | Spécification |
|---|---|
| Résolution cible | 1024 × 600 landscape (écran Elecrow 7" tactile) |
| Palette principale | warm-black + orange incandescent + magenta + vert acide + violet profond |
| Typographies | Archivo Black (display) + Inter (UI) + JetBrains Mono (data) |
| Layout | Top menu bar B1-B6 + voice grid central + encoder pastilles latérales E1-E8 |
| Distinction physique/virtuelle | Rouge pour les voix Perkons V1-V4, fuchsia/magenta pour les voix Teensy V5-V8 |
| Visualisation voix | Spectres de fréquences (et non waveforms) |
| Bloc « Activité en direct » | Mini grille 4×2 récapitulant les modulations actives par voix |

**Limite identifiée** : l'artifact bundlé exporté de Claude Design contient sa propre logique d'auto-fit interne (transform scale dans un `<div id="stage">`), inadaptée au déploiement hardware kiosk. Voir [§6 Fix temporaire](#ep1-stage-fit), et la décision de réimplémentation native via Claude Code à terme.

---

### §2. Configuration Raspberry Pi 5 — kiosk Chromium

**Date** : 2026-04-26
**Statut** : ✅ Validé et fonctionnel.

**Matériel** :
- Raspberry Pi 5
- Écran Elecrow 7" tactile (HDMI micro + USB touch)
- Ventilateur Synclum (connecteur JST 1.25mm 4 broches natif Pi 5)
- MicroSD SanDisk 64GB flashée Raspberry Pi OS 64-bit Bookworm

**Configuration via Raspberry Pi Imager** :
- OS : Raspberry Pi OS 64-bit (Bookworm)
- Hostname : `perkompanion`
- Utilisateur : `sauvignac`
- WiFi configuré au flash (SSID + mot de passe + pays FR)
- SSH activé (authentification par mot de passe)
- **Locale : `en_GB.UTF-8`** ← important, voir note ci-dessous

**⚠️ Erreur initiale à éviter — locale FR** :

La première tentative a configuré le Pi en français (locale FR, clavier FR). Cela génère des **conflits d'encodage UTF-8 entre la session SSH côté Pi et le terminal côté PC** (notamment Windows PowerShell, qui démarre en cp1252 par défaut). Les caractères accentués s'affichent en mojibake, certains messages d'erreur deviennent illisibles.

Tentatives qui n'ont pas suffi :
- `chcp 65001` côté PowerShell (passe en UTF-8) — ne corrige pas tout
- Réglages locale post-flash sur le Pi — laisse des résidus

**Solution adoptée** : reflasher le Pi en `en_GB.UTF-8` dès le début. Tout fonctionne. Le Pi reste un instrument technique en arrière-plan, sa locale n'a pas à être en français — le français reste dans l'**interface PërKompanion** elle-même (i18n), pas dans le système d'exploitation.

**Recommandation au public** : ne pas chercher à mettre le Pi en FR au flash initial.

**Mode de boot — Console Autologin** :

```
sudo raspi-config
→ 1 System Options → S5 Boot → B4 Console Autologin
```

Raison : les sessions graphiques Wayland (labwc) et X11 (LXDE-pi-x) du Pi OS Bookworm posent des problèmes de compatibilité avec lightdm en configuration kiosk. Le mode console + startx manuel est plus stable, plus léger, plus fiable pour un instrument dédié.

**Comportement au boot** :

```
Pi 5 s'allume
→ Boot Bookworm (console, ~15-20s)
→ Login automatique sauvignac
→ ~/.bash_profile exécuté → startx
→ ~/.xinitrc exécuté
→ Chromium en kiosk sur localhost:3000
→ Interface PërKompanion plein écran
```

**Délai écran blanc normal** : 10-15 secondes entre boot et affichage de l'interface — temps de démarrage X11 + Chromium. Acceptable pour un instrument qu'on allume au début d'une session.

**Décisions UX issues de cette session** :

- **Aucun champ de saisie texte** dans l'interface PërKompanion — le clavier virtuel (squeekboard) sur écran tactile 7" est inutilisable. Sur un instrument live, aucune saisie texte n'est acceptable
- **Touch simple uniquement** — tap, pas de scroll, pas de pinch, pas de geste complexe
- L'utilisateur ne voit **jamais** le bureau Raspberry Pi OS. Le système est invisible, l'interface PërKompanion est tout ce qui existe visuellement

(Les détails de configuration — `~/.xinitrc`, `~/.bash_profile`, `/boot/firmware/config.txt`, paquets installés — sont dans le guide de reproduction.)

---

### §3. Architecture technique Pi 5 / Node.js / WebSocket / Teensy

**Date** : 2026-04-26
**Statut** : ✅ Décision validée.

**Stack choisi** :

```
┌─────────────────────────────────────────┐
│  Chromium (kiosk plein écran)           │
│  HTML / CSS / JavaScript                │
│  Interface PërKompanion                 │
└──────────────┬──────────────────────────┘
               │ HTTP (chargement interface)
               │ WebSocket (temps réel bidirectionnel)
               │ localhost:3000
┌──────────────▼──────────────────────────┐
│  Node.js — serveur local                │
│  - Sert les fichiers HTML/CSS/JS        │
│  - Gère les WebSockets                  │
│  - Traduit événements UI → commandes    │
└──────────────┬──────────────────────────┘
               │ Serial USB / SPI / I2C
┌──────────────▼──────────────────────────┐
│  Teensy 4.1                             │
│  - Firmware C++                         │
│  - Moteur de modulation                 │
│  - Protocole param_id                   │
└──────────────┬──────────────────────────┘
               │ MIDI / SPI / I2C
┌──────────────▼──────────────────────────┐
│  Hardware                               │
│  - Perkons HD-01                        │
│  - Encodeurs / LEDs / OLEDs             │
│  - MCP23S17 (expanders I/O)             │
│  - PCM1808 / PCM5102A (audio)           │
└─────────────────────────────────────────┘
```

**Pourquoi HTTP + WebSocket** :

- **HTTP** sert l'interface HTML/CSS/JS depuis localhost. Simple, natif, aucune dépendance.
- **WebSocket** assure la communication temps réel bidirectionnelle :
  - Latence <1ms sur localhost — acceptable pour un instrument live
  - Bidirectionnel — le Pi envoie commandes au Teensy ET reçoit données en retour
  - Natif dans Chromium — pas de librairie JS externe nécessaire
  - Géré nativement par Node.js via `ws`

**Pourquoi pas Electron ou une app native** :

- Chromium kiosk + Node.js = même résultat sans la complexité d'Electron
- Séparation claire UI (Chromium) / logique (Node.js) / hardware (Teensy)
- Débogable depuis n'importe quel navigateur sur le réseau local
- Open source, standard web, pas de dépendance propriétaire

---

### §4. C'est quoi Node.js, Express, PM2 et WebSocket ?

Section pédago pour les viewers qui découvrent ces termes.

**Node.js** — un environnement qui permet d'exécuter du **JavaScript côté serveur** (en dehors du navigateur). Habituellement, JavaScript tourne dans un navigateur web. Node.js permet à JavaScript de tourner directement sur le système (Pi, PC, serveur), pour faire des choses comme servir des pages web, gérer des fichiers, communiquer avec des appareils. C'est le moteur qui anime le **cerveau** du PërKompanion.

**Express** — un framework Node.js minimaliste pour **servir du HTTP**. Il transforme Node.js en serveur web léger : il écoute les requêtes des navigateurs, sert des fichiers HTML/CSS/JS, et renvoie des réponses. C'est lui qui livre l'interface PërKompanion au navigateur Chromium.

**WebSocket** (`ws` est la librairie utilisée) — un protocole de communication **temps réel bidirectionnel**. Contrairement à HTTP qui fait « le navigateur demande, le serveur répond, fin de la conversation », WebSocket maintient un canal **ouvert en permanence** entre le navigateur et le serveur. Les deux peuvent s'envoyer des messages à tout moment, avec une latence inférieure à la milliseconde sur localhost. C'est le **système nerveux** du PërKompanion : tout ce qui bouge à l'écran ou tout ce qui se passe côté hardware passe par ce canal.

**PM2** — un gestionnaire de processus pour Node.js. Sans PM2, si le serveur Node plante ou si le Pi reboote, il faut le relancer à la main. **PM2 garantit** :
- **Démarrage automatique** du serveur au boot du Pi
- **Redémarrage automatique** si le serveur plante
- **Exécution en arrière-plan** détachée de la session SSH
- **Gestion des logs** avec rotation et rétention (pour ne pas saturer la SD card)
- **Monitoring** : `pm2 status` montre l'état en temps réel

C'est la **garantie de fiabilité** : on allume le Pi, le PërKompanion est là.

(Les commandes d'installation et de configuration sont dans le guide de reproduction.)

---

### §5. Serveur HTTP Node.js + WebSocket — implémentation

**Date** : 2026-04-26 (HTTP) + 2026-04-27 (WebSocket)
**Statut** : ✅ Fonctionnel — PM2 autostart validé.

**Stack final** :
- **Node.js v24** (installé via NVM — Node Version Manager, qui permet d'avoir plusieurs versions et de basculer facilement)
- **Express** pour servir les fichiers statiques de l'interface
- **`ws`** pour la communication WebSocket bidirectionnelle
- **PM2** pour la persistance et la gestion du processus

**Structure du projet** :

```
/home/sauvignac/perkompanion/
├── server.js          ← serveur Express + WebSocket
├── package.json       ← config npm
├── node_modules/      ← dépendances (installées par npm)
└── public/
    └── index.html     ← interface PërKompanion (Home Dashboard)
```

**Choix d'implémentation WebSocket** :

| Choix | Raison |
|---|---|
| Path `/ws` | Sépare proprement HTTP statique et WebSocket, évite toute ambiguïté de routing |
| Listen sur `0.0.0.0` | Permet le debug WebSocket depuis le laptop sur le réseau local |
| Heartbeat 500ms | Même cadence que le protocole Pi↔Teensy défini dans PHASE0 — simplifiera la logique « WS connecté ET Teensy connecté ? » plus tard |
| Helper `broadcast()` exposé | Volontairement isolé pour le futur bridge Serial Teensy |

**Wrapper de logs avec timestamps + niveaux** :

```javascript
const log = (level, msg) => {
  const ts = new Date().toISOString().slice(11, 23); // HH:MM:SS.mmm
  console.log(`[${ts}] [${level}] ${msg}`);
};
```

Niveaux utilisés : `BOOT`, `INFO`, `RECV`, `ERROR`.

Rendu typique :

```
[14:42:07.412] [BOOT] PërKompanion HTTP+WS running on :3000
[14:42:31.117] [INFO] WS connect (1 clients) from ::ffff:192.168.1.42
[14:42:31.122] [RECV] hello from laptop
[14:43:18.890] [INFO] WS disconnect (0 clients)
```

**Décision importante** : **les heartbeats ne sont pas loggés**. À 500ms par broadcast, ça ferait 2 lignes/seconde de spam qui pousserait tout le reste hors de l'écran. Le heartbeat est du **trafic de fond** par design — on logge les événements, pas le pouls. Les heartbeats restent visibles dans la console DevTools du navigateur via `ws.onmessage`.

**PM2 — autostart + logrotate** :

PM2 est configuré pour démarrer automatiquement au boot du Pi (`pm2 startup` + `pm2 save`). Le module `pm2-logrotate` est installé avec rotation à 10 Mo, rétention 7 fichiers, compression gzip, rotation forcée quotidienne. La SD card ne saturera pas même en jam de plusieurs heures par jour pendant des mois.

**Validation** : test depuis le navigateur du laptop sur `http://perkompanion.local:3000`, console DevTools — `WS open`, message `hello`, `echo` après ping, `heartbeat` toutes les 500ms reçus correctement.

(Code complet du `server.js` et toutes les commandes d'installation : dans le guide de reproduction.)

---

<a id="ep1-stage-fit"></a>
### §6. Fix temporaire stage_fit — MutationObserver

**Date** : 2026-04-26
**Statut** : ✅ Fonctionnel — **fix temporaire**, à supprimer lors du passage à Claude Code.

**Problème** : l'artifact bundlé exporté de Claude Design (`index.html` servi par Express sur `localhost:3000`) s'affichait avec **des marges noires** autour du contenu sur l'écran Elecrow 7" (1024×600), malgré une résolution écran correctement détectée et Chromium en mode kiosk plein écran.

**Diagnostic** :

Test 1 — Page test minimale (rouge avec carrés verts dans les coins) → affichage plein écran parfait. Conclusion : Pi 5 + Chromium kiosk OK, le problème vient du bundle React lui-même.

Test 2 — Inspection du DOM bundlé : le bundle injecte un container `<div class="stage" id="stage">` avec un transform inline calculé en JavaScript. Ce bundle a sa propre logique d'auto-fit interne qui mesure son container parent, calcule un scale, et **réécrit le style inline du `#stage` à chaque tentative d'override externe**.

**Fix appliqué** : un `MutationObserver` attaché au `#stage` détecte les modifications de l'attribut `style` et force un scale calculé pour remplir 1024×600.

**Pourquoi c'est un sparadrap** :
1. L'artifact bundlé Claude Design est un proto visuel, pas un livrable production. Conçu pour s'afficher dans le panneau preview de Claude (iframe à dimensions fixes), pas pour s'adapter à du hardware cible
2. Le `MutationObserver` se bat contre la logique interne du bundle. Fragile, dépendant de la structure interne actuelle
3. **Claude Code va réimplémenter l'interface** en HTML/CSS/React natif, **responsive natif au 1024×600**, sans bundle, sans transform scale, sans MutationObserver

**À supprimer** lors du passage à Claude Code : tout le bloc script de fix, les overrides CSS du body, toute référence au `#stage` injecté.

---

### Teaser de fin d'épisode

> *« Le cerveau s'allume, l'interface s'affiche, le canal de communication est prêt — il pulse, il écoute. Pour l'instant, il parle tout seul. Dans le prochain épisode, on commence à câbler le hardware physique pour que quelque chose lui réponde. À très bientôt. »*

---

<a id="episode-2"></a>
## Épisode 2 — Premier signal hardware

**Statut** : placeholder — contenu à définir à réception du Teensy.

**Arc narratif pressenti** : le canal de communication construit en Épisode 1 reste pour l'instant un monologue (le Pi parle tout seul). En Épisode 2, on connecte le premier élément hardware physique — probablement le Teensy 4.1 sur Serial USB — et on observe le premier vrai aller-retour. Le Pi envoie une commande, le Teensy répond. Le système n'est plus une simulation : il commence à vivre dans le monde physique.

**Décisions techniques à venir** dans cet épisode :
- Bridge Serial USB Pi ↔ Teensy
- Implémentation du `broadcast()` côté Node.js connecté au flux Serial
- Premier protocole de test (ping/pong)
- Validation du timing aller-retour Pi → Teensy → Pi

(Détails à compléter au moment du tournage.)

---

<a id="a-faire"></a>
## À faire prochainement

### Court terme (sessions à venir, en attente composants)
- [ ] Tournage de la **Vidéo Trailer** (1-2 min) — première position playlist YouTube
- [ ] Variantes du logo Sauvignac (mono noir, blanc, favicon, SVG vectorisé)
- [ ] Intégration du logo définitif dans le Home Dashboard (itération 7 Claude Design)
- [ ] Validation du tactile sur l'interface HTML (tap simple sans délai 300ms, pas de zoom intempestif)
- [ ] Préparer environnement de dev Teensy (PlatformIO/Arduino IDE avec Teensyduino)
- [ ] Finaliser dessin AutoCAD de la plate principale (attendre mesures précises des composants critiques)
- [ ] **Stabilité Wi-Fi Pi 5** — ajouter un watchdog / reconnect auto (le Pi a décroché du réseau pendant la session du 27 avril)
- [ ] Activer le remote debugging Chromium (flag `--remote-debugging-port=9222` dans `~/.xinitrc`) pour debug à distance

### Moyen terme (après arrivée Teensy)
- [ ] Test circuit MIDI sur breadboard (MIDI IN + MIDI OUT)
- [ ] Commande Sculpteo : plate test puis plate principale + module peuplier
- [ ] Commande Bourns PEC11R sur TME au moment commande Sculpteo
- [ ] Implémenter debouncing logiciel encodeurs dans firmware Teensy
- [ ] Vérifier configuration pins PCM1808/PCM5102A à réception modules
- [ ] Premier firmware Teensy (boot, ping série, test OLED)
- [ ] **Passage à Claude Code** : réimplémentation native de l'interface 1024×600 — supprimer le fix `stage_fit` (Épisode 1 §6)
- [ ] Bridge Serial USB Pi ↔ Teensy + branchement sur le helper `broadcast()` (Épisode 2)
- [ ] Écrans UI suivants (Voice Edit LFO, Settings, Libraries, etc.)

### Long terme
- [ ] Tournage de l'Épisode 0 — Vision et fondations
- [ ] Roadmap PCB custom V3+ (apprentissage KiCad via projets Sauvignac DIY Phase 1-2)
- [ ] Évaluation besoin PCM3168A après 6 mois de jam avec V1
- [ ] Épisode YouTube « Migration PCB custom » (plus tard)

---

<a id="references"></a>
## Documents techniques de référence

Ce journal est complété par les documents techniques suivants (publiés dans le repo) :

- `PERKOMPANION_VISION.md` — vision projet et architecture cible
- `PERKOMPANION_PHASE0.md` — protocole Pi↔Teensy, MIDI, threads
- `PERKOMPANION_PROTOCOL_TABLES.md` — tables protocole détaillées
- `PERKOMPANION_SPEC_v0_2.md`, `v0_3.md`, `v0_4.md` — specs successives
- `panel_design.md` — design panneau physique
- `teensy_pinout.md` — pinout Teensy 4.1
- `midi_hardware.md` — circuiterie MIDI
- `PERKOMPANION_EPISODE1_GUIDE.md` — guide de reproduction de l'Épisode 1

---

*Sauvignac — On part de zéro. On construit.*
