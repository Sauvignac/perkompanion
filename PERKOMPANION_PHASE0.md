# PërKompanion — Phase 0

> Fondations techniques à poser avant toute implémentation
> Document technique — **v0.9** — avril 2026

---

## Changements clés v0.8 final → v0.9

Les fondations techniques définies en v0.8 final restent valides en v0.9. Les évolutions v0.9 sont principalement **ergonomiques et mécaniques**, sans impact sur l'architecture temps-réel MIDI/audio :

**Ergonomie panneau** :
- **Module violet 9×5** : intégration des 5 modes M1-M5 (colonne gauche) et des 8 voix V1-V8 (rangée haut) dans le même module violet que le pad 8×4, pour un accès simultané des pouces (gauche = mode, droit = voix) sans quitter la main du pad trigger
- **Module violet redimensionné** : 190×115 mm (au lieu de 180×100 mm) pour accueillir la grille 9×5
- **Uniformisation Cherry MX partout** : abandon des tact 6×6 sur voix tête colonne + B1-B6 écran, remplacés par Cherry MX (cohérence visuelle et feel identique)
- **Bande centrale à 7 boutons** : 5 globaux + 2 réservés pour évolution future (UNDO, COPY, RANDOMIZE, LOCK, etc.)

**OLED master** :
- Passage de I²C (via TCA9548A) à **SPI 7-pin** pour uniformisation avec les 8 OLEDs voix
- **Bus I²C totalement libéré** (pins 18, 19 disponibles pour extensions futures)
- **TCA9548A** conservé dans la BOM en réserve mais non utilisé en v0.9

**Sourcing Teensy** :
- ProtoSupplies ne livrant pas en France, passage par **MyUS.com** (transitaire US → France)
- Teensy 4.1 Fully Loaded 32MB PSRAM : 64€ chez ProtoSupplies (port US inclus vers MyUS) + ~70-90€ FedEx/TVA = **~135-155€ total**
- Alternative "souder PSRAM soi-même" évaluée et rejetée (complexité CMS, préférence produit testé/monté pro)

**Aucun changement architectural** : pinout Teensy, architecture audio SAI1/SAI2 + 2×PCM1808 + 6×PCM5102A + TPA6120, chaîne SPI MCP23S17, protocole MIDI, mode dégradé — **tout reste identique à la v0.8 final**.

---

## Préambule

Ce document contient les décisions d'architecture technique qui doivent être tranchées **avant la première ligne de code**. Chacune de ces décisions, si prise tardivement ou modifiée après coup, provoque des refactoring majeurs ou des bugs difficiles à diagnostiquer en conditions live.

Ce document est complété par :
- `PERKOMPANION_PROTOCOL_TABLES.md` : tables de référence exhaustives (IDs, commandes, mapping physique, encoding des valeurs)
- `PERKOMPANION_VISION.md` : vision produit et fonctionnelle
- `teensy_pinout.md` : assignation GPIO complète du Teensy 4.1
- `midi_hardware.md` : schéma hardware des interfaces MIDI DIN

Pour toute question sur les IDs, commandes, numéros de CC MIDI, endianness, formats de données, encoding des valeurs sur le fil, consulter `PERKOMPANION_PROTOCOL_TABLES.md`.

Lecteur cible : développeur firmware Teensy, développeur software Pi, intégrateur hardware.

---

## 1. Routing MIDI — décisions fondatrices

### Principe général

**Le Teensy 4.1 est le cœur temps-réel MIDI. Le Pi 5 n'intervient jamais dans le chemin MIDI time-critical.**

Toute décision doit respecter cette règle. Un message MIDI qui doit arriver avec une précision <1ms ne passe pas par le Pi.

### Topologie MIDI définitive

```
┌─────────────────────────────────────────────────────────┐
│  [Laptop FL Studio]                                     │
│       │ USB                                             │
│       ▼                                                 │
│  [ESI M8U eX]  ← routeur 16×16                          │
│                                                         │
│   IN 1   ← FL Studio (clock maître, CC manuels)         │
│   IN 2   ← KeyStep 37 (notes pour arp)                  │
│   IN 16  ← Teensy MIDI OUT (modulations, triggers arp)  │
│                                                         │
│   OUT 8  → Perkons HD-01 (clock + tous CCs)             │
│   OUT 15 → Teensy MIDI IN (clock + notes externes)      │
│                                                         │
└─────────────────────────────────────────────────────────┘

Le Pi 5 n'est PAS connecté au réseau MIDI.
Le Pi communique avec le Teensy uniquement via USB série (data non-RT).
```

### Chemin du clock MIDI

**Critique pour le jitter.**

Source (FL Studio ou mode standalone) → ESI → **en parallèle** :
- Vers Perkons HD-01 (OUT 8) pour sync du séquenceur Perkons
- Vers Teensy MIDI IN (OUT 15) pour sync des LFOs, augmentations, séquenceurs hotcue

Le clock n'entre jamais dans le Pi. Latence clock Teensy < 1 ms.

### Chemin des CCs de modulation

Toutes les modulations générées par PërKompanion (LFOs, augmentations, enveloppes) produisent des CCs qui ciblent le Perkons.

Teensy génère CC → Teensy MIDI OUT → ESI IN 16 → ESI OUT 8 → Perkons.

**Pas de passage par le Pi pour ces messages.** Latence totale < 2 ms.

### Canal MIDI par voix — Multi MIDI Channel

**Décision actée** : le Perkons est configuré en **Multi MIDI Channel** :
- Voix 1 : canal MIDI 1
- Voix 2 : canal MIDI 2
- Voix 3 : canal MIDI 3
- Voix 4 : canal MIDI 4

Les CCs 70-80 contrôlent V1 sur canal 1, CCs 81-91 contrôlent V2 sur canal 2, etc.

Configuration Perkons : `SHIFT + MOD` → Track 3 Step 2 (voir `perkons_reference.md`).

Détails dans `PROTOCOL_TABLES.md` section 1.

### Coexistence CCs Teensy et CCs FL Studio

**Problème** : FL Studio peut envoyer des CCs manuels via Perkons (utilisateur qui automate dans le DAW) en parallèle des CCs générés par le Teensy (modulations PërKompanion).

**Règle** : les deux peuvent coexister sur les mêmes canaux MIDI. Le Perkons applique le dernier CC reçu pour chaque (canal, CC#, voix).

**Conséquence** : si l'utilisateur automate le Tune V1 dans FL ET qu'une modulation Teensy est active sur Tune V1, le résultat sera erratique (dernière valeur reçue gagne).

**Recommandation utilisateur** : ne pas dupliquer les automations. Soit FL pilote certains paramètres (et Teensy évite), soit Teensy pilote (et FL évite). Convention à établir par projet.

Une future extension pourrait détecter les conflits et alerter l'utilisateur, mais ce n'est pas prioritaire pour v1.

### Configuration ESI verrouillée

**Problème à éviter** : les boucles MIDI créent des comportements erratiques difficiles à diagnostiquer en live.

**Routes autorisées (explicitement actives)** :

| Source | Destination | Contenu | Justification |
|--------|-------------|---------|---------------|
| FL Studio | Perkons (OUT 8) | Clock + CC manuels + notes | Session principale |
| FL Studio | Teensy (OUT 15) | Clock uniquement | Sync Teensy |
| KeyStep 37 | Teensy (OUT 15) | Notes | Arp externe |
| Teensy | Perkons (OUT 8) | CC modulations + notes arp | Cœur PërKompanion |

**Routes interdites (vérifier absence)** :

| Source | Destination | Risque |
|--------|-------------|--------|
| Teensy | FL Studio | FL retraite et renvoie → boucle |
| Perkons | Teensy (pour clock) | Dépendance circulaire |
| Perkons | FL Studio | FL mélangerait avec ses outputs |
| KeyStep | Perkons direct | Contourne l'arp du Teensy |

### Configuration persistante ESI

Le software ESI permet d'exporter la config en fichier. **Actions à prévoir** :

- Sauvegarder la config dans un preset nommé "PERKOMPANION_LIVE"
- Exporter le fichier de config dans le repo : `docs/midi_routing/esi_m8u_ex_config.bin`
- Documenter la procédure de restauration dans `docs/setup/midi_routing_setup.md`
- Sauvegarder sur clé USB de backup pour secours en live

### Détection défensive de boucle

**Approche basée sur le timing** (remplace l'idée précédente de tagging par canal 16) :

Le Teensy tient une petite trace des CCs qu'il vient d'envoyer (hash : `channel|cc#|value|timestamp`). Si un CC identique arrive en IN du Teensy dans un délai <50 ms après être sorti, le firmware :
- Log l'événement (count incrémenté)
- Envoie `WARNING` au Pi (code = "MIDI_LOOP_DETECTED")
- Ignore le message entrant pour éviter l'amplification

Cette détection est légère (cache de 32 entrées glissantes, O(1) lookup) et défensive. Elle ne garantit pas la détection de toutes les boucles mais attrape les cas les plus dangereux.

### Clock en mode standalone

**Scénario** : PërKompanion utilisé sans laptop/DAW, pour jams impromptus ou en déplacement.

**Solution retenue** :
- Le **Teensy génère son propre clock interne** (via timer hardware à 24 PPQN)
- Le clock sort sur la MIDI OUT du Teensy vers l'ESI
- Via l'ESI, le clock arrive au Perkons (OUT 8) et optionnellement à d'autres machines
- Le Teensy **n'écoute pas son propre clock** en retour (pas de boucle : le Teensy utilise sa référence interne directement, pas celle retournée par l'ESI)

**Switch entre modes couplé / standalone** :
- Commande `SET_CLOCK_SOURCE` du Pi au Teensy
- En couplé : Teensy écoute le clock entrant sur MIDI IN (venant de FL via ESI)
- En standalone : Teensy ignore MIDI IN, utilise son clock interne

**Robustesse** : si le Pi tombe en mode standalone, le Teensy continue à générer son clock. Tout continue de jouer. C'est le point de résilience critique en free party.

---

## 2. Protocole USB série Pi ↔ Teensy

### Objectif

Communication bidirectionnelle fiable à haute fréquence entre Pi 5 et Teensy 4.1. Utilisé pour :
- Pi → Teensy : config, paramètres, samples à charger, LEDs à mettre à jour
- Teensy → Pi : états continus, événements utilisateur, monitoring

**Pas de JSON**, pas de texte, format binaire compact.

### Paramètres physiques

- Interface : USB série (Teensy expose un port série virtuel USB)
- Baud rate : **2 Mbps** (configuré en `Serial.begin(2000000)` côté Teensy)
- Côté Pi : `pyserial` avec `baudrate=2000000`, `timeout=0` (non-bloquant)
- Pas de contrôle de flux hardware

### Structure de trame

```
[SYNC 0xF0][DIR 1 byte][CMD 1 byte][LEN 1 byte][PAYLOAD N bytes][CHECKSUM 1 byte][END 0xF7]
```

- DIR : `0x01` (Pi → Teensy), `0x02` (Teensy → Pi)
- CMD : identifiant de commande (1 byte)
- LEN : longueur du PAYLOAD (0-240)
- CHECKSUM : XOR de DIR + CMD + LEN + PAYLOAD
- Total : 6 bytes d'overhead + payload

### Endianness

**Little endian** partout.

### Encoding des valeurs (CRITIQUE)

Toutes les valeurs de paramètres, amounts de modulation, etc. transitent en **int16 signed little endian** représentant une valeur normalisée.

```
value_wire : int16, range [-32767, +32767]
value_float : normalisé -1.0 à +1.0 (bipolar) ou 0.0 à +1.0 (unipolar)

Conversion : value_wire = round(clamp(value_float) * 32767)
```

Mapping vers CC MIDI 7-bit : `cc_value = (value_wire >> 8) & 0x7F` (bits de poids fort).

Les paramètres **enum** (Algo, Mode, Filter, etc.) utilisent l'index direct (0, 1, 2...) sans mapping normalisé.

Les paramètres **integer** (Pitch semitones, BPM centimals) utilisent la valeur entière directe.

**Voir `PROTOCOL_TABLES.md` section 4** pour spécification complète et exemples.

### Allocation complète des commandes

**Voir `PROTOCOL_TABLES.md` section 5** pour la table exhaustive des commandes (blocs 0x00-0xFF dans les deux sens).

**Blocs d'allocation** :
- 0x00-0x0F : connexion (HELLO, HEARTBEAT, GOODBYE, REQUEST_SNAPSHOT)
- 0x10-0x1F : paramètres et modulations
- 0x20-0x2F : hotcues (v3+)
- 0x30-0x3F : LEDs
- 0x40-0x4F : OLEDs
- 0x50-0x5F : système et reset
- 0x60-0x6F : arpégiateur
- 0x70-0x7F : pad mapping (avec chunking pour layouts longs)
- 0x80-0x8F : DSP chains (v2+)
- 0x90-0x9F : Voice Kits et Perkons Snapshots (v1+)
- 0xF0-0xFF : maintenance et debug

### Transfert de samples hotcue (chunking)

Les samples hotcue peuvent faire plusieurs MB. Protocole de transfert :

**Pi → Teensy (chargement au démarrage)** :
1. Pi envoie `LOAD_HOTCUE_START` (0x20) avec hotcue_id + total_size + sample_rate + bit_depth
2. Teensy répond `HOTCUE_LOADED_ACK` avec status=3 (READY) ou status=2 (MEMORY_FULL)
3. Pi envoie `LOAD_HOTCUE_CHUNK` (0x21) × N (chunks de 240 bytes max)
4. Teensy ne répond pas à chaque chunk (gain de bande passante)
5. Pi envoie `LOAD_HOTCUE_END` (0x22) avec total_checksum
6. Teensy répond `HOTCUE_LOADED_ACK` avec status=0 (OK) ou status=1 (CHECKSUM_FAIL)

**Teensy → Pi (sauvegarde d'une capture)** :
1. Teensy envoie `HOTCUE_CAPTURED` (0xC1) au Pi
2. Pi répond `REQUEST_HOTCUE_DATA` (0x26) avec hotcue_id
3. Teensy envoie série de `HOTCUE_CHUNK` (0xC2) × N
4. Teensy envoie `HOTCUE_CHUNK_END` (0xC3) avec total_checksum
5. Pi vérifie checksum, écrit sur SD

Voir `PROTOCOL_TABLES.md` sections 5 et 7 pour détails et priorisation.

### Priorisation des messages

Pour éviter que les transferts de samples saturent le canal et retardent les messages utilisateur :

**3 niveaux de priorité** (voir `PROTOCOL_TABLES.md` section 7) :
- **Niveau 1 (critique)** : HEARTBEAT, ENCODER, BUTTON, PAD, ERROR
- **Niveau 2 (standard)** : PARAM_VALUE, LED_UPDATE, monitoring
- **Niveau 3 (bulk)** : chunks samples, SNAPSHOT, OLED_BITMAP

Entre chaque chunk bulk, vider les queues niveaux 1 et 2.

### Budget bande passante

- **Pi → Teensy** : ~50 msg/s nominal (200 KB/s pendant un transfert sample)
- **Teensy → Pi** : ~1000 msg/s nominal

À 2 Mbps, capacité théorique 20-40k msg/s. Utilisation réelle : 3-5%. Marge confortable.

**Optimisation** : utiliser PARAM_BULK (0x91) et LED_BULK (0x31) pour grouper plusieurs valeurs. Réduction 10-20× du nombre de messages.

### Heartbeat et détection de déconnexion

- Pi envoie `HEARTBEAT` (0x02) toutes les 500 ms
- Teensy envoie `HEARTBEAT_ACK` (0x82) toutes les 500 ms
- **Seuil de déconnexion** : 3 heartbeats consécutifs manqués OU aucun trafic pendant 3 secondes
- Évite le faux positif en cas de micro-hoquet USB (charge CPU ponctuelle, etc.)
- Tentative de reconnexion toutes les 2 secondes jusqu'à rétablissement

### Snapshots d'état

Pour synchroniser Pi et Teensy après handshake ou reconnexion, le Teensy envoie un SNAPSHOT complet de son état (params, modulations, hotcues chargés, etc.).

Le snapshot est chunké via `SNAPSHOT_BEGIN` (0xF3) + `SNAPSHOT_CHUNK` (0xF4) × N + `SNAPSHOT_END` (0xF5).

Format complet : voir `PROTOCOL_TABLES.md` section 6.

**Taille typique** :
- v1 sans hotcues : ~2 KB
- v3 avec 50 hotcues chargés : ~5 KB
- v4 complet : ~10-20 KB

---

## 3. Architecture multi-thread côté Pi 5

### Problème à résoudre

1100 msg/s entrants traités en Python Flask = risque de backlog si le thread UI bloque le thread série. Impact : freezes UI intermittents en live.

### Architecture recommandée

```
┌─────────────────────────────────────────────────┐
│  Process Pi 5 (Python)                          │
│                                                 │
│  ┌────────────────────┐                         │
│  │ Thread_Serial_Read │ ← priorité haute        │
│  │                    │   (os.nice -20)         │
│  │ pyserial non-block │                         │
│  │ Parse trames       │                         │
│  │ Push → RecvQueue   │                         │
│  └──────────┬─────────┘                         │
│             │                                   │
│             ▼                                   │
│  ┌────────────────────────────────────┐         │
│  │  RecvQueue (queue.Queue, size=500) │         │
│  │  Si plein → drop plus anciens      │         │
│  └──────────┬─────────────────────────┘         │
│             │                                   │
│             ▼                                   │
│  ┌────────────────────────┐                     │
│  │ Thread_State_Updater   │ ← priorité moyenne  │
│  │                        │                     │
│  │ Pop RecvQueue          │                     │
│  │ Update AppState        │                     │
│  │ Émet événements UI     │                     │
│  └──────────┬─────────────┘                     │
│             │                                   │
│             ▼                                   │
│  ┌────────────────────────────────────┐         │
│  │  AppState (dict + threading.Lock)  │         │
│  │  Sources de vérité de l'UI         │         │
│  └──────────┬─────────────────────────┘         │
│             │                                   │
│             ▼ (via WebSocket)                   │
│  ┌────────────────────────┐                     │
│  │ Thread_Flask_UI        │ ← priorité normale  │
│  │                        │                     │
│  │ HTTP server            │                     │
│  │ Render pages           │                     │
│  │ WebSocket broadcast    │                     │
│  └────────────────────────┘                     │
│             ▲                                   │
│             │ (user actions)                    │
│             ▼                                   │
│  ┌────────────────────────┐                     │
│  │  SendQueue (3 niveaux) │                     │
│  │  q_critical, q_std,    │                     │
│  │  q_bulk                │                     │
│  └──────────┬─────────────┘                     │
│             │                                   │
│             ▼                                   │
│  ┌────────────────────────┐                     │
│  │ Thread_Serial_Write    │ ← priorité haute    │
│  │                        │                     │
│  │ Priorisation 3 niveaux │                     │
│  │ Write USB              │                     │
│  └────────────────────────┘                     │
│                                                 │
│  ┌────────────────────────┐                     │
│  │ Thread_Monitoring      │                     │
│  │                        │                     │
│  │ Heartbeat toutes 500ms │                     │
│  │ Détecte déconnexion    │                     │
│  │ Autosave périodique    │                     │
│  └────────────────────────┘                     │
└─────────────────────────────────────────────────┘
```

### Règles d'architecture

**Thread série (read/write)** :
- Jamais bloqués par l'UI
- Communication uniquement via queues thread-safe
- Timeout court sur les opérations série (100ms max)
- Si la queue de réception est pleine, **drop les plus anciens** (pas bloquer)

**Thread Flask UI** :
- Jamais de lecture/écriture directe du port série
- Lit `AppState` via lock rapide
- Envoie les user actions via `SendQueue`
- WebSocket pour push temps-réel

**AppState** :
- Structure de données centrale
- Protégé par `threading.Lock`
- Mise à jour uniquement par `Thread_State_Updater`
- Lecture depuis tous les threads

### Accès Wi-Fi distant

Le serveur Flask écoute sur `0.0.0.0` pour permettre la connexion depuis laptop/téléphone sur le réseau local.

**Sécurité** : en conditions live (festival, free party, réseau Wi-Fi non fiable) :
- **Authentification basique** : mot de passe dans un fichier de config (`/data/config/wifi_password.txt`)
- Flask HTTP Basic Auth ou JWT léger
- Pas de HTTPS (overhead complexe pour du local), mais authentification obligatoire
- Interface de gestion du mot de passe dans l'UI locale

**Multi-clients** :
- Plusieurs clients WebSocket simultanés possibles (laptop + téléphone + écran local)
- Broadcast des mises à jour AppState à tous les clients
- Pas de gestion explicite des conflits : "last write wins"
- Si un utilisateur tweake un paramètre depuis le téléphone et un autre depuis l'écran, la dernière action gagne (comportement naturel et acceptable)

---

## 4. Cycle de vie système

### Phase 1 : démarrage Teensy (t = 0)

Le Teensy boote instantanément (<1 seconde).

**Actions au démarrage** :
1. Initialisation GPIO (matrice pad, encodeurs, boutons)
2. Initialisation SPI (OLEDs voix) et I2C (OLED master, TCA9548A)
3. Initialisation audio (codec I2S, v2+)
4. Initialisation MIDI (UART)
5. Scan initial de l'état hardware
6. Allumage LEDs en mode "boot" (pattern animation)
7. Écriture "Booting..." sur les OLEDs voix
8. Démarrage du timer principal
9. Attente de `HELLO` du Pi
10. Timer interne : si pas de HELLO reçu en **10 secondes**, passage en **mode dégradé standalone**

### Phase 2 : démarrage Pi (t = 0 à 45s)

Le Pi boote en 30-45 secondes.

**Pendant ce temps, mode dégradé Teensy** :
- Fonctionnement limité sans Pi
- Encodeurs voix → CCs directs Perkons (mapping par défaut, voir `PROTOCOL_TABLES.md` section 10)
- Pad en mode "Drum Trigger" : 4 rangées = 4 voix, 8 colonnes = 8 niveaux de vélocité (16 à 127)
- Affichage OLED : "Booting... (NN s)"
- **L'instrument est jouable minimalement** même sans Pi

### Phase 3 : handshake (t ≈ 45s)

1. Pi démarre, charge son application Python
2. Pi ouvre le port série, envoie `HELLO` (0x01) avec version
3. Teensy reçoit `HELLO`, envoie `HELLO_ACK` (0x81) avec :
   - Version firmware + version protocole
   - Taille du snapshot à venir
4. Teensy envoie le snapshot complet (chunks via `SNAPSHOT_BEGIN` / `SNAPSHOT_CHUNK` / `SNAPSHOT_END`)
5. Pi compare le snapshot Teensy avec le projet qu'il s'apprête à charger :
   - Si hotcues dans le snapshot Teensy absents du projet Pi → Pi requête `REQUEST_HOTCUE_DATA` (0x26) pour chacun et les sauvegarde
   - Si paramètres modifiés dans le Teensy (mode dégradé) → Pi met à jour son AppState
6. Pi envoie la config complète au Teensy (modulations définies, pad mapping, hotcues à charger)
7. Teensy applique et confirme via `PARAM_VALUE` en bulk
8. Pi affiche l'UI en plein écran
9. Heartbeat régulier commence

**Durée totale handshake** : < 2 secondes après disponibilité Pi (hors transferts de samples).

### Phase 4 : runtime normal

- Communication bidirectionnelle
- Heartbeat toutes les 500 ms
- Autosave périodique
- Monitoring CPU / MEM / audio

### Phase 5 : perte de connexion Pi (runtime)

**Détection** : Teensy ne reçoit pas `HEARTBEAT` pendant 3 secondes.

**Comportement Teensy** :
- Continue à fonctionner avec l'état actuel en mémoire
- Modulations actives continuent
- Hotcues en cours continuent de jouer
- Encodeurs envoient CCs directement au Perkons
- Pad conserve son mapping actuel (pas de retour au fallback sauf si Clean Slate pressé)
- Affichage OLED : petit indicateur "⚠ UI" sur chaque voix
- Tentative de reconnexion : `HELLO` envoyé toutes les 2 secondes en IN série (au cas où le Pi revient)

**Le live continue.** Règle fondamentale en free party.

### Phase 6 : reconnexion Pi

Quand le Pi revient :
1. Pi redémarre, charge son app
2. Envoie `HELLO` au Teensy
3. Teensy répond avec snapshot de son état **actuel** (qui peut avoir divergé si l'utilisateur a joué pendant la déconnexion)
4. **Gestion de la divergence** :
   - Pi compare snapshot Teensy avec son projet en SD
   - Hotcues en PSRAM absents de SD → Pi lance `REQUEST_HOTCUE_DATA` pour les récupérer et sauvegarder
   - Paramètres modifiés par l'utilisateur en mode dégradé → Pi accepte l'état Teensy comme source de vérité
   - Dialog UI : "État hardware conservé. X hotcues récupérés."
5. Retour en runtime normal

### Phase 7 : perte de connexion Teensy

**Détection** : Pi ne reçoit pas `HEARTBEAT_ACK` pendant 3 secondes.

**Comportement Pi** :
- UI affiche banner "Hardware disconnected - reconnecting..."
- Conserve l'état actuel en mémoire
- Tentative de reconnexion toutes les 2 secondes

**Quand Teensy revient** :
- Handshake comme Phase 3
- Pi renvoie la config complète (Teensy a redémarré vierge)
- UI retourne à l'état normal

---

## 5. Interaction encodeur physique × modulation active

### Modes disponibles

**Mode A — Override permanent**
- Encodeur touché → modulation désactivée (`enabled = false`)
- L'utilisateur doit réactiver manuellement
- **Réactivation** : SHIFT + encodeur → remet `enabled = true`
- Feedback OLED : indicateur "MOD OFF" tant que désactivée

**Mode B — Override temporaire (DÉFAUT)**
- Encodeur touché → prend le contrôle pendant 3-5 secondes après le dernier mouvement
- Puis transition douce (fade 300ms) vers la valeur modulée courante
- Usage : tweak ponctuel sans casser la modulation

**Mode C — Merge additif (SHIFT + encodeur)**
- Valeur finale = valeur encodeur + contribution modulation
- L'encodeur définit le centre, la modulation oscille autour
- Voir règle de combinaison dans `PROTOCOL_TABLES.md` section 9

### Règles de combinaison

Voir `PROTOCOL_TABLES.md` section 9 pour détails complets (somme bornée, sources bipolaires, valeurs neutral).

### Priorité en cas de conflit

Si l'utilisateur touche l'encodeur (mode B, override actif) ET qu'une augmentation démarre au même moment :
- **L'override gagne pendant sa fenêtre**
- La modulation continue de calculer en interne mais n'affecte pas la sortie
- Après la fenêtre d'override, fade vers `override_value + combined_mod`

Si le Pi envoie `SET_PARAM` pendant que l'utilisateur tourne l'encodeur (override_active = true) :
- **L'encodeur physique gagne**
- `SET_PARAM` est ignoré
- Le Pi est notifié via `WARNING` (code = "SET_PARAM_IGNORED_OVERRIDE_ACTIVE")

### Comportement du fade en mode B

Pendant le fade après override :
- Le fade interpole de `override_value` (fixe) vers `modulated_value(t)` (qui bouge en temps réel)
- Résultat : trajectoire animée, l'utilisateur voit la valeur converger vers la modulation en mouvement
- Durée : 300 ms par défaut, configurable par paramètre

### Feedback visuel

**En mode B, modulation active** :
- LED colonne voix : pulse lent de la couleur du mode (bleu pour LFO, orange pour augmentation, etc.)

**En mode B, encodeur pris en override** :
- LED colonne voix : flash blanc puis retour au pulse
- Indication "OVERRIDE" sur l'OLED voix pendant le délai

**En mode C activé (SHIFT maintenu)** :
- LED différente (jaune)
- Indication "MERGE" sur l'OLED voix

### Contrainte d'usage Pot Catch Perkons

**Contrainte matérielle** : le Pot Catch du Perkons HD-01 (firmware v1.2) ne fonctionne pas comme documenté. Un pot physique touché saute directement à sa position matérielle, ignorant la valeur CC courante.

**Conséquence pour PërKompanion** : si le Teensy module Tune V1 à la valeur 80 et que l'utilisateur touche le pot Tune du Perkons (physiquement à 40), le pot "reprend" à 40, créant un saut audible.

**Règle documentée** : **quand PërKompanion pilote un paramètre d'une voix, ne pas toucher le pot Perkons correspondant**. Documenter cette contrainte clairement dans le manuel utilisateur.

Aucune solution technique n'est possible sans feedback du Perkons (qui n'envoie pas d'info de pot touché en MIDI). À vivre comme contrainte d'usage.

### Panic, Freeze, REC — sémantique

**Panic voix V** (bouton 0x0200/0x0210/0x0220/0x0230)
- All Notes Off (CC 123) + All Sound Off (CC 120) sur le canal V
- Arrête immédiatement les sons en cours de la voix sans toucher aux autres voix
- Les modulations continuent (paramètres restent modulés)

**Panic global** (bouton 0x0400)
- Panic sur les 4 voix simultanément
- Arrêt de toutes les augmentations en cours
- Arrêt de tous les hotcues en cours (v3+)

**Freeze voix V** (bouton 0x0201/0x0211/0x0221/0x0231)
- Désactive toutes les modulations actives sur les paramètres de V (équivalent Mode A override sur tous les params de la voix)
- Les valeurs CC actuelles sont gelées
- Rappui sur Freeze → dégèle (réactive les modulations)

**Freeze global** (bouton 0x0401)
- Freeze sur les 4 voix simultanément

**Clean Slate** (bouton 0x0404)
- Stop toutes modulations/augmentations/hotcues en cours
- Ne modifie pas les patterns ou kits Perkons
- Voir section "Bouton Clean Slate" dans VISION

**REC** (bouton 0x0410)
- **En v1 (sans audio)** : arme l'enregistrement des **actions utilisateur** (encodeurs, pads, boutons) horodatées au step/beat. Replayable via PLAY.
- **En v2+ (avec audio)** : peut aussi enregistrer l'audio master sur SD (fichier WAV)
- Arme / désarme par appui court
- LED REC clignote quand armé, fixe quand enregistre, éteinte sinon

**PLAY** (bouton 0x0411)
- Démarre le replay des actions utilisateur enregistrées
- LED PLAY clignote au tempo pendant le playback

---

## 6. Accélération encodeur

### Algorithme

Voir `PROTOCOL_TABLES.md` section 5 pour le format de message ENCODER (qui inclut le multiplicateur d'accélération).

**Implémentation côté Teensy** :

```cpp
struct EncoderState {
  uint32_t last_tick_us;
  int8_t last_direction;
};

int8_t compute_accelerated_delta(EncoderState* e, int8_t physical_delta) {
  uint32_t now = micros();
  uint32_t dt = now - e->last_tick_us;
  e->last_tick_us = now;
  
  uint8_t multiplier;
  if (dt < 10000) multiplier = 8;
  else if (dt < 30000) multiplier = 4;
  else if (dt < 80000) multiplier = 2;
  else multiplier = 1;
  
  return physical_delta * multiplier;
}
```

### Configuration par paramètre

Chaque `param_id` a un profil d'accélération stocké dans une table configurable :

| Profil | Multiplicateurs [lent, moyen, rapide, très rapide] | Usage typique |
|--------|---------------------------------------------------|----------------|
| None | 1, 1, 1, 1 | Précision fine (pitch semitones) |
| Low | 1, 2, 2, 1 | Peu d'accélération |
| Medium | 1, 2, 4, 1 | Standard |
| High | 1, 4, 8, 1 | Plages larges (cutoff, time) |
| Custom | défini par utilisateur | Cas spéciaux |

La configuration par paramètre est sauvegardée dans les projets.

### Implémentation

**Côté Teensy uniquement.** Le Pi ne voit pas les deltas physiques, seulement les deltas accélérés.

Le message `ENCODER` (0xA0) envoyé au Pi contient :
- encoder_id
- delta (déjà accéléré)
- acceleration_mult (pour info UI si souhait d'afficher)

### Feedback visuel

L'OLED voix peut afficher temporairement un indicateur "×8" quand l'encodeur est en accélération forte, pour que l'utilisateur comprenne les sauts.

---

## 7. Courbes de mapping par paramètre

### Principe

Les valeurs des encodeurs (normalisées 0.0-1.0) sont transformées par une courbe avant d'être envoyées en MIDI CC ou vers les paramètres DSP.

### Courbes disponibles

```python
curves = {
    'linear':     lambda x: x,
    'exp':        lambda x: x ** 2,
    'exp_strong': lambda x: x ** 3,
    'log':        lambda x: sqrt(x),
    'log_strong': lambda x: x ** 0.33,
    'scurve':     lambda x: smooth_step(x),
    'inverse':    lambda x: 1 - x,
    'custom':     lambda x: lookup_table[int(x*255)],
}
```

### Courbes par défaut par paramètre

Voir `PROTOCOL_TABLES.md` section 2 (colonne "Courbe" des tables param_id).

**Récapitulatif Perkons** :
- Tune : linear
- Decay : exp
- Param 1/2 : linear (dépend du mode algo)
- Cutoff : log_strong
- Drive : scurve
- FX Send : exp
- Level : exp

### Configuration utilisateur

Chaque paramètre peut avoir sa courbe redéfinie via l'UI écran. Sauvegardé avec le projet.

Mode "Custom" : l'utilisateur dessine sa courbe au doigt sur l'écran tactile (128 points interpolés), stockée comme table de lookup 256 valeurs, envoyée au Teensy via `SET_PARAM_CURVE` (0x17).

### Implémentation

Les courbes sont appliquées **côté Teensy**, au moment de convertir la valeur interne vers CC ou DSP.

Optimisation : précalculer les courbes dans des tables de lookup 256 entrées au démarrage. Accès O(1) en runtime.

---

## 8. Politique d'autosave (hotcues et état)

### Stratégie à trois niveaux

**Niveau 1 : autosave événementielle (immédiate)**

Déclenchée par événements critiques :
- **Capture d'un hotcue** : `HOTCUE_CAPTURED` (0xC1) reçu du Teensy → Pi initie `REQUEST_HOTCUE_DATA` (0x26) → sauvegarde SD dans les 500ms
- **Création d'une modulation** : sauvegarde immédiate du projet JSON
- **Sauvegarde manuelle demandée** : écriture complète

**Niveau 2 : autosave périodique (safety net)**

Toutes les **3 minutes** :
- Snapshot complet de l'état
- Fichier horodaté dans `/data/autosave/session_YYYYMMDD_HHMMSS.json`
- Rotation : conserver les 10 derniers

**Niveau 3 : autosave opportuniste**

Déclenchée par changements structurels :
- Changement de kit Perkons
- Changement de pattern
- Création/suppression augmentation
- Toute action irrécupérable

### Mécanisme de transfert capture

Le Teensy capture un hotcue → immédiatement jouable (en PSRAM). Le transfert vers Pi se fait en background :

1. Teensy envoie `HOTCUE_CAPTURED` (0xC1)
2. Pi initie transfert : `REQUEST_HOTCUE_DATA` (0x26)
3. Teensy envoie chunks `HOTCUE_CHUNK` (0xC2) à priorité niveau 3
4. Pi écrit incrémentalement sur SD
5. `HOTCUE_CHUNK_END` (0xC3) avec checksum
6. Pi confirme et ajoute le hotcue au projet courant

**Pendant le transfert** : le hotcue est **déjà jouable**. Le transfert ne bloque rien.

### Structure de fichier autosave

```json
{
  "version": "0.6",
  "timestamp": "2026-04-22T14:35:12Z",
  "project_name": "Live session Dissay",
  "perkons_state": {
    "kit_active": 17,
    "pattern_active": 5,
    "bpm": 140
  },
  "modulations": [...],
  "pad_mapping": {...},
  "hotcues": [
    {
      "id": "v2_05",
      "voice": 2,
      "slot": 5,
      "audio_file": "audio/v2_05_captured.wav",
      "start_steps": 0,
      "length_steps": 16,
      "pitch": 0,
      "feedback": 60,
      "loop_mode": "continuous",
      "quantize": "1/16"
    }
  ],
  "dsp_fx_chains": [...],
  "arpeggiator_states": [...],
  "cpu_snapshot": {...}
}
```

### Recovery au démarrage

Si un fichier autosave est plus récent que la dernière sauvegarde manuelle :

```
┌─────────────────────────────────────────┐
│  Session interrompue détectée           │
│                                         │
│  Dernier autosave : il y a 8 minutes    │
│  2 hotcues non sauvegardés manuellement │
│                                         │
│   [RECUPÉRER]    [IGNORER]              │
└─────────────────────────────────────────┘
```

L'utilisateur choisit. Par défaut, proposer récupération.

---

## 9. Budget CPU et dégradation progressive

### Cibles

| Version | CPU nominal | CPU pic autorisé |
|---------|------------|------------------|
| v1 (pas de DSP) | <20% | <40% |
| v2 (DSP par voix) | <55% | <75% |
| v3 (+ hotcues basique + modulation hotcue) | <65% | <80% |
| v4 (+ séquenceurs hotcue + arp hotcue) | <75% | <90% |

**Marge de 15-20% en pic** pour absorber les pics transitoires sans xrun.

### Dégradation progressive simple

Règle unique, légère :

```cpp
// Appelée toutes les 100 ms
void check_cpu_usage() {
  if (cpu_usage > 85) {
    Hotcue* oldest = find_oldest_playing_hotcue();
    if (oldest && oldest->has_fx_chain) {
      disable_fx_chain(oldest);
      send_warning_to_pi(WARN_CPU_DEGRADATION, oldest->id);
    }
  }
}
```

**Pas de logique complexe**, pas de prédictif. Une action simple, répétable, réversible (l'utilisateur peut rallumer la chain).

Coût monitoring : <0.1% CPU.

### Approche "Monitor & React"

- xrun détecté → log + warning UI
- CPU > 85% sur 1 seconde → dégradation
- CPU redescend sous 70% → les FX chains désactivées peuvent être rétablies par l'utilisateur (jamais automatiquement)

### Monitoring CPU par effet

Chaque objet DSP du Teensy enregistre son temps d'exécution. Envoi régulier via `CPU_USAGE` (0xD0) :

```
[total_percent (1)][peak_percent (1)][effect_count (1)]
[effect_id (2) + usage_percent (1)] × effect_count
```

Le Pi affiche la décomposition par effet, avec code couleur par coût (vert/jaune/orange/rouge).

---

## 10. Hybrid Kit Mode et Perkons Snapshots

Ces deux features sont des piliers de la v1 et méritent leur propre section protocolaire.

### Voice Kits (Hybrid Kit Mode)

**Définition** : un Voice Kit est un ensemble des 11 valeurs CC pour une voix Perkons (Tune, Param1, Cutoff, FX_Send, Decay, Param2, Drive, Level, Algo, Mode, Filter), mémorisé comme preset sur SD, rappelable via MIDI en temps réel.

**Ce n'est pas un "kit Perkons natif"** (qui contient plus de paramètres non accessibles en MIDI, comme les paramètres Master). C'est un snapshot des 11 CCs contrôlables.

**Stockage côté Pi** : dossier `/data/voice_kits/`, un fichier JSON par kit.

```json
{
  "kit_id": 17,
  "name": "Dark Kick 1",
  "voice_reference": 1,
  "cc_values": {
    "tune": 48, "param1": 100, "cutoff": 32, "fx_send": 80,
    "decay": 64, "param2": 20, "drive": 120, "level": 100,
    "algo": 2, "mode": 1, "filter": 0
  }
}
```

**Workflow de création** :
1. Utilisateur configure une voix (V2) à son goût via le Perkons et/ou PërKompanion
2. Combo `SHIFT + Load + pad cible` → Pi reçoit un BUTTON_DOWN + contexte, envoie `REQUEST_SNAPSHOT` (scope=voice 2)
3. Teensy renvoie les 11 valeurs CC actuelles de V2 dans le snapshot
4. Pi sauvegarde en JSON via logique interne, et envoie `SAVE_VOICE_KIT` au Teensy pour référence
5. Le kit_id et le nom sont associés au slot pad cible

**Workflow de rappel (hot-swap)** :
1. Utilisateur en mode pad = LoadVoiceKit (via M4 ou autre)
2. Voix active = V2
3. Tap sur touche pad qui a un LoadVoiceKit binding → `LOAD_VOICE_KIT(voice=2, kit_id=17, quantize=1_bar)`
4. Le Teensy stocke le kit en attente
5. Au prochain step 1 de barre, le Teensy envoie les 11 CCs en burst rapide (tous en <5ms) sur le canal 2
6. V2 change de "kit" sans coupure audio

**Commandes** : voir `PROTOCOL_TABLES.md` bloc 0x90-0x9F.

### Perkons Snapshots

**Définition** : un Perkons Snapshot est identique à un Voice Kit mais sur les **4 voix simultanément** (44 CCs au total).

**Workflow** : identique, mais rappel envoie les 44 CCs au next bar sur les 4 canaux.

**Usage typique** : mémoriser un "moment d'inspiration" (tous les params bien réglés), le rappeler plus tard pendant le set.

### Pot Catch Perkons — contrainte rappelée

Quand PërKompanion pilote des paramètres Perkons (via Voice Kits, Snapshots, ou modulations), **ne pas toucher les pots physiques du Perkons correspondants**. Le Pot Catch ne fonctionne pas comme documenté en firmware v1.2 et créera des sauts.

---

## 11. Voix virtuelles V5-V8, polyphonie hotcues et motions

### Principe des voix virtuelles

Les voix V5-V8 sont **physiquement représentées sur le panneau** avec la même structure que V1-V4 (bouton voix tête + OLED + 8 encodeurs). À la différence de V1-V4 qui pilotent les 4 voix Perkons via MIDI CC, **V5-V8 pilotent des hotcues assignés dynamiquement** (samples stockés en PSRAM Teensy et joués par le DSP interne).

### Règles d'assignation hotcue → voix virtuelle

**Workflow utilisateur** :
1. Pad en mode M3 (Hotcue) — les 32 pads affichent 16 hotcues par page (via pagination V1-V4 en mode pad)
2. Maintenir le bouton voix tête de V5 (ou V6/V7/V8)
3. Tap sur un pad hotcue → hotcue assigné à cette voix virtuelle
4. L'OLED V5 affiche le numéro (00-127) et le nom du hotcue
5. Les encodeurs de V5 pilotent immédiatement les paramètres du hotcue assigné

**Commande protocole** : `ASSIGN_HOTCUE_TO_VIRTUAL_VOICE` (0x2A)
```
hotcue_id (2 bytes) + virtual_voice_id (1 byte, 5-8)
```

**Désassignation** : combo bouton voix tête V5 + long press sur pad du hotcue OU `UNASSIGN_VIRTUAL_VOICE` (0x2B) depuis l'écran.

### Persistance des modulations lors de réassignation

**Principe** : les modulations (LFO, DSP, FX sends, etc.) attachées à **la colonne voix V5** restent en place même si le hotcue change.

**Exemple** :
- V5 a un LFO sur son Pitch + un Filter Cutoff + un Reverb Send actifs
- Tu réassignes un autre hotcue à V5
- Les modulations sont **toujours là**, appliquées au nouveau hotcue
- Seul le sample source change

**Implémentation** : les modulations sont stockées dans la **structure de la voix V5** (VoiceState), pas dans la structure du hotcue. Le hotcue n'est qu'une source de données (sample + paramètres de base).

```cpp
struct VirtualVoiceState {
    uint8_t voice_id;                    // 5-8
    uint16_t assigned_hotcue_id;         // 0-127 ou 0xFFFF si rien
    Modulations active_modulations;      // LFO, Arp, etc.
    DSPChain dsp_chain;                  // Filter, Drive, FX
    EncoderMode current_mode;            // LFO/DSP/Arp/Hotcue
    // etc.
};
```

### Pagination hotcues au pad en mode Hotcue

**Mode pad M3 (Hotcue)** : 8 pages de 16 hotcues = 128 hotcues totaux accessibles.

**Navigation** :
- Les boutons V1-V4 deviennent des **paginateurs** en mode M3 :
  - V1 = pages 1-2 (hotcues 0-31)
  - V2 = pages 3-4 (32-63)
  - V3 = pages 5-6 (64-95)
  - V4 = pages 7-8 (96-127)
- Short press V1 → page 1 (hotcues 0-15)
- Double tap V1 → page 2 (hotcues 16-31)

**Alternative plus simple** : V1-V8 = 8 pages de 16 hotcues chacune (tap pour sélectionner la page).

**Commande protocole** : `SET_PAD_HOTCUE_PAGE` (0x78)
```
page (1 byte, 0-7)
```

**LEDs des pads** sur la page active :
- Gris : slot vide (hotcue non chargé)
- Bleu : hotcue chargé mais non actif
- Blanc : hotcue actif (en train de jouer)
- Violet : hotcue locké (protégé de l'auto-steal)
- Orange : hotcue assigné à une voix virtuelle (V5-V8)

### Lock des hotcues

**Concept** : protéger certains hotcues essentiels (nappe ambient, drone permanent) de la coupure automatique quand la polyphonie est atteinte.

**Workflow** :
- Long press sur le pad d'un hotcue en mode M3 → menu écran avec option "Lock"
- LED pad devient violette
- Re-long press pour délocker

**Commande protocole** : `SET_HOTCUE_LOCK` (0x29)
```
hotcue_id (2 bytes) + locked (1 byte, 0 ou 1)
```

**Effet algorithmique** : voir section suivante.

### Gestion de la polyphonie hotcues

**Paramètre système** : `System.MaxSimultaneousHotcues` (param_id 0x5030)
- Range : 1-32
- Default : 16
- **Ajustable en direct** via encodeur dédié dans la zone master

**Algorithme de coupure automatique** :
```
Quand un nouveau hotcue est déclenché :
  Si count(hotcues_actifs) < polyphonie_max :
    -> Démarrer le nouveau hotcue
  Sinon :
    Trouver le hotcue le plus ancien NON-LOCKÉ
    Si trouvé :
      -> Arrêter ce hotcue (avec fade-out rapide 10ms pour éviter clic)
      -> Démarrer le nouveau hotcue
      -> Envoyer HOTCUE_STEAL_NOTIFICATION au Pi pour log UI
    Sinon (tous lockés) :
      -> Refuser le nouveau hotcue
      -> Envoyer WARNING_POLYPHONY_SATURATED au Pi
      -> LED du pad du nouveau hotcue clignote rouge brièvement (feedback)
```

**Commande protocole** :
- `SET_POLYPHONY_LIMIT` (0x2E) : `limit (1 byte)`
- `HOTCUE_STEAL_NOTIFICATION` (0x2F) : notification Teensy → Pi quand un hotcue est coupé automatiquement

### REC de motions (v2+)

Les motions sont les **mouvements d'encodeurs capturés en live** pendant qu'on maintient le bouton REC. Concept inspiré du **parameter lock live recording** du Perkons.

**Workflow** :
1. Maintenir bouton **REC** (main gauche)
2. Tourner un ou plusieurs encodeurs (main droite)
3. PërKompanion capture chaque changement avec timestamp synced au BPM
4. Relâcher REC → motions enregistrées dans la session live

**Philosophie** :
- Les motions capturées sont **principalement destinées au live**, pas au save global
- Elles tournent en loop pendant la session, écrasables, effaçables
- **Pas de "save multi-motion global"** : n'aurait pas de sens hors contexte live

**Sauvegarde individuelle** :
- Une motion sur un encodeur peut être **extraite** via UI écran et sauvegardée dans la bibliothèque
- Commande : `MOTION_SAVE` (0xA3)

**Assignation à un slot Augmentation** :
- Une motion sauvegardée peut être assignée à un slot du pad
- Déclenchement du slot = playback de la motion
- Commande : `MOTION_ASSIGN_TO_AUG_SLOT` (0xA8)

**Longueur des motions** :
- Par défaut : **16 steps** (aligné Perkons)
- Modifiable : 32 ou 64 steps via `MOTION_SET_LENGTH` (0xAA)
- Quantize 1/16 par défaut (désactivable)

**Interpolation** :
- Linear (par défaut)
- Step (pas d'interpolation)
- Smooth (lissage exponentiel)
- Exponential

### Implémentation côté Teensy

**Capture de motion** :
```cpp
struct MotionEvent {
    uint32_t timestamp_ticks;  // position dans la séquence
    uint8_t voice_id;
    uint16_t param_id;
    int16_t value;  // normalisé
};

class LiveMotionRecorder {
    bool rec_active = false;
    std::vector<MotionEvent> current_session;

    void on_rec_pressed() {
        rec_active = true;
        session_start_tick = current_clock_tick;
    }

    void on_encoder_change(uint8_t voice, uint16_t param, int16_t value) {
        if (rec_active) {
            MotionEvent ev = {current_clock_tick - session_start_tick, voice, param, value};
            current_session.push_back(ev);
            // Envoi au Pi pour UI temps réel
            send_motion_event(ev);
        }
    }

    void on_rec_released() {
        rec_active = false;
        // Les motions tournent en loop pendant la session
        // Pas de sauvegarde automatique
    }
};
```

**Playback de motion assignée à un slot** :
```cpp
void on_pad_pressed(uint8_t pad_id) {
    Binding binding = get_binding(pad_id);
    if (binding.action_type == ACTION_TRIGGER_MOTION) {
        Motion motion = load_motion(binding.motion_id);
        start_motion_playback(motion, binding.target_voice, binding.target_param);
    }
}
```

### Interaction des voix virtuelles avec la polyphonie

**Particularité** : quand un hotcue est assigné à V5 et qu'il est déclenché (via le pad ou via V5), il compte **comme un hotcue actif** pour la polyphonie globale.

**Règle** : **les hotcues assignés à une voix virtuelle sont automatiquement lockés** (protégés de l'auto-steal).

Pourquoi : si tu as assigné un hotcue à V5 pour le jouer en live avec ses encodeurs, tu ne veux **pas** qu'il soit coupé automatiquement parce que tu as déclenché 16 autres hotcues sur le pad. Les 4 voix virtuelles consomment donc 4 slots de polyphonie **garantis** (sauf si l'utilisateur delock manuellement).

**Conséquence** : polyphonie effective pour les hotcues "libres" = `polyphonie_max - nb_voix_virtuelles_actives`.

Si polyphonie_max = 16 et 4 voix virtuelles assignées et actives : 12 hotcues peuvent être déclenchés en plus sur le pad.

### Mode Step Edit (M5)

**Nouveau mode pad** : M5 (ajouté à M1-M4 existants).

**Fonction** : édition du step sequencer d'un hotcue.

**Activation** :
- Tap M5 (bouton dédié en colonne gauche du pad)
- Ou depuis l'écran tactile lors de l'édition d'un hotcue (bouton "Step Edit" dans l'UI)

**Affichage au pad** :
- Les 32 pads affichent les 16 ou 32 steps du hotcue sélectionné
- LEDs : on/off selon step actif
- Couleur : indicateur de vélocité ou probability

**Édition** :
- Tap : toggle on/off
- Long press : sélectionne le step pour édition détaillée
- Paramètres du step affichés à l'écran : vélocité, probability, ratchet, pitch_mod, tie
- Ajustement via encodeurs contextuels écran (E1-E8)

**Commandes protocole** :
- `ENTER_STEP_EDIT_MODE` (0x76) : `hotcue_id (2)`
- `EXIT_STEP_EDIT_MODE` (0x77)
- `SET_HOTCUE_STEP` (0x27) : pour modifier un step spécifique

---

## 12. Contraintes hardware et validation

### I2C vs SPI pour les OLEDs voix

**Problème identifié en v0.6** : 4 OLEDs 128×64 sur bus I2C 400 kHz = goulot bandwidth. Full refresh à 10 fps demande 160 KB/s, I2C à 400 kHz ne fournit que ~50 KB/s.

**Décision** : les 4 OLEDs voix sont en **SPI** (pas I2C).
- Chaque OLED a son CS (chip select) GPIO Teensy
- MOSI, SCK, DC, RST partagés
- Refresh 50+ fps possible, dirty rect optionnel

L'OLED master BPM reste en I2C (taille plus petite, refresh peu fréquent, via TCA9548A).

### Validation hardware à faire en Phase 1

Avant d'écrire la logique applicative, valider les briques de base :

1. **Test MIDI clock** : clock FL → ESI → Teensy MIDI IN. Mesure jitter <1 ms via oscilloscope ou LED flash sur chaque tick.
2. **Test MIDI OUT Teensy** : Teensy génère CC test → ESI → Perkons. Vérifier arrivée sans transit Pi.
3. **Test 4 OLEDs SPI** : afficher "V1", "V2", "V3", "V4" simultanément sur les 4 OLEDs.
4. **Test I2C master OLED** : afficher "BPM 140" sur l'OLED 0.96" via TCA9548A.
5. **Test scan matrice pad** : toutes les 32 touches détectées sans ghosting.
6. **Test encodeurs** : 16 encodeurs voix détectés avec acceleration.
7. **Test USB série Pi ↔ Teensy** : HEARTBEAT bidirectionnel, 1000 msg/s sans perte.
8. **Test NeoPixel** : 32 LEDs du pad + 16 indicateurs contrôlés individuellement.

Chaque test est un "Hello World" matériel. Si l'un échoue, debugger avant de continuer.

### Test rig minimal

Préparer dès Phase 1 un **sketch Arduino minimaliste** :

```cpp
// test_midi_clock.ino
void setup() {
  MIDI.begin(MIDI_CHANNEL_OMNI);
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  if (MIDI.read()) {
    if (MIDI.getType() == midi::Clock) {
      digitalWrite(LED_BUILTIN, HIGH);
      delayMicroseconds(100);
      digitalWrite(LED_BUILTIN, LOW);
    }
  }
}
```

Tester à différents BPM, mesurer au scope.

---

## 13. Checklist de démarrage du développement

Avant d'écrire la première ligne de code applicatif, s'assurer que ces points sont validés :

### Hardware et routing

- [ ] Matériel commandé et reçu
- [ ] Routing MIDI ESI configuré et verrouillé (config sauvegardée)
- [ ] Clock MIDI arrive au Teensy direct (jitter <1ms validé)
- [ ] CCs sortants Teensy vont au Perkons sans transit Pi (test scope)
- [ ] 4 OLEDs voix en SPI fonctionnelles
- [ ] OLED master I2C fonctionnelle via TCA9548A
- [ ] Matrice pad 8×4 scanne sans ghosting
- [ ] 32 encodeurs détectés
- [ ] NeoPixel 32 LEDs contrôlables individuellement

### Protocole et communication

- [ ] Baud rate 2 Mbps configuré
- [ ] Architecture multi-thread Pi implémentée
- [ ] Heartbeat bidirectionnel fonctionnel
- [ ] Détection déconnexion (3 heartbeats manqués ou 3s timeout) testée
- [ ] Tables de référence (PROTOCOL_TABLES.md) implémentées des deux côtés
- [ ] Endianness little endian validée

### Logique applicative

- [ ] Mode dégradé boot Teensy fonctionnel (fonctionne sans Pi)
- [ ] Handshake Pi/Teensy validé avec échange de SNAPSHOT
- [ ] Accélération encodeur implémentée côté Teensy
- [ ] Courbes de mapping définies pour tous les paramètres Perkons
- [ ] Interaction encodeur × modulation : mode B par défaut implémenté
- [ ] Règles de combinaison de modulations implémentées
- [ ] Position-in-bar maintenue correctement sync MIDI clock

### Fiabilité

- [ ] Autosave événementielle fonctionnelle dès v1
- [ ] Autosave périodique (3 min) configurée
- [ ] Recovery au démarrage testée
- [ ] Budget CPU monitoré dès la v1
- [ ] Dégradation progressive testée (forcer surcharge artificielle)

### Features v1

- [ ] Clean Slate implémenté avec protection appui long
- [ ] Pad mapping par défaut mode dégradé fonctionnel
- [ ] Wi-Fi access avec authentification basique
- [ ] Tap Tempo en mode standalone

---

## 14. Historique des décisions v0.4 → v0.8

Les points suivants sont issus de relectures successives par un Claude fresh-context. Chacun a été identifié comme un angle mort qui aurait causé des bugs difficiles à diagnostiquer.

**Passes v0.4 → v0.5 (2 passes)** : protocole USB binaire, heartbeat bidirectionnel, budget CPU v4, jacks d'insert, clock MIDI direct Teensy, interaction encodeur × modulation, thread série Pi, autosave hotcues, Clean Slate, routing CCs, boucle MIDI, accélération encodeur, courbes mapping, synchronisation démarrage.

**Passes v0.5 → v0.6 (2 passes)** :
- Tables d'IDs complètes (param_id, encoder_id, pad_id, button_id, led_id)
- Numéros CC Perkons précis (70-80 V1, 81-91 V2, etc.)
- Canal MIDI Multi mode choisi (1-4 par voix)
- Commandes de chunking complétées (LOAD_HOTCUE_START/CHUNK/END, SNAPSHOT_*)
- Endianness explicitée (little endian)
- Priorisation 3 niveaux dans thread série
- Position-in-bar et quantification spécifiées
- Règles de combinaison multi-modulations (somme bornée)
- Priorités override vs modulation vs SET_PARAM
- Clock en standalone : Teensy génère, n'écoute pas son retour
- OLEDs voix passées en SPI (résout goulot I2C)
- Contrainte Pot Catch Perkons documentée
- Seuil heartbeat augmenté à 3s
- Reconnexion Pi : gestion de divergence
- Mapping pad par défaut mode dégradé précisé
- Sécurité Wi-Fi (auth basique)
- Nouvelles commandes : SET_ARP_STEP, LOAD_AUG_CURVE, LOAD_PAD_LAYOUT, SET_PARAM_CURVE
- Format snapshot détaillé

**Passe v0.6 → v0.7 (1 passe finale)** :
- **Encoding des valeurs 2 bytes formalisé** (int16 signed LE, mapping normalisé -1..+1 ou 0..1)
- **Voice Kits et Perkons Snapshots** : commandes 0x90-0x96 ajoutées, workflows documentés
- **Probability/Ratchet** : feature repensée — impossible via MIDI sur Perkons (hardware only), donc appliquée aux augmentations/hotcues/arps générés par PërKompanion
- **action_type énuméré** pour SET_PAD_BINDING (0x00-0x0D)
- **Codes WARNING et ERROR** énumérés avec data format
- **Chunking LOAD_PAD_LAYOUT** (commandes 0x73-0x75 pour layouts > 21 bindings)
- **Tie ajouté** à SET_HOTCUE_STEP
- **UNLOAD vs DELETE hotcue** clarifiés
- **Enum curves** : `none` pour paramètres énumérés (Algo, Mode, Filter, etc.)
- **Type de paramètre** explicité : continuous, continuous bipolar, enum, integer, boolean
- **Panic/Freeze par voix** définis (All Notes Off + All Sound Off / gel des paramètres)
- **REC en v1** : enregistre les actions utilisateur (pas d'audio avant v2)
- **Ordre cycle MODE corrigé** : LFO → DSP → Arp → Hotcue (alignement VISION)
- **"132 params" corrigé** : 11 CCs par voix × 4 = 44 paramètres Perkons
- **Documents hardware ajoutés** : teensy_pinout.md et midi_hardware.md
- **OLED V3/V4 CS** : correction conflit avec I2C, pins 22/23 en v1
- **CONFIG_PUSH_ACK** : ACK pour confirmer fin de push config au handshake

**Passe v0.7 → v0.8 (refonte architecturale majeure)** :
- **Architecture 4+4 voix** : ajout de 4 voix virtuelles (V5-V8) physiquement représentées sur le panneau, pilotent des hotcues assignés dynamiquement
- **Bibliothèque hotcues globale** : 128 hotcues (au lieu de 32 par voix), paginés 8 pages × 16 via boutons V1-V8 en mode pad Hotcue
- **Assignation hotcue → voix virtuelle** : combo voix tête V5-V8 + tap pad (commandes 0x2A/0x2B)
- **Lock hotcues** : protection contre auto-steal (commande 0x29 SET_HOTCUE_LOCK)
- **Polyphonie ajustable en direct** : encodeur dédié zone master (param 0x5030, commande 0x2E SET_POLYPHONY_LIMIT)
- **Auto-steal notification** : commande 0x2F HOTCUE_STEAL_NOTIFICATION (Teensy → Pi pour log UI)
- **Voix virtuelles auto-lockées** : les 4 hotcues assignés à V5-V8 comptent dans la polyphonie mais sont auto-protégés
- **Motions REC** : concept inspiré du Perkons parameter lock live recording
  - Maintenir REC + tourner encodeur = capture
  - Nouveau bloc commandes 0xA0-0xAC (11 commandes motions)
  - Sauvegarde individuelle possible (MOTION_SAVE 0xA3)
  - Assignable à slot Augmentation (MOTION_ASSIGN_TO_AUG_SLOT 0xA8)
  - Par défaut 16 steps, quantize 1/16
- **Action type 0x0E TriggerMotion** ajouté à SET_PAD_BINDING
- **Mode pad M5 Step Edit** : nouveau mode pour édition step sequencer hotcue
- **Layout panneau 45×37cm** : +2cm en hauteur vs Perkons strict pour accueillir 8 colonnes voix
- **Écran 7" Elecrow raw panel** : au lieu de 10.1" avec coque (intégration mécanique propre via trous de fixation)
- **OLEDs voix SH1107 128×128** : au lieu de SH1106 128×64 (2× plus de pixels, même bande passante SPI)
- **8 encodeurs par colonne voix** : 1 MODE + 1 USER + 6 contextuels (au lieu de 4)
- **Appui+rotation sur encodeurs contextuels** : accès à une fonction secondaire sans changer de mode
- **Bouton CLEAN SLATE ajouté** aux globaux : 5 boutons au total (SHIFT, PANIC, FREEZE, CLEAN SLATE, REC)
- **Boutons globaux placés dans la bande centrale** (entre pad et voix) pour accès live rapide
- **Zone master sous l'écran, à droite des colonnes voix** (pas à droite de l'écran)
- **Casque en façade** avec encodeur volume dédié (bas-droite panneau)
- **Document panel_design.md** créé : dimensions exactes pour dessin AutoCAD et usinage FabLab

**Passe v0.8 initial → v0.8 final (pivot architecture audio)** :

Rollback du choix PCM3168A pour raison de **constructibilité solo**. Le PCM3168A (HTQFP-64 10×10mm pitch 0.5mm avec pad thermique exposé) demande :
- Station à air chaud et expérience CMS fine
- Pas de module breakout fiable sur le marché (TI a discontinué l'EVM officielle)
- Apprentissage KiCad (1-2 semaines) + 2-3 respins PCB JLCPCB PCBA sur 2-3 mois

**Pivot** : architecture modulaire à base de breakouts **PCM1808 (ADC stéréo) + PCM5102A (DAC stéréo)** + **TPA6120 module MCU-612 (ampli casque)**.

**Décisions actées v0.8 final** :
- **2× PCM1808 breakout** (4 canaux in Perkons) via SAI1 RX
- **6× PCM5102A breakout** (12 canaux out = 6 stéréos) via SAI1 TX (4 DAC) + SAI2 TX (2 DAC)
- **1× TPA6120 module MCU-612** pour ampli casque dédié (700mW/32Ω, façade)
- **Anticipation finale sur la plate** : Master L+R + V1-V4 stéréo + V5-V8 mixées stéréo + casque = 13 jacks sortie + 4 jacks entrée Perkons + 2 DIN MIDI + 3 digital = 22 connecteurs
- **DSP par voix modulable** : 4 slots reconfigurables parmi 13 types d'effets (Filter, Distortion, Bitcrusher, Chorus, Phaser, Reverb, Delay, Compressor, EQ, Ring Mod, AutoPan, Tremolo, LoFi) + sends master
- **Bloc param_id 0x2000-0x2FFF** : 28 params par voix × 8 voix = 224 params DSP modulables au total
- **Matrice pad déplacée sur MCP23S17** (libère pins 2-9 Teensy pour audio SAI1/SAI2)
- **20 MCP23S17 DIP-28** (au lieu de 15) sur 5 perfboards, 3 chaînes SPI (CS pins 24, 25, 26)
- **Pinout Teensy clean** : chaque pin a une fonction dédiée, aucun déplacement forcé par conflit
- **Enregistrement via Tascam uniquement** (pas de Pi 5 USB audio gadget)
- **ICs critiques sur TME** (MCP23S17-E/SP authentiques, pas AliExpress pour éviter contrefaçon)
- **Jacks Neutrik** pour sorties audio (qualité pro, durabilité 10+ ans)
- **Budget v0.8 final** : ~1065-1135€ avec anticipation complète

**PCM3168A différé en v3+** :
- Conditions de réintroduction : besoin réel démontré en jam (capture simultanée master + 4 voix individuelles) + maturité KiCad d'Alex
- Architecture de transition pré-étudiée : les modules PCM1808/PCM5102A restent sur le PCB comme fallback
- Voie : PCB custom + JLCPCB PCBA avec service d'assemblage SMT

Toutes les décisions sont documentées ici et dans `PROTOCOL_TABLES.md` pour éviter la re-discussion et servir de référence pendant tout le développement.

---

*Fin de PERKOMPANION_PHASE0.md — v0.8 final, avril 2026*
