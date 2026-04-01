# MLI-3D — Document de reprise complet (v23.6)

## Qui suis-je

Responsable FabLab et technicien IT dans un lycée technique. Pas de compétences en codage (compréhension vague). Recherche la simplicité, la performance, une interface et un usage intuitif. Présente des workshops en français et en anglais.

## Le projet

**MLI-3D** (Model Live Intro 3D) est un générateur d'intros 3D pour livestreams YouTube/Twitch. C'est un fichier HTML unique (~2930 lignes) qui utilise Three.js r128 pour créer un habillage de stream en 3D avec modèles GLB/FBX/STL, éclairage cinématique, textes animés, musique réactive, et une fenêtre popup pour OBS.

- **Repo GitHub** : https://github.com/fablabloritz-coder/MLI-3D
- **GitHub Pages** : https://fablabloritz-coder.github.io/MLI-3D
- **Version actuelle** : v23.6
- **Fichier principal** : `index.html` (single-file, tout-en-un)
- **Utilisateur GitHub** : `fablabloritz-coder`

## Architecture technique

### Stack
- **Single HTML file** — pas de build, pas de dépendances, tout dans un fichier
- **Three.js r128** via CDN (threejs.org)
- **GLTFLoader + RGBELoader + STLLoader + FBXLoader** via CDN
- **fflate + NURBSUtils/NURBSCurve** (dépendances FBXLoader)
- **Post-processing** : EffectComposer, UnrealBloomPass, BokehPass (DoF)
- **Pas de framework** — vanilla JS, CSS inline

### Structure du code
1. **HTML** (lignes 1–541) — Panneau de configuration (sidebar gauche) + preview (droite, dans `#pvWrap > #pv`)
2. **JavaScript** (lignes 542–2930) — Tout le moteur

### Composants majeurs
- **getConfig()** — Lit tous les contrôles UI et retourne un objet config complet (~112 paramètres)
- **U()** — Fonction de mise à jour centrale, appelée à chaque changement de slider/toggle. Se termine par `_undoPush()`
- **SLOTS[0..2]** — 3 slots pour modèles 3D, chacun avec : `buf`, `type`, `name`, `origMats`, `mixer`, `clips`, `activeClip`, `baseScale`, `group`, `_cfg`
- **S** — Objet d'état global : `logoURL`, `bgURL/bgType/bgName`, `customHdriBuf/customHdriName`, `audioURL/audioName`, `particleURL`, `crowd3DBuf/crowd3DName/crowd3DType/_crowd3DParts`, `envBuf/envName`, `curMat`, `scenes`, + champs `_bg*` privés
- **modelStore** — `Map(nom → {buf, type})` pour mémoriser les modèles chargés en session (changement de scène)
- **loadModelSlot(idx, buf, type)** — Charge un GLB/FBX/STL dans un slot. ⚠️ FBX : nouvelle instance `new THREE.FBXLoader()` à chaque appel
- **buildPop()** — Génère le HTML complet de la popup OBS (template string géant ~800 lignes)
- **openPopup()** — Ouvre la popup via window.open() avec les bonnes dimensions selon le ratio
- **updateSceneBounds()** — Recalcule `mC` (centre), `mH` (hauteur), `mW` (largeur entre slots)
- **buildCrowdTexture()** — Génère la texture canvas de foule SVG
- **parseCrowd3DModel(buf, type)** — Parse un modèle 3D pour la foule (InstancedMesh), stocke dans `S._crowd3DParts`
- **buildCrowd3D(c)** — Instancie les InstancedMesh autour de la scène
- **loadEnv(buf, type)** — Charge un GLB environnement, normalise à ~20 unités, stocke bounds dans `envBounds`
- **getEnvCamPath(preset, el, spd)** — Calcule pos/target caméra pour les 5 presets environnement
- **getWP()** — Calcule le waypath caméra spline Catmull-Rom (mode object)
- **anim()** — Boucle d'animation (cappée 30fps)
- **loadC_fromObj(c)** — Restaure tous les paramètres depuis un objet config. Appelle `updateCamModeUI()`, `updateCrowdModeUI()`, `updateAppModeUI()`, `setRatio()` et `U()` en fin
- **exportCfg() / importCfg()** — Export/import JSON complet avec fichiers embarqués
- **shareCfg()** — Partage config via URL base64
- **doTransition(cb)** — Applique une transition (fade/slide/zoom) puis appelle le callback
- **toggleLock(group, btn)** — Verrouille/déverrouille un groupe de paramètres
- **undo() / redo()** — Navigation dans l'historique (50 niveaux, debounce 600ms)

### Rendu 3D
- **Preview** : canvas dans `#pv` (lui-même dans `#pvWrap`), 30fps cap, pixelRatio adaptatif
- **Popup OBS** : fenêtre séparée via window.open(), même cap 30fps
- **Camera** :
  - `continuous` — spline Catmull-Rom fluide
  - `cuts` — sauts aléatoires avec fondu noir
  - `orbit` — cercle statique à rayon fixe (slider hauteur orbite)
  - `travel` — travelling A→B linéaire (slider amplitude)
  - `handheld` — spline + micro-vibrations spring-damper organiques (slider intensité)
- **Mode Environnement 3D** : toggle Objet ↔ Environnement. En mode env, slots cachés, caméra à l'intérieur avec 5 presets : `exploration`, `survol`, `couloir`, `panoramique`, `orbital`
- **Éclairage** : Key, Rim, Fill, L2 (SpotLights) + Ambient
- **Shadow maps** : 1024×1024, PCFSoftShadowMap
- **Fog** : FogExp2, toggle ON/OFF
- **Sol** : PlaneGeometry avec réflexion optionnelle
- **Post-processing** : Bloom (UnrealBloomPass) + DoF (BokehPass) via EffectComposer

### Foule & Paparazzi
- **Mode SVG** : 8 silhouettes Bézier sur cylindre CylinderGeometry BackSide (canvas 2048px)
- **Mode 3D** : InstancedMesh à partir d'un GLB/FBX/STL uploadé. Matériaux convertis en `MeshStandardMaterial`. Toggle couleur de base (color picker) + variation aléatoire. Cap 80 instances
- Toggle SVG ↔ Modèle 3D dans l'interface
- **Flashes paparazzi** : pool de 8 PointLights, déclenchement aléatoire, decay exponentiel

### Audio
- Upload MP3/WAV/OGG/M4A
- Visualisation mel-scale + A-weighting sur canvas
- Pulsation par spectral flux, 3 modes : off, couleur unique, cycle de couleurs

### Export vidéo
- `MediaRecorder` sur le canvas WebGL → fichier `.webm` téléchargeable
- Bouton **⏺ REC** en bas à droite de l'aperçu (rouge pulsant pendant l'enregistrement, timer affiché)
- 8 Mbps VP9, pixelRatio monté à 2× pendant le recording
- Chrome/Edge uniquement (Firefox non supporté)

### Ratio aperçu
- Sélecteur Libre / 16:9 / 4:3 / 9:16 / 1:1
- `#pvWrap` centre `#pv` avec `aspect-ratio` CSS
- `openPopup()` adapte les dimensions de la fenêtre au ratio choisi
- Sauvegardé dans config (`viewRatio`)

### Système de config
- **localStorage** pour sauvegardes nommées et scènes live
- **Export/Import JSON** avec TOUS les fichiers embarqués en base64 :
  - Modèles 3D slots 0/1/2, foule 3D, environnement 3D
  - Musique, HDRI personnalisé, fond image/vidéo, logo, texture particules
  - Message d'avertissement orange si un fichier ne peut pas être restauré
- **Partage via URL** en base64
- **Scènes** (1-9) : snapshots de config, switchables via touches numériques dans la popup + transition animée

### Transitions entre scènes
- 3 types : Fondu noir (`fade`), Slide (`slide`), Zoom (`zoom`)
- Durée configurable (0.2s → 2s)
- S'applique dans le preview (clic sur une scène) et dans la popup OBS (touches 1-9)

### Annuler / Refaire
- 50 niveaux d'historique, debounce 600ms
- Raccourcis : Ctrl+Z (annuler), Ctrl+Y ou Ctrl+Shift+Z (refaire)
- Boutons ↩/↪ dans le panneau bas (grisés si indisponible)
- État initial poussé au démarrage (200ms delay)
- Ne couvre pas les chargements de fichiers 3D (trop lourds)

### Verrouillage de groupes
- Icône 🔓 dans l'en-tête de 7 groupes verrouillables
- Clic → 🔒, le groupe est protégé contre `loadC_fromObj`
- Groupes : `textes`, `couleurs`, `eclairage`, `camera`, `effets`, `hdri`, `audio`
- Implémenté via `_locks` (Set) + `_lockKeys` (table groupe → clés) + filtre au début de `loadC_fromObj`

### Transparence GLTF
- Détection auto des matériaux avec opacity<1, alphaMap, transparent flag

### FBX
- `new THREE.FBXLoader().parse(buf,'')` — **nouvelle instance à chaque parse** (évite les conflits d'état interne pour le multi-slots)
- Matériaux convertis automatiquement de `MeshPhongMaterial` → `MeshStandardMaterial` (compatible popup OBS)

### Popup OBS
- Fenêtre propre, ratio adapté au sélecteur courant
- Touche F = toggle plein écran (avec try/catch)
- Touches 1-9 = changement de scène avec transition
- Modes caméra : tous les 5 modes sont baked dans le template (continuous, cuts, orbit, travel, handheld)
- Conseil : Chrome masque la barre d'URL

## Particularités et pièges connus

### Valeurs divisées dans getConfig → loadC_fromObj
Certains sliders ont une plage entière mais getConfig divise pour stocker. loadC_fromObj doit remultiplier.

**÷10 :** `tOp`→×100, `gExp`, `keyI`, `rimI`, `ambI`, `l2I`, `cLookY`, `bloomI`, `dofAp`, `hdriInt`, `animDur`, `cutDur`, `orbitH`, `pulseBright`, `flashPow`, `transDur`

**÷100 :** `pulseDecay`, `dustSize`, `slots.scale`

**÷1000 :** `fogD`

**slots :** `posX/posZ/posY` ÷10, `scale` ÷100

### mW (largeur scène)
Calculé depuis l'écart entre les **positions** des slots (pas les bounding boxes). Sinon, agrandir un modèle fait reculer la caméra.

### Scale depuis le bas
Bounding box recalculé + `position.y = -box.min.y` pour garder le modèle au sol.

### buildPop()
Template string géant (~800 lignes) qui génère tout le HTML/CSS/JS de la popup. Les valeurs de config sont injectées au moment de la génération. Tout changement de feature doit être **dupliqué** dans buildPop(). Les modes caméra orbit/travel/handheld sont injectés conditionnellement via des ternaires imbriqués.

### FBX multi-slots
Toujours créer `new THREE.FBXLoader()` — ne jamais réutiliser une instance globale.

### FBX dans popup
Les `MeshPhongMaterial` du FBXLoader sont convertis en `MeshStandardMaterial` au chargement (sinon noir dans la popup).

### Changement de modèle entre scènes
Fonctionne via `modelStore` (Map en mémoire session). Si le modèle n'a pas été chargé dans la session courante, le slot n'est pas rechargé.

### modelStore
Clés : `nom_fichier` pour les modèles de slot, `'crowd3D:nom'` pour la foule 3D, `'env:nom'` pour l'environnement.

### Token GitHub
Format `ghp_xxxxx` — à usage unique, révoquer après chaque push.

## Features livrées (v14 → v23.6)

| Version | Feature |
|---------|---------|
| v14 | Fonds custom image/vidéo sur sphère 3D |
| v15 | Audio mel-scale, mini-map, pulsation spectral flux |
| v16 | Polices système, fix fond sombre |
| v17 | GPU tiling, flou par downscale |
| v18 | Optimisation GPU : 30fps cap, pause hidden |
| v19 | Foule SVG (8 silhouettes), flashes paparazzi |
| v20 | Scale depuis le bas, Position Y, animations GLB, transparence GLTF, brouillard |
| v21 | Compte à rebours, filtres couleur CSS, popup OBS propre |
| v22.0 | Transitions entre scènes (fade/slide/zoom) |
| v22.1 | Bug fix modèles scènes + export JSON complet avec fichiers |
| v22.2 | Support FBX (loader + popup + détection type) |
| v22.3 | Foule 3D InstancedMesh (GLB/FBX/STL) + couleur + variation |
| v22.4 | Mode Environnement 3D (5 presets caméra intérieur) |
| v22.5 | Fix FBX multi-slots + matériaux noirs popup |
| v23.0 | Modes caméra : Orbite, Travelling A→B, Handheld |
| v23.1 | Export vidéo .webm (MediaRecorder) |
| v23.2 | Mode responsive : sélecteur ratio Libre/16:9/4:3/9:16/1:1 |
| v23.3 | Annuler/Refaire (Ctrl+Z / Ctrl+Y, 50 niveaux) |
| v23.4 | Verrouillage de groupes de paramètres (7 groupes) |
| v23.5 | Audit : logo+particule dans export, viewRatio, init undo, fbxLoader nettoyé |
| v23.6 | Audit : popup orbit/travel/handheld, updateUI loadC, S._ nettoyés |

## Roadmap — Features à développer (suggestions)

Il n'y a plus de roadmap formelle. Suggestions possibles pour la suite :

### 1. ⏱️ Timeline auto
Enchaîner des scènes avec timing automatique. Ex : scène 1 pendant 30s → scène 2 → boucle. Interface : liste ordonnée + durée par scène + bouton play/pause.

### 2. 🎬 Transitions + Timeline
Va de pair avec la timeline. Fondu, slide, zoom entre les scènes enchaînées automatiquement.

### 3. 🌍 Amélioration mode Environnement
Actuellement GLB uniquement. Pistes : support HDRI sphérique comme background, caméra first-person avec touches directionnelles, navigation manuelle dans OBS.

### 4. 🎨 Presets de scène rapides
Bibliothèque de configs prêtes à l'emploi (dark/light/neon/minimal) avec prévisualisation miniature.

### 5. 📝 Export HTML standalone
Générer un HTML complet avec toutes les ressources embarquées, prêt à être capturé par OBS sans avoir besoin de la popup.

## Workflow de développement

1. **Copie de travail** : `cp /home/claude/MLI-3D/index.html /home/claude/w.html`
2. **Éditer** w.html avec str_replace ou python3 pour les remplacements complexes
3. **Syntax check** :
   ```bash
   START=$(grep -n "^<script>" /home/claude/w.html | head -1 | cut -d: -f1)
   START=$((START+1))
   END=$(grep -n "^function buildPop" /home/claude/w.html | cut -d: -f1)
   END=$((END-1))
   sed -n "${START},${END}p" /home/claude/w.html > /tmp/test.js
   node -c /tmp/test.js
   ```
   ⚠️ Ne pas vérifier `buildPop` — c'est un template string avec `${...}` qui n'est pas du JS valide seul
4. **Copier vers output** : `cp w.html /mnt/user-data/outputs/MLI-3D-vXX.html`
5. **Copier vers repo** : `cp w.html /home/claude/MLI-3D/index.html`
6. **Commit + push** :
   ```bash
   cd /home/claude/MLI-3D
   git config user.email "votre@email.com"
   git config user.name "fablabloritz-coder"
   git remote set-url origin https://fablabloritz-coder:TOKEN@github.com/fablabloritz-coder/MLI-3D.git
   git add -A
   git commit -m "emoji v23.X — description"
   git push origin main
   ```
7. **Partager** : `present_files` avec le fichier dans /mnt/user-data/outputs/
8. **⚠️ Révoquer le token immédiatement**

## Convention de nommage des versions
- **vX.0** = feature majeure
- **vX.Y** = itérations/fixes sur la même feature
- Commits : ⚡ perf, 👥 foule, 📐 géométrie, 🌫️ rendu, 🔧 fix, ⏳ timer, 🎨 visuel, 🪟 fenêtre, 📋 config, 🎬 transition, 📦 format, 🌍 env, 📹 caméra, 💾 export, 📱 responsive, ↩ undo, 🔒 lock, 🔍 audit

## Préférences de l'auteur
- Interface en français
- Solutions zéro-budget, open-source
- Pas de framework, pas de build, fichier unique
- Privilégier la simplicité et la performance
- Répondre en français
- Fournir le fichier HTML téléchargeable + push GitHub à chaque étape
- Pour les remplacements complexes dans buildPop (template strings imbriquées), utiliser `python3` plutôt que `str_replace`
