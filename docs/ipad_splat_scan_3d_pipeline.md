# Splat et scan 3D d'un environnement avec un iPad — guide et pipeline recommandé

> **Rapport établi le 2026-09-20.** Tous les prix, formats et compatibilités ont été vérifiés les 19 et 20 septembre 2026 sur les pages officielles citées en section 11. **Ce marché bouge très vite** : Scaniverse est passé au freemium en 2026, Polycam a déplacé des fonctions vers des paliers supérieurs, Postshot a supprimé l'export de son plan gratuit. Revérifiez tout prix avant de payer quoi que ce soit.

---

## 1. Réponse courte (la recommandation en 10 lignes)

> ### Avant tout : identifiez votre iPad (30 secondes)
> **Réglages → Général → Informations → ligne « Nom du modèle ».**
> **Seuls les modèles nommés « iPad Pro » de 2020 et postérieurs ont un LiDAR.** Aucun iPad Air, mini ou standard n'en a, y compris l'**iPad Air M4 de 2026**.
> **Repère visuel** : sur un iPad Pro à LiDAR, un petit capteur noir supplémentaire est visible à côté de l'objectif arrière.
> Toute la suite du rapport se lit différemment selon cette réponse : le §2 a deux colonnes, le §4 a deux variantes.

1. **Oui, c'est gratuit pour une pièce**, et une seule application suffit : **Scaniverse** (Niantic Spatial), en mode « Classic », calcule le splat **sur l'iPad** en 1–2 min, hors ligne, sans quota.
2. **Vos deux besoins sont servis par une seule capture.** C'est le cœur de toute la méthode : un scan Scaniverse correctement réglé (données brutes activées) se retraite en **splat** *et* en **maillage** sans refilmer. Ne menez pas deux campagnes de capture en parallèle — menez-en une, et retraitez-la deux fois (§4, étape 5).
3. **Le LiDAR n'est pas nécessaire** pour le splat. Il n'existe de toute façon que sur les **iPad Pro** (2020 et suivants) — voir l'encadré d'identification ci-dessus.
4. **L'astuce centrale** : activez la sauvegarde des **données brutes** *avant* le premier scan, faites **un seul scan de 90–120 s**, puis « **Reprocess Scan** ». Le support officiel est explicite : « *open your scan, then press the … icon in the upper-right corner. Select Reprocess Scan* », et cela n'est possible que « *provided you have the raw data saved* ».
5. **Étape zéro obligatoire** : le support Niantic ne garantit les splats que pour les « iPhones 12 or newer » et **ne dit rien des iPad**. Testez 60 s avant d'investir une demi-journée — et si le mode Splat n'apparaît pas, le plan B est décrit à l'étape 0.
6. **Posez un mètre pliant ouvert à 1,00 m dans la scène, dans TOUS les cas, y compris sur iPad Pro.** Le mesh LiDAR est à l'échelle réelle et mesurable dans l'app ; **l'échelle du splat, elle, n'est documentée officiellement nulle part**. Recalibrez ensuite dans SuperSplat.
7. Nettoyage, mise à l'échelle et partage web : **SuperSplat** (navigateur, gratuit, MIT) puis **superspl.at**.
8. **Non, un circuit complet n'est pas faisable** ainsi : la capture depuis un véhicule est explicitement non supportée par Niantic. Visez des tronçons et des points d'intérêt.
9. **Dépense la plus défendable si vous avez un Mac : aucune.** **3D Splat App** (Laan Labs) est gratuite, native Mac App Store, entraîne en local en Metal et exporte PLY/SPZ/SOG — c'est l'équivalent 0 € de RadianceKit (7,99 $). **3D Splat App et Brush ont tous deux une interface graphique** : 3D Splat App s'installe en un clic depuis le Mac App Store et propose des presets (Fast / Medium / HD) ; Brush se télécharge en binaire (Apple Silicon, Windows x64, Linux x64) ou se compile, s'utilise en ligne de commande avec `--with-viewer` pour ouvrir l'interface, et **tourne aussi sur PC AMD/Intel** — ce que 3D Splat App ne fait pas (Mac uniquement).
10. **Ne payez pas** : Postshot Indie 204 €/an, Polycam Basic 150 $/an (sans PLY du splat), Scaniverse Plus tant que vous n'avez pas épuisé le quota Free.
11. **Budget conseillé** : 0 € pour démarrer, 0 € ensuite dans la grande majorité des cas.

**Aiguillage selon votre matériel :**

| Votre situation | Chemin |
|---|---|
| iPad Pro (LiDAR), pas d'ordinateur | §4, variante LiDAR — splat + mesh mesuré (Scaniverse, ou Modelar pour le nuage E57/LAS) |
| iPad Air / mini / standard, pas d'ordinateur | §4, variante sans LiDAR — splat + mesh Polycam Space (replis : KIRI Photo Scan, RealityScan Mobile) |
| iPad + Mac Apple Silicon | §4 puis §6 : **3D Splat App (0 €)**, ou RadianceKit 7,99 $, ou Brush. **Mesh sans ligne de commande : Reality Composer Pro ou PhotoCatch** (§5.5) |
| iPad + PC NVIDIA, ou qualité maximale | §4 puis §5 : COLMAP + gsplat / LichtFeld |
| **PC Windows/Linux avec GPU AMD ou Intel** | **Brush en local** (binaires Windows x64 et Linux x64), ou **OpenSplat** (AGPLv3, ROCm/HIP) — pas besoin de cloud |
| **Aucun GPU du tout** | GPU cloud gratuit en §5.4 (Kaggle — vérification SMS obligatoire ; Lightning AI en alternative) |

> **Qui exige réellement NVIDIA ?** Seulement **Postshot**, **LichtFeld Studio** et **gsplat/nerfstudio**. Brush tourne « *on macOS/windows/linux, AMD/Nvidia/Intel cards, Android, and in a browser* » ; OpenSplat tourne sur « *NVIDIA, AMD and Apple (Metal) GPUs, but can also run entirely on the CPU (~100x slower)* ».

---

## 2. Comprendre en 2 minutes : splat (3DGS) vs scan 3D (mesh)

Un **Gaussian splat (3DGS)** est un nuage de millions de petites ellipses colorées. C'est une **représentation d'apparence** : photoréaliste, avec reflets, verre, feuillage. Mais ce n'est **pas de la géométrie** : on ne peut ni mesurer, ni faire une collision, ni l'ouvrir dans un logiciel de CAO sans conversion.

Un **scan 3D classique** produit un **maillage texturé** (mesh) et/ou un **nuage de points**. Moins joli, mais mesurable, exportable en OBJ/FBX/USDZ, utilisable pour de la collision, de la simulation ou de la modélisation.

**Les deux se complètent — et Scaniverse permet de tirer les deux d'une seule capture.** C'est la raison pour laquelle tout le §4 est construit autour d'**une seule prise de vue**.

**Ce que l'iPad sait faire :**

| | iPad Pro (LiDAR, 2020+) | iPad Air / mini / standard |
|---|---|---|
| Splat 3DGS | Oui, sur l'appareil, gratuit | Oui, sur l'appareil, gratuit |
| Mesh mesuré | Oui (LiDAR, portée ≈ 5 m), mesures dans l'app | Non — photogrammétrie seulement |
| Nuage de points LAS / E57 | Oui (Scaniverse LAS ; Modelar E57/LAS/PLY/CSV) | **Pas de LAS ni d'E57 ; nuage de points PLY/XYZ possible gratuitement via KIRI Photo Scan** (photogrammétrie cloud, échelle non garantie, rétention serveur courte) |
| Échelle du splat | **Non documenté** — posez un étalon | **Non documenté** — posez un étalon |

Le LiDAR d'Apple porte à **environ 5 mètres**. Au-delà, la géométrie est interpolée et non mesurée. Sa précision réelle est **centimétrique au mieux**, avec une dérive qui s'accumule : une pièce de 9 m peut mesurer +10 cm d'un côté et −8 cm de l'autre (mesures rapportées par ScanManifold, article de 2022 — **source ancienne**, mais le capteur n'a pas changé).

> **Point d'honnêteté sur l'échelle.** Aucune documentation éditeur ne garantit que le splat Scaniverse sort à l'échelle métrique. Ce qui est documenté, c'est que le **mesh LiDAR** permet de « *prendre des mesures précises* ». Pour le splat, traitez l'échelle comme inconnue jusqu'à recalibration manuelle dans SuperSplat (§4, étape 7) — sur iPad Pro comme sur iPad Air.

> **Second point d'honnêteté** : le mode Splat de Scaniverse n'est documenté officiellement que pour les iPhone 12+. Sur iPad Air 4+/iPad 10+/mini 6+/iPad Pro M-series c'est très probable ; sur iPad 8, mini 5 ou Air 3 (puces A12/A13), **à vérifier avant de s'engager** (voir §4, étape 0, et son plan B).

---

## 3. Panorama des solutions vérifiées

| Solution | Splat | Mesh / scan | Traitement | iPad / LiDAR | Gratuit ? / prix 2026 | Droits commerciaux | Exports | Verdict environnement |
|---|---|---|---|---|---|---|---|---|
| **Scaniverse** (Niantic Spatial) | Oui, sur l'appareil, 1–2 min | Oui (LiDAR ou photogrammétrie) | Appareil + cloud optionnel | iPadOS 16.6+, A12+ ; LiDAR facultatif | **Gratuit illimité** (Classic) ; cloud Free 20 000 crédits/mois, Plus 20 $/mois ou 200 $/an, Pro 50 $/mois ou 500 $/an | **Non en gratuit** — Pro ou Enterprise exigé | SPZ, PLY, OBJ, FBX, USDZ, LAS | **Excellent pour une pièce** ; par morceaux pour un bâtiment ; non pour une piste. Cloud : **5 min et 500 m² max par scan** |
| **Polycam** | Oui, **payant** (cloud) | Oui (LiDAR + Space non-LiDAR, local) | Appareil + cloud | iPadOS 18+ ; LiDAR facultatif | Free 0 $ (GLTF seul, **pas de splat**) ; Basic 150 $/an ; Business 400 $/an | Non documenté | GLTF (Free) ; OBJ/FBX/STL/USDZ (Basic) ; PLY/LAS (Business) | Bon pour pièces ; crashs mémoire dès 2–3 pièces, pertes de scans après mise à jour (§4, étape 5) |
| **KIRI Engine** | Oui, **Pro uniquement** | Oui, gratuit (Photo Scan 150 photos / 2 Go ; LiDAR) | Cloud (sauf LiDAR) | iPadOS 15+ ; LiDAR pour Room/Scene | Basic 0 $ ; **Pro 17,99 $/mois ou 79,99 $/an** | Non documenté | OBJ, STL, FBX, GLTF, GLB, USDZ, **PLY, XYZ** (nuage) | Correct pour pièce ; 3 min de vidéo max ; **rétention 3 jours documentée côté API** |
| **Modelar** (3D LiDAR Scanner) | **Non** | Oui — nuage + maillage texturé, sur l'appareil | Appareil | **iPad Pro LiDAR uniquement** | **Gratuit, 0 €**, sans abonnement, sans pub, sans filigrane | Non documenté | **Nuage : E57, LAS, PLY, CSV** ; **mesh : GLB, USDZ, OBJ, STL, PLY** | Le seul chemin gratuit vers E57/LAS sur iPad ; **aucun splat, aucun repli photogrammétrie** |
| **RealityScan Mobile** (Epic) | **Non** | Oui (objets) | Cloud | iPadOS 16+ ; LiDAR non utilisé | **Gratuit**, « all exports completely free » | **Oui** (EULA : « any lawful purpose ») | OBJ, USDZ | Objets seulement, 300 images max — utile en **repli mesh** |
| **Apple Object Capture** — Reality Composer Pro / PhotoCatch | **Non** | **Oui, mesh texturé** | Ordinateur (Mac) | Photos prises avec **n'importe quel iPad**, LiDAR ou non | **Gratuit** (Reality Composer Pro livré avec Xcode ; PhotoCatch Mac gratuit avec restrictions non documentées) | Non documenté | **USDZ, OBJ** | **Seule voie native Mac sans ligne de commande.** Conçu pour objets et zones : au-delà d'**≈ 1,80 m** la qualité se dégrade sauf traitement à un niveau de détail supérieur ; plafond **≈ 2 000 images** ; **pas d'échelle réelle** si les photos ne viennent pas d'un appareil LiDAR |
| **Meshroom** (AliceVision) | Non (plugin expérimental, non utilisable) | **Oui, mesh texturé + nuage densifié** | Ordinateur | — | **Gratuit, MPL2** | **Oui** | OBJ/MTL + textures, Alembic, nuages | Alternative libre à RealityScan sur PC. **Interface graphique nodale.** Deux réserves : **sans GPU NVIDIA CUDA on se limite au « Draft Meshing »**, qui maille le seul nuage épars et donne un aperçu grossier inexploitable ; **aucun build macOS officiel** |
| **Teleport by Varjo** | Oui | **Non** | Cloud | iPadOS 17+ ; sans LiDAR | Essai 5 captures puis **à partir de 30 $ prépayés**, facturation au nombre d'images | Non documenté | PLY (payant) | Bon pour lieux, mais aucun mesh |
| **Voxelio** | Oui, sur l'appareil | Oui (LiDAR) | Appareil | **LiDAR requis** | Gratuit limité ; Pro à vie 179,99 € | Non documenté | SPZ, OBJ, PLY, USDZ, STL | Splat « petites scènes » seulement |
| **3D Scanner App** | Non | Oui | Les deux | LiDAR pour l'essentiel | **Abonnement de fait** (≈ 35–80 €/an) | Non documenté | OBJ, GLTF, USDZ, PLY, LAS, DXF | À éviter en 2026 (paywall, réseau obligatoire) |
| **Gaussian SplatKing** | Capture seule | Nuage LiDAR seul | Ordinateur ensuite | iPadOS 18+ ; LiDAR facultatif | **Gratuit** | Oui (vous restez propriétaire des captures) | COLMAP, JPEG/RAW, MOV | Capteur pour pipeline PC |
| **Record3D** | Capture RGBD seule | Nuages PLY par frame | Ordinateur ensuite | **iPad Pro LiDAR uniquement** (« FaceID ou LiDAR », absents des iPad Air/mini/standard) | ≈ **4–6 €** (achat unique) | Non documenté | EXR+JPG + poses, PLY, OBJ, FBX, glTF | **Poses ARKit ingérables par `ns-process-data record3d` — supprime COLMAP.** Pendant payant de Spectacular Rec |
| **Spectacular Rec** + `sai-cli` | Via dataset Nerfstudio | Nuage / mesh OBJ (beta) | Ordinateur (Win/Linux x86) | **iPadOS 17+, LiDAR non requis** — *mais voir réserve ci-dessous* | **Gratuit**, SDK « free for non-commercial use » | **Non — usage non commercial** | transforms.json + images + COLMAP texte + sparse_pc.ply | **Supprime l'étape COLMAP** ; échelles documentées « table » et « room » seulement ; app iOS figée depuis mars 2024. **L'iPad figure dans la compatibilité App Store mais n'apparaît nulle part dans la documentation de l'éditeur, qui ne parle que d'iPhone : à tester avant de bâtir un pipeline dessus** |
| **3D Splat App** (Mac) | **Oui, local Metal** | **Non** | Ordinateur | — (Mac M1+, macOS 15.6+) | **Gratuit, 0 €**, aucun achat intégré | Non documenté | **PLY, PLY compressé, SPZ, SOG** | Pièces et bâtiments : 150–400 images pour une pièce, 400+ pour un bâtiment |
| **RadianceKit** (Mac) | Oui, local Metal | Non | Ordinateur | — | **7,99 $ US, achat unique** | Non documenté | PLY, SPZ, SOG, glTF, .splat | Pièce et grande scène, sans plafond |
| **SplatScene** (Mac) | Oui, local | Non | Ordinateur | — | 3 exports gratuits ; 34,99 $/an | Non documenté | PLY | Non documenté |
| **Brush** | Oui, local | Non | Ordinateur | — | **Gratuit, Apache-2.0** | **Oui** | PLY | Scènes entières ; **Mac/Windows/Linux, AMD/Nvidia/Intel** ; **binaires précompilés**, interface via `--with-viewer` |
| **OpenSplat** | Oui, local | Non (consomme un SfM) | Ordinateur | — | **Gratuit (à compiler)** ; binaire Windows 29 $ | **Oui** (AGPLv3) | **PLY, .splat, SPZ, .rad** | Scènes entières ; **CUDA / ROCm / Metal / CPU** ; ≈ 2 Go de VRAM par million de gaussiennes |
| **Postshot** | Oui | Non | Ordinateur | — | Free non commercial **sans export** ; Indie 204 €/an | Non en Free ; oui en Indie/Studio | PLY, SPZ (payant) | **Windows + NVIDIA obligatoires** |
| **LichtFeld Studio** | Oui | Non | Ordinateur | — | Gratuit compilé ; binaire **30 $ US** | Oui (GPLv3) | PLY, SOG, SPZ, HTML, USD | Scènes 10 M gaussiennes ; **NVIDIA CC 7.5+** |
| **gsplat / nerfstudio** | Oui | Non (splatfacto) | Ordinateur | — | **Gratuit, Apache-2.0** | Oui | PLY (avec `--save_ply`) | Excellent, **GPU NVIDIA requis** |
| **COLMAP 4.2** | Poses seulement | Oui (dense, CUDA) | Ordinateur | — | **Gratuit, BSD** | Oui | COLMAP, PLY, LAS | Toutes tailles ; lent |
| **RealityScan desktop 2.2** | Non | Oui, excellent | Ordinateur | — | **Gratuit < 1 M$ de revenus** | Oui sous ce seuil | OBJ, FBX, GLB, USDZ, LAS… | Meilleur mesh gratuit ; **pas de macOS** |
| **ODM / WebODM** | Via OpenSplat | Oui + orthophoto + DSM | Ordinateur | — | **Gratuit, AGPLv3** | Oui | PLY, LAZ, OBJ, GeoTIFF | Extérieurs, drone, pistes |
| **MeshLab / CloudCompare** | Non | **Post-traitement** (nettoyage, décimation, Poisson) | Ordinateur | — | **Gratuit** (GPL) | Oui | OBJ, PLY, STL, E57, LAS, GLTF… | **Indispensable en aval du mesh** (§5.5) |
| **SuperSplat** | Édition/publication | Non | Navigateur | Safari 26+ possible | **Gratuit, MIT** | Oui | PLY, SOG, SPZ, HTML | Indispensable en aval |
| **splat-transform** | Conversion/fusion | Collision voxel + GLB | Ordinateur (Node) | — | **Gratuit, MIT** | Oui | PLY, SOG, SPZ, GLB, CSV | Assemblage multi-zones |
| **Splatware** | Oui (cloud) | Payant | Cloud | Navigateur | Free 0 € (Lite illimité, 5 projets, 3 exports/jour) ; Creator 9,95 € puis 19,95 €/mois | Oui en Free | PLY, .splat | Pièce en gratuit, **sans app ni LiDAR** |

---

## 4. Pipeline recommandé (gratuit) — étape par étape

**Durée réaliste : 35 à 50 minutes. Coût : 0 €.**
**Une seule capture, deux livrables** (splat + mesh) : c'est l'étape 5 qui matérialise cette promesse.

### Étape 0 — Test de compatibilité (10 min, obligatoire)

Installez **Scaniverse** et **restez sur l'expérience « Classic / Legacy »** : ne créez pas de compte, ne migrez pas vers « New Scaniverse » (cloud à crédits). Faites un scan **Splat** de 60 s dans une pièce. Vérifiez trois choses : le mode Splat existe, le traitement aboutit en 1–2 min sans réseau, l'export PLY/SPZ fonctionne.

> **Où cliquer ?** Le libellé exact de l'écran d'accueil varie selon la version. Le test qui tranche est le suivant : si le sélecteur de mode propose **Splat** à côté de Mesh, vous êtes bien dans l'expérience Classic.

> **Piège** : la migration vers New Scaniverse est à sens unique ; des avis 2026 signalent l'impossibilité de revenir à Classic sans désinstaller et perdre ses scans.

> ### Plan B si le mode Splat n'apparaît pas (iPad 8, mini 5, Air 3 — puces A12/A13)
> Trois replis, par ordre d'effort croissant :
> 1. **Splatware Free** — 0 €, **100 % navigateur, sans app** : déposez une vidéo (jusqu'à 3 vidéos / 0,5 Go par projet), modèle « Lite » illimité, 5 projets, 3 exports/jour, usage commercial autorisé. C'est le chemin le plus court vers un splat depuis un iPad qui ne sait pas en calculer.
> 2. **Teleport by Varjo** — 5 captures d'essai gratuites, iPadOS 17+, sans LiDAR. Attention : aucun mesh, et l'export .ply est payant.
> 3. **Le pipeline §5** — filmez avec l'iPad, traitez sur ordinateur. C'est la seule voie qui ne dépend d'aucune app iPad.

### Étape 1 — Réglages avant le premier scan (3 min)

Dans Scaniverse : **activez la sauvegarde des données brutes**. Sans ce réglage, « Reprocess Scan » n'apparaîtra pas et il faudra refaire un scan complet. **Ce réglage n'est pas rétroactif.**

> **Le libellé exact varie selon la version.** Cherchez dans les Réglages une option du type *Save raw data* / *Keep capture data*. **Le test qui prouve que c'est actif** : faites un scan de 20 s, ouvrez-le, pressez l'icône « **…** » en haut à droite — si l'entrée **Reprocess Scan** est présente, le réglage est bon. Le support officiel formule la condition ainsi : le retraitement n'est possible que « *provided you have the raw data saved* ».

Activez « **Ignore Lidar** » si votre iPad n'a pas de LiDAR, ou si la pièce dépasse 5 m, ou en cas d'erreur « Scan Error » avec rayures rouges (contournement officiel donné par le support en mars 2026).

Côté iPadOS : **Ne pas déranger** (une notification casse le suivi), mode Avion, 10–15 Go libres, batterie > 80 %.

### Étape 2 — Préparer la scène (5 min)

Allumez toutes les lumières et n'y touchez plus. Fermez les stores côté fenêtre : **Scaniverse n'a aucun verrou d'exposition**, un contre-jour ruine le splat. Dégagez un chemin de circulation. Sortez personnes et animaux.

**Posez un mètre pliant ouvert à 1,00 m au sol, bien visible — y compris sur iPad Pro.** L'échelle du splat n'est garantie nulle part : c'est votre étalon pour l'étape 7.

Repérez un point de départ **riche en texture** — bibliothèque, tapis à motifs, plan de travail. Jamais un mur blanc nu.

### Étape 3 — Le scan splat (2–3 min)

Mode **Splat**. Trois passes, en **marchant** — la doc officielle les décrit ainsi : « *Hold the device level to capture mid-level features such as walls and furniture. Tilt the device upwards to capture the ceiling. Tilt the device downwards to capture the floor.* »

1. Tour du périmètre à hauteur des yeux, caméra vers le centre.
2. Tour en inclinant vers le **haut** (plafond).
3. Tour en inclinant vers le **bas** (sol, plinthes).

Image mentale du support officiel : « imaginez que vous scannez un objet invisible au milieu de la pièce ». Tournez le **corps**, pas le poignet. Visez **90 à 120 s**.

> **Piège** : ne dépassez jamais 180 s — l'app avertit elle-même, et le staff Niantic confirme que « very large splats can fail to process ». Ne pivotez jamais sur place : sans parallaxe, la reconstruction échoue. Une pièce = un scan.

### Étape 4 — Contrôle immédiat (2–4 min)

Le calcul se fait sur l'iPad. Inspectez plafond, sol, les quatre coins et les abords des fenêtres. Si c'est troué, **refaites tout de suite**, tant que l'éclairage est identique.

> Si l'app se ferme pendant le traitement : splat trop gros. Faites deux scans de 60 s au lieu d'un de 150 s.

### Étape 5 — Le mesh, depuis le même scan (3–5 min)

**C'est ici que la capture unique paie deux fois.** Ouvrez le scan → icône « **…** » en haut à droite → « **Reprocess Scan** » → **Mesh**. Le même jeu de données est recalculé en maillage texturé.

- **iPad Pro (LiDAR)** : mesh mesuré, à l'échelle réelle, mesures possibles dans l'app.
- **iPad sans LiDAR** : bascule en photogrammétrie — texturé, mais échelle non garantie.

**Alternative pour iPad Pro, si vous voulez un vrai nuage de points** : **Modelar — 3D LiDAR Scanner**, gratuit, sans abonnement, sans publicité et sans filigrane, traitement 100 % sur l'appareil. Exports **E57, LAS, PLY, CSV** pour le nuage et **GLB, USDZ, OBJ, STL, PLY** pour le maillage. C'est la seule solution gratuite de ce tableau qui donne de l'E57 et du LAS sur iPad. **Limites** : iPad Pro LiDAR uniquement, **aucun splat**, **aucun mode photogrammétrie de repli**.

**Variante sans LiDAR, mesh de meilleure qualité** : utilisez **Polycam**, mode « **Space** » non-LiDAR (lancé le 14/01/2026 ; iPad Air 5 / mini 6 / iPad 10e gén. et plus récents, 4 Go de RAM). Traitement **local et hors ligne**, gratuit, export **GLTF** — seul format du plan Free, mais lisible par Blender, MeshLab et CloudCompare.

> ### ⚠ Pièges Polycam 2026 — à lire avant de lancer une capture
> Les retours 2026 sont sévères et documentés. Protocole minimal pour éviter la déception :
> - **Une pièce par capture.** « *Never was able to scan more than 2-3 rooms before it quit* » ; « *crashes before I can finish a 750 sqft space* ». Polycam reconnaît que les plantages au traitement viennent « *presque toujours* » d'un manque de mémoire.
> - **Sessions ≤ 30 min**, c'est la recommandation de l'éditeur lui-même.
> - **Ne quittez pas l'app et ne laissez pas l'écran s'éteindre pendant le traitement** : « *Closing the app or letting your phone sleep… can cause it to fail* ».
> - **Batterie ≥ 80 %**, batterie externe au-delà de 20–30 min, **10–15 Go libres**, **Ne pas déranger** activé.
> - **Synchronisez et exportez AVANT toute mise à jour de l'app.** « *Updated the app and lost 80% of my scans* » — plusieurs témoignages concordants en 2026.
> - Le quota de captures du plan Free n'est **pas chiffré publiquement** : ne bâtissez pas un projet dessus.
>
> **Replis si Polycam plante ou exige un compte** : **KIRI Engine Photo Scan** (gratuit, jusqu'à **150 photos / 2 Go par scan**, exports illimités et sans filigrane en OBJ/STL/FBX/GLTF/GLB/USDZ, plus les **nuages de points PLY et XYZ**) ou **RealityScan Mobile** (gratuit, « *the application and all exports are completely free of charge* », exports OBJ et USDZ sur iOS). Pensez à télécharger vite chez KIRI : **une rétention de 3 jours est documentée côté API — appliquez-la par prudence à l'app**.

### Étape 6 — Exporter immédiatement (5 min)

Splat → **SPZ** (format ouvert MIT, ≈ 10× plus léger) **et PLY** (maître, 64 à 632 Mo). Mesh → **USDZ** (pour l'AR), **OBJ/FBX**, **LAS** pour le nuage.

> **Piège structurel** : les images et poses brutes **ne sortent jamais** de Scaniverse (« Scaniverse does not support exporting or transferring the raw capture data »). Un scan supprimé est perdu, « Transfer Scans » est signalé comme non fiable, et un scan raté **ne pourra jamais être rattrapé sur ordinateur**. Exportez le jour même.

> ### Étape 6 bis — Où ranger tout ça (5 min, à faire une seule fois)
> Un PLY de splat pèse **64 à 632 Mo par zone**, et une zone traitée au §5 mobilise **15 à 20 Go**. À trois ou quatre pièces, vous avez un problème de rangement, pas un problème de 3D. Arborescence qui se tient sur la durée :
>
> ```
> ~/scans3d/
> └── 2026-09-20_appartement/
>     ├── 00_ETALON.txt          ← longueur réelle du mètre pliant + repère utilisé
>     ├── zone01_salon/
>     │   ├── raw/               ← vidéo .mov, ou export brut s'il existe
>     │   ├── splat/             ← zone01.ply (maître) + zone01.spz (échange)
>     │   ├── mesh/              ← zone01.usdz, zone01.obj (+ textures)
>     │   └── work/              ← images/, undistorted/, sparse/ — SUPPRIMABLE
>     ├── zone02_cuisine/
>     └── assemblage/            ← maison.sog, transforms notés à l'étape 9
> ```
>
> **Trois règles qui évitent les regrets :**
> 1. **`work/` est jetable, `raw/` et `splat/` ne le sont pas.** Vous pouvez toujours régénérer `undistorted/` depuis la vidéo ; vous ne pouvez jamais régénérer un scan Scaniverse supprimé.
> 2. **Gardez le PLY comme maître, diffusez en SPZ ou SOG.** Ne supprimez jamais le PLY après compression : SPZ et SOG sont des formats avec perte.
> 3. **Sauvegardez `raw/` + `splat/` hors de la machine** (disque externe ou cloud) le jour de la capture. C'est ≈ 1 Go par zone — négligeable — et c'est la seule chose irremplaçable.

### Étape 7 — Nettoyer, mettre à l'échelle, redresser (15–25 min)

Ouvrez **superspl.at/editor** dans Chrome, Edge ou Safari 26+ (SuperSplat 3.0 exige WebGPU).

1. **Nettoyer** : supprimez la grosse sphère de splats lointains que génère Scaniverse, puis les floaters. Outils : rectangle, brosse, polygone, lasso, pipette, flood, sphère, boîte — plus la sélection par **attributs** (faible opacité, échelle extrême), qui abat l'essentiel du travail.
2. **Mettre à l'échelle** : outil **Measure**. Placez deux marqueurs sur les deux extrémités de votre mètre pliant, puis saisissez la longueur réelle : « *SuperSplat uniformly rescales the whole active splat around the midpoint of the two markers* ».
3. **Redresser** : outil **Orient**. Trois points sur une surface qui devrait être de niveau : « *The whole active splat rotates and translates so the picked plane lands exactly on the grid plane* ».
4. **Exporter** : PLY (maître), SPZ (échange), SOG (web).

### Étape 8 — Partager, sans compte (5 min)

- **Mesh** : le `.usdz` s'ouvre dans **Fichiers → Quick Look**, bouton AR pour le poser à l'échelle dans la vraie pièce. Aucune installation.
- **Splat** : bouton Partager de Scaniverse → lien de la carte publique Classic.
- **Splat, version soignée** : dans SuperSplat, **Publish** vers superspl.at (compte PlayCanvas gratuit) — compression SOG, streaming LOD au-delà d'un million de gaussiennes, lien « unlisted ». Ou export **Viewer App** en HTML autonome.

### Étape 9 — Plusieurs zones : la méthode d'alignement, pas à pas

Une pièce = un scan. Traversez les portes **très lentement** en gardant visible une portion de la pièce précédente.

> **Cause n°1 d'échec, à connaître avant de capturer** : **sans zone de recouvrement réellement filmée entre deux scans, aucun alignement n'est possible a posteriori** — ni à la main, ni en cloud. Preuve chiffrée côté cloud : un utilisateur a soumis 13 scans à la fusion, **3 seulement ont été intégrés**, pour 30 210 crédits consommés, faute de recouvrement. **Le recouvrement se gagne à la capture, jamais au montage.**

Procédure d'alignement dans SuperSplat, en quatre temps :

1. **Poser la zone de référence.** Ouvrez `zone01.ply` seul. Mettez-le à l'échelle (**Measure**, sur le mètre pliant) et redressez-le (**Orient** → Align to Grid). **Ne touchez plus jamais à zone01** : c'est votre repère monde.
2. **Importer la zone suivante dans la même scène.** Importez `zone02.ply` par-dessus. Sélectionnez-le dans le **Scene Manager** (panneau de scène), puis utilisez les outils **Move / Rotate / Scale** : « *Drag the gizmo handles in the viewport. For precise values, edit Position, Rotation, and uniform Scale in the Transform panel below the Scene Manager.* » Appuyez-vous sur un élément physiquement commun aux deux scans — **un chambranle de porte est le meilleur repère** : il est vertical, à arêtes franches, et visible des deux côtés.
3. **Relever les valeurs.** Une fois l'alignement visuellement correct, **notez les valeurs de Position et de Rotation affichées dans le panneau Transform**. Ce sont elles, et elles seules, qui vous permettront de reproduire l'opération.
4. **Rejouer en ligne de commande** pour un export reproductible et scriptable (§8) : `splat-transform` accepte exactement ces valeurs via `-t` (translation) et `-r` (rotation). C'est ce qui transforme un montage manuel en procédure rejouable.

> **Limite dure** : Scaniverse Classic **ne fusionne pas** les scans — chaque zone reste un fichier isolé.
> **Côté cloud** : le cloud Scaniverse sait fusionner plusieurs scans dans un **Site**, mais **la page tarifs ne précise pas à partir de quel palier cette fonction est ouverte**, et des sources secondaires l'attribuent à Plus/Pro. Le plan Free donne 20 000 crédits/mois (≈ 10–11 min de capture) : **testez la fusion sur deux scans courts avant d'en dépendre, et n'achetez Plus qu'après confirmation.**

---

## 5. Pipeline « qualité supérieure » (ordinateur ou GPU cloud gratuit)

À n'entreprendre que si vous butez sur les trois murs du gratuit. Comptez **une journée** pour la première pièce.

> ### ⚠ Ce qu'il faut installer d'abord, AVANT de filmer
> Chaque commande de cette section suppose un outil déjà installé. Les installer après avoir filmé et transféré plusieurs gigaoctets est le meilleur moyen d'abandonner en cours de route. **Comptez 30 à 60 minutes d'installation la première fois.**
>
> **Pour tout le monde :**
> - **ffmpeg** (extraction des images, §5.2) : `brew install ffmpeg` (macOS) · `winget install ffmpeg` (Windows) · `sudo apt install ffmpeg` (Linux).
> - **COLMAP** (poses caméra, §5.3) : voir les commandes d'installation au début du §5.3.
> - **Node.js LTS** (pour `splat-transform`, §8) : depuis nodejs.org, puis `npm install -g @playcanvas/splat-transform`.
>
> **Selon la voie d'entraînement choisie (§5.4) — une seule suffit :**
> - **3D Splat App** (Mac) : rien à installer d'autre, un clic sur le Mac App Store. **C'est la voie la plus courte.**
> - **Brush, binaire** : rien à installer, téléchargez l'archive de votre système. **Voie recommandée hors Mac.**
> - **Brush, compilé** : `rustup` (rustup.rs), Rust 1.88+, puis `cargo run --release`. **À ne faire que si vous voulez la branche main.**
> - **gsplat** (PC NVIDIA) : **Python 3.10**, un environnement virtuel ou **conda**, et le **toolkit CUDA correspondant à votre pilote** — la compilation CUDA de gsplat se fait **en JIT au premier lancement**, donc une version de CUDA incohérente ne se révèle qu'à ce moment-là.
>
> **Testez chaque outil avec `ffmpeg -version`, `colmap -h`, `node -v` AVANT de partir filmer.**

### 5.1 Capture vidéo à exposition verrouillée

Scaniverse n'a pas de verrou d'exposition. Filmez avec **Blackmagic Camera** (gratuit, iPadOS 18+) : **4K 30 fps H.265**, ISO et balance des blancs fixes, appui long pour verrouiller AF+AE. *(Sur iPad le plafond est 4K ; le 8K annoncé est réservé à Android.)*

**Vitesse d'obturation :** assez courte pour figer le mouvement — en pratique **1/250 s et plus**. La seule consigne éditeur vérifiée est celle de Postshot : « *Prefer short exposure times and small apertures to avoid both motion blur and defocus blur* », et il vaut mieux monter l'ISO que rallonger la pose, car « *radiance fields tend to tolerate noise better than blur* ». **Les valeurs chiffrées qui circulent en communauté (1/250 en photo, 1/400–1/500 en vidéo) n'ont aucune source éditeur** — traitez-les comme des ordres de grandeur, pas comme un réglage prescrit.

### 5.1 bis — Sortir les rushes de l'iPad (5 min, à régler AVANT de filmer)

Le §5.2 lance `ffmpeg -i zone01.mov` comme si le fichier était déjà sur l'ordinateur. Il ne l'est pas, et une vidéo 4K de 2 min pèse de l'ordre du **gigaoctet**.

**Avant de filmer**, dans Blackmagic Camera : **Réglages → Media → Save Clips To → `Files`** (les trois options sont *In-App Only*, *In-App and Photo Library*, et *Files to internal or external storage*). **Avec le réglage par défaut, vos rushes ne seront pas dans l'app Fichiers** et vous les chercherez longtemps.

**Ensuite, trois voies de transfert :**
- **SSD USB-C** : app Fichiers → glisser le clip vers le disque. La plus rapide pour plusieurs gigaoctets.
- **AirDrop** vers un Mac.
- **Partage réseau** (SMB via l'app Fichiers) vers un PC.

**Vérifiez que le fichier s'ouvre sur l'ordinateur avant de libérer de la place sur l'iPad.**

### 5.2 Extraire et trier les images

```bash
ffmpeg -i zone01.mov -vf fps=3 -qscale:v 1 -qmin 1 zone01/images/%05d.jpg
```

Puis **éliminez les images floues** avec Sharp Frames (gratuit, navigateur ou Python). **150 à 300 images nettes valent mieux que 1000 redondantes**, et réduisent fortement le temps de calcul.

> ### Place disque et temps : à quoi s'attendre avant de lancer
> Ordres de grandeur pour **une zone** de la taille d'une pièce (estimations, sauf mention contraire) :
>
> | Élément | Place disque |
> |---|---|
> | Vidéo 4K30 H.265 de 2 min | ≈ 0,7 à 1,5 Go |
> | 250–300 JPEG extraits en qualité 1 | ≈ 1,5 à 3 Go |
> | `undistorted/` (COLMAP recopie les images) | ≈ autant que les JPEG |
> | Base COLMAP + `sparse/` | quelques centaines de Mo |
> | PLY de splat en sortie | **64 à 632 Mo** (plage mesurée sur des splats Scaniverse) |
> | **Total à prévoir, par zone** | **15 à 20 Go libres** |
>
> Côté **temps**, le seul repère publié et vérifiable : un salon entraîné avec Brush sur un MacBook Air a demandé **478 s de COLMAP puis 3 h 34 d'entraînement** pour 30 000 itérations. Sur un PC NVIDIA récent, comptez plutôt des dizaines de minutes. **Une pièce = une demi-journée de machine, pas dix minutes.**

### 5.3 Poses caméra avec COLMAP 4.2

Installation : `brew install colmap` (macOS Apple Silicon), `conda install -c conda-forge colmap` (Linux), ou zip précompilé Windows.

```bash
DS=~/splat/zone01

colmap feature_extractor \
  --database_path $DS/database.db \
  --image_path $DS/images \
  --ImageReader.camera_model OPENCV \
  --ImageReader.single_camera 1 \
  --SiftExtraction.max_image_size 3200

colmap sequential_matcher --database_path $DS/database.db

# OBLIGATOIRE pour des frames extraites d'une vidéo (pas d'EXIF de focale)
colmap view_graph_calibrator --database_path $DS/database.db

colmap global_mapper \
  --database_path $DS/database.db \
  --image_path $DS/images \
  --output_path $DS/sparse

colmap model_analyzer --path $DS/sparse/0

colmap image_undistorter \
  --image_path $DS/images \
  --input_path $DS/sparse/0 \
  --output_path $DS/undistorted \
  --output_type COLMAP --max_image_size 3200
```

> **Pourquoi `view_graph_calibrator` n'est pas optionnel ici.** La documentation COLMAP est explicite : « *The global mapper depends on reasonably good focal length priors to perform well.* » Or les JPEG produits par `ffmpeg` à l'étape 5.2 **n'ont aucune métadonnée EXIF de focale**. Sans cette étape, l'alignement échoue ou se fragmente en plusieurs modèles — c'est le point de rupture le plus coûteux de tout le pipeline, puisqu'il survient après l'extraction et avant l'entraînement.
> `view_graph_calibrator` « *estime les focales et autres paramètres intrinsèques à partir des relations géométriques par paires* ». **Il modifie la base de données en place** : travaillez sur une copie.
> **Si vos images viennent d'un appareil photo avec EXIF complet, cette étape est facultative. Si elles viennent de ffmpeg, elle est obligatoire.**

Le `global_mapper` est GLOMAP, intégré à COLMAP depuis la 4.0.0 (mars 2026). Vous devez obtenir **un seul** dossier `sparse/0` et près de 100 % d'images enregistrées. Sinon : essayez le `mapper` incrémental, ou **hloc** (SuperPoint + LightGlue) sur les scènes peu texturées.

> ### Raccourci : sauter COLMAP entièrement
> Deux apps enregistrent la vidéo **avec les poses caméra déjà calculées**, ce qui supprime l'étape la plus lente et la plus fragile du pipeline.
>
> **Spectacular Rec** — gratuit, iPadOS 17+, **LiDAR non requis**. Sur PC, `sai-cli process --key_frame_distance 0.15 <capture> <sortie>` (0,15 m = réglage documenté pour une **pièce** ; 0,05 m pour une scène de la taille d'une table) produit directement un dataset Nerfstudio, prêt pour Brush, gsplat ou OpenSplat.
> **Limites** : SDK « *free for non-commercial use* » ; **roues Python disponibles uniquement pour Windows x86-64 et Linux x86-64 — pas de macOS** ; échelles documentées « table-sized » et « room-sized » seulement ; **app iOS figée depuis la v1.2.0 du 22 mars 2024** ; et **l'iPad figure dans la compatibilité App Store mais n'apparaît nulle part dans la documentation de l'éditeur, qui ne parle que d'iPhone : à tester avant de bâtir un pipeline dessus.**
>
> **Record3D** — son pendant payant (≈ **4–6 €**, achat unique), et la voie la mieux documentée. Il capture en RGBD avec les poses ARKit, que Nerfstudio ingère directement via `ns-process-data record3d --data <capture> --output-dir <sortie> [--ply-dir ...]`, **sans COLMAP**.
> **Limite dure** : Record3D exige « *FaceID ou LiDAR* », **donc un iPad Pro** — les iPad Air, mini et standard n'ont ni l'un ni l'autre et sont exclus.

### 5.4 Entraîner le splat

**Mac Apple Silicon — 3D Splat App** (gratuit, interface graphique, aucune ligne de commande) : importez le dossier COLMAP produit en 5.3, choisissez un preset (**Fast ≈ 2 min, Medium ≈ 5 min, HD ≈ 20 min**) ou réglez itérations / résolution / nombre max de splats à la main, puis exportez en PLY, PLY compressé, SPZ ou SOG. C'est le chemin le plus court pour un utilisateur Mac. **Elle ne produit aucun mesh.**

**Mac / Windows / Linux, tout GPU — Brush** (Apache-2.0, aucun GPU NVIDIA requis).

**Téléchargez le binaire correspondant à votre système sur la page Releases du dépôt** — dernière release **v0.3.0**, avec `brush-app-aarch64-apple-darwin.tar.xz` (Apple Silicon), `brush-app-x86_64-pc-windows-msvc.zip` (Windows x64) et `brush-app-x86_64-unknown-linux-gnu.tar.xz` (Linux x64). **Sur ce binaire, le drapeau s'appelle `--total-steps`.** Brush dispose d'une interface graphique : ajoutez `--with-viewer` à n'importe quelle commande pour l'ouvrir (« *Every CLI command can work with `--with-viewer` which also opens the UI, for easy debugging* »). Il fait aussi office de visionneuse de splats, y compris dans le navigateur (démo : `arthurbrussee.github.io/brush-demo`).

*Variante avancée* — **ne compilez que si vous voulez la branche main 1.0.0**, dont le drapeau est `--total-train-iters`. Cela suppose d'installer `rustup` au préalable :

```bash
git clone https://github.com/ArthurBrussee/brush && cd brush
cargo run --release -- --help          # IMPÉRATIF : les noms de flags ont changé
cargo run --release -- ~/splat/zone01/undistorted \
  --max-resolution 1920 --total-train-iters 30000 \
  --sh-degree 3 --export-path ~/splat/zone01/out --with-viewer
```

> Binaire v0.3.0 → `--total-steps`. Branche main 1.0.0 → `--total-train-iters`. Vérifiez toujours avec `--help`.

**PC NVIDIA — gsplat** (Apache-2.0) :

```bash
pip install gsplat
git clone https://github.com/nerfstudio-project/gsplat && cd gsplat/examples
pip install -r requirements.txt --no-build-isolation
python simple_trainer.py default \
  --data_dir ~/splat/zone01/undistorted --data_factor 1 \
  --save_ply True \
  --result_dir ~/splat/zone01/results
```

> ### ⚠ `--save_ply True` n'est pas optionnel
> Dans le code source de `examples/simple_trainer.py` (branche `main`), le défaut est `save_ply: bool = False` — **sans ce drapeau, aucun fichier `.ply` n'est écrit**, seulement des checkpoints `.pt` aux `save_steps`. Vous iriez au bout des 30 000 itérations, c'est-à-dire plusieurs heures de GPU, pour ne rien pouvoir ouvrir dans SuperSplat.
> Avec le drapeau, les PLY sont écrits aux itérations **7 000 et 30 000** (valeur par défaut de `ply_steps`), dans **`<result_dir>/ply/`**.

**Troisième voie, tous GPU confondus — OpenSplat** (AGPLv3, donc **utilisable commercialement**, contrairement à 2DGS et SuGaR) : il tourne sur **NVIDIA (CUDA), AMD (ROCm/HIP), Apple Silicon (Metal) et même en CPU pur (~100× plus lent)**, lit nativement les projets **ODX, OpenSfM, COLMAP, OpenMVG et Nerfstudio**, et exporte en **`.ply`, `.splat`, `.spz` ou `.rad`** (option `-o`). Coût GPU annoncé : « *~2GB of GPU memory for each million gaussians* » — comptez donc plusieurs Go pour une pièce. La compilation est gratuite sur toutes les plateformes ; un **binaire Windows précompilé et signé est vendu 29,00 $ US** en paiement unique.

**Sans GPU — trois options, de la plus gratuite à la plus rapide :**

| Service | Gratuit ? | GPU | Bon pour |
|---|---|---|---|
| **Kaggle** | **Oui**, ≈ 30 h/semaine, sessions 12 h | T4 ×2 (16 Go chacun) | Le premier essai. **Vérification SMS obligatoire** |
| **Lightning AI** | **Oui**, 15 crédits/mois (≈ 80 h GPU en interruptible), 1 Studio toujours actif avec cycle de 4 h | T4 0,19 $/h, L4 0,48 $/h | Environnement persistant type VS Code ; **prévoyez des checkpoints** à cause du cycle de 4 h |
| **RunPod** | Non (prépayé, ≈ 5 $ de bonus de parrainage) | RTX 4090 0,34 $/h | Le plus simple à coût quasi identique à Vast.ai sur 4090 |
| **Vast.ai** | Non (dépôt minimum 5 $) | RTX 4090 ≈ 0,33 $/h **on-demand** | **Son seul vrai avantage est l'A100 80 Go** (≈ 0,67–0,74 $/h contre 1,19–1,39 $/h chez RunPod) — utile précisément pour les **grandes scènes** que ce rapport recommande de découper |

> **Sur Vast.ai, méfiez-vous du prix « spot ».** Les 0,16 $/h souvent cités ne valent que pour les instances **interruptibles**, qui peuvent être reprises en cours d'entraînement. Sur un entraînement 3DGS de plusieurs heures sans reprise automatique, l'économie ne vaut pas le risque.
>
> **Règle d'or : ne faites jamais tourner COLMAP sur Kaggle** (4 cœurs CPU). Faites-le en local et n'envoyez que le dossier `undistorted`.

### 5.5 Mesh depuis le même jeu de photos

- **Mac, sans ligne de commande — Apple Object Capture.** C'est la **seule voie native, gratuite et sans terminal** pour obtenir un maillage texturé sur Mac à partir de photos prises avec n'importe quel iPad, LiDAR ou non. Deux frontends :
  - **Reality Composer Pro**, livré gratuitement avec **Xcode** : *File → New → Object Capture Model*.
  - **PhotoCatch** (Mac, gratuit avec des restrictions non documentées), qui accepte aussi une vidéo directement.

  **Méthode** : photos prises librement avec **≥ 70 % de recouvrement**, puis reconstruction **100 % locale**, export **USDZ/OBJ**. Les niveaux de détail *medium / full / raw* et les textures 16K sont **réservés au Mac** (iOS ne fait que *reduced*).
  **Limites à connaître avant de s'engager** : conçu pour objets et zones — Apple prévient qu'« *areas larger than 6 feet may have reduced mesh and texture quality unless processed at a higher detail level on Mac* », soit environ **1,80 m** ; plafond d'environ **2 000 images** (« *on Mac models with lots of unified memory you can process up to 2000 images* ») ; et **pas d'échelle réelle** si les photos ne viennent pas d'un appareil LiDAR.

- **PC Windows/Linux — RealityScan 2.2** : même alignement, mesh texturé, exports OBJ/FBX/GLB/USDZ/LAS. Meilleure qualité gratuite sous le seuil de 1 M$ de revenus. **Pas de version macOS.**

- **PC Windows/Linux, alternative libre à RealityScan — Meshroom** (AliceVision, **MPL2, gratuit, interface graphique nodale**) : photogrammétrie complète produisant un **maillage texturé** et des **nuages de points densifiés**, sans compte ni cloud. C'est la seule option gratuite à interface graphique si RealityScan est écarté — par exemple **au-delà du seuil de 1 M$ de revenus**.
  **Deux réserves vérifiées** : (1) **sans GPU NVIDIA compatible CUDA**, on se limite au **« Draft Meshing »**, qui maille le seul nuage épars et donne un **aperçu grossier inexploitable** comme scan ; (2) **il n'existe aucun build macOS officiel**.

- **PC NVIDIA, tout COLMAP** : `patch_match_stereo` → `stereo_fusion` → `poisson_mesher`. **Impossible sur Mac** (CUDA requis).

- **Mac ou PC sans GPU, extérieur** : **ODM** via Docker → nuage LAZ géoréférencé, mesh OBJ texturé, orthophoto GeoTIFF, DSM/DTM.

- **Partout, depuis le splat** : **3DGS-to-PC** (Apache-2.0), qui échantillonne un nuage **dense** puis maille par Poisson, avec un renderer Python ne nécessitant aucun CUDA.

> ### ⚠ Un mesh de scan brut n'est utilisable nulle part en l'état
> Quelle que soit la voie ci-dessus, la sortie est bruitée, trouée, et bien trop lourde pour un moteur de jeu, un simulateur ou de la CAO. **Prévoyez systématiquement une passe de nettoyage** — c'est une étape du pipeline, pas une finition optionnelle :
>
> - **MeshLab** (gratuit, GPL, Windows/macOS/Linux) pour le maillage : *Filters → Remeshing → **Simplification: Quadric Edge Collapse Decimation***, et sa variante **« (with texture) »** qui préserve les UV ; **Close Holes** pour les trous ; **Screened Poisson** pour refermer une surface. Attention sur une scène ouverte (extérieur, piste) : Poisson extrapole et crée des « bulles » — rognez ensuite par le champ scalaire de densité.
> - **CloudCompare** (gratuit, GPL) pour les nuages : filtre **SOR** (points aberrants), sous-échantillonnage, recalage **ICP** entre zones, **Cross Section** pour extraire des tranches, **Rasterize** pour produire une heightmap ou un DEM.
> - **Blender**, modificateur **Decimate** (mode *Collapse* ratio 0,1–0,3, ou *Planar* pour murs et sols) quand la cible est un moteur de jeu ou Gazebo (§9).
>
> **Ni MeshLab ni CloudCompare ne comprennent un splat**, seulement les maillages et les nuages — voir le piège des normales nulles au §8.

### 5.6 Contrôle qualité chiffré

Réservez une image sur 8 pour l'évaluation et lisez le **PSNR** rapporté par Brush ou gsplat. Repère publié et vérifiable : un salon entraîné avec Brush sur MacBook Air atteint **26,87 dB** après 3 h 34 (COLMAP : 478 s). *(Les valeurs souvent citées de 25,2 dB pour Scaniverse et 28 dB pour les implémentations de référence ne sont pas sourcées — à ne pas prendre pour argent comptant.)* Second repère : un PLY exporté **< 1 Mo** signale une capture ratée ; 20–50 Mo est correct.

---

## 6. Option « je paie un peu » : quand ça vaut le coup, et quand non

### Ce qui vaut le coup

| Dépense | Prix (devise, relevé) | Condition | Ce que ça lève |
|---|---|---|---|
| **RadianceKit** | **7,99 $ US**, achat intégré **unique** (App Store US, 2026-09-20) | macOS **26.0+**, Mac **M1+** | Les 3 murs : durée de scan, absence de fusion, absence de données brutes. Vidéo sans limite, entraînement Metal local, export PLY/SPZ/SOG/glTF. **Mais comparez d'abord avec 3D Splat App, gratuite** |
| **Scaniverse Plus, 1 mois** | **20 $/mois, ou 200 $/an en annuel** (page tarifs, 2026-09-19) | Projet ponctuel | 40 000 crédits/mois (≈ 22 min de capture) + splats depuis vidéo 360° + export USDZ. **Vérifiez d'abord avec le quota Free si la fusion multi-scans vous est bien ouverte** (voir encadré ci-dessous) |
| **LichtFeld Studio, binaire** | 30 $ US, paiement unique | PC NVIDIA CC 7.5+ | Interface complète ; **gratuit si vous compilez** |
| **OpenSplat, binaire Windows** | **29,00 $ US**, paiement unique | Windows, pour éviter la compilation | Gain de temps seulement — **le code est gratuit sur toutes les plateformes** |
| **KIRI Engine Pro** | 17,99 $/mois ou **79,99 $/an** | **Ni** Mac Apple Silicon **ni** PC NVIDIA | Splat + « 3DGS to Mesh » sans ordinateur |
| **Record3D** | ≈ **4–6 €**, achat unique | iPad **Pro** (LiDAR), pipeline Nerfstudio | Supprime COLMAP : poses ARKit ingérées par `ns-process-data record3d` |
| **RunPod RTX 4090** | 0,34 $/h (Community, 2026-09-13) | Aucun GPU local | ≈ 0,35 à 1 $ par scène |
| **Vast.ai A100 80 Go** | ≈ 0,67–0,74 $/h | **Grandes scènes uniquement** | ≈ 40 % moins cher que RunPod sur cette carte précise |

**Équivalents 0 €** pour chaque ligne :
- **3D Splat App** (Laan Labs, Mac App Store, **gratuite, aucun achat intégré**, M1+ / macOS 15.6+, interface graphique, presets Fast/Medium/HD, export PLY/SPZ/SOG) remplace **RadianceKit**. **Elle ne produit aucun mesh.**
- **Brush** (Apache-2.0, Mac/Windows/Linux, AMD/Nvidia/Intel, binaires précompilés, interface via `--with-viewer`) remplace RadianceKit et Postshot — et couvre en plus les PC AMD/Intel, ce que 3D Splat App ne fait pas.
- **Reality Composer Pro** (gratuit avec Xcode) ou **PhotoCatch** remplacent une app de scan payante pour le mesh sur Mac.
- **Meshroom** remplace RealityScan sur PC si vous dépassez le seuil de 1 M$ de revenus.
- **OpenSplat compilé** et **LichtFeld compilé** remplacent leurs binaires payants.
- **Kaggle** ou **Lightning AI** remplacent RunPod.
- **Spectacular Rec** (gratuit, non commercial) remplace Record3D — mais sans LiDAR et avec la réserve iPad du §5.3.
- **CorbeauSplat** (freddewitt/CorbeauSplat, MIT, macOS Apple Silicon, orchestre COLMAP + Brush + SuperSplat + splat-transform) existe bel et bien — *vérifié le 2026-09-20 — mais sans retour d'usage indépendant : à tester avant d'en dépendre.*

> **Avant de payer Plus pour un grand site, lisez ceci.**
> **Point 1 — les plafonds par scan ne s'achètent pas.** Le cloud Scaniverse impose des limites **par scan**, indépendantes du plan : « *Each scan supports up to five minutes of recording time and can cover up to 500 square meters.* » Payer ne vous dispense pas de découper — cela vous donne seulement plus de crédits pour traiter les morceaux.
> **Point 2 — le palier de la fusion multi-scans n'est pas établi.** Le cloud Scaniverse sait fusionner plusieurs scans dans un **Site**, mais la page tarifs mentionne « *combining multiple scans into one asset* » uniquement dans son texte de présentation, **sans rattacher la fonction à aucun palier** — ni Free, ni Plus, ni Pro — tandis que des sources secondaires l'attribuent à Plus/Pro. **Testez la fusion sur deux scans courts avec vos 20 000 crédits Free, et n'achetez Plus qu'après confirmation.**

### Ce qui ne vaut pas le coup

| À ne pas payer | Prix | Pourquoi |
|---|---|---|
| **Postshot Indie** | 204 €/an | Même service que 3D Splat App (gratuite) ou Brush (gratuit) ; Windows + NVIDIA obligatoires ; le plan gratuit n'exporte **ni PLY ni SPZ** |
| **Polycam Basic** | 150 $/an | **Ne sort pas le PLY du splat** — classé Business (400 $/an) par le Help Center du 16/09/2026 |
| **Scaniverse Plus** | **20 $/mois, ou 200 $/an en annuel** | Plafonné à ≈ **22 min de capture cloud par mois** (40 000 crédits à 30 crédits/s), et les limites de 5 min / 500 m² par scan restent. Ne le prenez qu'après avoir épuisé le Free |
| **Polycam, essai 7 jours** | — | Bascule automatique en abonnement annuel ; dizaines d'avis 2026 |
| **Teleport by Varjo** | **à partir de 30 $ prépayés, puis facturation au nombre d'images (prix unitaire non public)** | Aucun mesh, aucun plan gratuit durable |

> **Droits commerciaux** : le mode Classic gratuit de Scaniverse n'en ouvre **aucun**. La page tarifs est explicite : « *For commercial rights, including resale, you must be on a Pro plan or Enterprise contract* » (50 $/mois ou 500 $/an). **2DGS et SuGaR** sont sous licence Inria non commerciale, et **Spectacular Rec** sous SDK non commercial. À l'inverse, la chaîne **COLMAP (BSD) + Brush/gsplat (Apache-2.0) + OpenSplat (AGPLv3) + Meshroom (MPL2) + SuperSplat (MIT)** est libre de tout usage, y compris commercial — c'est la colonne « Droits commerciaux » du tableau §3.

---

## 7. Bonnes pratiques de capture — checklist

☐ **Translater, jamais pivoter sur place.** Postshot est catégorique : « *Always move the camera* », sans panoramique ni rotation sans déplacement, la reconstruction reposant sur la triangulation depuis des positions différentes. Confirmé par le papier LighthouseGS : « *panorama-style motion usually fails to correctly perform COLMAP* ».
☐ **Trois passes par zone** : hauteur des yeux, vers le plafond, vers le sol.
☐ **Recouvrement** : **30 à 50 %** entre photos selon Postshot, 70 à 80 % entre frames vidéo ; chaque surface vue dans **au moins 3 images**. Pour Apple Object Capture, visez **≥ 70 %**.
☐ **1 à 3 minutes maximum par scan Scaniverse** ; l'app avertit à 180 s.
☐ **Obturateur court, ouverture fermée, ISO élevé plutôt que pose longue** — « *Prefer short exposure times and small apertures* », et « *radiance fields tend to tolerate noise better than blur* ».
☐ **Démarrer sur une zone texturée**, jamais un mur nu. Collez des post-it sur les grands aplats si nécessaire.
☐ **Lumière homogène et constante.** Stores fermés côté fenêtre. Ciel couvert en extérieur.
☐ **Recouvrement réel entre zones** : pause avant et après chaque porte, traversée lente. **Sans recouvrement filmé, aucun alignement n'est récupérable.**
☐ **Mètre pliant ouvert à 1,00 m posé dans le champ, dans tous les cas** — l'échelle du splat n'est garantie nulle part.
☐ **Ne pas déranger** activé, batterie > 80 %, 10–15 Go libres.
☐ **Régler `Save Clips to → Files` dans Blackmagic Camera AVANT de filmer** (§5.1 bis), sinon vos rushes resteront introuvables dans l'app Fichiers.
☐ **Renoncer d'avance** aux miroirs, vitres, eau, carrosseries, grillages fins et personnes en mouvement.
☐ **Exporter le jour même**, en SPZ **et** PLY, et ranger selon l'arborescence du §4 étape 6 bis — et **télécharger vos résultats KIRI sans tarder : une rétention de 3 jours est documentée côté API, appliquez-la par prudence à l'app.**
☐ **Extérieur** : boucle de périmètre puis quadrillage, en gardant des repères fixes et texturés dans le cadre (la doc officielle reconnaît la « Large outdoor drift » en zone ouverte).

---

## 8. Visualiser, partager, éditer, convertir

**Formats.** PLY = maître, universel, lourd (64–632 Mo). **SPZ** = format ouvert MIT de Niantic, ≈ 10× plus léger, lu par SuperSplat, Postshot, Babylon.js, Adobe, Blender 5.3. **SOG** = 15 à 20× plus léger que le PLY, format de diffusion web. **.splat / .ksplat** = viewers historiques.

**Conversion et assemblage — splat-transform** (MIT, `npm install -g @playcanvas/splat-transform` — **nécessite Node.js LTS**, voir l'encadré d'installation du §5). Options vérifiées le 2026-09-20 : `-t/--translate`, `-r/--rotate`, `-s/--scale`, `-H/--filter-harmonics <0|1|2|3>`, `-N/--filter-nan`, `-B/--filter-box`, `-d/--decimate`, `-F/--filter-floaters`, `-C/--filter-cluster`. Fusion de plusieurs zones — **les valeurs `-r` et `-t` sont celles que vous avez relevées dans le panneau Transform de SuperSplat à l'étape 9 du §4** :

```bash
splat-transform zone01.ply \
  zone02.ply -r 0,90,0 -t 4.2,0,-1.1 \
  zone03.ply -t 9.6,0,-1.4 \
  maison.sog
```

**Splat → mesh.** Trois voies gratuites :

1. **3DGS-to-PC** (Apache-2.0) — nuage dense puis Poisson via Open3D, renderer Python sans CUDA. **Voie recommandée.**
2. **splat-transform**, collision voxelisée. **Le fichier de sortie `.voxel.json` est un argument positionnel obligatoire** : sans lui, la commande ne produit rien.
   ```bash
   splat-transform piste.ply -C --seed-pos 0,0,0 \
     --voxel-floor-fill --collision-mesh smooth piste.voxel.json
   ```
   La sortie est double : `piste.voxel.json` (métadonnées) + `piste.voxel.bin` (octree binaire), **et le maillage de collision est écrit à côté, en `piste.collision.glb`**. `--collision-mesh` accepte une valeur optionnelle `[smooth|faces]` (défaut : `smooth`). **Choisissez le bon remplissage** : `--voxel-floor-fill` remplit chaque colonne depuis le bas — c'est le mode des **scènes extérieures** ; `--voxel-external-fill` scelle les voxels extérieurs par inondation depuis la frontière — c'est le mode des **intérieurs**. Valeurs par défaut : `--voxel-size 0.05`, `--voxel-opacity 0.1`. **Ces valeurs sont en unités monde** : sans la mise à l'échelle de l'étape 7, le résultat est inexploitable.
3. **2DGS / SuGaR** — meilleure fidélité géométrique, mais **licence Inria + MPII, recherche et évaluation uniquement, usage commercial interdit**. SuGaR est de plus gelé depuis septembre 2024.

> **Piège à connaître absolument** : dans un PLY 3DGS, les champs `nx`, `ny`, `nz` valent **zéro**. Une reconstruction de Poisson lancée directement dans CloudCompare ou MeshLab rend un calque **vide**, silencieusement. Il faut supprimer puis recalculer les normales, ou utiliser 3DGS-to-PC.

**Blender.** La 5.3 apporte l'import natif PLY/SPZ et le rendu 3DGS (Workbench, EEVEE, Cycles) — mais elle est **en alpha jusqu'au 30/09/2026**, sortie stable annoncée le 10/11/2026, **sans export**, avec des performances signalées comme non idéales. *(Un défaut d'« Apply Transform » sur l'échelle, la rotation et les harmoniques sphériques est **rapporté mais non vérifié** dans cette revue — testez avant de vous y fier.)* Pour travailler aujourd'hui : **Blender 5.2 LTS + add-on « 3DGS Render » de KIRI** (gratuit, GPL, PLY 3DGS uniquement). Pour la décimation d'un mesh de scan, le modificateur **Decimate** est pleinement valable dès la 5.2 LTS.

**Unity.** `aras-p/UnityGaussianSplatting` (MIT) : menu *Tools → Gaussian Splats → Create GaussianSplatAsset*. Exige D3D12, Metal ou Vulkan — DirectX 11 ne fonctionne pas. *L'auteur indique ne plus prévoir de développements significatifs : projet vivant par sa communauté.*

**Unreal.** `xverse-engine/XScene-UEPlugin` (Apache 2.0, rendu Niagara). *Le dépôt annonce « Unreal Engine 5.0+ » et ses notes citent UE 5.2 à 5.4 — vérifiez la compatibilité avec votre version avant de vous engager.* Le plugin Luma AI est de facto abandonné.

**Web.** **Spark** (World Labs, MIT, three.js) avec son CLI `build-lod` et le format streamable `.RAD` : des scènes de 73 à 106 millions de splats sont rendues en temps réel, y compris sur mobile.

**Sur iPad.** Safari suffit pour un lien superspl.at. Sinon **MetalSplatter**, app App Store gratuite et open source (PLY, SPZ, .splat). *Évitez Spatial Fields à 24,99 $ : des avis signalent des plantages précisément sur les splats de 500 Mo — votre cas d'usage.*

---

## 9. Aller plus loin : intégrer l'environnement scanné dans ROS / Gazebo / RViz

*Section pertinente uniquement si votre projet est bien la collection de circuits virtuels ROS de ce dépôt.*

> ### ⚠ Ce dépôt est un paquet **catkin ROS 1**, pas ROS 2
> Vérifié dans les fichiers : `package.xml` au format 2, `config/racetrack.rviz` avec des classes `rviz/*`, fichiers de lancement en XML, topic `/shape`, Fixed Frame `/map`, namespace `MWS`, markers `type: 4` et `id: 0`, règles `install()` commentées dans le `CMakeLists.txt`.
> **Aucune commande ROS 2 ne s'applique telle quelle ici.** Le tableau ci-dessous sépare explicitement les deux mondes. ROS 1 Noetic est par ailleurs en fin de support depuis mai 2025 : si vous démarrez un nouveau projet, la colonne de droite est la bonne.

| Besoin | **ROS 1 Noetic (ce dépôt)** | **ROS 2 / Gazebo moderne (si migration)** |
|---|---|---|
| Sauver une carte 2D | `rosrun map_server map_saver -f ma_carte` → `ma_carte.pgm` + `ma_carte.yaml` | `ros2 run nav2_map_server map_saver_cli` |
| Nuage → grille d'occupation | `octomap_server` (OctoMap 3D incrémentale ; publie une projection 2D sur le topic `projected_map`, en `nav_msgs/OccupancyGrid`), puis `map_saver` | `pcd2pgm`, ou Nav2 + `map_saver_cli` |
| Simulateur | **Gazebo Classic 11** | Gazebo Harmonic |
| Chemin des modèles | **`GAZEBO_MODEL_PATH`** (chemins séparés par `:`), avec `<include><uri>model://…</uri></include>` | `GZ_SIM_RESOURCE_PATH` |
| Version SDF | **1.6 / 1.7** | 1.11 |
| Visualiseur | RViz (ROS 1) | RViz2 |

**Afficher un mesh scanné dans RViz (ROS 1).** Placez `piste.dae` **et ses textures** dans un dossier `meshes/`.

> ### ⚠ Ne partez pas d'un fichier launch vide
> **Dupliquez un fichier de lancement existant** (par exemple `launch/race-track-26NKzM0j.launch`). Vérifié dans le dépôt : ces fichiers déclarent déjà en tête les arguments `h`, `orientation`, `scale`, `color`, `position`, `pose` et `lt`. L'extrait ci-dessous **les utilise sans les redéclarer** : collé dans un fichier vierge, il produit une erreur `roslaunch`.
> **Procédure** : dupliquez le fichier, remplacez le bloc de coordonnées entre les commentaires `start/end of race track coordinates`, remplacez le nœud `pub1`, puis ajoutez les lignes suivantes — en gardant `ns: 'MWS'` mais avec un `id` différent de 0 :

```xml
<arg name="mesh_uri" value="mesh_resource: package://virtual_racetracks_collection/meshes/piste.dae"/>
<arg name="mesh_marker" value="'{$(arg h), ns: 'MWS', id: 1, type: 10, action: 0,
     $(arg pose), scale: {x: 1.0, y: 1.0, z: 1.0}, $(arg color), $(arg lt),
     $(arg mesh_uri), mesh_use_embedded_materials: true}'" />
<node name="pub_mesh" pkg="rostopic" type="rostopic"
      args="pub /shape visualization_msgs/Marker $(arg mesh_marker)"/>
```

`type: 10` est `MESH_RESOURCE`. `scale: 1 1 1` signifie taille native. RViz délègue le chargement des DAE/OBJ/STL à Assimp et résout les URI `package://`. *Note : toutes les règles `install()` du CMakeLists sont commentées, donc l'URI se résout depuis l'espace source.*

**Rester au format actuel du dépôt.** Extrayez la ligne centrale du scan avec **CloudCompare** (outil *Cross Section* : tranche horizontale puis extraction des contours, disponible depuis la 2.12), exportez en CSV, puis :

```bash
python3 scripts/racetrack_arg_converter.py piste.csv
```

> ### Ce que fait réellement le script (vérifié dans le code)
> - **Il n'affiche rien à l'écran** : il écrit sa sortie dans un fichier **`<nom>.txt` créé à côté du CSV** (`csv_file_name.replace(".csv", ".txt")`), et se contente d'afficher « *Copy the covertion result from …* ».
> - **Il saute la première ligne** du CSV (`next(reader)`) : votre fichier doit avoir une ligne d'en-tête, sinon le premier point est perdu.
> - **Il exige exactement deux colonnes.** La ligne `X, Y = map(float, row)` lève une exception dès qu'une ligne en contient trois ou plus — **ce qui est le cas par défaut d'un export CloudCompare** (X, Y, Z, et souvent des champs scalaires). **Supprimez les colonnes surnuméraires avant de lancer le script.**
> - **Il force `z: 0.0`** : il ne convient qu'à un tracé sensiblement plan.
>
> Le contenu de `<nom>.txt` est à coller entre les commentaires `start/end of race track coordinates`.

**Gazebo Classic 11** (le simulateur de ce dépôt). Modèle statique, visuel détaillé et collision décimée, en **SDF 1.6/1.7** :

```xml
<collision name="collision">
  <geometry>
    <mesh>
      <uri>model://piste_scan/meshes/piste_collision.dae</uri>
    </mesh>
  </geometry>
</collision>
```

Placez le dossier du modèle dans un répertoire listé par **`GAZEBO_MODEL_PATH`** (chemins séparés par `:` ; vérifiez avec `echo $GAZEBO_MODEL_PATH`), puis référencez-le par `<include><uri>model://piste_scan</uri></include>`.

**Décimez impérativement le mesh avant** — voir l'encadré de nettoyage du §5.5 : modificateur **Decimate** de Blender (mode Collapse ratio 0,1–0,3, ou mode Planar pour murs et sols), ou *Quadric Edge Collapse Decimation* dans MeshLab. Un mesh de scan brut saturera la mémoire du moteur physique.

*Si vous migrez vers Gazebo Harmonic, c'est là que `<mesh optimization="convex_decomposition">` et `GZ_SIM_RESOURCE_PATH` (SDF 1.11) deviennent pertinents — pas avant.*

**Cartes de navigation (ROS 1).** PLY → PCD dans CloudCompare, puis `octomap_server` pour construire la grille et publier `projected_map`, enfin `rosrun map_server map_saver` → PGM + YAML. Terrain : CloudCompare *Rasterize* → PNG niveaux de gris **carré**, redimensionné en 2ⁿ+1 pour Gazebo Classic, puis `<heightmap>`.

**Les deux idées transposables hors ROS** : la mise à l'échelle métrique (§4, étape 7) et la génération d'un mesh de collision directement depuis le splat (§8) — c'est la seule voie « géométrie » qui ne dépende ni du LiDAR ni d'un GPU.

---

## 10. Récapitulatif des coûts et matrice de décision

### Coûts (devise d'origine, relevé les 19–20 septembre 2026)

| Poste | Gratuit | Payant |
|---|---|---|
| Capture splat sur iPad | Scaniverse Classic — **0 €, illimité** | — |
| Capture mesh sur iPad | Scaniverse LiDAR / Polycam Space non-LiDAR / **Modelar (iPad Pro, E57+LAS)** / KIRI Photo Scan (+ PLY/XYZ) / RealityScan Mobile — **0 €** | Polycam Basic 150 $/an (inutile ici) |
| Vidéo à exposition verrouillée | Blackmagic Camera — **0 €** | — |
| Capture avec poses (sans COLMAP) | **Spectacular Rec + sai-cli — 0 €** (non commercial, Win/Linux x86, réserve iPad) | **Record3D ≈ 4–6 €** (iPad Pro uniquement) |
| Entraînement local Mac | **3D Splat App**, Brush, CorbeauSplat — **0 €** | RadianceKit **7,99 $ US une fois** |
| Entraînement local PC/AMD/Intel | **Brush (binaire précompilé)**, **OpenSplat compilé** — **0 €** | OpenSplat binaire Windows 29 $ |
| Entraînement local PC NVIDIA | gsplat, LichtFeld compilé — **0 €** | LichtFeld binaire 30 $ ; Postshot Indie 204 €/an |
| GPU cloud | Kaggle ≈ 30 h/sem (**vérification SMS obligatoire**), **Lightning AI 15 crédits/mois ≈ 80 h interruptible**, Colab 12 h — **0 €** | RunPod 0,34 $/h ; **Vast.ai A100 80 Go ≈ 0,67–0,74 $/h** (grandes scènes) |
| Poses caméra | COLMAP — **0 €** | — |
| Mesh photogrammétrie | RealityScan < 1 M$, **Meshroom (MPL2)**, ODM, **Reality Composer Pro / PhotoCatch (Mac)** — **0 €** | Metashape Standard 179 $ |
| Nettoyage et décimation du mesh | **MeshLab, CloudCompare, Blender — 0 €** | — |
| Édition, conversion, hébergement | SuperSplat, splat-transform, superspl.at — **0 €** | — |
| **Fusion multi-scans automatique** | **Cloud uniquement, palier non précisé par l'éditeur — à tester avec le quota Free** (20 000 crédits/mois ≈ 10–11 min de capture) | Plus 20 $/mois (ou 200 $/an) **si le test confirme que Free ne suffit pas** |
| Droits commerciaux | Chaîne COLMAP + Brush/gsplat/OpenSplat + Meshroom + SuperSplat — **0 €** | Scaniverse Pro 50 $/mois ou 500 $/an |

> **Avertissement sur la fusion cloud** : **sans recouvrement réellement filmé entre les scans, les scans excédentaires sont silencieusement exclus de l'asset final**. Cas documenté : 13 scans soumis, **3 seulement intégrés**, **30 210 crédits consommés** — soit davantage que le quota mensuel du plan Free, dépensés pour rien. Testez la fusion sur 2 scans courts avant d'y engager tous vos crédits.

> **Rétention côté serveur — à ne pas découvrir trop tard.** **KIRI Engine** : une rétention de **3 jours** est documentée **côté API** (« *your models are stored on our servers for 3 days. After this period, they'll be automatically deleted* ») — **appliquez-la par prudence à l'app**, faute de documentation équivalente. **Scaniverse cloud** : les traitements sont longs et sans délai garanti (des scans bloqués « en traitement » pendant près d'une semaine ont été rapportés sur le forum officiel). **Lancez un traitement cloud, puis téléchargez dès qu'il aboutit** — un aller-retour d'une semaine peut tout faire perdre.

**Total conseillé : 0 € pour démarrer, 0 € ensuite dans la grande majorité des cas (7,99 $ au maximum si vous préférez RadianceKit à 3D Splat App).**

### Matrice de décision

| Votre situation | Splat | Mesh | Coût | Section |
|---|---|---|---|---|
| iPad Pro seul, une pièce | Scaniverse Classic | Reprocess Scan → Mesh LiDAR (ou Modelar pour E57/LAS) | 0 € | §4 |
| iPad Air/mini/standard seul, une pièce | Scaniverse Classic | Polycam Space non-LiDAR (replis : KIRI PLY/XYZ, RealityScan) | 0 € | §4 |
| iPad sans mode Splat (A12/A13) | **Splatware Free** (navigateur) ou §5 | Polycam / KIRI | 0 € | §4 étape 0 |
| Appartement / maison | Scaniverse par pièce + SuperSplat ; fusion cloud **à tester avec le quota Free** | idem, par pièce | 0 € | §4 étape 9 |
| iPad + Mac Apple Silicon | **3D Splat App (0 €)**, Brush, ou RadianceKit | **Reality Composer Pro / PhotoCatch (0 €, sans terminal)**, 3DGS-to-PC ou ODM | 0–7,99 $ | §5, §6 |
| iPad + PC NVIDIA | gsplat ou LichtFeld | RealityScan 2.2, ou Meshroom | 0 € | §5 |
| **PC Windows/Linux, GPU AMD ou Intel** | **Brush, binaire précompilé**, ou **OpenSplat (ROCm/HIP)** | RealityScan 2.2 (AMD supporté depuis la 2.2) | 0 € | §5.4 |
| **PC sans GPU NVIDIA, mesh seulement** | — | **Meshroom limité au Draft Meshing** → préférez RealityScan ou ODM | 0 € | §5.5 |
| **Aucun GPU** | Kaggle, Lightning AI, ou Splatware Free | ODM Docker | 0 € | §5.4 |
| Extérieur étendu, piste, circuit | **Drone + ODM/RealityScan** ; **Vast.ai A100 80 Go** si la scène est très lourde | idem | 0 € + drone | §5.5 |
| Usage professionnel | Chaîne COLMAP + Brush/OpenSplat + SuperSplat | RealityScan, Meshroom | 0 € (licences libres) | §5, §6 |

### Ce qui ne marchera pas — à savoir avant de commencer

- **Circuit ou piste complète : non.** La capture depuis un véhicule en mouvement est **explicitement non supportée** par Niantic (échec confirmé en juillet 2026 sur une vidéo 360° de 4 min 57 filmée en voiture : « *scan creation from a moving vehicle is currently unsupported* »). Une vidéo 360° donne **un seul** splat non fusionnable. Repli honnête : tronçons et points d'intérêt (virage, stand, ligne d'arrivée), ou drone + pipeline ordinateur.
- **Métrologie : non.** Comptez ±5 à 10 cm bruts sur une pièce de 6 m, LiDAR limité à ≈ 5 m, et un mesh dont la géométrie « privilégie l'aspect visuel sur la précision ». **L'échelle du splat, elle, n'est garantie par aucune documentation** — d'où l'étalon physique.
- **Droits commerciaux : non** en gratuit chez Scaniverse (ni avec 2DGS/SuGaR, ni avec Spectacular Rec). Oui avec COLMAP + Brush/gsplat/OpenSplat + Meshroom + SuperSplat.
- **Fusion automatique de scans : non** en version Classic. **Côté cloud, le palier n'est pas précisé par l'éditeur** : la fonction est mentionnée dans le texte de présentation mais rattachée à aucun plan dans le tableau comparatif, et des sources secondaires l'attribuent à Plus/Pro. **À tester avec les 20 000 crédits Free (≈ 10–11 min de capture) avant d'en dépendre ou d'acheter.** Dans tous les cas, les plafonds de 5 min et 500 m² par scan s'appliquent.
- **Un mesh de scan brut directement exploitable : non.** Il faut systématiquement une passe de nettoyage et de décimation (MeshLab, CloudCompare, Blender) avant tout usage en moteur, en simulation ou en CAO (§5.5).
- **Rattrapage d'un scan raté sur ordinateur : impossible** — les données brutes ne sortent jamais de l'app.

---

## 11. Sources

**Scaniverse / Niantic Spatial**
https://dev.scaniverse.com/support · https://www.nianticspatial.com/pricing · https://www.nianticspatial.com/docs/scaniverse/techniques/ · https://www.nianticspatial.com/docs/scaniverse/troubleshoot/ · https://www.nianticspatial.com/docs/scaniverse/quickstart/ · https://www.nianticspatial.com/docs/scaniverse/360camera/ · https://www.nianticspatial.com/blog/usdz-scaniverse · https://apps.apple.com/us/app/scaniverse-3d-scanner/id1541433223 · https://community.nianticspatial.com/t/processing-fails-on-a-couple-of-my-splats/5734 · https://community.nianticspatial.com/t/generated-assets-from-13-scans-but-only-3-scans-appear-in-the-final-asset/5761 · https://community.nianticspatial.com/t/scan-render-failure-generic-failed-status-for-8k-360-hevc-video-gopro-max-2/5758 · https://community.nianticspatial.com/t/are-there-any-plans-for-this-to-be-available-in-a-license-for-students/5747 · https://community.nianticspatial.com/t/test-scans-stuck-in-vps-processing-for-almost-a-week/5375

**Polycam**
https://poly.cam/pricing · https://learn.poly.cam/hc/en-us/articles/43933482446996-How-to-Use-Space-Mode-Non-LiDAR-Devices · https://learn.poly.cam/hc/en-us/articles/27756102599572-What-File-Types-Can-Polycam-Export · https://learn.poly.cam/hc/en-us/articles/34295907278996-How-to-Access-Developer-Mode · https://learn.poly.cam/hc/en-us/articles/27426630160148-App-Crashing-While-Processing-Captures · https://learn.poly.cam/hc/en-us/articles/48538689020692-Preparing-to-Scan · https://poly.cam/press-release/space-mode-access-expanded-2026 · https://raw.githubusercontent.com/PolyCam/polyform/main/README.md

**KIRI Engine**
https://www.kiriengine.app/pricing · https://www.kiriengine.app/features/export-formats · https://www.kiriengine.app/features/photo-scan · https://www.kiriengine.app/faq/can-i-export-a-point-cloud-of-my-3d-model · https://www.kiriengine.app/blog/kiri-engine-basic-vs-pro · https://docs.kiriengine.app/asset-retention/ · https://github.com/Kiri-Innovation/3dgs-render-blender-addon

**Autres apps iPad de scan**
https://modelar.ai/ · https://apps.apple.com/us/app/modelar-3d-lidar-scanner/id1572844190 · https://apps.apple.com/us/app/realityscan-mobile/id1584832280 · https://www.realityscan.com/mobile · https://apps.apple.com/us/app/spectacular-rec/id6473188128 · https://apps.apple.com/us/app/gaussian-splatking/id6759175085 · https://radiancefields.com/splatking/guide/technique · https://get.teleport.varjo.com/pricing · https://www.voxelio.app · https://record3d.app/ · https://apps.apple.com/us/app/record3d-3d-videos/id1477716895 · https://record3d.app/presskit

**Apple Object Capture (Mac, sans ligne de commande)**
https://developer.apple.com/videos/play/wwdc2024/10107/ · https://developer.apple.com/videos/play/wwdc2023/10191/ · https://developer.apple.com/documentation/realitykit/capturing-photographs-for-realitykit-object-capture · https://developer.apple.com/documentation/realitykit/realitykit-object-capture · https://developer.apple.com/forums/thread/732290 · https://apps.apple.com/us/app/reality-composer/id1462358802 · https://www.photocatch.app/ · https://www.photocatch.app/press · https://9to5mac.com/2021/06/23/photocatch-lets-you-easily-create-3d-models-using-apples-new-object-capture-api/ *(source de 2021)*

**SuperSplat / PlayCanvas / splat-transform**
https://superspl.at/editor · https://developer.playcanvas.com/user-manual/supersplat/editor/editing-splats/ · https://developer.playcanvas.com/user-manual/supersplat/editor/transform-measure-align/ · https://developer.playcanvas.com/user-manual/supersplat/editor/publishing/ · https://github.com/playcanvas/splat-transform · https://raw.githubusercontent.com/playcanvas/splat-transform/main/README.md · https://blog.playcanvas.com/new-in-supersplat-editor-3-0-rebuilt-on-webgpu/

**Entraîneurs desktop**
https://3dsplatapp.com/ · https://apps.apple.com/us/app/3d-splat-app/id6760239941 · https://apps.apple.com/us/app/radiancekit/id6760346035?mt=12 · https://www.radiancekit.de/ · https://github.com/bkindler/radiancekit · https://splatscene.app/ · https://github.com/freddewitt/CorbeauSplat · https://github.com/ArthurBrussee/brush · https://raw.githubusercontent.com/ArthurBrussee/brush/main/README.md · **https://github.com/ArthurBrussee/brush/releases** · https://github.com/ArthurBrussee/brush/releases/tag/v0.3.0 · https://arthurbrussee.github.io/brush-demo · https://github.com/nerfstudio-project/gsplat · https://raw.githubusercontent.com/nerfstudio-project/gsplat/main/examples/simple_trainer.py · https://github.com/WebODM/OpenSplat · https://raw.githubusercontent.com/WebODM/OpenSplat/main/README.md · https://sites.fastspring.com/masseranolabs/product/opensplatforwindows · https://github.com/MrNeRF/LichtFeld-Studio · https://lichtfeld.io/ · https://portal.lichtfeld.io/signup/ · https://www.jawset.com/shop/pricing/ · https://www.jawset.com/docs/d/Postshot+User+Guide/Capturing+Guidelines · https://splatware.com/pricing

**COLMAP / photogrammétrie / capture avec poses**
https://colmap.github.io/cli.html · https://colmap.github.io/install.html · https://github.com/cvg/Hierarchical-Localization · https://spectacularai.github.io/docs/sdk/tools/nerf.html · https://pypi.org/project/spectacularai/ · https://www.realityscan.com/license · https://dev.epicgames.com/documentation/realityscan/realityscan-2-2 · https://opendronemap.org/download/ · https://docs.opendronemap.org/outputs/ · **https://github.com/alicevision/Meshroom** · https://alicevision.org/view/meshroom.html · https://github.com/alicevision/Meshroom/releases · https://github.com/alicevision/meshroom/issues/1226 · https://github.com/alicevision/Meshroom/wiki/Error:-This-program-needs-a-CUDA-Enabled-GPU · https://meshroom-manual.readthedocs.io/en/latest/first-steps/install/requirements.html

**Splat → mesh, nettoyage et décimation**
https://github.com/Lewis-Stuart-11/3DGS-to-PC · https://github.com/hbb1/2d-gaussian-splatting · https://raw.githubusercontent.com/hbb1/2d-gaussian-splatting/main/LICENSE.md · https://github.com/Anttwo/SuGaR · **https://www.meshlab.net/** · https://www.cloudcompare.org/doc/wiki/index.php/Poisson_Surface_Reconstruction_(plugin) · https://docs.blender.org/manual/en/latest/modeling/modifiers/generate/decimate.html

**Formats, viewers, moteurs**
https://github.com/nianticlabs/spz · https://www.nianticspatial.com/blog/spz4 · https://github.com/sparkjsdev/spark · https://sparkjs.dev/docs/new-features-2.0/ · https://developer.blender.org/docs/release_notes/5.3/rendering/ · https://github.com/aras-p/UnityGaussianSplatting · https://github.com/xverse-engine/XScene-UEPlugin · https://github.com/scier/MetalSplatter

**Capture, matériel, transfert, GPU cloud**
https://www.blackmagicdesign.com/products/blackmagiccamera · **https://www.blackmagicdesign.com/products/blackmagiccamera/techspecs** · https://forum.blackmagicdesign.com/viewtopic.php?f=2&t=196533 · https://help.sketchup.com/en/gaussian-splats-best-practices · https://arxiv.org/html/2507.06109v1 · https://sharp-frames.reflct.app/ · https://www.apple.com/ipad-pro/specs/ · https://www.apple.com/ipad-air/specs/ · https://caseadri.com/roomkit/guides/which-iphones-have-lidar/ · https://help.roomsketcher.com/hc/en-us/articles/29949063142045-Does-My-Phone-or-Tablet-Have-LiDAR · https://www.kaggle.com/docs/notebooks · https://www.kaggle.com/docs/efficient-gpu-usage · https://www.kaggle.com/discussions/product-announcements/735239 · https://research.google.com/colaboratory/faq.html · **https://lightning.ai/pricing/** · https://lightning.ai/docs/platform/overview/faq/billing · https://www.runpod.io/pricing · **https://vast.ai/pricing** · https://www.scanmanifold.com/blog-posts/lidar-on-iphone-how-accurate-is-it-plus-the-biggest-errors-that-manifold-corrects *(source de 2022)*

**ROS 1 Noetic (ce dépôt)**
https://index.ros.org/p/map_server/ · https://github.com/ros-planning/navigation/blob/noetic-devel/map_server/src/map_saver.cpp · https://index.ros.org/p/octomap_server/ · https://github.com/OctoMap/octomap_mapping/blob/kinetic-devel/octomap_server/src/OctomapServer.cpp · https://classic.gazebosim.org/tutorials?tut=model_structure · https://www.cloudcompare.org/doc/wiki/index.php/Cross_Section · https://www.cloudcompare.org/doc/wiki/index.php/Rasterize

**ROS 2 / Gazebo moderne (uniquement en cas de migration)**
https://github.com/ros2/common_interfaces/blob/rolling/visualization_msgs/msg/Marker.msg · https://github.com/ros2/rviz/blob/rolling/rviz_rendering/src/rviz_rendering/mesh_loader.cpp · https://github.com/gazebosim/sdformat/blob/main/sdf/1.11/mesh_shape.sdf · https://gazebosim.org/docs/harmonic/sdf_worlds/ · https://github.com/LihanChen2004/pcd2pgm · https://github.com/ros-navigation/navigation2/blob/main/nav2_map_server/README.md

**Divers**
https://docs.nerf.studio/quickstart/custom_dataset.html
---

## Annexe — comment ce rapport a été produit, et ce qu'il ne garantit pas

**Méthode.** Le document est le produit d'un workflow multi-agents exécuté les 19 et 20 septembre 2026, en six phases :

1. **Recherche** — 11 agents en parallèle, un par angle : apps iPad de splat, apps iPad de scan, capture avec poses ARKit, traitement cloud gratuit, traitement local, viewers et conversion, plateforme Apple, bonnes pratiques de capture, tarifs, retours d'utilisateurs, réutilisation ROS.
2. **Consolidation** — 110 solutions canoniques dégagées des constats bruts, classées par priorité.
3. **Vérification sceptique** — un agent par solution, chargé non pas de confirmer mais de **réfuter** chaque affirmation sur les sources officielles (site éditeur, page de tarifs, App Store, documentation, GitHub, changelog). **59 fiches produites.** De nombreuses affirmations ont effectivement été réfutées : le splat gratuit de KIRI Engine (en réalité réservé au plan Pro), la disponibilité des splats sur le plan gratuit de Polycam, l'obligation d'un GPU NVIDIA pour RealityScan (AMD accepté depuis juin 2026), plusieurs dates de version, les prix de l'iPad Pro (hausse Apple de juin 2026).
4. **Limites réelles** — une passe supplémentaire sur les six solutions les plus importantes (Scaniverse, Polycam, KIRI Engine, RealityScan Mobile, 3D Scanner App, Gaussian SplatKing) pour recenser les retours d'utilisateurs et les limites de terrain.
5. **Conception et jury** — quatre pipelines candidats rédigés indépendamment (tout-iPad gratuit, qualité maximale gratuite, usage régulier, réutilisation ROS), puis notés par trois juges aux points de vue distincts (débutant pressé, expert en splatting, pragmatique économe) sur la simplicité, le coût, la qualité et la fiabilité.
6. **Rédaction et critique** — rédaction à partir du pipeline gagnant enrichi des meilleures idées des autres, puis deux tours de critique de complétude et de révision.

**Chiffres de la dernière exécution :** 70 agents, aucun en erreur, environ 5,2 millions de jetons, 368 appels d'outils. Les exécutions précédentes, interrompues par des limites de facturation, avaient déjà produit les phases 1 à 4.

**Ce qui est solide.** Les faits sur les solutions listées en section 3 proviennent de pages officielles réellement ouvertes, et chacun a survécu à une tentative de réfutation. Les affirmations sur ce dépôt (arguments déjà déclarés dans les fichiers de lancement, règles `install()` toutes commentées dans le CMakeLists, sortie du convertisseur écrite dans un fichier `.txt`) ont été relues dans le code.

**Ce qui l'est moins.**

- **51 solutions de priorité basse n'ont pas été vérifiées individuellement** : elles n'apparaissent pas dans le rapport, ou seulement en mention.
- La dernière critique de complétude rendait encore un verdict « à réviser » avec une note de 8 sur 10 ; la révision correspondante a été appliquée, mais **aucune critique supplémentaire n'a validé cette dernière passe**.
- Reddit était inaccessible aux agents : les retours d'utilisateurs ont été lus via les flux RSS officiels ou une archive, et sont présentés comme des témoignages, non comme des faits.
- Plusieurs pages de tarifs sont rendues en JavaScript et n'ont pas pu être lues intégralement ; les points concernés sont signalés dans le texte.
- Tout ce qui touche aux prix vieillit vite. Scaniverse est passé au freemium, Polycam a déplacé des fonctions vers des paliers supérieurs et Postshot a retiré l'export de son plan gratuit, tout cela en 2026. **Revérifiez avant de payer.**
