# PërKompanion — Device Profile Schema

> Spécification du format Device Profile — **v1.0.0** — avril 2026
>
> Document narratif accompagnant `profile_template_with_docs.yaml`. Sert
> aux lecteurs qui veulent comprendre l'architecture du framework sans
> lire de YAML.

---

## 1. Pourquoi un format de profile

PërKompanion a démarré comme companion dédié à l'Erica Synths Perkons HD-01.
Avant la première ligne de code applicatif, le projet a été refondu en
**framework générique de companion MIDI** + **profiles de devices**
chargeables. Le Perkons devient le profile de référence (premier
implémenté), pas le projet lui-même.

Un **Device Profile** est un fichier YAML qui décrit la grammaire MIDI
d'une drum machine, d'un synthé ou d'un sampler hardware : ses tracks,
ses paramètres, leurs courbes de réponse, leurs modes d'adressage. Le
runtime PërKompanion charge un ou plusieurs profiles simultanément
(jusqu'à 4 en v1), expose leurs paramètres au moteur de modulation, et
les pilote via les commandes du protocole USB binaire Pi↔Teensy.

Le profile décrit **ce qu'un device est**, pas **comment le runtime
l'utilise**. Cette séparation est un principe central du format : tout
ce qui relève d'une politique d'usage (quand préfixer, comment combiner
plusieurs profiles, comment reprendre après une déconnexion) appartient
au runtime, pas au profile.

---

## 2. Principes de conception

### 2.1 Bottom-up

Le format n'a pas été conçu *a priori* mais dérivé du stress-test
progressif sur quatre devices physiquement disponibles, choisis pour
couvrir un spectre large de paradigmes MIDI :

| # | Device | Paradigme testé |
|---|--------|----------------|
| 1 | Erica Synths Perkons HD-01 | Multi MIDI Channel, voix isomorphes, CC pur |
| 2 | Elektron Digitakt II | Multi Channel, tracks hétérogènes, density élevée |
| 3 | Korg Electribe 2 Sampler | Single Channel + Part Select, NRPN, deux groupes FX |
| 4 | Behringer TD-3-MO | Minimalisme : 1 track, 0 globals, 13 params |

À chaque profile, les frottements observés ont durci ou enrichi le
modèle. Les profiles précédents ont été réécrits quand nécessaire.
Aucune abstraction n'est introduite avant qu'un device réel l'exige.

### 2.2 Format de données, pas DSL

Le YAML est un format de **données**. Aucun mécanisme de templating
runtime, aucune substitution dynamique, aucune expression à évaluer.
Quand un device a 16 parts qui ne diffèrent que par un index, les 16
sont listées explicitement. La perte en concision est compensée par
un gain en debuggabilité et en simplicité de parsing.

Les anchors et aliases YAML (`&name` / `*name`) sont autorisés et
encouragés pour factoriser les structures isomorphes. C'est un mécanisme
**du format YAML lui-même**, résolu au parsing, pas un templating ad hoc.

### 2.3 Format Pi-only

Le Teensy ne parse jamais le YAML. Le Pi (Node.js) parse le profile,
le valide, le sérialise en binaire compact, et le transmet au Teensy
via le protocole USB. Le Teensy reçoit une représentation alignée et
compacte qu'il stocke en EEPROM (mode dégradé) et en RAM (mode normal).

Conséquence : le coût de parsing côté embarqué n'est pas un critère du
format. La **lisibilité humaine** et la **richesse documentaire** le sont.

### 2.4 Documentation des absences

Un profile décrit autant ce qu'il ne couvre pas que ce qu'il couvre.
Le bloc `notes` est conçu pour rendre explicite :

- les hypothèses (`midi_mode_assumed`)
- les modes alternatifs ignorés (`alternate_modes_known`)
- les valeurs non vérifiées (`to_validate_against_midi_impl_chart`)
- les features hardware non adressables en MIDI (`not_midi_exposed`)
- les messages MIDI acceptés mais non modélisés (`midi_messages_not_modeled`)

Cette discipline est précieuse pour les contributeurs MIT qui reprennent
ou étendent un profile existant : ils savent immédiatement où sont les
trous assumés et lesquels ils peuvent combler.

---

## 3. Vue d'ensemble de la grammaire

Un profile est un document YAML avec sept sections, dont quatre sont
obligatoires et trois sont optionnelles.

| Section | Statut | Rôle |
|---------|--------|------|
| `profile` | obligatoire | identité, version, couleur, slot, scopes |
| `track_labels` | obligatoire | vocabulaire UI pour les tracks |
| `notes` | optionnelle | documentation libre |
| `tracks` | obligatoire | collection des unités pilotables |
| `global_param_groups` | optionnelle | paramètres globaux du device |
| `quirks` | optionnelle | comportements connus, bugs firmware |

L'absence d'une section optionnelle se signale par l'**omission complète**
(pas de stub vide). Un device sans globals n'a tout simplement pas de
clé `global_param_groups` dans son fichier.

---

## 4. Section `profile` — identité

| Champ | Type | Statut | Description |
|-------|------|--------|-------------|
| `id` | string | obligatoire | identifiant stable, snake_case, unique |
| `name` | string | obligatoire | nom commercial affiché en UI |
| `version` | string SemVer | obligatoire | version du profile (≠ firmware device) |
| `device_firmware` | string | obligatoire | firmware cible du device, format libre |
| `author` | string | obligatoire | auteur (pseudonyme accepté) |
| `license` | string | obligatoire | licence (MIT recommandé) |
| `default_color` | string hex | obligatoire | couleur d'accent device, palette restreinte |
| `preferred_slot` | integer 0..3 | optionnelle | hint runtime, non bloquant |
| `snapshot_scopes` | array | obligatoire | scopes de snapshot supportés |

La couleur est restreinte à la palette Cyber Brutalist Sauvignac (8 à 12
couleurs validées). L'utilisateur peut surcharger via ses settings, mais
le profile lui-même n'accepte pas de valeur libre arbitraire.

`preferred_slot` est un **hint**, pas une réservation. Le runtime peut
allouer un autre slot en cas de conflit avec un profile déjà chargé.

`snapshot_scopes` énumère les niveaux de snapshot pertinents pour ce
device. Valeurs : `track`, `device`, `page`, `global`. Un device avec une
seule track expose tout de même `track` et `device` — la sémantique reste
distincte côté UI.

---

## 5. Section `track_labels` — vocabulaire UI

L'abstraction commune au format s'appelle `tracks`. Chaque device choisit
le nom vernaculaire affiché à l'utilisateur final. Deux champs
obligatoires : `singular` et `plural`.

Exemples observés : voice/voices (Perkons, TD-3-MO), track/tracks
(Digitakt), part/parts (Electribe).

Le runtime utilise ces labels uniquement pour l'UI. Toute la logique
interne référence l'abstraction `track`.

---

## 6. Section `notes` — documentation libre

Tout est optionnel. Aucune clé n'est imposée par le format. Le runtime
ne lit aucun champ de cette section. Conventions observées dans les
profiles de référence :

- `midi_mode_assumed` — mode MIDI assumé par le profile
- `midi_mode_setup` — comment configurer ce mode sur le device
- `alternate_modes_known` — modes alternatifs hors-scope
- `to_validate_against_midi_impl_chart` — points à vérifier
- `not_midi_exposed` — features hardware non MIDI
- `midi_messages_not_modeled` — messages MIDI acceptés mais non décrits
- `audio_in_routing` — informations audio (hors-MIDI)

Un contributeur peut ajouter ses propres clés sous `notes` sans casser
le format.

---

## 7. Section `tracks` — unités pilotables

Une track représente une unité de contrôle adressable : voix, track
audio, track MIDI, part, monosynth complet.

### 7.1 Champs d'une track

| Champ | Type | Statut | Description |
|-------|------|--------|-------------|
| `id` | string | obligatoire | identifiant stable, snake_case |
| `label` | string | obligatoire | label court UI (1-3 chars idéal) |
| `kind` | string libre | obligatoire | catégorie pour groupage UI |
| `addressing` | bloc | obligatoire | comment cibler la track en MIDI |
| `cc_offset` | integer | optionnelle | offset appliqué à tous les CCs |
| `params` | array | obligatoire | paramètres pilotables |

### 7.2 Le champ `kind`

`kind` est un **string libre**, jamais validé par le format ni par le
runtime. Sa valeur sert au groupage visuel UI. Diversité observée dans
les profiles de référence :

- `"voice"` — Perkons, TD-3-MO
- `"audio"`, `"midi_out"` — Digitakt II
- `"part"` — Electribe 2 Sampler
- `"monosynth"` — TD-3-MO

Un contributeur peut introduire toute valeur de `kind` qui lui semble
pertinente pour son device.

### 7.3 Le bloc `addressing`

Décrit comment cibler la track en MIDI.

| Sous-champ | Type | Statut | Description |
|------------|------|--------|-------------|
| `midi_channel` | integer 1..16 | obligatoire | canal MIDI |
| `preface` | array | optionnelle | messages préfixe avant chaque param |

`preface` est une liste de messages MIDI déclaratifs envoyés avant chaque
message de paramètre, pour adresser la bonne track. Cas typique : Single
Channel + Part Select sur Korg Electribe (un NRPN qui pré-sélectionne la
part avant d'envoyer la valeur du paramètre).

Le format de chaque entrée du `preface` suit la grammaire du `mechanism`
(voir section 8). Pour un NRPN preface :

```yaml
preface:
  - { mechanism: "nrpn", msb: 0, lsb: 1, data_msb: 5 }
```

Le `data_msb` ici représente la valeur de sélection (par exemple, l'index
de la part Korg, 0..15).

`preface` est une **liste**, pas un singleton, pour permettre des cas
exotiques futurs (deux NRPN successifs, sysex + CC, etc.) sans changer
la grammaire. Le coût d'une liste à un élément est minime.

### 7.4 Le champ `cc_offset`

Optionnel. Quand présent, le runtime applique cet offset à **tous** les
CCs des params de la track. Permet de réutiliser un anchor de params
entre tracks isomorphes quand le device numérote ses voix par offset
constant.

Exemple Perkons HD-01 : la voix 1 utilise les CCs 70..80, la voix 2
les CCs 81..91, etc. L'anchor `voice_params` définit les CCs de la voix
1 (70..80), et chaque voix supérieure applique un `cc_offset` (11, 22,
33). Le runtime calcule `actual_cc = cc + cc_offset`.

À ne pas utiliser quand les tracks ne sont pas isomorphes — Digitakt
audio et Digitakt MIDI ont des structures de params différentes, donc
deux anchors distincts, pas un anchor + offset.

---

## 8. Grammaire d'un `param`

Un param décrit un paramètre pilotable. Champs communs :

| Champ | Type | Statut | Description |
|-------|------|--------|-------------|
| `name` | string | obligatoire | identifiant lisible humain |
| `mechanism` | string | optionnelle (défaut `"cc"`) | type de message MIDI |
| `curve` | string | obligatoire | courbe de réponse encodeur→MIDI |
| `type` | string | obligatoire | type sémantique |
| `range` | array `[a, b]` | optionnelle | bornes (selon `type`) |
| `values` | array de strings | obligatoire si `enum` | labels successifs |
| `resolution` | integer | optionnelle | résolution effective en bits |

### 8.1 Mécanismes supportés

**v1 supporte deux mécanismes :**

`mechanism: "cc"` (défaut implicite si omis) — message Control Change MIDI
standard 7 bits. Champ supplémentaire :
- `cc` (integer 0-127, obligatoire)

`mechanism: "nrpn"` — message NRPN structuré (Non-Registered Parameter
Number) 14 bits. Champs supplémentaires :
- `nrpn_msb` (integer 0-127, obligatoire)
- `nrpn_lsb` (integer 0-127, obligatoire)

Quand un param est utilisé dans un `preface` (sélection de track), la
grammaire est légèrement différente : le `mechanism` reste `"cc"` ou
`"nrpn"`, mais les champs deviennent `msb`, `lsb`, `data_msb` (la valeur
à envoyer pour sélectionner la track). Cette dissymétrie est volontaire :
un preface entry est un message MIDI complet déclaratif, pas une
description de paramètre.

### 8.2 Courbes

Valeurs admises pour `curve` :

| Valeur | Usage |
|--------|-------|
| `"none"` | aucune interpolation (pour `enum` et `boolean`) |
| `"linear"` | linéaire pur |
| `"exp"` | exponentielle (perception volume, time, decay) |
| `"log"` | logarithmique douce |
| `"log_strong"` | logarithmique forte (Cutoff fréquentiel) |
| `"scurve"` | sigmoïde (drive, saturation) |

Les courbes sont implémentées par le runtime, pas par le profile. Le
profile choisit, le runtime applique.

### 8.3 Types sémantiques

| Valeur | Description | Range par défaut |
|--------|-------------|------------------|
| `"continuous"` | continu unipolar | `[0, 1]` |
| `"continuous_bipolar"` | continu bipolar | `[-1, +1]` |
| `"integer"` | entier discret | `[0, 127]` |
| `"boolean"` | bool 0/1 | `[0, 1]` |
| `"enum"` | énumération | indices `0..len(values)-1` |

Le `range` peut surcharger le défaut quand pertinent (par exemple, un
integer borné `[0, 31]` pour un Sample Slot limité).

### 8.4 Resolution

`resolution` indique le nombre de bits effectifs du paramètre. Défaut
implicite : 7 pour CC, 14 pour NRPN. À spécifier explicitement quand un
param a une résolution non-standard (par exemple, un CC qui n'utilise
en réalité que les valeurs 0..15, soit 4 bits).

---

## 9. Section `global_param_groups` — paramètres globaux

Optionnelle. Si le device n'expose aucun paramètre global en MIDI, la
section est absente du fichier.

Un groupe est un cluster cohérent de paramètres globaux (FX bus, Master,
Common, etc.). Plusieurs groupes peuvent coexister sur des canaux MIDI
différents — d'où la structure en **liste de groupes** plutôt qu'en liste
plate de params.

| Champ | Type | Statut | Description |
|-------|------|--------|-------------|
| `id` | string | obligatoire | identifiant stable, snake_case |
| `label` | string | obligatoire | nom collectif affiché en UI |
| `addressing` | bloc | obligatoire | même grammaire que tracks.addressing |
| `params` | array | obligatoire | même grammaire que tracks.params |

Le bloc `addressing` est symétrique entre tracks et global_param_groups.
Un groupe peut donc avoir son propre preface (cas du "Common" zone Korg
Electribe).

---

## 10. Section `quirks` — comportements connus

Optionnelle. Dictionnaire ouvert. Aucune clé n'est réservée par le
format. Sert à signaler des bugs firmware connus, des limitations, des
particularités d'implémentation.

Conventions observées dans les profiles de référence :

| Clé | Type | Signification |
|-----|------|--------------|
| `pot_catch_broken` | bool | Pot Catch défectueux (Perkons fw v1.2) |
| `midi_only_partial` | bool | features hardware non en MIDI |
| `has_nrpn_only_params` | bool | params haute-rés NRPN-only |
| `requires_part_select_preface` | bool | Single Channel mode |

Le runtime peut consulter certaines de ces clés si elles existent. Un
contributeur peut ajouter ses propres clés sans casser le format.

---

## 11. Allocation des `param_id`

Le bloc `0x0000-0x0FFF` du protocole USB binaire Pi↔Teensy est partitionné
en **4 slots de 1024 paramètres** chacun :

| Slot | Plage param_id | Capacité |
|------|---------------|----------|
| 0 | `0x0000-0x03FF` | 1024 params |
| 1 | `0x0400-0x07FF` | 1024 params |
| 2 | `0x0800-0x0BFF` | 1024 params |
| 3 | `0x0C00-0x0FFF` | 1024 params |

À l'intérieur d'un slot, le runtime alloue les `param_id` séquentiellement
selon l'ordre de déclaration des params dans le profile (parcours :
tracks puis global_param_groups, dans l'ordre de définition).

Un profile peut déclarer un `preferred_slot` (0..3) comme hint
d'allocation. Le runtime résout les conflits : si deux profiles préfèrent
le même slot, le second se voit assigner un autre slot disponible.

Capacité par profile observée dans les profiles de référence :

| Profile | Params | Marge |
|---------|--------|-------|
| Perkons HD-01 | 44 | < 5% |
| Digitakt II | ~414 | ~40% |
| Electribe 2 Sampler | ~234 | ~23% |
| TD-3-MO | 13 | < 2% |

Aucun profile observé ne sature un slot. Si un device futur dépassait
1024 params, le mécanisme d'extension serait un bloc additionnel
(`0x7000+` réservé en v5+ pour cela).

Le bloc `0x6000-0x6FFF` est réservé aux voix virtuelles internes V5-V8
de PërKompanion (relocalisation issue de l'ancien bloc Perkons).

---

## 12. Conventions YAML

| Convention | Justification |
|-----------|---------------|
| YAML 1.2 strict | exclure les ambiguïtés YAML 1.1 |
| Indent 2 espaces | uniformité |
| Strings en double-quotes | éviter le "Norway problem" (`no` → `false`) |
| Anchors et aliases autorisés | factorisation des structures isomorphes |
| Commentaires `#` recommandés | documentation in-file pour contributeurs |

Pas de tabulations. Pas de YAML flow style en mélange chaotique avec
block style — l'inline `{ key: val, key: val }` est utilisé seulement
pour les listes courtes (params, tracks à une ligne) où la lisibilité
verticale serait perdue.

---

## 13. Versionnage des profiles

Chaque profile a sa propre version SemVer dans le champ `profile.version` :

- **Major** : breaking change dans la sémantique d'un param existant
  (renommage, suppression, changement de courbe radical, changement de
  CC). Les snapshots utilisateurs sont invalidés.
- **Minor** : ajout de params, ajout de tracks, ajout de groupes globaux.
  Les snapshots utilisateurs restent compatibles.
- **Patch** : corrections de valeurs CC erronées, mise à jour de `notes`,
  ajout de `quirks`.

Le profile inclut aussi `device_firmware`, qui indique le firmware cible
du device. Un même profile peut couvrir plusieurs firmwares mineurs ;
si un firmware majeur change la grammaire MIDI, on crée un profile
distinct (`perkons_hd01_v2.yaml`) plutôt que d'embarquer un branchement
au runtime.

---

## 14. Extensions futures envisagées

Les éléments suivants ne sont **pas** supportés par le schéma v1 mais
sont documentés ici pour signaler les directions d'évolution prévues :

**`mechanism: "pitch_bend"`** — message Pitch Bend natif (14 bits).
Déclencheur pressenti : profile Hydrasynth (instrument utilisant
massivement le Pitch Bend pour l'expressivité).

**`mechanism: "aftertouch"`** — Channel Aftertouch et/ou Polyphonic
Aftertouch. Déclencheur pressenti : profile Hydrasynth.

**`mechanism: "sysex"`** — messages SysEx structurés pour devices qui
exposent certains paramètres uniquement en SysEx. Format à définir le
moment venu, probablement avec template binaire et placeholders pour
la valeur.

**Slots > 4 simultanés** — extension du bloc `0x7000+` si plus de 4
profiles externes simultanés en v5+.

**Templating intégré** — explicitement refusé en v1. Le seul mécanisme
de factorisation est l'anchor YAML natif. Si le besoin se manifeste
plus tard (devices à 64+ tracks isomorphes par exemple), il faudrait
réévaluer — mais en première approche, lister explicitement reste le
choix par défaut.

Toutes ces extensions seront introduites par stress-test bottom-up
(un device qui en a réellement besoin), pas par anticipation.

---

## 15. Quatre profiles de référence

Les quatre profiles utilisés pour valider le format sont publiés dans
`profiles/` :

- `perkons_hd01.yaml` — référence, voix isomorphes, CC pur
- `elektron_digitakt_ii.yaml` — tracks hétérogènes, density élevée
- `korg_electribe_2_sampler.yaml` — Single Channel + Part Select, NRPN
- `behringer_td3_mo.yaml` — minimaliste, 1 track, 0 globals

Lire ces profiles aux côtés du présent document est la meilleure manière
de comprendre la grammaire en action.

---

*Fin de device_profile_schema.md — v1.0.0, avril 2026*
