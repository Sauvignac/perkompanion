# Livrable 4 — Décisions transverses propagées (chef d'orchestre)

> Issu de la conversation "PërKompanion — Réflexion architecture framework"
> du 28 avril 2026.
>
> Document à reporter au journal de décisions transverse
> (`PERKOMPANION_DECISIONS.md` ou conv chef d'orchestre Claude.ai), pour
> consolidation avec les autres décisions du projet.
>
> Toutes les décisions ci-dessous ont été **actées par le chef d'orchestre
> dans la conv source**. Ce document les propage formellement, il ne les
> resoumet pas au débat.

---

## §F1. Pivot framework + Device Profiles

**Décision actée** : PërKompanion est refondu en **framework générique de
companion MIDI** + **Device Profiles** chargeables. Le Perkons HD-01
devient le profile de référence (premier implémenté), pas le projet
lui-même.

**Justification** :
- Décision prise *avant* la première ligne de code applicatif → aucune
  dette technique.
- Les abstractions existantes (param_id 16-bit avec blocs réservés, moteur
  de modulation device-agnostic, protocole USB binaire sans noms hardcodés)
  facilitent ce pivot.
- Quatre devices physiquement disponibles à l'atelier (Perkons, Digitakt II,
  Electribe 2 Sampler, TD-3-MO) ont permis de stress-tester progressivement
  le format en bottom-up.

**Naming public** : on garde "PërKompanion" comme nom de projet pour
toute la phase 1. Le pivot framework reste **invisible côté public**
(chaîne YouTube, branding, pitch). Le pivot deviendra un événement
narratif au moment du lancement de la 2e machine supportée.

**Reporté hors-scope** :
- Choix du nom de la marque-mère framework.
- Stratégie de communication du pivot.

---

## §F2. Format des Device Profiles

**Décision actée** : YAML 1.2 strict, double-quotes systématiques,
indent 2 espaces, anchors et aliases natifs autorisés.

**Justification** :
- Commentaires natifs indispensables pour les contributeurs MIT.
- Anchors évitent 200-400 lignes de duplication sur les devices à tracks
  isomorphes (Perkons, Digitakt audio, Electribe).
- Stack `js-yaml` côté Node.js Pi, pas de friction écosystème.
- Format Pi-only : le Teensy ne parse jamais le YAML, donc le coût de
  parsing embarqué n'est pas un critère.

**Alternatives écartées** : JSON (pas de commentaires, pas d'anchors),
JSON5 (pas d'anchors), TOML (mauvais en hiérarchie profonde), DSL custom
(anti-pattern).

**Documents associés** :
- `device_profile_schema.md` — spec narrative.
- `profile_template_with_docs.yaml` — template auto-documenté.

---

## §F3. Allocation des `param_id`

**Décision actée** : restructuration du bloc `0x0000-0x0FFF`.

**Avant** : bloc Perkons hardcodé avec sous-bloc `0x0500-0x08FF` pour
voix virtuelles internes V5-V8.

**Après** :
- `0x0000-0x0FFF` : profiles externes uniquement, partitionné en **4 slots
  × 1024 paramètres**.
- `0x6000-0x6FFF` : voix virtuelles V5-V8 internes (relocalisées).
- `0x7000-0x7FFF` : réserve double — modulations complexes / séquenceurs
  hotcue (v4+) **et** extension profiles si plus de 4 profiles simultanés
  (v5+).
- `0x8000-0xFFFF` : inchangé (réservé utilisateur / plugins).

**Hint utilisateur** : un profile YAML peut déclarer un champ
`preferred_slot: N` (0-3) pour suggérer un slot d'accueil au runtime. Le
runtime résout les conflits (autre slot disponible).

**Capacité observée** : aucun profile testé ne sature un slot (Digitakt II
le plus dense à ~414 params, soit ~40% d'un slot 1024).

**Documents associés** :
- `PERKOMPANION_PROTOCOL_TABLES.md` v1.0.0 §2.

---

## §F4. Nouvelle famille de commandes protocole 0xC0-0xCF

**Décision actée** : ajout du **bloc 0xC0-0xCF Pi → Teensy "Device Profiles"**
pour le chargement, déchargement et listing des profiles externes.

**Commandes** :

| CMD | Nom | Justification |
|-----|-----|---------------|
| 0xC0 | LOAD_DEVICE_PROFILE_BEGIN | initie un transfert chunké |
| 0xC1 | LOAD_DEVICE_PROFILE_CHUNK | charge un chunk (≤240 bytes) |
| 0xC2 | LOAD_DEVICE_PROFILE_END | finalise et valide CRC |
| 0xC3 | UNLOAD_DEVICE_PROFILE | libère un slot |
| 0xC4 | LIST_LOADED_PROFILES | retourne via 0xCA |
| 0xC5 | SET_ACTIVE_PROFILE_FOR_DEGRADED_MODE | persistance EEPROM |

**Côté retour Teensy → Pi** : sous-bloc 0xCA-0xCC.

| CMD | Nom |
|-----|-----|
| 0xCA | DEVICE_PROFILE_LIST_ENTRY |
| 0xCB | PROFILE_LOAD_ACK |
| 0xCC | PROFILE_UNLOAD_ACK |

**Choix du bloc 0xC0-0xCF** : libre Pi → Teensy avant v1.0.0, adjacent
thématiquement aux blocs 0x80 (DSP) et 0x90 (Voice Kits) qui sont aussi
de la "configuration de slot". Le bloc 0xC0-0xCF Teensy → Pi est déjà
utilisé pour les hotcues (0xC0-0xC4) ; les nouvelles CMD ACK profile
0xCA-0xCC y sont insérées sans conflit.

**Format binaire du blob transféré** : non spécifié dans cette décision.
Document `device_profile_binary.md` à produire en pré-implémentation
firmware Teensy.

**Documents associés** :
- `PERKOMPANION_PROTOCOL_TABLES.md` v1.0.0 §6 (commandes) et §14 (vue
  d'ensemble Device Profile Schema).

---

## §F5. Settings utilisateur séparés du Device Profile

**Décision actée** : les **settings utilisateur** (overrides de couleurs,
mappings personnalisés des CCs user-assignables type Digitakt User_CC1..8,
labels custom) vivent dans un fichier séparé `settings.yaml` côté Pi, et
**ne sont jamais mélangés au Device Profile**.

**Justification** :
- Un Device Profile décrit *ce qu'un device est* (propriétés intrinsèques).
- Les settings décrivent *comment l'utilisateur veut le piloter* (préférences
  contextuelles).
- Cette séparation préserve la pureté du profile pour partage open source MIT,
  et permet à un utilisateur de cumuler un profile communautaire + ses
  préférences locales sans collision.

**Conséquences** :
- Les CCs `User_CC1..8` du Digitakt restent **opaques** dans son profile.
- Un futur écran "Settings utilisateur" côté UI proposera l'override des
  labels et le mapping vers les machines cibles.

**Documents associés** :
- `profiles/elektron_digitakt_ii.yaml` — exemple concret de CCs opaques.
- À venir : `settings_schema.md` (hors v1).

---

## §F6. Mode dégradé multi-profile (Option A + fallback B)

**Décision actée** : le mode dégradé du Teensy pilote le **dernier profile
actif** (Option A), avec fallback automatique sur Perkons hardcodé
(Option B) si EEPROM corrompue / vide.

**Mécanisme** :
- Le Teensy garde en EEPROM (4 KB sur Teensy 4.1) un snapshot binaire
  minimal du profile actif : CCs, channels, ranges, courbes par paramètre.
  Pas l'UI, pas les noms.
- Mise à jour automatique de l'EEPROM à chaque changement de page côté
  Pi (commande 0xC5).
- Validation au boot Teensy via CRC + version.
- Si EEPROM invalide → fallback automatique sur Perkons hardcodé (machine
  fondatrice toujours présente dans le setup).
- Indicateur visuel sur OLED master pour signaler quel mode est actif.

**Justifications du choix** :
- Option C ("safe minimal sans MIDI externe") rejetée : casserait trop fort
  un set live en cas de crash Pi.
- Option B seule rejetée : conceptuellement régressive, Perkons redeviendrait
  privilégié dans le code framework.
- Option A + fallback B : compromis qui minimise la disruption tout en
  garantissant qu'on n'est jamais bloqué.

**Contrainte EEPROM** : taille profile minimal sérialisé estimée 200-500
bytes. 4 KB largement suffisants même pour stocker plusieurs profiles
en cache.

**Risque connu** : EEPROM peut devenir obsolète après mise à jour de
profile côté Pi sans propagation. Synchronisation à gérer via `version`
+ `crc32` dans le profile.

**Documents associés** :
- `PERKOMPANION_VISION.md` v1.0.0 §10 nouveau sous-chapitre "Mode dégradé
  multi-profile".
- `PERKOMPANION_PHASE0.md` v1.0.0 §13 "Framework et Device Profiles".

---

## §F7. Snapshots — généralisation v3+ avec scope

**Décision actée** : les concepts v0.7 "Voice Kit" et "Perkons Snapshot"
sont fusionnés en un concept unique **"Snapshot"** avec paramètre
`scope` ∈ {voice, device, page, global}.

**Modèle** :
```
Snapshot = {
  scope: enum [voice / device / page / global],
  scope_target: id de la cible selon scope,
  parameters: [{param_id, value}, ...],
  metadata: {name, color, created_at, ...}
}
```

**Pour la v1** : seul le scope `voice` est exposé dans l'UI. L'utilisateur
continue à voir "Voice Kit" si c'est plus parlant. Mais le protocole et
le code interne utilisent déjà la commande générique avec scope.

**Pour v2-v3** : ouverture progressive vers Device Snapshot, Page Snapshot,
Global Snapshot.

**Conséquence sur les profiles** : chaque profile déclare ses
`snapshot_scopes` applicables (typiquement `["track", "device"]` pour les
4 profiles testés). L'UI ne propose à l'utilisateur que les scopes
déclarés par le profile actif.

**Conséquence protocole v1** : la commande `LOAD_SNAPSHOT` (0x91 actuel)
et `SAVE_SNAPSHOT` doivent être conçues dès v1 avec paramètre scope, même
si v1 ne l'utilise qu'en mode voice.

**Questions ouvertes pour v3** :
- Un snapshot peut-il être incomplet (capturer seulement certains
  paramètres) ?
- Les modulations actives sont-elles capturées dans un snapshot ?

**Documents associés** : tous les profiles `profiles/*.yaml` déclarent
`snapshot_scopes`.

---

## §F8. Couleur par profile + palette restreinte

**Décision actée** : chaque profile de machine a une couleur associée
dans son champ `default_color`, qui se propage sur tous les éléments
visuels (OLEDs voix, LEDs pad, accents écran 7"). Modèle "default +
overrides" : settings utilisateur peuvent surcharger.

**Contrainte palette** : choix limité aux 8-12 couleurs validées de la
charte Cyber Brutalist Sauvignac. Pas de choix libre hexadécimal pour
préserver l'identité visuelle.

**Palette pressentie** (à finaliser avec contributeurs) :
- `#FF0040` Rouge Sauvignac — Perkons (réservé)
- `#A8FF00` Vert acide — Digitakt (réservé)
- `#8B00FF` Violet — Electribe (réservé)
- `#FFD700` Jaune — TD-3-MO (réservé)
- `#FF00AA` Magenta — hotcues internes / V5-V8
- `#FF4500` Orange — signature Sauvignac universelle
- `#EEEEEE` Blanc cassé — fallback / neutre

**Documents associés** :
- `profile_template_with_docs.yaml` — palette listée pour contributeurs.
- 4 profiles de référence — chacun déclare `default_color`.

---

## §F9. Étiquetage des jacks d'entrée audio

**Décision actée** : les 4 jacks d'entrée audio sur la tranche arrière
de la plate doivent être étiquetés **IN 1 / IN 2 / IN 3 / IN 4** (gravure
laser FabLab), **pas** V1 IN / V2 IN / V3 IN / V4 IN.

**Justification** : étiquetage neutre qui ne fige pas conceptuellement les
entrées sur les voix Perkons. Cohérent avec la roadmap multi-machine v2-v4
où les entrées audio physiques pourront être routées vers n'importe quel
slot (sampling live d'une TD-3, monitoring DSP d'un Hydrasynth, etc.).

**Statut** : pas urgent (la plate ne part pas en gravure avant validation
prototype), mais à ne pas oublier.

**Documents associés** :
- `panel_design.md` — à mettre à jour avant envoi gravure.
- `PERKOMPANION_VISION.md` v1.0.0 §13 (note ajoutée).

---

## §F10. Extensions futures envisagées (non implémentées en v1)

Documentées pour signaler les directions d'évolution sans les coder :

**`mechanism: "pitch_bend"`** — message Pitch Bend natif (14 bits).
Déclencheur pressenti : profile **Hydrasynth**.

**`mechanism: "aftertouch"`** — Channel Aftertouch et/ou Polyphonic.
Déclencheur pressenti : profile **Hydrasynth**.

**`mechanism: "sysex"`** — messages SysEx structurés. Format à définir
le moment venu.

**Slots > 4 simultanés** — extension du bloc `0x7000+` si plus de 4
profiles externes simultanés en v5+.

**Templating intégré au format YAML** — explicitement refusé en v1. Si
le besoin se manifeste plus tard (devices à 64+ tracks isomorphes), à
réévaluer. Pour l'instant : lister explicitement reste le choix par défaut.

**Toutes ces extensions seront introduites par stress-test bottom-up**
(un device qui en a réellement besoin), pas par anticipation.

---

## Liste des documents produits ou modifiés en v1.0.0

### Nouveaux documents

- `device_profile_schema.md` — spécification narrative complète du format
  Device Profile.
- `profile_template_with_docs.yaml` — template auto-documenté pour
  contributeurs.
- `profiles/perkons_hd01.yaml` — profile de référence Perkons HD-01.
- `profiles/elektron_digitakt_ii.yaml` — profile Digitakt II.
- `profiles/korg_electribe_2_sampler.yaml` — profile Electribe 2 Sampler.
- `profiles/behringer_td3_mo.yaml` — profile TD-3-MO.
- `DECISIONS_TRANSVERSES_FRAMEWORK.md` (ce document) — propagation
  chef d'orchestre.

### Documents modifiés

- `PERKOMPANION_PROTOCOL_TABLES.md` v0.9 → v1.0.0 (cf. changelog en tête)
- `PERKOMPANION_PHASE0.md` v0.9 → v1.0.0 (cf. changelog en tête)
- `PERKOMPANION_VISION.md` v0.9 → v1.0.0 (cf. changelog en tête)

### Documents non modifiés mais à réévaluer

- `panel_design.md` — étiquetage jacks IN 1-4 (cf. §F9).
- `PERKOMPANION_DECISIONS.md` — peut intégrer les §F1-F10 ci-dessus selon
  le rythme et le format souhaité par le chef d'orchestre.
- `teensy_pinout.md` — à priori inchangé (le pivot framework n'impacte
  pas le pinout hardware).

---

*Fin de DECISIONS_TRANSVERSES_FRAMEWORK.md — v1.0.0, avril 2026*
