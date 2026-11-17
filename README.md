# Ubuntu 26.04 Desktop — Autoinstall (setup Jules)

Provisionne une machine Ubuntu identique à la mienne sans rien faire à la main :
GNOME dark minimal, Tela + Starship + Nerd Font, dock auto-masqué, Vitals, NetBird,
Chrome, Slack, Discord, Spotify, Lens, zsh/oh-my-zsh, et les fixes Electron/Wayland.

## Ce qui est automatisé

- **Base** : Ubuntu Desktop minimal, FR, pilotes NVIDIA + codecs
- **Apps** : Chrome, Spotify, Lens, NetBird (dépôts apt) + Slack, Discord (.deb)
- **Look** : mode sombre, icônes Tela, polices Inter / JetBrainsMono Nerd Font,
  horloge avec secondes, dock auto-masqué, bouton accessibilité masqué
- **Terminal** : Ptyxis dark + Nerd Font, prompt Starship deux lignes
- **Shell** : zsh par défaut + oh-my-zsh + plugins (autosuggestions, syntax-highlighting…)
- **Extensions GNOME** : Dash to Dock, Blur my Shell, Just Perfection, Vitals
- **Fixes** : override `.desktop` X11 pour Lens et Slack (crash Electron sur Wayland),
  collage au clic molette (`gtk-enable-primary-paste`)

## À faire AVANT

1. **Mot de passe** — *rien à faire*. Aucun mot de passe n'est stocké dans le fichier :
   pas de bloc `identity` (le schéma exige un password), l'installeur demande
   nom, hostname, username et mot de passe à la main.
   C'est ce qui rend le fichier **sûr à stocker publiquement** (pas de hash exposé).
   > Saisir le username `jules` — le script de setup s'appuie dessus.
2. Vérifier `locale` / clavier dans `autoinstall.yaml`.
3. **Slack** est épinglé à une version (`4.49.89`). Mettre à jour l'URL si besoin
   depuis https://slack.com/downloads/linux
4. **Clé Spotify** : si l'install échoue sur une erreur `NO_PUBKEY`, la clé a tourné.
   Récupérer le bon `pubkey_<ID>.gpg` et l'ajuster dans `jules-setup.sh`.

## Comment l'utiliser

### Option A — clé USB "cidata" (le plus simple, pas de ré-empaquetage ISO)

1. Flasher l'ISO officielle Ubuntu 26.04 Desktop sur une clé (Rufus, `dd`, etc.).
2. Sur une 2e clé USB (ou une partition), créer un volume **étiqueté `CIDATA`** (FAT32)
   contenant deux fichiers :
   - `user-data`  → le contenu de `autoinstall.yaml`
   - `meta-data`  → fichier vide
3. Booter sur l'ISO avec les deux clés branchées. L'installeur détecte l'autoinstall.

> Astuce : on peut passer `autoinstall` aux paramètres du noyau pour forcer le mode
> sans interaction.

### Option B — ISO ré-empaquetée (autoinstall embarqué)

Glisser `autoinstall.yaml` dans l'ISO et l'exposer en `/autoinstall.yaml`.
Voir : https://linuxconfig.org/how-to-repackage-an-ubuntu-iso-image-for-autoinstall-using-yaml

### Valider la syntaxe avant de graver

```bash
# Vérifie que le YAML est conforme au schéma autoinstall
sudo snap install subiquity --classic
subiquity.subiquity-configure-run --help   # ou : cloud-init schema --config-file user-data
```

## Architecture du fichier

- `autoinstall.*` : exécuté par l'installeur (partitionnement, user, paquets de base, snaps).
- `autoinstall.user-data` : **cloud-init**, joué au **premier boot** (réseau dispo).
  - `write_files` : dconf système, `.zshrc`, `starship.toml`, overrides `.desktop`,
    le script `jules-setup.sh` et son service systemd.
  - `runcmd` : active le service `jules-setup.service`.
- `jules-setup.service` : one-shot, tourne **une seule fois** (se désactive et pose
  un flag `/var/lib/jules-setup.done`), installe dépôts/apps/fonts/extensions.

## Notes / limites

- `user-data` cloud-init sur Ubuntu **Desktop** est moins documenté que sur Server ;
  si le setup ne se déclenche pas, lancer manuellement une fois :
  `sudo /usr/local/sbin/jules-setup.sh`
- Les **secrets** ne sont volontairement pas inclus. Le `.zshrc` source `~/.secrets`
  (non versionné) — y mettre `BW_*`, `NETBIRD_TOKEN`, `NPM_TOKEN`, etc.
- Les extensions GNOME sont installées avec `gext` qui récupère la version compatible
  avec le shell installé (plus robuste qu'un zip figé).
