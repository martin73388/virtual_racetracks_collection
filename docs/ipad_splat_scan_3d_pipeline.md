# Splat et scan 3D d'un environnement avec un iPad — guide et pipeline recommandé

> Étude réalisée le **19 septembre 2026**. Prix, versions et fonctionnalités ont été vérifiés à cette date sur les pages officielles (App Store, sites éditeurs, GitHub, documentation). Ce marché bouge très vite : revérifiez les tarifs avant d'acheter quoi que ce soit.
>
> Méthode : un workflow multi-agents a balayé le web sur 11 angles (apps iPad, capture avec poses ARKit, cloud gratuit, traitement local, viewers, plateforme Apple, bonnes pratiques, tarifs, retours d'utilisateurs, réutilisation ROS), a identifié 110 solutions, puis un « vérificateur sceptique » par solution a tenté de réfuter chaque affirmation sur les sources officielles (34 fiches vérifiées, ~1 260 pages ouvertes). Les points restés incertains sont signalés **« à vérifier »**. Voir l'annexe pour les limites de l'étude.

---

## 1. Réponse courte

**Pipeline 100 % gratuit, tout sur l'iPad (recommandé pour démarrer) :**

1. **Scaniverse** (Niantic Spatial, gratuit, iPadOS 16.6+, LiDAR facultatif) en mode « Classic » : le splat est calculé **sur l'iPad** en 1 à 2 minutes, sans compte ni abonnement, exportable en **PLY / SPZ**. La même app produit un **maillage texturé** (mode LiDAR sur iPad Pro, mode photogrammétrie sur les autres iPad), exportable en **OBJ / FBX / USDZ / LAS** (GLB cité par l'App Store).
2. Scannez **pièce par pièce ou zone par zone**, 1 à 3 minutes par scan (l'app avertit au-delà de 180 s ; les très gros scans échouent).
3. Nettoyez, compressez et publiez le splat gratuitement dans **SuperSplat** (éditeur web MIT, hébergement gratuit sur superspl.at).
4. Avec un iPad Pro (LiDAR), ajoutez **KIRI Engine Basic** (gratuit) pour le mode « Scene Scan » LiDAR local (mesh OBJ/USDZ) et **Modelar** (gratuit) si vous voulez un nuage de points E57/LAS.

**Pour monter en qualité sans payer, il faut un ordinateur :**

- **Mac Apple Silicon** : filmez en 4K avec l'iPad (exposition verrouillée), puis entraînez le splat dans **3D Splat App** (gratuit, Mac App Store, Metal) ou **COLMAP + Brush** (gratuits, open source).
- **PC Windows avec GPU NVIDIA (ou AMD récent)** : **RealityScan 2.2** (gratuit sous 1 M$ de revenus) donne un maillage de n'importe quelle taille et exporte les caméras au format COLMAP, que **Brush**, **OpenSplat** ou **LichtFeld Studio** transforment en splat.
- **Sans ordinateur puissant** : **Splatware Free** (cloud, modèle « Lite » illimité, 3 exports/jour) ou un notebook **Kaggle / Google Colab** gratuit avec COLMAP + OpenSplat/gsplat.

**Ce qu'il ne faut PAS croire en 2026 :** Polycam gratuit **ne crée plus de splats** (export GLTF uniquement) ; KIRI Engine réserve **tout** le 3DGS au plan Pro ; Luma AI a abandonné la capture 3D (app iPhone uniquement, Genie fermé le 1er janvier 2026) ; 3D Scanner App est passée sous abonnement ; Postshot est Windows/NVIDIA uniquement et son plan gratuit n'exporte pas le splat.

**Le seul achat qui change vraiment la donne si vous ne voulez pas d'ordinateur :** KIRI Engine Pro (49,99 $/an via l'App Store, 79,99 $/an sur le site) pour obtenir **splat + maillage issu du splat** d'une même vidéo, sans LiDAR.

---

## 2. Comprendre en 2 minutes

### Splat (3DGS) ou scan 3D (mesh) ?

| | Gaussian splat (3DGS) | Scan 3D classique (maillage / nuage de points) |
|---|---|---|
| Ce que c'est | Des millions de « gaussiennes » colorées qui reproduisent l'apparence photoréaliste d'une scène, y compris reflets et végétation | Une surface triangulée texturée (OBJ, GLB, USDZ…) ou un nuage de points (PLY, LAS, E57) avec une géométrie mesurable |
| Points forts | Rendu très réaliste, tolère les matériaux difficiles, léger en SPZ/SOG | Géométrie exploitable : mesures, collisions, impression 3D, simulateurs (Gazebo), CAO |
| Points faibles | Pas de vraie surface : mesure et collision difficiles, conversion en mesh imparfaite | Échoue sur vitres, miroirs, murs unis ; textures moins réalistes |
| Formats | PLY (lourd), SPZ (≈10× plus petit), SOG (15-20× plus petit), SPLAT, KSPLAT | OBJ, FBX, GLB/GLTF, USDZ, STL, DAE ; nuages PLY, LAS, E57, PCD |

Les deux sont complémentaires : capturez **les deux** lors de la même visite (LiDAR ou photogrammétrie pour la géométrie et l'échelle, vidéo/photos pour le splat).

### Ce que votre iPad sait faire

| Modèle d'iPad | LiDAR | Conséquence |
|---|---|---|
| **iPad Pro** 11" 2e gén. / 12,9" 4e gén. (2020, A12Z), M1 (2021), M2 (2022), M4 (2024), M5 (2025) | **Oui** (portée ≈ 5 m) | Mesh LiDAR sur l'appareil, plans RoomPlan, échelle réelle, poses ARKit exportables (Polycam raw, SplatKing, Record3D). Object Capture guidé d'Apple exige en plus une puce A14+, donc iPad Pro 2021 ou plus récent |
| iPad Air (toutes gén., y compris M4 2026), iPad mini (A17 Pro), iPad (A16) | **Non** | Splat et photogrammétrie par photos/vidéo uniquement (Scaniverse « Ignore LiDAR », RealityScan Mobile, KIRI Photo Scan, Polycam Space Mode sans LiDAR) ; pas d'échelle réelle automatique |

Autres faits utiles :

- L'iPad Pro M4/M5 n'a qu'une **seule caméra arrière 12 MP grand-angle** (pas d'ultra grand-angle) : il faut plus de passes qu'avec un iPhone Pro. La vidéo 4K 30 fps est parfaitement adaptée.
- Pas de ProRAW sur iPad ; des JPEG/HEIC bien exposés ou une vidéo 4K suffisent largement.
- **iPadOS 27** (sorti le 14 septembre 2026) et **RealityKit 27** ajoutent le **rendu natif des Gaussian splats** (API `GaussianSplatComponent`, déjà active sur visionOS 27, annoncée « dans une prochaine version » pour iOS/iPadOS/macOS 27 dans les notes de version). Apple ne fournit **aucun outil de capture** de splats : la création reste du ressort d'apps tierces. OpenUSD 26.03 (mars 2026) a standardisé un schéma USD pour les splats.
- Prix Apple France au 19/09/2026 (hausse de juin 2026) : iPad Pro M5 11" **à partir de 1 319 €** (13" : 1 669 €), reconditionné Apple iPad Pro 11" M4 256 Go **1 019 €** ; iPad Air M4 819 € (sans LiDAR) ; iPad A16 509 € ; iPad mini A17 Pro 689 €. Un iPad Pro d'occasion 2020-2022 est la voie LiDAR la moins chère.

---

## 3. Panorama des solutions vérifiées

Légende : « sur appareil » = calcul sur l'iPad ; « cloud » = envoi obligatoire sur les serveurs de l'éditeur ; « ordinateur » = Mac/PC nécessaire.

### 3.1 Apps iPad

| Solution | Splat | Mesh / scan | Traitement | iPad / LiDAR | Gratuit ? Prix 2026 | Exports | Verdict environnement |
|---|---|---|---|---|---|---|---|
| **Scaniverse** (Niantic Spatial) v5.2.8 | Oui, sur l'appareil, illimité (« Classic ») ; cloud « New Scaniverse » à crédits | Oui : mesh LiDAR (iPad Pro) ou photogrammétrie (sans LiDAR), nuage LAS | Sur appareil (Classic) ou cloud | iPadOS 16.6+, puce A12+ ; LiDAR facultatif. Splat garanti sur iPhone 12+ ; sur iPad A12/A13 (iPad 8, mini 5, Air 3) **à tester** | **Gratuit** et illimité sur l'appareil. Cloud : Free 20 000 crédits/mois (≈ 11 min de capture), Plus 20 $/mois (200 $/an), Pro 50 $/mois (500 $/an, droits commerciaux) | Splat PLY, SPZ ; mesh OBJ, FBX, USDZ, LAS (+ GLB) ; cloud : + USDZ splat+mesh (payant). Pas d'export des images/poses | **Le meilleur point d'entrée gratuit.** Scans de 1-3 min (max 5 min / 500 m²) : découper une maison ou un circuit en zones ; fusion multi-scans uniquement en cloud payant |
| **Polycam** v7.0.2 | Oui mais **cloud et payant** (Basic et plus) ; Free : non | Oui : Space Mode LiDAR (mesh + plans), Space Mode sans LiDAR (mesh), photogrammétrie cloud | Sur appareil (mesh LiDAR/Space) ou cloud | iPadOS 18+ ; LiDAR : iPad Pro 2020+ ; Space sans LiDAR : iPad mini 6, iPad 10, Air 5+ | Free 0 $ (150 images, **export GLTF seul**, pas de splat) ; Basic 150 $/an (12,50 $/mois) ou 30 $/mois ; Business 400 $/an ; plan « Pro » supprimé | Free : GLTF ; Basic : OBJ, FBX, DAE, STL, USDZ ; Business : + PLY, LAS, XYZ, DXF, **Splat PLY** (à confirmer dans l'app) ; raw data (images + poses + profondeur) en Developer Mode sur LiDAR, tous plans | Excellent produit mais sa version gratuite est une démo. Utile gratuitement pour l'export « raw data » (poses ARKit) vers un pipeline PC |
| **KIRI Engine** v4.2.7 | Oui, cloud, **Pro uniquement** (vidéo ≤ 1080p / 3 min ou 20-300 photos) | Oui : Photo Scan gratuit (150 photos), LiDAR Scan gratuit et local (Room / Scene / Object), 3DGS→Mesh 3.0 (Pro) | Cloud (photo, 3DGS) ; sur appareil (LiDAR) | iPadOS 15+ ; LiDAR requis seulement pour les modes LiDAR (iPad Pro 2020+) | Basic gratuit, exports illimités sans filigrane ; Pro 17,99 $/mois ou 79,99 $/an (site), **49,99 $/an via l'App Store** | Mesh OBJ, FBX, STL, GLB, GLTF, USDZ, PLY, XYZ ; splat PLY ; raw dataset Nerfstudio (LiDAR + Pro) | Très bon pour le mesh gratuit (Scene Scan LiDAR sans limite documentée). Le splat est payant mais c'est l'option « splat + mesh sans ordinateur » la moins chère |
| **RealityScan Mobile** (Epic) v1.9 | Non | Oui : photogrammétrie cloud, 20-300 photos, orienté objets | Cloud | iPadOS 16+, tout iPad, sans LiDAR | **Gratuit**, tout export gratuit, usage commercial autorisé | OBJ + USDZ direct ; GLB/glTF via Sketchfab | Objets et gros objets ; une pièce ou un circuit dépasse les 300 photos : scanner par zones |
| **Teleport by Varjo** v2.11.1 | Oui, cloud, conçu pour les lieux (pièces → quartiers, jusqu'à 100 M de splats) | Non | Cloud | iPadOS 17+, sans LiDAR | 5 captures d'essai (sans export) ; puis « pay per capture » à partir de 30 $ ; export PLY réservé aux payants | PLY (payant), vidéo fly-through | La meilleure solution cloud pour un **grand extérieur** (drone accepté), mais payante |
| **Gaussian SplatKing** v1.4.1 | Ne calcule pas le splat : app de **capture** (photo, vidéo 4K, mode LiDAR) | Non (nuage de points LiDAR seulement) | Sur appareil (données), entraînement sur ordinateur | iPadOS 18+ ; mode LiDAR = iPad Pro ; autres iPad : mono-objectif | **Gratuit**, sans achat intégré (dons) | Mode LiDAR : dossier **COLMAP prêt à entraîner** (poses ARKit) ; photo/vidéo : images + métadonnées | Idéal pour sauter l'étape COLMAP sur iPad Pro ; projet d'un seul développeur (pérennité à surveiller) |
| **Voxelio 3D LiDAR Scanner** v1.3.0 | Oui, sur l'appareil, **petites scènes seulement** ; export du dataset (poses) pour PC | Oui : mesh OBJ, nuage PLY, RoomPlan, Space Mode | Sur appareil | iPadOS 18+, **LiDAR requis** | Gratuit avec quota hebdomadaire non publié et filigrane sur plans/vidéos ; Pro Lifetime 179,99 € ou 2,99 / 9,99 / 99,99 € | SPZ, OBJ+MTL, STL, PLY, USDZ, plan 2D | Prometteur mais très jeune (juillet 2026) ; pas pour un environnement en splat |
| **Modelar** v3.1.2 | Non | Oui : nuage de points et mesh LiDAR temps réel | Sur appareil | iPadOS 18.6+, **iPad Pro LiDAR uniquement** | **Gratuit**, sans achat intégré | Mesh GLB, USDZ, OBJ, STL, PLY ; nuage **E57, LAS**, PLY, CSV | Seule app gratuite trouvée avec E57/LAS : pièces et bâtiments parcourus à pied |
| **3D Scanner App** v2.5 | Non | Oui (LiDAR, photo, RoomPlan) | Les deux | iPadOS 14.1+ ; LiDAR pour le mode LiDAR | Téléchargement gratuit mais **abonnement de fait depuis mai 2026** (34,99 € ou 79,99 €/an en France), app revendue en 2025 | USDZ, OBJ, GLTF, GLB, DAE, STL, PTS, PCD, PLY, XYZ, LAS, DXF | À éviter désormais (avis massivement négatifs sur le paywall) |
| **Reality Composer / Object Capture** (Apple) | Non | Oui, **objets uniquement** (USDZ, niveau « reduced ») | Sur appareil | iPad Pro 2021+ (LiDAR + A14) | Gratuit, mais plus mis à jour depuis octobre 2023 | USDZ | Inadapté aux environnements (mode « area » limité à ≈ 1,8 m) |
| **Record3D** v1.11 | Non (capture RGBD + poses ARKit) | Nuage/mesh image par image, pas de fusion | Ordinateur | iPad Pro LiDAR | Basic 4,99 €, Full 5,99 € (achat unique) | EXR + JPG + poses → `ns-process-data record3d` | Source de poses fiable pour Nerfstudio, LiDAR obligatoire |
| **Spatial Fields** v1.3.0 | Visionneuse seulement (PLY, SPZ, LCC, USDZ) avec **mode AR** | Non | Sur appareil | iPadOS 18.6+ | 29,99 € (achat unique) | Aucun | Optionnel : le plus beau viewer AR natif iPad ; plantages signalés sur gros splats |

Écartés après vérification : **Luma 3D Capture** (iPhone seulement, plus de splat dans l'app, Genie fermé) ; **Splatcatcher** (ex-SplatCam, iPhone Pro seulement) ; **NeRFCapture** (abandonné depuis 2023) ; **Spectacular Rec** (app figée en 2024, SDK Windows/Linux non commercial) ; **PhotoCatch** (maintenance minimale, abonnements opaques).

### 3.2 Traitement sur ordinateur

| Solution | Splat | Mesh / scan | Plateforme / GPU | Gratuit ? Prix 2026 | Exports | Verdict |
|---|---|---|---|---|---|---|
| **3D Splat App** (Laan Labs) v0.3.1 | Oui, entraînement local Metal depuis photos, vidéo ou vidéo 360 (Fast ≈ 2 min, Medium ≈ 5, HD ≈ 20) | Non | **Mac M1+**, macOS 15.6+, 16-32 Go RAM | **Gratuit** (Mac App Store, sans achat intégré) | PLY, PLY compressé, SPZ, SOG, vidéo | Le maillon « splat gratuit sur Mac » le plus simple (COLMAP intégré) ; app jeune (mars 2026), sans doc officielle |
| **Brush** (A. Brussee) v0.3.0 | Oui, tout GPU (Apple, AMD, NVIDIA, Intel), visualisation en direct, jusqu'à 10 M de splats | Non | Mac Apple Silicon, Windows, Linux, Chrome/Edge | **Gratuit** (Apache-2.0) | PLY (lisible par SuperSplat) | Exige des poses COLMAP ; ≈ 3 h 30 pour 30 000 itérations sur un MacBook Air (témoignage unique) ; binaires figés à v0.3.0, le code récent (1.0.0) se compile |
| **OpenSplat** (WebODM) v1.2.2 | Oui (CLI), entrée COLMAP / ODM / Nerfstudio | Non (voir ODX/WebODM pour le mesh) | Mac (Metal), Linux (CUDA/ROCm), Windows | Gratuit à compiler (AGPL) ; **binaire Windows 29 $** (une fois) | PLY, SPLAT, SPZ, RAD | Solide, sans interface ; ≈ 2 Go de VRAM par million de gaussiennes |
| **LichtFeld Studio** (MrNeRF) v0.5.3 | Oui, GUI complète (entraînement, édition, LOD, HTML) | Non | **Windows/Linux, NVIDIA RTX 20+ (CC 7.5+)** | Source gratuite (GPLv3, à compiler) ; binaire Windows via portail **30 $ minimum** ; ancien binaire v0.4.2 gratuit | PLY, SOG, SPZ, HTML, USD, RAD | La référence open source sur PC NVIDIA, très active |
| **Postshot** (Jawset) v1.1.69 | Oui, GUI, SfM intégré | Non | **Windows 10+, NVIDIA RTX 2060+ uniquement** | Free : non commercial, filigrane, **pas d'export PLY/SPZ** ; Indie 17 €/mois (204 €/an) ou 26 €/mois ; Studio 39 €/mois (468 €/an) | Indie : PLY, SPZ v4, HTML ; plugins Unreal 5.4-5.8 et After Effects | Très confortable, mais inutilisable gratuitement pour partager un splat |
| **RealityScan 2.2 desktop** (ex-RealityCapture, Epic) | Non (exporte les caméras COLMAP pour un entraîneur) | **Oui, toute échelle** (photos, vidéo, laser, drone) | **Windows** (NVIDIA CUDA ou AMD RDNA 3/4 depuis juin 2026), Linux CLI expérimental, pas de Mac | **Gratuit** sous 1 M$ de revenus annuels (1 250 $/siège/an sinon) ; compte Epic | OBJ, PLY, GLB, STL, USDZ, FBX, DAE, LAS, XYZ… + COLMAP, CSV/PLY (Postshot) | La voie mesh gratuite la plus puissante pour un bâtiment ou un circuit |
| **COLMAP 4.2.0** (+ GLOMAP intégré) | Non (poses + nuage épars pour Brush / OpenSplat / LichtFeld / Nerfstudio) | Nuage dense et mesh **avec CUDA/ROCm seulement** | Windows (binaires), **Mac arm64** (binaire, Homebrew), Linux | **Gratuit** (BSD) | Modèle sparse BIN/TXT, PLY | Brique universelle ; le « global mapper » (GLOMAP) est 10-100× plus rapide que l'incrémental |
| **Apple Object Capture** (Reality Composer Pro, PhotoCatch) | Non | Oui, objets et zones ≤ ≈ 1,8 m ; jusqu'à 2 000 images sur Mac | Mac (Apple Silicon) | Gratuit (Xcode) | USDZ, OBJ | Excellent pour un objet ou un mur, pas pour une pièce entière |
| **Nerfstudio** (splatfacto) v1.1.5 | Oui | Non depuis splatfacto | Linux/Windows, NVIDIA (≈ 6 Go VRAM) | Gratuit | PLY | **Projet quasi à l'arrêt** (dernière release nov. 2024, dernier commit juil. 2025) ; ses convertisseurs `ns-process-data polycam / record3d` restent utiles |
| **Agisoft Metashape Standard** 2.3.2 | Non (export caméras COLMAP) | Oui, robuste, toute échelle | Windows, **Mac**, Linux | 179 $ (licence perpétuelle), essai 30 jours | Nombreux formats + COLMAP | La photogrammétrie payante la moins chère qui tourne sur Mac |

### 3.3 Cloud

| Solution | Ce qu'on obtient | Gratuit ? | Limites du gratuit | Verdict |
|---|---|---|---|---|
| **Splatware** (Berlin) | Splat entraîné dans le cloud depuis vidéos/photos iPad, éditeur, viewer, lien de partage | Free : modèles « Lite » illimités (≤ 1,5 M gaussiennes, 8-12 min) ; Creator 9,95 € le 1er mois puis 19,95 €/mois ; Pro 79,95 €/mois | 5 projets, 3 exports/jour, 500 images ou 3 vidéos (0,5 Go) par projet, pas de mesh, plan « modifiable sans préavis » | La seule voie **cloud gratuite sans ordinateur** vérifiée pour transformer une vidéo iPad en splat exportable (PLY, .SPLAT) ; qualité « Lite » |
| **Google Colab** | Notebook GPU (T4 16 Go) pour COLMAP + OpenSplat / gsplat / 3DGS | Gratuit (GPU non garanti, ≤ 12 h) ; Pro 9,99 $/mois (100 unités), Pro+ 49,99 $ | Coupures, GPU parfois indisponible, pas d'exécution en arrière-plan | Faisable pour une pièce ou une façade (100-300 images réduites) ; notebook Nerfstudio officiel cassé, préférer OpenSplat |
| **Kaggle Notebooks** | Idem avec 2 × T4 | Gratuit ≈ 30 h GPU/semaine, sessions 12 h, vérification SMS | 4 cœurs CPU (COLMAP lent), 20 Go de sortie | Plus généreux que Colab en heures ; P100 retiré le 15/09/2026 |
| **Niantic Spatial cloud** (New Scaniverse) | Splats cloud « plus réalistes », fusion multi-scans en « Sites », 360°, USDZ splat+mesh | Free 20 000 crédits/mois (≈ 11 min) ; Plus 20 $/mois ; Pro 50 $/mois | 30 crédits/s de capture ; 360° et USDZ réservés à Plus/Pro ; droits commerciaux en Pro | Le prolongement naturel de Scaniverse pour un bâtiment entier |
| RunPod / Vast.ai (non revérifiés) | GPU à l'heure (RTX 3090/4090 ≈ 0,12-0,34 $/h) | Payant à l'usage | Compétences Docker/SSH | < 1 € par scène si l'on sait s'en servir |

### 3.4 Visualiser, éditer, convertir

| Solution | Rôle | Gratuit ? | Notes |
|---|---|---|---|
| **SuperSplat 3.3** (PlayCanvas) | Nettoyage, crop, couleur, rendu, export, publication superspl.at | Gratuit (MIT), compte PlayCanvas gratuit pour publier | Navigateur WebGPU (Chrome/Edge, Safari 26+, donc Safari sur iPadOS 26/27 en principe, sans validation officielle) ; import PLY/SPZ/SOG/SPLAT/KSPLAT/LCC ; export PLY, SOG, SPZ, HTML autonome |
| **splat-transform** (CLI PlayCanvas) | Conversion PLY/SPZ/SOG/SPLAT/KSPLAT/LCC, fusion, LOD, transformation | Gratuit (MIT) | `npm install -g @playcanvas/splat-transform` (non revérifié) |
| **SPZ 4** (Niantic Spatial) | Format compressé ouvert, ≈ 10× plus petit que PLY | Gratuit | Convertisseur web local sur nianticspatial.com/spz-converter |
| **Spark 2.0** (World Labs, three.js) | Viewer web à intégrer dans un site | Gratuit (MIT) | Fonctionne sur iOS (WebGL2) ; remplace GaussianSplats3D (non maintenu). Non revérifié |
| Unity / Unreal / Blender / Godot | gsplat-unity (maintenu) ; XGRIDS LCC-3DGS plugin (UE 5.1-5.8, gratuit) ; add-on **3DGS Render by KIRI Engine** (gratuit) puis import natif prévu dans Blender 5.3 ; GDGS | Gratuits | Non revérifiés individuellement ; le plugin Unreal de Luma est abandonné |

---

## 4. Pipeline recommandé (gratuit, tout sur l'iPad)

Objectif : obtenir, en moins d'une heure, un splat et un maillage d'une pièce (ou d'une zone extérieure de 200 à 500 m²), sans ordinateur ni abonnement.

### Étape 0 — Préparer (5 min)

- Installez **Scaniverse** (App Store, gratuit). Sur iPad Pro, installez aussi **KIRI Engine** (compte gratuit « Basic ») et, si vous voulez des nuages E57/LAS, **Modelar**.
- Batterie > 80 %, 10 à 15 Go libres, lentille nettoyée, mode Ne pas déranger.
- Allumez toutes les lumières, fermez les stores (ou sortez par ciel couvert), retirez les personnes et les objets mobiles de la zone. Sur les murs blancs, collez quelques repères texturés temporaires (post-it, affiches) pour aider le suivi.
- Découpez mentalement la scène : **une capture par pièce** (ou par tronçon de 30-50 m en extérieur), avec un chevauchement visible aux portes et aux transitions.

### Étape 1 — Capturer le splat avec Scaniverse (3 à 5 min par zone)

1. Ouvrez Scaniverse, choisissez le mode **Splat**. Sur un iPad Pro, laissez le LiDAR actif (meilleurs résultats) ; sur un iPad sans LiDAR, l'app bascule d'elle-même (option « Ignore LiDAR » disponible dans les réglages).
2. Commencez sur une zone riche en détails (pas sur un mur uni).
3. **Déplacez-vous avec les pieds**, jamais en pivotant sur place : faites le tour du périmètre, puis traversez le centre, en 3 passes (à niveau, en visant vers le haut pour le plafond, vers le bas pour le sol). Variez hauteur, inclinaison et distance (restez à 0,5-3 m des surfaces).
4. Marchez lentement (5 à 10 cm/s, deux fois moins vite que naturel), gardez toujours dans le cadre des éléments déjà capturés, ralentissez et marquez une pause avant et après chaque porte.
5. **Arrêtez entre 1 et 3 minutes** (l'app avertit à 180 s ; plafond 5 min et 500 m²). Un scan court et bien couvert vaut mieux qu'un scan long avec des trous.
6. Lancez le traitement (1 à 2 min sur l'appareil, hors ligne). Vérifiez : zones floues = manque de couverture, refaites la passe correspondante. L'amélioration peut être relancée (les utilisateurs conseillent une qualité d'amélioration ≤ 5).
7. Répétez pour chaque pièce / tronçon.

### Étape 2 — Capturer le maillage (5 à 10 min par zone)

- **iPad Pro (LiDAR)** : dans Scaniverse, mode **Mesh** (portée « Range » 5 m max), mêmes trajectoires, 60 s à 3 min par pièce ; ou **KIRI Engine › LiDAR Scan › Scene Scan** (gratuit, local, hors ligne, export OBJ/USDZ) ; ou **Modelar** pour un nuage de points E57/LAS et un mesh GLB.
- **iPad sans LiDAR** : dans Scaniverse, mode **Mesh** en photogrammétrie (le mesh sera plus lisse et moins détaillé que le cloud de Polycam) ; ou **KIRI Engine › Photo Scan** (150 photos par scan, gratuit, cloud) pour un objet ou une petite zone ; ou **RealityScan Mobile** (20-300 photos, gratuit, cloud, orienté objets).
- Si vous prévoyez de retraiter plus tard, activez dans Scaniverse la conservation des données brutes (« Reprocess Scan » permet de recalculer en splat ou en mesh).

### Étape 3 — Exporter (2 min)

- Splat : **SPZ** (léger, pour le web et le partage) et **PLY** (universel, pour Blender/Unity/Unreal/SuperSplat). Enregistrez dans Fichiers ou iCloud Drive.
- Mesh : **OBJ** (+ textures) ou **USDZ** (Quick Look AR sur iPad) et **GLB** ; nuage de points en **LAS** si nécessaire.
- Partage immédiat : lien Scaniverse, ou USDZ ouvert en réalité augmentée directement dans Fichiers.

### Étape 4 — Nettoyer et publier avec SuperSplat (10 à 20 min)

1. Ouvrez `https://superspl.at/editor` (idéalement sur un Mac/PC ; Safari 26+ sur iPad prend en charge WebGPU mais l'éditeur n'est pas validé officiellement sur tablette).
2. Importez le PLY ou le SPZ. Supprimez les « floaters » (brosse, sphère, lasso, ou sélection par faible opacité dans le panneau Splat Data), recadrez avec la boîte de crop, ajustez l'orientation et l'échelle.
3. Exportez en **SOG** (15-20× plus petit) ou **SPZ**, ou générez un **viewer HTML autonome** ; ou publiez gratuitement sur superspl.at (compte PlayCanvas gratuit, scène « unlisted » par défaut, lien partageable, AR/VR WebXR).

### Étape 5 — Archiver

Conservez : les exports PLY (source), les données brutes Scaniverse, le mesh OBJ/GLB. Notez la date et le modèle d'iPad : les apps changent vite.

**Coût total : 0 €.** Durée pour une pièce : 15 à 30 min de bout en bout.

**Pièges fréquents**

- Splat troué ou pièces déconnectées : manque de recouvrement (portes, couloirs) ou trop peu de changement de point de vue.
- Dérive en extérieur ouvert : inclure des repères stables (arbres, bancs, arêtes), faire plusieurs passes.
- Plein soleil, vitres, miroirs, surfaces brillantes : les points faibles de toutes les apps mobiles.
- Très gros scan qui plante : découper en morceaux (réponse officielle de Niantic).
- Fichiers Scaniverse « Free » : usage non commercial (les droits commerciaux du cloud sont dans le plan Pro à 50 $/mois ; le mode Classic n'énonce pas de droits explicites).

---

## 5. Pipeline « qualité supérieure » (ordinateur ou GPU cloud gratuit)

Le splat calculé sur l'iPad plafonne (benchmark tiers : ≈ 85 % de la qualité d'un entraînement complet sur GPU). Pour un rendu nettement meilleur, capturez sur iPad et entraînez ailleurs.

### 5.1 Capture soignée sur iPad (commune à toutes les variantes)

- Vidéo **4K 30 fps** avec **exposition, ISO, balance des blancs et mise au point verrouillés** : appui long dans l'app Appareil photo (verrou AE/AF), ou mieux l'app gratuite **Blackmagic Camera** (iPadOS 18+, obturateur fixe 1/100 à 1/250 selon la lumière, H.265 pour limiter la taille ; 4K maximum sur iPad).
- Ou des **photos** (100 à 300 par pièce, 400 et plus pour un bâtiment) : plus de contrôle et de netteté ; chaque surface vue depuis au moins 3 positions, recouvrement 70-80 %.
- Pas d'ultra grand-angle, pas de flou, pas de zoom ; trajectoires identiques à l'étape 1.
- Sur iPad Pro, **Gaussian SplatKing** (gratuit) enregistre en mode LiDAR un **dossier COLMAP prêt à entraîner** (poses ARKit) : vous sautez l'étape la plus longue. En mode photo/vidéo il fournit des images bien contrôlées et une note de qualité par image, mais COLMAP reste nécessaire.
- Alternative sur iPad Pro : **Polycam** en mode LiDAR ou Room, avec « Developer mode » activé **avant** la capture, puis Export › Raw data (.zip, disponible sur le plan gratuit d'après le README officiel de Polyform) → `ns-process-data polycam`.

### 5.2 Variante Mac Apple Silicon (0 €)

**Chemin simple :** transférez la vidéo (AirDrop) et ouvrez-la dans **3D Splat App** (Mac App Store, gratuit, M1+, macOS 15.6+). Choisissez le preset **HD** (≈ 20 min ; une pièce détaillée peut prendre 20 min à 1 h selon le Mac). Exportez en SPZ/SOG/PLY, puis nettoyez dans SuperSplat.

**Chemin open source :** COLMAP puis Brush.

```bash
# 1) Extraire 2 images/s de la vidéo, redimensionnées à 1920 px de large
brew install ffmpeg colmap
mkdir -p scene/images
ffmpeg -i capture.mov -vf "fps=2,scale=1920:-2" -q:v 2 scene/images/frame_%04d.jpg

# 2) Poses avec COLMAP 4.x (global mapper = GLOMAP intégré, 10-100x plus rapide)
cd scene
colmap feature_extractor --database_path db.db --image_path images \
  --ImageReader.single_camera 1 --ImageReader.camera_model OPENCV
colmap sequential_matcher --database_path db.db
# frames vidéo sans EXIF : calibrer d'abord les focales
colmap view_graph_calibrator --database_path db.db
colmap global_mapper --database_path db.db --image_path images --output_path sparse
# (vérifiez les noms d'options avec `colmap global_mapper --help` : ils évoluent entre versions)

# 3) Entraîner le splat dans Brush (binaire Apple Silicon v0.3.0 ou compilation de main)
#    Interface : Load › choisir le dossier "scene" (images + sparse/0) › Train
#    Export PLY automatique toutes les 5 000 itérations, 30 000 par défaut
```

Compter environ 3 h 30 pour 30 000 itérations sur un MacBook Air (témoignage août 2026) ; bien moins sur un Mac Studio. **OpenSplat** (compilation Metal) est l'alternative en ligne de commande : `opensplat ./scene -n 30000 -o salon.ply`.

**Mesh sur Mac :** COLMAP ne fait pas de reconstruction dense sans CUDA. Utilisez le mesh LiDAR de l'iPad (Scaniverse/KIRI), **Object Capture** pour un objet ou une zone (Reality Composer Pro › File › New › Object Capture Model, jusqu'à 2 000 images), ou **Metashape Standard** (179 $, essai 30 jours) pour un bâtiment.

### 5.3 Variante PC Windows avec GPU (0 € à 30 $)

1. **RealityScan 2.2** (Epic Games Launcher, gratuit sous 1 M$ de revenus ; NVIDIA CUDA ou AMD RDNA 3/4) : importez photos ou vidéo, Align, puis **mesh texturé** de toute taille (export OBJ/GLB/STL/USDZ/FBX/DAE/LAS) et **Export › Registration › COLMAP** (dossier standard, images non distordues, masques).
2. Entraînez le splat depuis l'export COLMAP avec, au choix :
   - **Brush** (gratuit, tout GPU) ;
   - **LichtFeld Studio** (NVIDIA RTX 20+/GTX 16+ ; source gratuite à compiler, binaire Windows v0.5.x pour 30 $, ou ancien binaire v0.4.2 gratuit) : GUI, édition, export PLY/SOG/SPZ/HTML ;
   - **OpenSplat** (binaire Windows 29 $, ou gratuit compilé) ;
   - **Postshot Free** uniquement pour entraîner et regarder (pas d'export du splat, filigrane, non commercial).
3. Nettoyage et publication dans SuperSplat.

Ordres de grandeur communautaires : 30 min sur RTX 4090, 2-3 h sur RTX 3060 pour ≈ 1 M de gaussiennes ; ≈ 8 h pour 1 000 photos de drone sur RTX 3060.

### 5.4 Variante sans GPU : cloud gratuit

**Le plus simple : Splatware Free.** Créez un compte sur splatware.com, uploadez la vidéo (MP4/MOV, ≤ 0,5 Go, 3 vidéos par projet) ou jusqu'à 500 photos, lancez un entraînement **Lite** (≈ 8-12 min, ≤ 1,5 M gaussiennes), éditez, partagez par lien, exportez en PLY (3 exports par jour). Qualité inférieure aux modèles premium (Creator 19,95 €/mois après le 1er mois à 9,95 €, qui ajoute 3 entraînements « Cinematic/Ultra » et un mesh physique).

**Le plus puissant : Kaggle (≈ 30 h GPU/semaine, 2 × T4) ou Google Colab (T4 non garanti, 12 h).**

```bash
# Dans un notebook (Kaggle : Settings › Accelerator › GPU T4 x2 ; téléphone vérifié)
# 1) Uploader le dossier images/ (dataset Kaggle ou Google Drive)
# 2) COLMAP (paquet apt souvent < 4.0 : sans global_mapper, utiliser mapper)
apt-get install -y colmap ffmpeg
colmap automatic_reconstructor --workspace_path /kaggle/working/scene \
  --image_path /kaggle/input/scene/images --data_type video --quality medium
# 3) OpenSplat (notebook Colab officiel lié dans le README) ou gsplat
git clone https://github.com/WebODM/OpenSplat && cd OpenSplat && mkdir build && cd build \
  && cmake -DCMAKE_BUILD_TYPE=Release .. && make -j$(nproc)
./opensplat /kaggle/working/scene -n 30000 -d 2 -o /kaggle/working/scene.ply
```

Réduisez les images (facteur 2 à 4, ≈ 1 600 px) et limitez-vous à 100-300 photos sur un T4 16 Go ; sauvegardez le PLY dès qu'il est produit (coupures de session). Le notebook Colab officiel de Nerfstudio est cassé depuis fin 2025 (Python 3.12) : ne perdez pas de temps dessus.

### 5.5 Variante « poses ARKit » avec Nerfstudio (iPad Pro, PC NVIDIA)

```bash
pip install nerfstudio            # projet ralenti : dernière release v1.1.5, gsplat épinglé 1.4.0
ns-process-data polycam --data raw_polycam.zip --output-dir data/salon   # ou record3d
ns-train splatfacto --data data/salon
ns-export gaussian-splat --load-config outputs/salon/splatfacto/<date>/config.yml --output-dir exports/salon
```

Le fichier `exports/salon/splat.ply` s'ouvre dans SuperSplat. Nerfstudio n'exporte **pas** de mesh depuis splatfacto ; entraînez `nerfacto` en parallèle et utilisez `ns-export poisson` si vous voulez un OBJ texturé.

---

## 6. Option « je paie un peu » : quand ça vaut le coup

| Dépense | Prix vérifié (09/2026) | Ça vaut le coup si… | Inutile si… |
|---|---|---|---|
| **KIRI Engine Pro** | 49,99 $/an (App Store, promos 35,99-47,99 $) ; 79,99 $/an ou 17,99 $/mois sur le site | Vous voulez **splat + mesh dérivé du splat** d'une même vidéo (≤ 3 min, 1080p) **sans ordinateur** et sans LiDAR, avec plugin Blender gratuit | Vous avez un Mac ou un PC GPU (les outils gratuits font mieux) ; la file d'attente cloud (parfois > 1 h) vous gêne |
| **Scaniverse Plus** | 20 $/mois ou 200 $/an (40 000 crédits ≈ 22 min de capture cloud/mois) | Vous devez **fusionner plusieurs scans** d'un bâtiment en un seul « Site », ou traiter une vidéo 360° (Insta360 X4/X5) ; export USDZ | Une pièce à la fois vous suffit (le mode Classic gratuit fait le travail) |
| **Scaniverse Pro** | 50 $/mois ou 500 $/an | Usage **commercial** explicite, 360° jusqu'à 10 min | Usage personnel |
| **Polycam Basic** | 150 $/an (12,50 $/mois) ou 30 $/mois | Vous tenez à l'écosystème Polycam (plans 2D/3D, splats cloud jusqu'à 300 images, exports mesh 6 formats) | Vous voulez le PLY du splat : il semble réservé à **Business (400 $/an)** ; KIRI Pro est 3 fois moins cher pour du splat cloud |
| **Postshot Indie** | 17 €/mois (204 €/an) ou 26 €/mois | PC Windows NVIDIA et vous préférez une GUI léchée avec export PLY/SPZ et plugin Unreal | Vous acceptez LichtFeld Studio (30 $ une fois) ou Brush (gratuit) |
| **OpenSplat Windows** / **LichtFeld portail** | 29 $ / 30 $ (une fois) | PC Windows, vous ne voulez pas compiler | Vous savez compiler (gratuit) |
| **Teleport Professional** | à partir de 30 $ prépayés, facturé au nombre d'images | Grand extérieur, quartier, drone, jusqu'à 100 M de splats, sans matériel | Scènes de la taille d'une pièce |
| **Splatware Creator** | 9,95 € le 1er mois puis 19,95 €/mois | 3 entraînements premium/mois + mesh physique, sans ordinateur | Le modèle Lite gratuit vous suffit |
| **Metashape Standard** | 179 $ (perpétuel) | Mesh photogrammétrique robuste **sur Mac** à l'échelle d'un bâtiment, export caméras COLMAP | Vous avez un PC Windows (RealityScan gratuit) |
| **Spatial Fields** | 29,99 € (une fois) | Vous voulez poser vos splats en **AR** dans la pièce, sur iPad et Vision Pro | Le viewer web Scaniverse / superspl.at vous suffit |
| **Record3D** | 4,99-5,99 € (une fois) | Pipeline Nerfstudio avec poses LiDAR sans COLMAP | SplatKing (gratuit) couvre déjà le besoin |
| RadianceKit (Mac, 7,99 $) / SplatScene (Mac, 4,99 $/mois) | non revérifiés | Entraînement Metal « un clic » sur Mac, RadianceKit exige macOS 26 | 3D Splat App est gratuit |

À ne pas acheter en 2026 : abonnement 3D Scanner App (l'app a changé de mains et de modèle), Polycam pour les splats, Luma.

---

## 7. Checklist de capture d'un environnement

**Avant**
- [ ] Batterie > 80 %, 10-15 Go libres, lentille propre, mode avion si le scan est local.
- [ ] Lumière constante et diffuse : tous les plafonniers allumés et stores fermés à l'intérieur ; ciel couvert ou soleil du même côté pendant toute la prise à l'extérieur ; pas de vent fort.
- [ ] Personnes, animaux, véhicules et objets mobiles hors champ.
- [ ] Repères texturés temporaires sur les murs unis ; référence d'échelle (mètre ruban, feuille A4) visible si vous n'avez pas de LiDAR.
- [ ] Parcours planifié et reconnaissance à pied sans filmer.

**Pendant**
- [ ] **Translation, pas rotation** : « capturer avec les pieds, pas avec les bras ». Un panorama sur place fait échouer COLMAP et ARKit.
- [ ] Lentement (5-10 cm/s), mouvement continu, pas de à-coups ; pause avant et après chaque porte.
- [ ] Périmètre puis centre ; 3 passes (niveau / vers le haut / vers le bas) ; 2 à 5 boucles à hauteurs différentes pour un objet ou un point d'intérêt (orbiter en visant vers l'intérieur).
- [ ] Chaque surface vue depuis ≥ 3 positions ; recouvrement 70-80 % entre images vidéo (30-50 % entre photos).
- [ ] Distance 0,5 à 3 m des surfaces ; ≥ 30 cm de tout obstacle ; garder les éléments déjà capturés dans le cadre.
- [ ] Exposition / focus / balance des blancs verrouillés ; 4K 30 fps ; tout net (pas de bokeh, pas de flou de bougé : obturateur court, ISO plutôt que flou).
- [ ] Éviter vitres, miroirs, surfaces brillantes, plafonds blancs uniformes ; ne pas rester immobile dans un reflet.
- [ ] Scaniverse : 1-3 min par scan, jamais plus de 5 min ni 500 m² ; démarrer sur une zone détaillée.

**Après**
- [ ] Inspecter murs blancs, portes, miroirs, plafond avant de quitter les lieux ; refaire les passes manquantes.
- [ ] Une capture par pièce ; extérieur séparé de l'intérieur ; découper un site dès que la lumière change ou que le parcours devient difficile à recouvrir.
- [ ] Grand extérieur (circuit, terrain, façade) : vidéo/photos au sol le long du tracé **plus** passes drone obliques (80 % recouvrement frontal, 70 % latéral, GSD ≤ 3 cm, jamais uniquement du nadir) ; la fusion sol + aérien reste imparfaite dans les outils grand public, privilégiez la capture au sol pour les vues « pilote ».

Limites pratiques d'une capture mobile (source tierce, mars 2026) : 50-200 m² en intérieur, 200-500 m² en extérieur par capture ; précision dimensionnelle 2 à 5 % à l'échelle d'un bâtiment ; LiDAR Apple fiable jusqu'à ≈ 5 m, dérive de plusieurs cm sur une pièce de 9 m si l'on scanne trop longtemps.

---

## 8. Visualiser, partager, éditer, convertir

**Voir un splat sur l'iPad**
- Dans Scaniverse (natif, y compris en AR) ; dans Safari via superspl.at, Reflct (15 scènes gratuites, non revérifié) ou un viewer HTML exporté par SuperSplat ; en AR native avec Spatial Fields (payant).
- Les apps natives iPad exploitant l'API RealityKit 27 devraient se multiplier avec iPadOS 27.

**Formats et conversion**
- PLY = source de référence (lourd). SPZ 4 (ouvert, Niantic, mai 2026) ≈ 10× plus petit ; SOG (PlayCanvas) 15-20× plus petit, avec streaming LOD sur superspl.at.
- Convertir : SuperSplat (Convert, dans le navigateur) ou `splat-transform` (CLI). SPZ ↔ PLY : convertisseur web de Niantic Spatial.
- Nettoyer un splat mobile réduit souvent la taille de manière spectaculaire (retours utilisateurs : jusqu'à ≈ 90 %, puis encore ≈ 90 % en ne gardant qu'une bande d'harmoniques sphériques — non vérifié précisément).

**Moteurs 3D**
- Blender : add-on gratuit **3DGS Render by KIRI Engine** ; import natif PLY/SPZ annoncé pour Blender 5.3 (novembre 2026, non revérifié).
- Unity : **gsplat-unity** (maintenu) plutôt que UnityGaussianSplatting (figé).
- Unreal : plugin **XGRIDS LCC-3DGS** (gratuit, UE 5.1-5.8) ou plugin Postshot (.psht) ; le plugin Luma est abandonné.
- Web : Spark 2.0 (three.js) ou SuperSplat Viewer/Studio ; Cesium 3D Tiles pour les scènes géoréférencées.

**Splat → mesh (et inverse)**
- Le plus simple : **KIRI Engine Pro** (« 3DGS to Mesh 3.0 », option à cocher avant l'upload) ; **Scaniverse cloud** produit depuis juillet 2026 un USDZ **splat + mesh aligné** (captures 360°, plans payants).
- Gratuit mais technique (GPU NVIDIA, ré-entraînement depuis les photos + poses) : **2DGS**, **SuGaR**, PGSR, GOF, MILo (non revérifiés).
- Sans GPU, grossier : exporter les centres des gaussiennes en CSV/PLY (splat-transform), calculer les normales puis Poisson dans **CloudCompare** ou **MeshLab** ; suffisant pour une collision ou une heightmap, pas pour un rendu.
- Mesh → splat : LichtFeld Studio (mesh-to-splat) et Splatware (« mesh only by conversion »).

---

## 9. Aller plus loin : intégrer l'environnement scanné dans ROS / Gazebo / RViz

Ce dépôt affiche des circuits sous forme de `visualization_msgs/Marker` de type `LINE_STRIP` (type 4) publiés par `rostopic pub` dans un fichier `launch/race-track-XXXXXXXX.launch`, avec un script `scripts/racetrack_arg_converter.py` qui transforme un CSV (X,Y) en arguments `pN`. Trois façons d'y brancher un scan iPad :

### 9.1 Afficher le maillage scanné dans RViz (marker MESH_RESOURCE)

1. Dans Blender (gratuit) : importer l'OBJ/GLB/USDZ exporté par Scaniverse/KIRI ; unités en **mètres**, **+Z vers le haut**, **+X vers l'avant** (les scans Apple sont en Y-up : rotation de 90° autour de X) ; centrer à l'origine ; modificateur **Decimate** (Collapse, ratio 0,1-0,3 ; Planar pour les murs et sols) ; exporter en **DAE (Collada)** avec la texture PNG à côté, ou en GLB.
2. Placer le fichier dans un dossier `meshes/` du paquet et publier un marker de type **10** :

```xml
<arg name="mesh" value="'{header: {frame_id: map}, ns: scan, id: 1, type: 10, action: 0,
  pose: {position: {x: 0.0, y: 0.0, z: 0.0}, orientation: {w: 1.0, x: 0.0, y: 0.0, z: 0.0}},
  scale: {x: 1.0, y: 1.0, z: 1.0}, color: {a: 1.0, r: 1.0, g: 1.0, b: 1.0},
  mesh_resource: \"package://virtual_racetracks_collection/meshes/piste.dae\",
  mesh_use_embedded_materials: true}'" />
<node name="pub_mesh" pkg="rostopic" type="rostopic" args="pub /shape visualization_msgs/Marker $(arg mesh)"/>
```

RViz (ROS 1 comme ROS 2) charge DAE/OBJ/STL via Assimp ; `scale 1 1 1` = 1 m. Les constantes de type sont identiques en ROS 2.

### 9.2 Conserver le format actuel (ligne centrale extraite du scan)

1. Ouvrir le nuage de points (PLY/LAS) ou le mesh dans **CloudCompare** (gratuit) : outil **Cross Section** (≥ 2.12) pour extraire les contours ou une tranche, ou tracer une polyligne ; exporter en CSV (X,Y).
2. Sous-échantillonner, puis `python3 scripts/racetrack_arg_converter.py piste.csv` génère les `<arg name="pN">` et la liste `pts` à coller dans un nouveau `race-track-XXXXXXXX.launch` (branche du même nom, conformément au CONTRIBUTING.md).

### 9.3 Simuler dans Gazebo

- **Modèle statique** : dossier `piste_scan/` avec `model.config`, `model.sdf`, `meshes/piste.dae`, `materials/textures/` ; dans le monde SDF : `<include><static>true</static><uri>model://piste_scan</uri></include>` ; variable `GZ_SIM_RESOURCE_PATH` pointant sur le dossier parent (Gazebo Harmonic, LTS jusqu'en 2029). GLB accepté nativement depuis Gazebo Garden ; DAE reste le format le plus éprouvé. Gazebo Classic est en fin de vie depuis janvier 2025.
- **Collision automatique** (SDF 1.11) : réutiliser le même mesh avec `<mesh optimization="convex_decomposition">` dans `<collision>`, et `<scale>` pour corriger un export en cm/mm.
- **Terrain** : CloudCompare › **Rasterize** le nuage LiDAR en image niveaux de gris carrée, redimensionnée à 256/512 px (Ogre2 / Harmonic exige 2^n ; Classic exigeait 2^n+1) → `<heightmap><uri>…png</uri><size>L l h</size></heightmap>` ; DEM GeoTIFF possible via GDAL.
- **Carte 2D de navigation** : PLY → PCD (CloudCompare), puis `pcd2pgm` (ROS 2 Humble) ou `octomap_server`, sauvegarde avec `ros2 run nav2_map_server map_saver_cli -f piste` (YAML + PGM) ; carte d'élévation 2.5D avec `grid_map_pcl`.

Formats bruts utiles côté ROS : LAS/E57 (Modelar, Scaniverse), PLY (tous), USDZ splat + mesh « prêt pour Isaac Sim » (Scaniverse cloud, payant).

---

## 10. Récapitulatif des coûts et matrice de décision

### Coûts (au 19/09/2026)

| Poste | Gratuit | Le moins cher qui change quelque chose |
|---|---|---|
| Capture + splat sur iPad | Scaniverse Classic (0 €) | KIRI Pro 49,99 $/an (splat + mesh cloud) ; Scaniverse Plus 20 $/mois (fusion, 360°) |
| Mesh sur iPad | Scaniverse (0 €), KIRI Basic LiDAR (0 €), Modelar (0 €), RealityScan Mobile (0 €) | — |
| Entraînement Mac | 3D Splat App, Brush, OpenSplat, COLMAP (0 €) | RadianceKit 7,99 $ (non revérifié) |
| Entraînement PC | Brush, LichtFeld source, OpenSplat source, COLMAP (0 €) ; Postshot Free (sans export) | OpenSplat Windows 29 $ ; LichtFeld binaire 30 $ ; Postshot Indie 204 €/an |
| Mesh sur ordinateur | RealityScan desktop (0 € < 1 M$), Object Capture Mac (0 €), Meshroom / WebODM (0 €, non revérifiés) | Metashape Standard 179 $ (Mac) |
| Cloud | Splatware Lite (0 €), Kaggle (0 €), Colab (0 €) | Colab Pro 9,99 $/mois ; Splatware Creator 19,95 €/mois ; Teleport dès 30 $ |
| Édition / partage | SuperSplat + superspl.at (0 €) | Spatial Fields 29,99 € (AR) |
| Matériel | Votre iPad actuel | iPad Pro reconditionné M4 1 019 € ou d'occasion 2020-2022 pour le LiDAR |

### Matrice de décision

| Ma situation | Pipeline conseillé |
|---|---|
| Un iPad (avec ou sans LiDAR), pas d'ordinateur, 0 € | **Section 4** : Scaniverse (splat + mesh) → SuperSplat dans Safari ou plus tard sur un ordinateur |
| iPad + Mac Apple Silicon | Section 4 pour le mesh + **section 5.2** (3D Splat App ou COLMAP + Brush) pour un splat de meilleure qualité |
| iPad + PC Windows avec RTX / Radeon récente | **Section 5.3** : RealityScan (mesh toute échelle + COLMAP) → Brush / LichtFeld / OpenSplat |
| iPad + vieux PC sans GPU | **Section 5.4** : Splatware Lite, ou Kaggle/Colab avec OpenSplat |
| iPad Pro et envie de poses ARKit sans COLMAP | SplatKing (gratuit, dossier COLMAP) ou Polycam raw → section 5.2 / 5.3 / 5.5 |
| Je veux splat + mesh sans rien installer et j'accepte ≈ 50 $/an | KIRI Engine Pro |
| Bâtiment entier ou circuit complet | Découper en zones (section 7) ; fusion via Scaniverse Plus, ou drone + vidéo au sol → RealityScan/COLMAP → Brush/LichtFeld ; Teleport si budget |
| Utilisation dans ROS/Gazebo | Mesh LiDAR (Scaniverse/KIRI/Modelar) → Blender → DAE/GLB → section 9 |

---

## 11. Sources

Toutes les URL ci-dessous ont été réellement ouvertes le 19 septembre 2026 par les agents de vérification (pages officielles en priorité).

**Scaniverse / Niantic Spatial** — https://www.nianticspatial.com/products/capture · https://www.nianticspatial.com/pricing · https://www.nianticspatial.com/en/faq/scaniverse · https://dev.scaniverse.com/support · https://www.nianticspatial.com/docs/scaniverse/techniques/ · https://www.nianticspatial.com/docs/scaniverse/troubleshoot/ · https://www.nianticspatial.com/docs/scaniverse/360camera/ · https://www.nianticspatial.com/blog/usdz-scaniverse · https://www.nianticspatial.com/blog/spz4 · https://community.nianticspatial.com/t/processing-fails-on-a-couple-of-my-splats/5734 · https://apps.apple.com/us/app/scaniverse-3d-scanner/id1541433223 · https://dev.scaniverse.com/news/creating-splats-which-app-to-choose · https://github.com/nianticlabs/spz

**Polycam** — https://poly.cam/pricing · https://poly.cam/tools/gaussian-splatting · https://learn.poly.cam/hc/en-us/articles/27425185907348-How-to-Use-Object-Mode · https://learn.poly.cam/hc/en-us/articles/36655587097620-How-to-Use-Space-Mode-LiDAR-Devices · https://github.com/PolyCam/polyform · https://apps.apple.com/us/app/polycam-3d-scanner-lidar-360/id1532482376

**KIRI Engine** — https://www.kiriengine.app/pricing · https://www.kiriengine.app/blog/kiri-engine-basic-vs-pro · https://www.kiriengine.app/blog/Best_Free_3D_Scanner_Apps_2026 · https://www.kiriengine.app/blog/how-to-capture-3d-gaussian-splats-kiri-engine · https://www.kiriengine.app/blog/3DGSvsPhotogrammetryvsLiDAR · https://www.kiriengine.app/faq/scan-processing-time · https://apps.apple.com/us/app/kiri-engine-3d-scanner-app/id1577127142

**RealityScan (Epic Games)** — https://www.realityscan.com/mobile · https://www.realityscan.com/license · https://www.realityscan.com/news/realityscan-2-2-is-here-with-full-amd-gpu-support-download-today · https://dev.epicgames.com/documentation/realityscan/realityscan-2-2 · https://dev.epicgames.com/documentation/realityscan/realityscan-2-1-1 · https://dev.epicgames.com/documentation/realityscan-mobile/realityscan-mobile-1-7-release-notes · https://apps.apple.com/us/app/realityscan-mobile/id1584832280

**Autres apps iPad** — Teleport : https://get.teleport.varjo.com/pricing · https://teleport.varjo.com/docs/quick-start-guide/ · https://apps.apple.com/us/app/teleport-by-varjo/id6450445339 — Gaussian SplatKing : https://radiancefields.com/splatking · https://radiancefields.com/splatking/guide/pipelines · https://radiancefields.com/splatking/guide/output · https://apps.apple.com/us/app/gaussian-splatking/id6759175085 — Voxelio : https://www.voxelio.app · https://apps.apple.com/us/app/voxelio-3d-lidar-scanner/id6764829442 — Modelar : https://modelar.ai/ · https://apps.apple.com/us/app/modelar-3d-lidar-scanner/id1572844190 — 3D Scanner App : https://apps.apple.com/us/app/3d-scanner-app/id1419913995 · https://labs.laan.com/apps · https://www.3dscannerlidar.com/ — Reality Composer : https://apps.apple.com/us/app/reality-composer/id1462358802 — Record3D : https://record3d.app/ · https://apps.apple.com/us/app/record3d-3d-videos/id1477716895 — Spatial Fields : https://spatialfields.app/ · https://apps.apple.com/app/id6745549629 — Splatcatcher : https://apps.apple.com/us/app/splatcam-lidar-capture/id6759800588 — Luma : https://apps.apple.com/us/app/luma-ai/id1615849914 — NeRFCapture : https://github.com/jc211/NeRFCapture — Spectacular Rec : https://spectacularai.github.io/docs/sdk/tools/nerf.html · https://pypi.org/project/spectacularai/ — Blackmagic Camera : https://www.blackmagicdesign.com/products/blackmagiccamera · https://www.blackmagicdesign.com/products/blackmagiccamera/techspecs

**Traitement sur ordinateur** — 3D Splat App : https://3dsplatapp.com · https://apps.apple.com/app/3d-splat-app/id6760239941 — Brush : https://github.com/ArthurBrussee/brush · https://radiancefields.com/platforms/brush — OpenSplat : https://github.com/WebODM/OpenSplat · https://sites.fastspring.com/masseranolabs/product/opensplatforwindows — LichtFeld Studio : https://lichtfeld.io/ · https://github.com/MrNeRF/LichtFeld-Studio · https://portal.lichtfeld.io/signup/ · https://radiancefields.com/platforms/lichtfeld-studio — Postshot : https://www.jawset.com/shop/pricing · https://www.jawset.com/docs/d/Postshot+User+Guide/Importing+Images · https://www.jawset.com/docs/d/Postshot+User+Guide/Capturing+Guidelines · https://radiancefields.com/platforms/postshot — COLMAP : https://colmap.github.io/ · https://colmap.github.io/install.html · https://github.com/colmap/colmap/releases · https://github.com/colmap/glomap — Nerfstudio : https://docs.nerf.studio/quickstart/custom_dataset.html · https://docs.nerf.studio/quickstart/export_geometry.html · https://github.com/nerfstudio-project/nerfstudio/releases · https://github.com/nerfstudio-project/gsplat — Apple Object Capture : https://developer.apple.com/documentation/realitykit/realitykit-object-capture · https://developer.apple.com/videos/play/wwdc2024/10107/ · https://developer.apple.com/videos/play/wwdc2021/10076/ — PhotoCatch : https://www.photocatch.app/ — Metashape : https://www.agisoft.com/buy/online-store/ · https://github.com/agisoft-llc/metashape-scripts/blob/master/src/export_for_gaussian_splatting.py — 3DF Zephyr : https://www.3dflow.net/3df-zephyr-feature-comparison/ — ODM/WebODM : https://opendronemap.org/download/ · https://docs.opendronemap.org/installation/ — hloc : https://github.com/cvg/Hierarchical-Localization — RadianceKit : https://apps.apple.com/app/id6760346035 — SplatScene : https://splatscene.app/

**Cloud** — Splatware : https://splatware.com/pricing · https://splatware.com/docs/uploading-and-data-preparation — Google Colab : https://research.google.com/colaboratory/faq.html · https://colab.research.google.com/signup · https://github.com/googlecolab/colabtools/issues/563 — Kaggle : https://www.kaggle.com/docs/notebooks · https://www.kaggle.com/docs/efficient-gpu-usage · https://www.kaggle.com/discussions/product-announcements/735239 · https://www.kaggle.com/code/stpeteishii/a-loft-colmap-gaussian-splatting — RunPod : https://www.runpod.io/pricing — Vast.ai : https://computeprices.com/providers/vast — Thunder Compute (comparatif Colab) : https://www.thundercompute.com/blog/colab-alternatives-for-cheap-deep-learning-in-2025

**Viewers, formats, moteurs** — SuperSplat : https://superspl.at/ · https://developer.playcanvas.com/user-manual/supersplat/ · https://developer.playcanvas.com/user-manual/supersplat/editor/editing-splats/ · https://developer.playcanvas.com/user-manual/supersplat/editor/import-export/ · https://developer.playcanvas.com/user-manual/supersplat/editor/publishing/ · https://github.com/playcanvas/supersplat/releases · https://blog.playcanvas.com/new-in-supersplat-editor-3-0-rebuilt-on-webgpu/ · https://github.com/playcanvas/splat-transform — Spark / World Labs : https://docs.worldlabs.ai/marble/export/gaussian-splat/unreal — Swyvl : https://swyvl.io/blog/best-gaussian-splat-viewers/ · https://swyvl.io/blog/gaussian-splat-formats-ply-spz-ksplat/ · https://swyvl.io/blog/how-to-create-gaussian-splats/ — MetalSplatter : https://github.com/scier/MetalSplatter — 2DGS : https://github.com/hbb1/2d-gaussian-splatting — SuGaR : https://github.com/Anttwo/SuGaR — KIRI Blender add-on : https://www.kiriengine.app/blog/kiri-engine-basic-vs-pro

**Plateforme Apple et matériel** — https://developer.apple.com/videos/play/wwdc2026/279/ · https://developer.apple.com/tutorials/data/documentation/realitykit/gaussiansplatresource.json · https://developer.apple.com/tutorials/data/documentation/realitykit/gaussiansplatcomponent.json · https://developer.apple.com/videos/play/wwdc2025/287/ · https://developer.apple.com/tutorials/data/documentation/macos-release-notes/macos-26-release-notes.json · https://aousd.org/blog/openusd-v26-03/ · https://www.apple.com/ipad-pro/specs/ · https://www.apple.com/ipad-air/specs/ · https://www.apple.com/iphone-18-pro/specs/ · https://www.apple.com/shop/buy-ipad/ipad-pro · https://www.apple.com/newsroom/2025/10/apple-introduces-the-powerful-new-ipad-pro-with-the-m5-chip/ · https://www.apple.com/newsroom/2020/03/apple-unveils-new-ipad-pro-with-lidar-scanner-and-trackpad-support-in-ipados/ · https://support.apple.com/en-us/119916 · https://radiancefields.com/platforms/apple · https://www.mactrast.com/2026/09/apple-releases-ios-27-ipados-27-macos-27-watchos-27-tvos-27-and-visionos-27-to-the-general-public/ · https://caseadri.com/roomkit/guides/which-iphones-have-lidar/

**Bonnes pratiques** — https://help.sketchup.com/en/gaussian-splats-best-practices · https://www.freegaussian.ai/blog/best-practices-capture · https://realhorizons.ai/blog/indoor-gaussian-splatting-capture-guide/ · https://realhorizons.ai/blog/outdoor-gaussian-splatting-capture-guide/ · https://realhorizons.ai/blog/gaussian-splatting-from-photos-vs-video/ · https://archgyan.com/luma-ai-3d-capture-architects-existing-buildings/ · https://arxiv.org/html/2507.06109v1 (LighthouseGS) · https://na.mipmap3d.com/blogs/from-field-to-model-capturing-high-quality-3d-gaussian-splats-with-uav-imagery/ · https://sainingzhang.github.io/project/uc-gs/ · https://www.captures.studio/interactive-capture-tutorial · https://www.reshot.ai/3d-gaussian-splatting · https://github.com/NVlabs/instant-ngp/blob/master/docs/nerf_dataset_tips.md · https://www.polyvia3d.com/guides/gaussian-splatting-mobile-capture · https://www.polyvia3d.com/guides/gaussian-splatting-tools-comparison · https://www.polyvia3d.com/guides/best-gaussian-splatting-apps · https://www.skyebrowse.com/news/posts/polycam-vs-scaniverse · https://www.skyebrowse.com/news/posts/gaussian-splat-from-video · https://www.scanmanifold.com/blog-posts/lidar-on-iphone-how-accurate-is-it-plus-the-biggest-errors-that-manifold-corrects

**Retours d'utilisateurs (Reddit, lus via flux RSS / archive)** — https://www.reddit.com/r/GaussianSplatting/comments/1sqdze2/mobile_gaussian_splatting_capture_thats_easy/ · https://www.reddit.com/r/GaussianSplatting/comments/1kpt60s/top_5_tools_for_gaussian_splatting_compared/ · https://www.reddit.com/r/3DScanning/comments/1no11h6/polycam_vs_scaniverse_scanned_wall_scanned_using/ · https://www.reddit.com/r/GaussianSplatting/comments/1qqyhki/best_approachsoftware_for_highest_quality/ · https://www.reddit.com/r/GaussianSplatting/comments/1j80nhk/how_to_get_started_with_gaussian_splatting/ · https://www.reddit.com/r/GaussianSplatting/comments/1sfgtgn/noob_needs_help/ · https://www.reddit.com/r/GaussianSplatting/comments/1o4ockj/big_continuous_splat_of_indoors_or_outdoors_area/ · https://www.reddit.com/r/GaussianSplatting/comments/1szy7bm/iphone_app_with_guided_capture_for_high_quality/

**ROS / Gazebo / RViz** — https://classic.gazebosim.org/tutorials?tut=import_mesh · https://gazebosim.org/docs/harmonic/sdf_worlds/ · https://gazebosim.org/docs/harmonic/release-features/ · https://github.com/gazebosim/sdformat/blob/main/sdf/1.11/mesh_shape.sdf · https://github.com/gazebosim/gz-rendering/blob/gz-rendering8/ogre2/src/Ogre2Heightmap.cc · https://classic.gazebosim.org/tutorials?tut=dem · https://github.com/ros2/rviz/blob/rolling/rviz_rendering/src/rviz_rendering/mesh_loader.cpp · https://github.com/ros2/common_interfaces/blob/rolling/visualization_msgs/msg/Marker.msg · https://docs.blender.org/manual/en/3.6/modeling/modifiers/generate/decimate.html · https://www.cloudcompare.org/doc/wiki/index.php/Rasterize · https://www.cloudcompare.org/doc/wiki/index.php/Cross_Section · https://www.cloudcompare.org/doc/wiki/index.php/Poisson_Surface_Reconstruction_(plugin) · https://github.com/LihanChen2004/pcd2pgm · https://github.com/OctoMap/octomap_mapping · https://github.com/ANYbotics/grid_map/tree/master/grid_map_pcl · https://github.com/ros-navigation/navigation2/blob/main/nav2_map_server/README.md · https://github.com/ros-perception/perception_pcl

---

## Annexe — méthode et limites de l'étude

- **Workflow** : 11 agents de recherche web en parallèle (un angle chacun) → consolidation en 110 solutions canoniques → un agent « sceptique » par solution chargé de **réfuter** les affirmations sur les sources officielles → fiches produit à jour. Les phases prévues ensuite (panel de 4 pipelines candidats notés par 3 juges, rédaction automatique, critique de complétude) ont été interrompues par un plafond de dépense du compte ; la synthèse de ce document a donc été rédigée directement à partir des 34 fiches vérifiées et des 11 synthèses de recherche.
- **Vérifiées par un sceptique (34)** : Scaniverse, Polycam, KIRI Engine, RealityScan Mobile, RealityScan desktop, Teleport, Postshot, Gaussian SplatKing, Splatcatcher, 3D Splat App, Voxelio, Spatial Fields, Modelar, 3D Scanner App, Reality Composer, Apple Object Capture, PhotoCatch, Record3D, Nerfstudio, Spectacular Rec, NeRFCapture, COLMAP, Google Colab, Kaggle, OpenSplat, LichtFeld Studio, Brush, SuperSplat, Splatware, matériel iPad, bonnes pratiques de capture, pipeline DIY, Blackmagic Camera, grandes scènes / drone.
- **Non revérifiées (issues d'une seule passe de recherche, à confirmer avant de s'y fier)** : Lightning AI, RunPod, Vast.ai, gsplat, RadianceKit, SplatScene, Reflct, Swyvl, Spatial Studio, splat-transform, Spark, SPZ (détails), add-on Blender KIRI, Blender 5.3, 2DGS, SuGaR, RealityKit 27 (détails), Meshroom, ODM/WebODM, MeshLab, CloudCompare (détails), Metashape (fiche partielle).
- **Points marqués « à vérifier »** dans le texte : mode splat Scaniverse sur iPad A12/A13 ; export « Splat PLY » Polycam réservé à Business ; quota hebdomadaire gratuit de Voxelio ; date exacte de l'API splats RealityKit sur iPadOS 27.
- Reddit était bloqué pour les agents : les retours d'utilisateurs ont été lus via les flux RSS officiels ou une archive, et sont présentés comme des témoignages, pas comme des faits.
