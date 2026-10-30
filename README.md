# 🍫 CacaoTrack — Performance Supply Chain du Cacao en Côte d'Ivoire

> **Power BI · Modélisation en étoile · DAX avancé · Storytelling décisionnel pour un Comité de Direction**

De la parcelle au port : un rapport exécutif de 10 pages qui identifie **+1,73 Md XOF de marge nette par campagne** (3,5 Md XOF cumulés sur 3 campagnes) à capter d'ici 2028/29.

![Power BI](https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-136%20mesures-7A5F4C)
![Modèle](https://img.shields.io/badge/Mod%C3%A8le-Sch%C3%A9ma%20en%20%C3%A9toile-A98B76)
![Données](https://img.shields.io/badge/Donn%C3%A9es-149%20098%20mouvements-BABF94)

---

## 📑 Sommaire

1. [Contexte métier](#-1-contexte-métier)
2. [Architecture des données](#-2-architecture-des-données)
3. [Préparation des données (Power Query)](#-3-préparation-des-données-power-query)
4. [Indicateurs de performance (KPIs)](#-4-indicateurs-de-performance-kpis)
5. [Aperçu visuel](#-5-aperçu-visuel)
6. [Compétences démontrées](#-6-compétences-démontrées)
7. [Guide de reproduction rapide](#-7-guide-de-reproduction-rapide)
8. [Structure du dépôt](#-8-structure-du-dépôt)
9. [Auteur](#-9-auteur)

---

## 🎯 1. Contexte métier

Un exportateur de cacao ivoirien pilote une chaîne logistique en 7 maillons : **Ferme → Coopérative → Stockage → Contrôle qualité → Transfert → Réception port → Export**. Le périmètre analysé couvre **149 098 mouvements**, **120 coopératives**, **60 transporteurs** et **2 ports** (Abidjan, San-Pédro), sur les campagnes **2022/23 à 2025/26** (campagne = octobre → septembre).

### Problématiques business adressées

| # | Question de la Direction | Réponse apportée par le rapport |
|---|---|---|
| 1 | La collecte croît-elle vraiment ? | Non : elle plafonne à ~15 000 t. La hausse de +37 % affichée vient de **campagnes partielles** (2022/23 = jan–sep, 2025/26 = oct–déc). |
| 2 | Où se perd la marge en logistique ? | Le coût est de **~48 XOF/kg (4,6 % du prix)**. Le levier n'est ni le mode ni la flotte, mais la **dispersion des tarifs** (0 à 87 XOF/kg pour un même service). |
| 3 | Comment créer de la valeur à prix figé ? | Par le **mix variété × grade** : Criollo Premium vs Forastero Standard = +247 XOF/kg (+26 %). |
| 4 | Où et pourquoi perd-on du volume ? | Pertes diffuses (~1,77 %) mais **concentrées sur 1 mouvement sur 3** ; l'humidité > 8 % coûte ~12 XOF/kg. |
| 5 | Les certifications paient-elles ? | Levier de **marge** (+50 XOF/kg), pas de fidélisation. Priorité à Gôh-Djiboua (29 % certifié). |
| 6 | Quel plan, quel ROI, quels risques ? | 3 priorités simulables (curseurs what-if), scénario prudent à −30 % de volume, matrice de 7 risques. |

### Trois leviers de création de valeur

1. **Valoriser chaque kg** : certification, mix Premium, humidité ≤ 8 % (+0,28 Md XOF/an)
2. **Protéger le flux** : plafond de démarque à 2,5 %, appel d'offres transport (+0,10 Md XOF/an)
3. **Sécuriser l'origine** : +3 % de volume/campagne via contrats coopératives (+1,35 Md XOF/an en 2028/29)

---

## 🏗️ 2. Architecture des données

### Schéma en étoile

Un seul fait central, relié à **10 dimensions** par des relations **1-à-plusieurs, filtrage unidirectionnel**. Chaque ligne du fait trouve une correspondance dans les 10 dimensions grâce à un membre **« Inconnu » (clé -1)**.

```
                    dim_Date ──────┐
                   dim_Produit ────┤
                      dim_Lot ─────┤
               dim_Cooperative ────┤
                    dim_Ferme ─────┼──►  fact_Mouvements
                     dim_Site ─────┤     (149 098 lignes)
             dim_Transporteur ─────┤
                    dim_Port ──────┤
               dim_Evenement ──────┤
                   dim_Devise ─────┘
```

### Tables

| Type | Table | Rôle |
|---|---|---|
| **Fait** | `fact_Mouvements` | 1 ligne = 1 mouvement logistique (poids, humidité, pertes, coûts unitaires XOF/kg, valeur brute, primes de certification) |
| Dimension | `dim_Date` | Calendrier oct. 2022 → sept. 2026, hiérarchie **Campagne → Trimestre campagne → Mois** (mois triés octobre → septembre) |
| Dimension | `dim_Evenement` | 7 maillons de la chaîne, triés par `Ordre étape` |
| Dimension | `dim_Produit` | Hiérarchie **Variété → Grade → Produit** (Forastero, Trinitario, Criollo × Standard, Premium) |
| Dimension | `dim_Cooperative` | Hiérarchie **Région → Département → Ville → Coopérative** |
| Dimension | `dim_Lot` · `dim_Ferme` | Lots (certification : RA, Fairtrade, Bio, UTZ, Aucune) et fermes |
| Dimension | `dim_Transporteur` | Mode de transport, type de flotte (propre / sous-traitance / non référencé) |
| Dimension | `dim_Site` · `dim_Port` · `dim_Devise` | Sites géolocalisés, ports, devises |
| Support | `Cascade`, `Plan Campagnes`, `Plan Priorités`, `Hypothèses`, `Risques` | Tables de calcul pour la cascade de valeur, le plan d'action et le registre des risques |
| What-if | `p Cible Certif`, `p Shift Premium`, `p Résorption`, `p Croissance Volume`, `p Réduction Tarif` | 5 paramètres de simulation (curseurs) |
| Mesures | `_Mesures` | Table dédiée : **136 mesures** réparties en 12 dossiers d'affichage |

### Bonnes pratiques de modélisation appliquées

- **Staging séparé** : requêtes `src_*` non chargées, groupées en `01_Staging` → `02_Dimensions` → `03_Fact` → `99_Parameters`
- **Paramètres** (`SourcePath`, `pThreshold_Humidite = 8`, `pThreshold_Perte = 0.025`) : aucun chemin ni seuil codé en dur
- **Colonnes techniques masquées** : seuls les drapeaux et classes utiles restent visibles ; tout le reste est rangé par dossiers (Quantités, Valeurs, Coûts, Qualité)
- **Colonnes de tri** dédiées pour les classes d'humidité et de tarif transport
- **Auto date/time désactivé** : table de dates explicite uniquement
- **Culture `fr-FR`** et noms métier en français pour les utilisateurs finaux

---

## 🧹 3. Préparation des données (Power Query)

Source : classeur Excel de 10 feuilles (~150 k mouvements). **14 anomalies** de qualité identifiées et traitées dans Power Query :

| Problème | Traitement |
|---|---|
| 1 800 identifiants de mouvement dupliqués | Dédoublonnage (151 800 → 149 098 lignes valides après exclusion des lignes invalides) |
| Dates mêlant vraies dates et texte | Reconstruction d'une date propre, **0 ligne sans date** |
| Devise `XO F` | Normalisation en `XOF` |
| Poids et nombre de sacs négatifs, lignes sans étape | Exclusion des lignes invalides |
| Clés manquantes ou hors plage | Redirection vers le membre `Inconnu (-1)` de chaque dimension |
| Régions des coopératives en casse incohérente, `d’Abidjan` | Mise en forme propre, apostrophes harmonisées |
| Coûts stockés en XOF/kg | **Montants recalculés** = coût unitaire × poids |

Contrôles de recette automatisés (valeurs attendues vs obtenues) : lignes du fait, transporteurs/sites inconnus, tonnages collecté et exporté 2024/25, CA brut, prix moyen pondéré, humidité moyenne, intégrité des 10 relations.

---

## 📊 4. Indicateurs de performance (KPIs)

Les **136 mesures** sont rangées dans la table `_Mesures`, par dossiers. Le code commenté est dans [`dax/`](dax/).

### Collecte et volumes

| KPI | Valeur (2024/25) | Valeur pour la Direction |
|---|---|---|
| Volume collecté (t) | **15 023 t** (+1,0 % vs 2023/24) | Mesure de la capacité d'approvisionnement ; comparaison **à périmètre comparable** (YTD campagne) pour éviter l'effet campagnes partielles |
| Volume exporté (t) | **14 689 t** | Base de calcul de la valeur et du plan |
| Δ Collecte % par région / coopérative | Montagnes −8,8 %, Lagunes +7,8 % | Identifie moteurs de croissance et alertes |

### Logistique

| KPI | Valeur | Valeur pour la Direction |
|---|---|---|
| Coût logistique / kg | **~48 XOF/kg** (4,6 % du prix) | Pilotage du coût unitaire transport + stockage |
| Part volume vs part coût par tranche de tarif | 34 % du volume = 46 % du coût au-delà de 40 XOF/kg | Cible de l'appel d'offres transport |
| Part sous-traitance | 32 % du volume | Mesure la dépendance, sans surcoût constaté |

### Export et prix

| KPI | Valeur | Valeur pour la Direction |
|---|---|---|
| CA brut / net export | **15,41 Md XOF** brut | Chiffre d'affaires et conversion brut → net |
| Prix moyen pondéré (PMP) | **~1 046 XOF/kg** | Prix réellement obtenu, pondéré par les volumes |
| Écart PMP vs Forastero Standard | Criollo Premium **+247 XOF/kg** | Quantifie le levier mix variété × grade |

### Démarque et qualité

| KPI | Valeur | Valeur pour la Direction |
|---|---|---|
| Taux de perte | **1,77 %** | Indicateur de démarque global et par maillon |
| Part des mouvements > 2,5 % de perte | 31 % des mouvements = 67 % des pertes | Concentration : cible d'un plafond de démarque |
| Taux de perte plafonné à 2,5 % | 1,37 % | Gain potentiel (what-if) |
| Humidité moyenne / Décote > 8 % | 7,49 % / ~12 XOF/kg | Coût de la non-qualité |

### Certifications

| KPI | Valeur | Valeur pour la Direction |
|---|---|---|
| Part du volume exporté certifié | **54 %** (RA 22, Fairtrade 16, Bio 8, UTZ 8) | Positionnement de l'offre |
| Prime moyenne / kg certifié | +50 XOF/kg (+4,8 %) | Valeur nette de la certification |
| Part certifiée par région | Gôh-Djiboua **29 %** | Priorisation des investissements |

### Plan d'action et ROI (simulable)

| KPI | Valeur | Valeur pour la Direction |
|---|---|---|
| Gain par priorité (P1, P2, P3) | 283 M / 95 M / 1 353 M XOF | Arbitrage budgétaire |
| Gain cible 2028/29 | **+1,73 Md XOF** (+11,5 % de marge nette) | Ambition chiffrée |
| Gain cumulé 3 campagnes | **3,5 Md XOF** | ROI global |
| Gain prudent (−30 % de volume) | 1,33 Md XOF | Robustesse du plan |

> Les valeurs ci-dessus proviennent du modèle et sont contrôlées contre la spécification de recette (tolérance ±2–5 % liée au nettoyage).

---

## 🖼️ 5. Aperçu visuel

Rapport de **10 pages + 1 infobulle**, thème sur mesure « *CacaoTrack Executive* » (palette cacao/moka, titres Georgia, corps Segoe UI), titres d'action pilotés par mesures DAX, navigation par boutons, filtre Campagne synchronisé.

> 📌 Ajoutez vos captures dans `docs/images/` puis décommentez les lignes ci-dessous.

| Page | Aperçu |
|---|---|
| **Couverture & sommaire** (9 boutons de navigation) | `![Couverture](docs/images/00_couverture.png)` |
| **01 · Synthèse exécutive** | `![Synthèse](docs/images/01_synthese.png)` |
| **02 · Diagnostic de la collecte** (Top 5 / Flop 5) | `![Collecte](docs/images/02_collecte.png)` |
| **03 · Performance routière & sous-traitance** | `![Logistique](docs/images/03_logistique.png)` |
| **04 · Flux d'exportation & valorisation prix** | `![Export](docs/images/04_export.png)` |
| **05 · Démarque & qualité** | `![Démarque](docs/images/05_demarque.png)` |
| **06 · Impact économique des certifications** (cascade) | `![Certifications](docs/images/06_certifications.png)` |
| **07 · Plan d'action & ROI** (5 curseurs what-if) | `![Plan](docs/images/07_plan.png)` |
| **08 · Hypothèses & risques** (matrice probabilité × impact) | `![Risques](docs/images/08_risques.png)` |
| **09 · Décisions attendues** | `![Décisions](docs/images/09_decisions.png)` |

<!-- Exemple de GIF de navigation :
![Démo navigation](docs/images/demo_navigation.gif)
-->

---

## 🧠 6. Compétences démontrées

| Domaine | Démonstration dans ce projet |
|---|---|
| **Modélisation** | Schéma en étoile, 10 dimensions, membres « Inconnu », hiérarchies, colonnes de tri, relations 1-N unidirectionnelles |
| **DAX avancé** | 136 mesures organisées en dossiers ; variation à périmètre comparable (YTD campagne), pondérations, cascade de valeur, Top/Flop N, what-if, mesures de formatage conditionnel et de titres dynamiques |
| **Ingénierie de données** | Power Query paramétré, staging, dédoublonnage, typage, normalisation, recette automatisée contre une spécification |
| **Data storytelling** | Titres d'action, 1 insight par page, hiérarchie visuelle, couleurs sémantiques (alerte, positif, partiel), accessibilité (texte alternatif, contrastes AA) |
| **Automatisation** | Rapport généré de façon reproductible au format PBIR via script Node.js, validé par le CLI de création de rapports |

---

## 🚀 7. Guide de reproduction rapide

> ℹ️ Le fichier Power BI (`.pbix` / `.pbip`) et les données sources **ne sont pas publiés** (volume et confidentialité). Ce dépôt contient la **logique** : modèle documenté, DAX et scripts de préparation.

### Pour comprendre le modèle en 5 minutes

```bash
git clone https://github.com/mbozoomengo/Cacao_CI_Supply_Chain_Report.git
cd Cacao_CI_Supply_Chain_Report
```

1. Lire la section [Architecture des données](#-2-architecture-des-données) ci-dessus.
2. Parcourir [`dax/`](dax/) : les mesures sont classées par thème, avec leur logique en commentaire.
3. Parcourir [`scripts/`](scripts/) : transformations Power Query (M) de nettoyage et de construction des dimensions.

### Pour recréer le modèle dans Power BI Desktop

1. Préparer un classeur Excel avec les 10 feuilles (mouvements, coopératives, fermes, lots, sites, transporteurs, produits, événements, ports, devises).
2. Dans Power BI Desktop, créer les paramètres `SourcePath`, `pThreshold_Humidite` (8) et `pThreshold_Perte` (0,025).
3. Coller les requêtes M de `scripts/` dans l'éditeur avancé (staging, dimensions, fait).
4. Créer les 10 relations 1-N unidirectionnelles `dim_* → fact_Mouvements`.
5. Créer une table vide `_Mesures`, puis y coller les mesures de `dax/`.
6. Désactiver *Auto date/time* et *Détecter automatiquement les relations* (Fichier → Options → Fichier actuel).
7. Contrôler : 149 098 lignes dans le fait ; collecte 2024/25 ≈ 15 023 t ; PMP ≈ 1 046 XOF/kg.

---

## 📁 8. Structure du dépôt

```
Cacao_CI_Supply_Chain_Report/
├── README.md
├── .gitignore
├── dax/
│   └── mesures_cles.dax        # Mesures stratégiques commentées
├── scripts/
│   └── ...                     # Requêtes M (Power Query) et scripts de préparation
└── docs/
    └── images/                 # Captures d'écran et GIF du rapport
```

---

## 👤 9. Auteur

**William Mbozo'o** — Data Engineer & Consultant Business Intelligence
📍 Projet : *Cacao CI · Stratégie & Performance Supply Chain* — Octobre 2026
🔗 GitHub : [@mbozoomengo](https://github.com/mbozoomengo)

*Les données de ce projet sont des données d'étude ; les chiffres servent à démontrer la méthode d'analyse.*
