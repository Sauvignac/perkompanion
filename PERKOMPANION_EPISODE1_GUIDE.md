# PërKompanion — Épisode 1 — Guide de reproduction

> Guide pratique pour reproduire l'installation présentée dans l'Épisode 1 de la chaîne YouTube Sauvignac.
> Tous les fichiers, toutes les commandes, dans l'ordre.
>
> **Pré-requis matériel** :
> - Raspberry Pi 5
> - Écran Elecrow 7" tactile (HDMI micro + USB touch)
> - MicroSD ≥ 32 GB
> - Câble micro HDMI vers HDMI
> - Alimentation officielle Pi 5 (5V/5A USB-C)
>
> **Pré-requis logiciel** :
> - Raspberry Pi Imager installé sur ton PC
> - Un client SSH (Windows : PowerShell, Linux/macOS : terminal natif)
> - Un client SFTP pour le transfert de fichiers (FileZilla, WinSCP, ou rsync)

---

## Sommaire

1. [Flash de la MicroSD](#flash)
2. [Premier boot et configuration de base](#first-boot)
3. [Configuration de l'écran Elecrow 7"](#elecrow)
4. [Mode console autologin + kiosk Chromium](#kiosk)
5. [Installation Node.js + Express + WebSocket](#nodejs)
6. [Installation et configuration PM2](#pm2)
7. [Déploiement de l'interface](#interface)
8. [Configuration PM2 logrotate](#logrotate)
9. [Validation finale](#validation)

---

<a id="flash"></a>
## 1. Flash de la MicroSD

⚠️ **Erreur à éviter — locale FR au flash initial**

La première tentative de ce projet a configuré le Pi en français (locale FR, clavier FR). Cela génère des conflits d'encodage UTF-8 entre la session SSH côté Pi et le terminal côté PC (notamment Windows PowerShell qui démarre en cp1252). Les caractères accentués deviennent illisibles, les messages d'erreur sont corrompus.

**Le `chcp 65001` côté PowerShell ne suffit pas.** Le seul correctif fiable est de reflasher en `en_GB.UTF-8`.

**Recommandation** : configure le Pi en anglais dès le départ. Le français reste dans l'interface PërKompanion elle-même (i18n), pas dans le système d'exploitation.

### Configuration Raspberry Pi Imager

1. Lancer Raspberry Pi Imager
2. Choisir l'OS : **Raspberry Pi OS (64-bit)** (Bookworm)
3. Choisir la carte SD
4. Cliquer sur l'engrenage (paramètres avancés) :
   - Hostname : `perkompanion`
   - Activer SSH avec authentification par mot de passe
   - Username : `sauvignac`
   - Password : (au choix)
   - WiFi SSID + password + pays : `FR`
   - **Locale : `en_GB.UTF-8`**
   - **Layout clavier : `gb` (UK)**
5. Confirmer et flasher

---

<a id="first-boot"></a>
## 2. Premier boot et configuration de base

Insérer la carte dans le Pi, brancher l'écran et l'alimentation. Au premier boot, le Pi se connecte au Wi-Fi configuré au flash.

### Connexion SSH depuis le PC

```bash
ssh sauvignac@perkompanion.local
```

Si la résolution `.local` ne fonctionne pas, trouve l'IP du Pi via l'interface admin de ta box.

### Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

### Paquets nécessaires pour le kiosk

```bash
sudo apt install -y xorg xinit x11-xserver-utils unclutter chromium-browser xdotool
sudo apt remove -y gnome-keyring  # évite le popup d'auth Chromium au démarrage
```

---

<a id="elecrow"></a>
## 3. Configuration de l'écran Elecrow 7"

L'écran Elecrow 7" (1024×600) nécessite une configuration HDMI explicite dans `/boot/firmware/config.txt`.

### Édition du fichier

```bash
sudo nano /boot/firmware/config.txt
```

### Ajouter en fin de fichier

```ini
# Écran Elecrow 7" 1024x600
hdmi_group=2
hdmi_mode=87
hdmi_cvt=1024 600 60 6 0 0 0
hdmi_drive=1
disable_overscan=1
```

Sauvegarder (`Ctrl+O`, `Entrée`, `Ctrl+X`).

### Reboot

```bash
sudo reboot
```

L'écran doit afficher la console à la résolution native 1024×600.

---

<a id="kiosk"></a>
## 4. Mode console autologin + kiosk Chromium

### Activer l'autologin console

```bash
sudo raspi-config
```

Naviguer : `1 System Options` → `S5 Boot / Auto Login` → `B2 Console Autologin` → `OK` → `Finish` → ne pas rebooter tout de suite.

> **Pourquoi pas le mode graphique** : les sessions Wayland (labwc) et X11 (LXDE-pi-x) du Pi OS Bookworm ont des soucis de compatibilité avec lightdm en configuration kiosk. Console + startx manuel = plus stable, plus léger, plus fiable pour un instrument dédié.

### Fichier `~/.xinitrc`

Crée le fichier qui sera exécuté par `startx` :

```bash
nano ~/.xinitrc
```

Contenu :

```bash
#!/bin/bash
# Désactivation de l'économiseur d'écran et du DPMS
xset s off
xset -dpms
xset s noblank

# Cache le curseur après 0s d'inactivité
unclutter -idle 0 &

# Lancement de Chromium en mode kiosk plein écran
chromium-browser \
  --noerrdialogs \
  --disable-infobars \
  --kiosk \
  --app=http://localhost:3000
```

Le rendre exécutable :

```bash
chmod +x ~/.xinitrc
```

### Fichier `~/.bash_profile`

Pour que `startx` se lance automatiquement après le login console :

```bash
nano ~/.bash_profile
```

Contenu :

```bash
# Sourcing du .bashrc standard
source ~/.bashrc

# Lancement automatique de X11 si on est sur tty1 (login console)
if [ -z "$DISPLAY" ] && [ "$(tty)" = "/dev/tty1" ]; then
    startx
fi
```

### Reboot pour valider

```bash
sudo reboot
```

Au reboot, le Pi doit :
1. Booter en console
2. Se logger automatiquement comme `sauvignac`
3. Lancer X11 puis Chromium en kiosk
4. Afficher une page d'erreur (normal — le serveur n'est pas encore installé)

---

<a id="nodejs"></a>
## 5. Installation Node.js + Express + WebSocket

### Installation de NVM (Node Version Manager)

NVM permet d'avoir plusieurs versions de Node.js et de basculer facilement.

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

**Fermer puis rouvrir la session SSH** pour que NVM soit chargé.

### Installation de Node.js LTS

```bash
nvm install --lts
node --version  # devrait afficher v24.x.x
npm --version
```

### Création du projet

```bash
mkdir -p ~/perkompanion/public
cd ~/perkompanion
npm init -y
```

### Installation des dépendances

```bash
npm install express ws
```

### Fichier `~/perkompanion/server.js`

```bash
nano ~/perkompanion/server.js
```

Contenu complet :

```javascript
const express = require('express');
const path = require('path');
const http = require('http');
const { WebSocketServer } = require('ws');

// --- Logger minimal avec timestamp + niveau ---
const log = (level, msg) => {
  const ts = new Date().toISOString().slice(11, 23); // HH:MM:SS.mmm
  console.log(`[${ts}] [${level}] ${msg}`);
};

const app = express();
app.use(express.static(path.join(__dirname, 'public')));

const server = http.createServer(app);
const wss = new WebSocketServer({ server, path: '/ws' });
const clients = new Set();

wss.on('connection', (ws, req) => {
  clients.add(ws);
  const ip = req.socket.remoteAddress;
  log('INFO', `WS connect (${clients.size} clients) from ${ip}`);

  // Hello initial
  ws.send(JSON.stringify({ type: 'hello', server: 'perkompanion', ts: Date.now() }));

  ws.on('message', (data) => {
    const text = data.toString();
    log('RECV', text);
    // Echo pour validation skeleton — sera remplacé par routage Teensy
    ws.send(JSON.stringify({ type: 'echo', payload: text, ts: Date.now() }));
  });

  ws.on('close', () => {
    clients.delete(ws);
    log('INFO', `WS disconnect (${clients.size} clients)`);
  });

  ws.on('error', (err) => log('ERROR', `WS error: ${err.message}`));
});

// Heartbeat broadcast — cadence 500ms, alignée sur le protocole Pi↔Teensy
// (volontairement non loggé : trafic de fond, sinon spam illisible)
setInterval(() => {
  const msg = JSON.stringify({ type: 'heartbeat', ts: Date.now() });
  for (const ws of clients) {
    if (ws.readyState === ws.OPEN) ws.send(msg);
  }
}, 500);

// Helper pour le futur bridge Serial Teensy
function broadcast(obj) {
  const msg = JSON.stringify(obj);
  for (const ws of clients) {
    if (ws.readyState === ws.OPEN) ws.send(msg);
  }
}

server.listen(3000, '0.0.0.0', () => {
  log('BOOT', 'PërKompanion HTTP+WS running on :3000');
});
```

### Test rapide

```bash
node server.js
```

Tu dois voir : `[HH:MM:SS.mmm] [BOOT] PërKompanion HTTP+WS running on :3000`.

`Ctrl+C` pour arrêter, on va passer à PM2 pour la mise en service permanente.

---

<a id="pm2"></a>
## 6. Installation et configuration PM2

### Installation globale de PM2

```bash
npm install -g pm2
```

### Démarrage du serveur via PM2

```bash
pm2 start ~/perkompanion/server.js --name server
pm2 status
```

Tu dois voir le process `server` en statut `online`.

### Configurer PM2 pour démarrer au boot du Pi

```bash
pm2 startup
```

PM2 affiche une commande à copier-coller (qui crée un service systemd). Exécute-la, puis :

```bash
pm2 save
```

Cette dernière commande sauvegarde la liste des processus pour qu'ils soient relancés au boot.

### Vérification

```bash
sudo reboot
```

Après reboot, sans aucune action :
- Le Pi boot
- Login automatique
- PM2 démarre `server.js`
- X11 lance Chromium en kiosk
- Chromium se connecte à `localhost:3000`
- L'interface s'affiche

---

<a id="interface"></a>
## 7. Déploiement de l'interface

L'interface (un fichier `index.html` autonome, par exemple un export d'artifact Claude Design) doit être placée dans `~/perkompanion/public/`.

### Transfert via FileZilla (SFTP)

- Host : `perkompanion.local` (ou IP)
- Port : 22
- Username : `sauvignac`
- Password : celui défini au flash
- Naviguer vers `/home/sauvignac/perkompanion/public/`
- Glisser-déposer le fichier `index.html` (le nom doit être exactement `index.html`)

### Transfert via SCP en ligne de commande

```bash
scp index.html sauvignac@perkompanion.local:/home/sauvignac/perkompanion/public/
```

### Refresh de Chromium kiosk

Pour rafraîchir l'affichage sur le Pi sans rebooter :

```bash
DISPLAY=:0 xdotool key F5
# ou hard reload :
DISPLAY=:0 xdotool key ctrl+shift+r
```

### Note sur les artifacts Claude Design

Si tu déploies un artifact bundlé exporté de Claude Design, il est probable qu'il s'affiche avec des marges noires malgré la résolution 1024×600 correctement détectée. C'est dû à la logique d'auto-fit interne du bundle (transform scale dans un `<div id="stage">`). Ce point est documenté dans `PERKOMPANION_DECISIONS.md` Épisode 1 §6 — un fix temporaire `MutationObserver` est applicable, mais la solution propre est de réimplémenter l'interface en HTML/CSS/React natif via Claude Code (à venir).

---

<a id="logrotate"></a>
## 8. Configuration PM2 logrotate

Pour éviter que les logs ne saturent la SD card à long terme.

### Installation et configuration

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 10M
pm2 set pm2-logrotate:retain 7
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:dateFormat YYYY-MM-DD_HH-mm-ss
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'
pm2 save
```

### Vérification

```bash
pm2 conf pm2-logrotate
```

| Paramètre | Valeur | Effet |
|---|---|---|
| `max_size` | 10M | Rotation dès qu'un log dépasse 10 Mo |
| `retain` | 7 | Garde les 7 derniers fichiers |
| `compress` | true | Anciens logs gzippés (gain ~10×) |
| `rotateInterval` | `0 0 * * *` | Rotation forcée tous les jours à minuit |

### Localisation des logs

```
~/.pm2/logs/server-out.log    ← stdout (console.log)
~/.pm2/logs/server-error.log  ← stderr (erreurs)
```

`pm2 describe server` donne les chemins exacts.

---

<a id="validation"></a>
## 9. Validation finale

### Sur le Pi (SSH)

```bash
pm2 status
pm2 logs server --lines 5
```

Tu dois voir :

```
[HH:MM:SS.mmm] [BOOT] PërKompanion HTTP+WS running on :3000
```

### Sur le laptop (test WebSocket)

Ouvre `http://perkompanion.local:3000` dans Chrome/Firefox, F12 (DevTools), onglet Console, colle :

```javascript
const ws = new WebSocket(`ws://${location.hostname}:3000/ws`);
ws.onopen = () => console.log('✅ WS open');
ws.onmessage = (e) => console.log('← recv:', e.data);
ws.onclose = () => console.log('✗ WS close');
setTimeout(() => ws.send('ping from browser'), 500);
```

Réponses attendues :
- `✅ WS open`
- Un message `hello` initial
- Un message `echo` après le `ping`
- Des `heartbeat` toutes les 500ms

Côté Pi en parallèle (`pm2 logs server`) :

```
[HH:MM:SS.mmm] [INFO] WS connect (1 clients) from ::ffff:192.168.x.x
[HH:MM:SS.mmm] [RECV] ping from browser
```

Si tout ça fonctionne : **l'épisode 1 est validé**. 🎉

---

## Récapitulatif des fichiers créés

| Chemin | Rôle |
|---|---|
| `/boot/firmware/config.txt` | Configuration HDMI pour Elecrow 7" (lignes ajoutées) |
| `~/.xinitrc` | Script lancé par `startx` — Chromium kiosk |
| `~/.bash_profile` | Lance `startx` automatiquement au login console tty1 |
| `~/perkompanion/server.js` | Serveur Express + WebSocket |
| `~/perkompanion/package.json` | Configuration npm (créé par `npm init -y`) |
| `~/perkompanion/public/index.html` | Interface PërKompanion (Home Dashboard) |
| `~/.pm2/` | Configuration PM2 (créé automatiquement) |

---

## Commandes utiles au quotidien

| Commande | À quoi ça sert |
|---|---|
| `pm2 status` | Vue rapide : process, uptime, restart count, mémoire, CPU |
| `pm2 logs server` | Logs live (Ctrl+C pour sortir) |
| `pm2 logs server --lines 100` | Les 100 dernières lignes |
| `pm2 logs server --err` | Seulement les erreurs |
| `pm2 monit` | Dashboard interactif temps réel |
| `pm2 describe server` | État détaillé du process |
| `pm2 restart server` | Relance |
| `pm2 reload server` | Relance « graceful » (sans coupure) |
| `pm2 flush` | Vide tous les logs accumulés |
| `DISPLAY=:0 xdotool key F5` | Refresh Chromium depuis SSH |
| `DISPLAY=:0 xdotool key ctrl+shift+r` | Hard reload Chromium depuis SSH |

---

## Dépannage

### Le SSH ne répond plus mais l'écran fonctionne

Le Wi-Fi du Pi a probablement décroché. C'est un problème connu sur Pi 5. Solutions :
1. Tenter `arp -a` côté PC pour retrouver l'IP du Pi (préfixes MAC Pi : `b8:27:eb`, `dc:a6:32`, `d8:3a:dd`, `2c:cf:67`, `e4:5f:01`)
2. Vérifier l'interface admin de la box pour l'IP
3. En dernier recours : reboot brutal (débrancher/rebrancher l'alim) — PM2 redémarre automatiquement au boot grâce à `pm2 startup` + `pm2 save`

### `pm2 logs server` ne montre rien après modification de `server.js`

Penser à relancer : `pm2 restart server`. PM2 ne recharge pas le code à chaud par défaut.

### L'interface s'affiche avec des marges noires sur l'Elecrow

C'est probablement un artifact bundlé Claude Design avec auto-fit interne. Voir Épisode 1 §6 dans `PERKOMPANION_DECISIONS.md` pour le fix temporaire `MutationObserver`. La solution durable est la réimplémentation native via Claude Code.

### Le Pi décroche du SSH pendant l'édition d'un fichier avec nano

Privilégier l'édition locale + push via FileZilla au lieu de nano sur SSH. Si ça arrive : au prochain `nano server.js`, refuser la récupération du fichier swap, puis `rm ~/perkompanion/.server.js.swp`.

---

*Sauvignac — On part de zéro. On construit.*
