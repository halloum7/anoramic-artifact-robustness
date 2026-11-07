# Article 6 — Plan de travail

**Titre provisoire :** *Physics-informed artifact taxonomy and robustness benchmark of deep learning models in real-world dental panoramic radiography: a single-device dose-aware study*

**Revues cibles (vérifier le quartile SJR au moment de soumettre) :** Dentomaxillofacial Radiology · Physica Medica · Medical Physics · Journal of Dentistry · Computers in Biology and Medicine

---

## 1. Données (état au 04/11/2026)

| Élément | Valeur |
|---|---|
| Appareil | Owandy I-Max Plus, capteur direct, logiciel mj32 5.08 |
| Fichiers | 198 DICOM (157 traités à ce jour) → 148 patients, 5 doublons, 1 image sans paramètres |
| Période | 2018 – 2026 |
| Format | ≈ 1536 × 2650 px, pixel 0,078 mm, 8 bits « FOR PRESENTATION » (image déjà traitée par le logiciel) |
| Protocoles | 9 couples kV/mA : 64–74 kV, 4–7 mA, 13,9 ou 14,6 s ; 72 kV/4 mA dans ~70 % des cas |
| Dose (DAP) | 3,8 → 10,0 µGy·m² (lue sur le bandeau incrusté ; absente de l'en-tête DICOM) |
| Âge | disponible pour ~50 % (en-tête DICOM ou intervalle dans le nom de fichier) |
| Annotations | grille d'artefacts remplie et corrigée par le dentiste (151 images, version finale du 04/11/2026) ; « autre artefact » = dose d'exposition inadaptée au patient (55 images) ; relecture de 30 images prévue à J+14 |

Anonymisation : noms remplacés par IMG_xxxx / PAT_xxx, bandeau (date, paramètres) et logo masqués, table de correspondance séparée (CONFIDENTIEL, à ne jamais partager).

## 2. Question de recherche et hypothèses

- **Q :** quels artefacts physiques et erreurs de positionnement propres à la panoramique dégradent le plus les modèles d'IA, et une augmentation de données fondée sur la physique de l'acquisition corrige-t-elle cette fragilité ?
- **H1 :** la performance de segmentation chute de façon significative et spécifique selon le type d'artefact (fantôme métallique, rotation/inclinaison, plan d'occlusion, rachis).
- **H2 :** un simulateur physique de ces artefacts, utilisé en augmentation, réduit cette chute davantage qu'une augmentation générique.
- **H3 (angle dose) :** à protocole plus faible (kV, mAs, DAP), la qualité perçue et la performance de l'IA baissent ; on quantifie ce compromis dose/performance.

## 3. Contributions annoncées

1. Taxonomie annotée des artefacts sur une base clinique réelle (151 images, double lecture partielle, kappa de Cohen).
2. Banc d'essai de robustesse stratifié par artefact, pour plusieurs architectures (U-Net / ResNet50 de vos travaux antérieurs, un modèle Transformer, SAM2 adapté).
3. Simulateur d'artefacts fondé sur la géométrie panoramique (canal focal, grandissement horizontal/vertical, images fantômes, bruit lié au mAs), en code ouvert.
4. Analyse dose / qualité / performance IA à partir des paramètres d'exposition et du DAP.

## 4. Méthode

**4.1 Données d'entraînement (décision du 04/11/2026) :**
- **STS-2D-Tooth** (Wang et al., *Scientific Data* 2025) : 4 000 panoramiques dont 900 avec masques de dents, adultes + enfants, téléchargement libre (Zenodo / Hugging Face) → base principale d'entraînement.
- **Tufts Dental Database** (Panetta et al., *IEEE JBHI* 2022) : 1 000 panoramiques avec masques de dents, accès sur formulaire, usage recherche non commercial → 2e source d'entraînement (diversité d'appareils).
- Optionnel : **OdontoAI / O²PR** (2 000 panoramiques annotées, inscription sur la plateforme) ; **DENTEX** non retenu pour la segmentation (boîtes et diagnostics uniquement).
- **Vos images = test externe indépendant**, jamais vues à l'entraînement ; 40–50 masques de dents à produire pour le Dice.

**4.2 Annotation (dentiste + Claude) :**
- grille `grille_annotation_artefacts.xlsx` (12 artefacts codés 0/1/2, qualité 1–3, dentition) ;
- 30 images relues une seconde fois → kappa ;
- masques de dents sur 40 à 50 images pour mesurer le Dice (pré-segmentation automatique, correction par le dentiste).

**4.3 Simulateur physique :**
- rotation/inclinaison de la tête → déformation non linéaire selon la position dans le canal focal ;
- avancée/recul → grandissement horizontal différentiel des incisives ;
- image fantôme → réplique controlatérale agrandie, floue et décalée en hauteur d'un objet dense ;
- variation de dose → bruit quantique proportionnel à 1/√mAs (calibré sur vos 9 protocoles).

**4.4 Expériences :**
- E1 : performance de référence sur les bases publiques ;
- E2 : test sur vos images, stratifié par artefact (Dice, HD95, détection) ;
- E3 : entraînement avec augmentation générique vs. physique → gain sur E2 ;
- E4 : régression performance ~ artefacts + kV + mAs + DAP + âge (modèle mixte) ;
- statistiques : IC à 95 % par bootstrap, test de Wilcoxon apparié, correction de Holm.

**4.5 Explicabilité :** cartes Grad-CAM/attention sur les cas d'échec, pour montrer que les erreurs se concentrent sur les zones d'artefact (lien avec X-MedNet).

## 5. Répartition du travail

| Tâche | Qui |
|---|---|
| Anonymisation, inventaire, métadonnées | Claude ✅ |
| Annotation des artefacts (151 images) | Dentiste (≈ 3–4 h) |
| Double lecture 30 images | Dentiste à J+14 ou 2e lecteur |
| Simulateur, code d'entraînement, notebooks | Claude |
| Exécution de l'entraînement sur GPU | Kamal (Google Colab ou PC avec GPU) — l'environnement de Claude n'a pas de GPU |
| Masques de dents (40–50 images) | pré-segmentation Claude, correction dentiste |
| Analyse statistique, figures, rédaction | Claude, relecture Kamal |
| Avis du comité d'éthique | Kamal |

## 6. Prochaines étapes

1. Lancer `anonymiser_panoramiques.py` sur le PC pour traiter les 198 fichiers.
2. Donner la grille au dentiste.
3. Choisir la base publique d'entraînement (Claude vérifie licences et annotations disponibles).
4. Claude écrit le simulateur physique et le notebook Colab d'entraînement.
