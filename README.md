# MLI-3D — Model Live Intro 3D

**Générateur d'intro live 3D pour le streaming**

MLI-3D est une application web autonome qui permet de créer des intros animées professionnelles en 3D pour vos livestreams YouTube, Twitch ou toute autre plateforme. Chargez un modèle 3D, personnalisez l'éclairage, les textes, les effets — puis lancez la scène dans une fenêtre dédiée à capturer avec OBS.

![MLI-3D](https://img.shields.io/badge/version-1.0-c9a87c) ![License](https://img.shields.io/badge/license-MIT-blue) ![Three.js](https://img.shields.io/badge/Three.js-r128-green)

---

## Fonctionnalités

### Modèle 3D
- Chargement de fichiers `.glb` / `.gltf` par glisser-déposer
- Orientation manuelle (0°–360°)
- Auto-rotation réglable
- **6 matériaux** : Original, Bronze, Marbre, Chrome, Or, Noir mat
- Statue par défaut intégrée si aucun modèle n'est chargé

### Éclairage
- **8 presets** : Classique, Dramatique, Froid, Coucher de soleil, Nuit bleue, Plat (soft), Clair-obscur, Néon
- Contrôle individuel : couleur et intensité de la lumière principale, du contre-jour, et de l'ambiante
- Orbite de l'éclairage (rotation du rig de lumières autour du modèle)

### Environnement HDRI
- **8 environnements Poly Haven** (CC0) intégrés : studios et extérieurs
- Upload de **fichiers .hdr personnalisés**
- Affichage en fond 360° (optionnel)
- Intensité des reflets réglable
- Brouillard désactivé automatiquement en mode fond HDRI

### Caméra cinématique
- Trajectoire en **spline Catmull-Rom** continue (pas de coupure entre les plans)
- **5 presets** : Contemplatif, Standard, Dynamique, Intime, Panoramique
- Vitesse et distance ajustables
- Léger tremblement organique (effet "handheld")

### Textes & Logo
- Titre, sous-titre, textes bas gauche/droite, badge live
- **7 polices** : Cormorant Garamond, Playfair Display, Cinzel, Montserrat, Bebas Neue, Oswald, Raleway
- Tailles individuelles réglables
- Logo par glisser-déposer (PNG transparent recommandé)
- Positionnement du logo aux 4 coins (intégré au layout, pas superposé)

### Effets visuels
- **Bloom** (halo lumineux cinématique) avec intensité réglable
- Barres cinématiques (hauteur ajustable, padding auto)
- Grain pellicule
- Vignette
- Particules de poussière flottantes
- Lignes décoratives

### Sol
- Couleur personnalisable
- Toggle de visibilité
- Mode réfléchissant (miroir)
- Auto-masquage quand le HDRI est affiché en fond

### Sauvegarde
- Sauvegarde/chargement de configurations nommées (localStorage)
- Tous les réglages conservés sauf les fichiers (modèle, logo, HDRI)

---

## Utilisation

### Installation

Aucune installation requise. C'est un fichier HTML unique et autonome.

```
git clone https://github.com/fablabloritz-coder/MLI-3D.git
```

Puis ouvrir `index.html` dans **Google Chrome** (recommandé).

Ou pour une utilisation en ligne sans installation/téléchargement
```
https://fablabloritz-coder.github.io/MLI-3D/
```

Pour encore plus de flexibilité et enlever l'interface inutile d'un nagivateur (barre d'url), il est possible de créer un raccourcie pointant (Cible) vers ceci :

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" --app=https://fablabloritz-coder.github.io/MLI-3D
```
La raccourcie lancera (en tout cas avec Google Chrome) la dernière version de MLI-3D en mode Application

### Workflow OBS

1. Ouvrez `index.html` dans Chrome
2. Configurez votre scène (modèle, textes, éclairage, effets...)
3. Cliquez **"Lancer la fenêtre OBS"**
4. Dans la popup, **cliquez pour passer en plein écran** (masque la barre d'URL)
5. Dans OBS : ajoutez une source **"Capture de fenêtre"** pointant sur la popup
6. Lancez votre stream !

> **Astuce** : Gardez la fenêtre principale ouverte — vous pouvez ajuster les réglages en temps réel pendant que la popup tourne.

### Où trouver des modèles 3D ?

- **[Sketchfab](https://sketchfab.com)** — Nombreux modèles gratuits et payants (télécharger en `.glb`)
- **[Poly Haven](https://polyhaven.com/models)** — Modèles CC0
- **[Pixabay](https://pixabay.com/3d-models/search/glb/)** - Modèle glb libre de droit
- **Photogrammétrie** — Scannez un objet réel avec Polycam, Luma AI, ou RealityCapture

### Où trouver des HDRIs ?

- **[Poly Haven HDRIs](https://polyhaven.com/hdris)** — Des centaines de HDRIs CC0 gratuits
- Téléchargez en format `.hdr` (1K ou 2K suffisent)
- Glissez-déposez dans la zone "HDRI personnalisé" de l'application

---

## Technologies

- **[Three.js](https://threejs.org/)** r128 — Moteur 3D WebGL
- **GLTFLoader** — Chargement de modèles `.glb`
- **RGBELoader** — Chargement d'environnements `.hdr`
- **UnrealBloomPass** — Post-processing bloom
- **PMREMGenerator** — Réflexions d'environnement
- **Poly Haven** — Environnements HDRI (CC0)
- **Google Fonts** — Typographies

---

## Structure

```
MLI-3D/
├── index.html    ← Application complète (fichier unique)
├── README.md     ← Ce fichier
└── LICENSE        
```

Tout est contenu dans un seul fichier HTML. Pas de build, pas de dépendances à installer, pas de serveur requis.

---

## Licence

MIT — Libre d'utilisation, modification et redistribution.

---

## Crédits

- Développé pour le **Lycée Henri Loritz** (Nancy)
- Environnements HDRI par **[Poly Haven](https://polyhaven.com)** (CC0)
- Polices par **[Google Fonts](https://fonts.google.com)**

---

*Projet créé au FabLab du Lycée Henri Loritz*
