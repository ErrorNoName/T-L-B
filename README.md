1. **L’installation et le placement des fichiers BepInEx.**  
2. **La configuration du lanceur Steam (pour un exécutable Windows via Proton ou pour la version Linux si le jeu en propose une).**  
3. **La structure de dossiers recommandée et la gestion des plugins.**

> **Remarque importante :** D’après la présence de `REPO.exe`, de `winhttp.dll` et d’un dossier `MonoBleedingEdge`, il est fort probable que le jeu tourne en fait sous Windows (via Proton). Les instructions pour la version native Linux (si elle existe vraiment) sont indiquées plus bas (section *Utiliser BepInEx sur un binaire Linux natif*). Cependant, dans la plupart des cas sur Steam/Arch Linux, on fait tourner la version Windows via Proton.  

---

## 1. Vérifier si le jeu tourne via Proton ou en natif

1. **Ouvrez Steam** et allez sur votre Bibliothèque.
2. **Clic droit** sur *REPO* → **Propriétés**.
3. Dans **Compatibilité**, regardez si l’option *“Forcer l’utilisation d’un outil de compatibilité Steam Play”* (Proton) est activée.  
   - Si c’est coché et qu’un Proton (ex. “Proton Experimental” ou “Proton 8.x”) est sélectionné, alors le jeu tourne sous Wine/Proton en version *Windows*.
   - Si ce n’est pas coché mais que vous avez quand même des .exe, c’est peut-être que le jeu dispose d’un script Linux qui appelle un binaire Windows (cas rare).  
   - La plupart du temps, la présence de `REPO.exe`, `UnityPlayer.dll` et `winhttp.dll` signifie que le jeu tourne bel et bien sous Proton, et BepInEx s’utilise alors via la méthode “winhttp”.

---

## 2. Installation de BepInEx (version Windows/Proton)

> **Si vous avez déjà des fichiers `winhttp.dll`, `libdoorstop.so`, `doorstop_config.ini` et le dossier `BepInEx` dans `~/.local/share/Steam/steamapps/common/REPO`, passez directement à la section [3. Configuration Proton dans Steam](#3-configuration-proton-dans-steam).**  

### 2.1. Récupérer la bonne version de BepInEx

1. Rendez-vous sur la page [BepInEx Releases sur GitHub](https://github.com/BepInEx/BepInEx/releases) (normalement vous feriez ça en dehors de Steam, mais sous Arch Linux, vous pouvez aussi trouver des paquets AUR, par exemple `bepinex-bin`, *mais* souvent ces paquets ne sont pas “directement” prêts pour votre jeu. Le plus simple est de télécharger la version manuellement).  
2. Choisissez **BepInEx x64 (5.x ou 6.x)** suivant ce que la documentation ou la communauté mod recommande. La version stable la plus courante pour la majorité des jeux Unity est la branche 5.4.x ou 6.x (BepInEx 5 étant encore la plus utilisée).  
3. **Décompressez** l’archive téléchargée.

Dans l’archive, vous trouverez un dossier du type `BepInExPack_xxx`. À l’intérieur de ce dossier, on retrouve normalement :

- `winhttp.dll`
- `doorstop_config.ini`
- `BepInEx/` (avec `core`, `plugins`, `patchers`, `config`, etc.)
- (Éventuellement) `libdoorstop_x64.so` ou `libdoorstop.so`
- ... et d’autres fichiers.

### 2.2. Copier BepInEx dans le dossier du jeu

1. Allez dans votre dossier du jeu :  
   ```bash
   cd ~/.local/share/Steam/steamapps/common/REPO
   ```
2. **Copiez tous les fichiers** suivants à la racine de ce dossier (là où se trouve `REPO.exe`, `run.sh`, `UnityPlayer.dll`, etc.) :

   - `winhttp.dll`  
   - `libdoorstop.so` (s’il est fourni, il peut servir dans certains cas sous Linux)  
   - `doorstop_config.ini`  
   - Le dossier `BepInEx` **en entier** (contenant `core/`, `plugins/`, `config/`, etc.)
   
   Après cette copie, on s’assure d’avoir la structure suivante :
   ```
   ~/.local/share/Steam/steamapps/common/REPO
   ├── REPO.exe
   ├── winhttp.dll
   ├── libdoorstop.so       (facultatif si fourni)
   ├── doorstop_config.ini
   ├── BepInEx/
   │    ├── config/
   │    ├── core/
   │    ├── plugins/
   │    └── ...
   └── ...
   ```

3. Vérifiez que **`doorstop_config.ini`** est correct :  
   - `enabled = true`  
   - `targetAssembly = BepInEx/core/BepInEx.dll`  
   - (Le chemin est relatif : “BepInEx/core/BepInEx.dll”).  
   - En général, le reste peut être laissé par défaut.

---

## 3. Configuration Proton dans Steam

Pour que BepInEx soit bien chargé, nous devons dire à Proton/Wine d’utiliser `winhttp.dll` (installé par BepInEx) **comme DLL native**. Sinon, il se peut que Wine/Proton utilise sa propre version interne et n’injecte jamais BepInEx.

1. **Ouvrez Steam** et retournez dans les **Propriétés** du jeu REPO.
2. Dans l’onglet **Général**, trouvez le champ **Options de lancement** (*Launch Options*).
3. **Ajoutez** (ou remplacez) la ligne suivante :

   ```bash
   WINEDLLOVERRIDES="winhttp=n,b" %command%
   ```
   
   - Le `n,b` signifie : “natif, puis intégré” (native, builtin). Cela fait en sorte que la `winhttp.dll` de BepInEx soit chargée en premier.
   - Le `%command%` est nécessaire pour dire à Steam de continuer à lancer le jeu ensuite.

> **Exemple complet**  
> ```
> WINEDLLOVERRIDES="winhttp=n,b" %command%
> ```
> 
> Si vous avez d’autres variables d’environnement, vous pouvez les chaîner. Par exemple :
> ```
> PROTON_LOG=1 WINEDLLOVERRIDES="winhttp=n,b" %command%
> ```

4. **Fermez** la fenêtre de propriétés et lancez le jeu.

### 3.1. Vérifier si BepInEx se lance bien

- Normalement, au premier démarrage, un dossier `BepInEx/logs/` va être créé.  
- Vous devriez voir un fichier `LogOutput.log` dans `~/.local/share/Steam/steamapps/common/REPO/BepInEx/`.  
- Lancement réussi : dans ce fichier, vous trouverez des lignes mentionnant BepInEx, par exemple :  
  ```
  [Message:   BepInEx] BepInEx 5.4.x - REPO (Unity x.x.x)
  [Info   :   BepInEx] Running under Unity vX.X.XXXX
  ...
  ```

---

## 4. Installer et gérer vos mods

1. Placez les fichiers `.dll` de vos mods dans `BepInEx/plugins`.  
   - Certains mods peuvent venir avec plusieurs fichiers ou un dossier entier. Respectez la hiérarchie fournie par le mod, la plupart se contentent de mettre un `NomDuMod.dll` dans `plugins/`.  
2. (Optionnel) Si un mod vous demande de placer un *patcher* dans `BepInEx/patchers`, suivez cette instruction. C’est moins courant mais ça arrive pour des mods avancés.  
3. (Optionnel) Configurez certains mods dans `BepInEx/config/`. Les `.cfg` seront générés au premier lancement (si le mod en prévoit un).  
4. Redémarrez le jeu pour chaque nouveau mod. Vérifiez à nouveau dans `LogOutput.log` si le mod se charge (la plupart des mods écrivent un message au démarrage).

---

## 5. Utiliser BepInEx sur un binaire Linux natif (rare)

Certains jeux Unity publiés sur Linux incluent un exécutable ELF (souvent nommé `GameName.x86_64` ou un script `run.sh` qui appelle un binaire ELF). Dans ce cas :

1. À la racine du dossier du jeu, vous devriez retrouver un script de lancement (ex. `run.sh`) ou un exécutable (ex. `REPO.x86_64`).  
2. BepInEx pour Linux se sert de `libdoorstop.so` ou `libdoorstop_x64.so` pour faire l’injection.  
3. Il vous faut alors **précharger** cette bibliothèque avant de lancer le jeu, par exemple :  
   ```bash
   LD_PRELOAD="$PWD/libdoorstop.so" \
   DOORSTOP_ENABLE=1 \
   DOORSTOP_INVOKE_DLL_PATH="$PWD/BepInEx/core/BepInEx.dll" \
   ./run.sh
   ```
   ou
   ```bash
   LD_PRELOAD="$PWD/libdoorstop.so" \
   DOORSTOP_ENABLE=1 \
   DOORSTOP_INVOKE_DLL_PATH="$PWD/BepInEx/core/BepInEx.dll" \
   ./REPO.x86_64
   ```
4. Pour le lancer directement depuis Steam :  
   - Ouvrez les Propriétés du jeu.  
   - Dans **Options de lancement**, mettez par exemple :  
     ```bash
     LD_PRELOAD="$PWD/libdoorstop.so" \
     DOORSTOP_ENABLE=1 \
     DOORSTOP_INVOKE_DLL_PATH="$PWD/BepInEx/core/BepInEx.dll" \
     %command%
     ```
   - Assurez-vous que `doorstop_config.ini` (s’il existe) est adapté à Linux ou que les variables d’environnement ci-dessus suffisent (ce qui est souvent le cas).  
5. La structure de dossiers et le placement des mods (`BepInEx/plugins/`) restent les mêmes.

---

## 6. Questions fréquentes et dépannage

### 6.1. BepInEx ne se lance pas, pas de `LogOutput.log`

- **Avez-vous bien configuré** `WINEDLLOVERRIDES="winhttp=n,b"` si vous tournez via Proton ?  
- **Avez-vous le bon BepInEx (x64 vs x86)** ? Vérifiez que votre jeu est en 64 bits.  
- **Le fichier** `doorstop_config.ini` est-il présent à la racine du dossier du jeu ? Est-ce que `enabled = true` est bien renseigné ?  
- Essayez de **supprimer** d’anciennes versions de `winhttp.dll` ou `libdoorstop.so` si vous aviez fait un test précédent.

### 6.2. Certains mods ne fonctionnent pas

- Vérifiez les **dépendances** du mod (ex. certaines libs complémentaires).  
- Vérifiez que le mod est **compatible** avec la version du jeu.  
- Vérifiez que vous avez la **même version majeure** de BepInEx requise par le mod (BepInEx 5.x vs 6.x).

### 6.3. Où se trouve la console BepInEx ou la fenêtre de log ?

- Sur Linux + Proton, vous ne verrez pas forcément la console DOS/terminal comme sur Windows.  
- Vous trouverez les logs dans `BepInEx/LogOutput.log` et parfois `BepInEx/logs/`.

---

## Récapitulatif rapide (check-list)

1. **Activer Proton** (si c’est un .exe) :  
   - Bibliothèque Steam → clic droit sur le jeu → Propriétés → Compatibilité → Choisir Proton.  

2. **Placer BepInEx** (version Windows 64 bits) dans `~/.local/share/Steam/steamapps/common/REPO` :  
   - `winhttp.dll`, `doorstop_config.ini`, dossier `BepInEx`.  

3. **Configurer Steam** (Options de lancement) :  
   ```bash
   WINEDLLOVERRIDES="winhttp=n,b" %command%
   ```

4. **Lancer le jeu** → Vérifier `BepInEx/LogOutput.log`.

5. **Ajouter des mods** dans `BepInEx/plugins` → Relancer → Vérifier log.
