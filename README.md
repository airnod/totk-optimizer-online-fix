# TotK – NX Optimizer + Online Mod fix (60 FPS + multiplayer)

🇫🇷 [Français](#français) · 🇬🇧 [English](#english)

---

## Français

### Le problème
Dans **Zelda: Tears of the Kingdom** sur émulateur (testé sur Yuzu EA 4176), utiliser
**NX Optimizer / TOTK Optimizer** (60 FPS, UltraCam…) en même temps que le **TotK Online Mod**
(multijoueur) fait planter le jeu, ou le laisse bloqué sur la barre de chargement.
Chaque mod fonctionne très bien tout seul.

Dans le log de l'émulateur on voit :
```
Userspace PANIC!
nn::diag::detail::Abort(nn::Result const*)
nn::socket::detail::Initialize(...)   ou   nn::socket::detail::Socket(int, int, int)
```

### La cause
Les deux mods utilisent le réseau de la Switch (`nn::socket`) :
- l'**Online Mod** en a besoin pour se connecter au serveur ;
- **NX Optimizer** (module UltraCam, `exefs/subsdk3`) ouvre une **console de debug** réseau vers `127.0.0.1`.

`nn::socket` ne peut être initialisé qu'une seule fois. L'Optimizer démarre en premier, le conflit fait planter le jeu.

### Le correctif
Ce mod est un simple patch `.pchtxt` qui **désactive uniquement la console réseau de debug de NX Optimizer**.
Le 60 FPS, UltraCam et tous les réglages de l'Optimizer continuent de fonctionner.
Aucun fichier du jeu ni de l'Optimizer n'est fourni ou modifié : le patch est appliqué en mémoire par l'émulateur.

### Installation (Yuzu et forks)
1. Copie le dossier `Optimizer Online Fix` dans :
   `%APPDATA%\yuzu\load\0100F2C0115B6000\`
2. Dans l'émulateur : clic droit sur TotK → Propriétés → vérifie que
   **Optimizer Online Fix**, **TOTK Optimizer** et **Online Mod** sont cochés.
3. Lance le jeu.

### ⚠️ Compatibilité
Le patch cible **une version précise** du module de NX Optimizer :
`subsdk3` build ID **`D152076FB85E0B999DDEA822E27FC25BF95907D5`** (version d'octobre 2026).
Si NX Optimizer est mis à jour, le build ID change et le patch ne s'applique simplement plus
(aucun risque, mais le crash revient). Ouvre une issue pour demander une mise à jour.

Pour vérifier : dans le log de Yuzu, cherche
`Applying IPSwitch patch from mod "Optimizer Online Fix"`.

---

## English

### The problem
In **Zelda: Tears of the Kingdom** on emulator (tested on Yuzu EA 4176), running
**NX Optimizer / TOTK Optimizer** (60 FPS, UltraCam…) together with the **TotK Online Mod**
(multiplayer) crashes the game or leaves it stuck on the loading bar. Each mod works fine alone.

Emulator log shows:
```
Userspace PANIC!
nn::diag::detail::Abort(nn::Result const*)
nn::socket::detail::Initialize(...)   or   nn::socket::detail::Socket(int, int, int)
```

### The cause
Both mods use the Switch network stack (`nn::socket`):
- the **Online Mod** needs it to talk to the server;
- **NX Optimizer** (UltraCam module, `exefs/subsdk3`) opens a network **debug console** to `127.0.0.1`.

`nn::socket` can only be initialized once. The Optimizer starts first and the conflict aborts the game.

### The fix
This mod is a small `.pchtxt` patch that **only disables NX Optimizer's network debug console**.
60 FPS, UltraCam and every Optimizer setting keep working.
No game or Optimizer files are included or modified — the emulator applies the patch in memory.

### Install (Yuzu and forks)
1. Copy the `Optimizer Online Fix` folder into:
   `%APPDATA%\yuzu\load\0100F2C0115B6000\`
2. In the emulator: right-click TotK → Properties → make sure
   **Optimizer Online Fix**, **TOTK Optimizer** and **Online Mod** are all enabled.
3. Launch the game.

### ⚠️ Compatibility
The patch targets **one specific build** of NX Optimizer's module:
`subsdk3` build ID **`D152076FB85E0B999DDEA822E27FC25BF95907D5`** (October 2026 release).
If NX Optimizer updates, the build ID changes and the patch simply stops applying
(harmless, but the crash comes back). Open an issue to request an update.

To check it works: search the Yuzu log for
`Applying IPSwitch patch from mod "Optimizer Online Fix"`.

### Technical details
Patched functions in `subsdk3` (offsets in memory, `offset_shift 0x100`):

| Offset | Patch | Role |
|---|---|---|
| `0x18DF0` | `NOP` | skip the network console init call (nifm + socket + connect) |
| `0x1D48C` | `mov w0,#0; ret` | logger → `nn::socket::Send` |
| `0x1A1F4` | `mov w0,#0; ret` | logger → `nn::socket::Send` |
| `0x1A4A4` | `mov w0,#0; ret` | console command receive → `nn::socket::Recv` |
| `0x1A5CC` | `mov w0,#0; ret` | console command receive → `nn::socket::Recv` |

Not affiliated with Nintendo, NX Optimizer or the TotK Online Mod authors.
