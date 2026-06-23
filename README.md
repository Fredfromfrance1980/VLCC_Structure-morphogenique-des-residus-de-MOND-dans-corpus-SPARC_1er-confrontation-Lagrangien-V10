# Section spéciale VLCC : Analyse observationnelle des trajectoires morphogéniques dans le corpus SPARC :
Premier test observationnel du Lagrangien VLCC V10 (Variable Lagrangian of Cosmic Chronotropy) sur 175 galaxies SPARC. Pipeline de reproduction SPARC avec le candidat opératoire H2 : couplage ν_bar → (τ, σ), 22 paramètres globaux améliore MOND de 27,6 % (ΔBIC = −189) via le couplage de la fréquence orbitale ν_bar au champ tri-phasé T = (τ, σ, Δt). Trois résultats empiriques indépendants. 
Pipeline Python complet + CSV.

Zenodo. DOI : https://doi.org/10.5281/zenodo.20816833
Auteur : Frédérick Vronsky  
ORCID : https://orcid.org/0009-0003-5719-9604  
Cadre théorique : Variation Lagrangienne de la Chronotropie Cosmique (VLCC V10)  
Licence code : Creative Commons BY-NC-SA 4.0 International | Licence données SPARC : © Lelli et al. 2016
---
Contenu du dépôt
```
vlcc-sparc-H2/
├── vlcc_H2_model.py                  # Script principal — modèle H2 complet + optimisation
├── sparc_vlcc_final.csv              # Corpus SPARC enrichi (variables VLCC, MOND, M/L)
├── sparc_vlcc_H2_predictions.csv     # Prédictions pré-calculées H2 (6 seeds, 89 colonnes)
├── sparc_vlcc_clustering.csv         # Résultats GMM non supervisé (section 5, ARI/NMI)
└── README.md
```
> **Note :** `vlcc_H2_results.json` (résultats d'optimisation bruts par seed) n'est pas inclus dans ce dépôt.
> Tous les résultats sont entièrement reproductibles via `python vlcc_H2_model.py`.
---
Démarrage rapide
```bash
pip install numpy pandas scipy scikit-learn

# Vérification rapide (~3 min, seed=42 uniquement)
python vlcc_H2_model.py --fast

# Protocole complet (6 seeds, ~8h)
python vlcc_H2_model.py
```
Dépendances : `numpy`, `pandas`, `scipy`, `scikit-learn` — bibliothèques standard, aucun appel externe.
Résultats attendus (seed=42, protocole complet 40 redémarrages × 8000 itérations) :
Métrique	H2 candidat	MOND	Gain
RMSE test (km/s)	26.47	37.03	+26.6 %
R²	0.918	~0.84	
ΔBIC	−168	0	
G1.2 gain (corpus complet)	45.5 %	—	
G1.3 gain (corpus complet)	62.8 %	—	
Explosions numériques	0/6	—	
Performance globale (moyenne 6 seeds) : RMSE = 30.36 ±5.20 km/s vs MOND 41.72 ±5.13 km/s,
gain 27.6 % [IC 95 % : 25.1–30.1 %], ΔBIC = −189 ±62.
---
Description des fichiers
`vlcc_H2_model.py` — Script principal
Implémente le candidat opératoire H2 dans son intégralité :
Calcul des variables VLCC (χ₁, χ₃, ν_bar, T_env, M_i, w_i)
Optimisation Nelder-Mead adaptatif (Gao & Han 2012), 40 redémarrages, 8000 itérations
Split galaxy-safe GroupShuffleSplit (15 % test, seeds [42, 0, 1, 2, 3, 4])
Comparaison stricte MOND / H2 (même split, même jeu test, même traitement M/L)
Export des résultats et prédictions
---
`sparc_vlcc_final.csv` — Corpus SPARC enrichi (3021 lignes)
Corpus de travail après filtrage qualité (R > 0, Vobs > 0, e_Vobs > 0, Σ_L > 0, Ω_bar > 0,
g_bar > 0, χ₁ > 0, χ₃ > 0, V²_bar > 0, ν_bar > 0). Contient toutes les variables
observationnelles et VLCC nécessaires à la reproduction.
Groupe de colonnes	Description
`galaxy, distance_mpc, inc_deg, hubble_type, ...`	Métadonnées galactiques SPARC
`R, Vobs, e_Vobs, Vgas, Vdisk, Vbul, Sigma_disk`	Courbes de rotation observées
`V_bar, V2_bar, g_bar, nu_bar, Sigma_L, Omega_bar, f_gas`	Variables baryoniques
`chi1, chi3`	Coordonnées morphogéniques VLCC (plan χ₁, χ₃)
`V_MOND, V_MOND_ml`	Prédictions MOND (fonction d'interpolation simple, a₀ = 3.086 × 10⁻¹⁴ (km/s)²/kpc)
`m2l_disk, has_bulge`	Rapport masse/luminosité et présence de bulbe
`V2_disk_ml, V2_bul_ml, V2_bar_ml`	Contributions M/L ajustées
Source SPARC : Lelli, F., McGaugh, S.S., Schombert, J.M. (2016). AJ, 152, 157.
DOI: 10.3847/0004-6256/152/6/157
---
`sparc_vlcc_H2_predictions.csv` — Prédictions H2 pré-calculées (3021 lignes × 89 colonnes)
Résultats complets du candidat H2 sur les 6 seeds. Permet la reproduction de toutes les
figures et tableaux de l'article sans relancer l'optimisation (~8h).
Groupe de colonnes	Description
`Galaxy, R, Vobs, e_Vobs, ...`	Variables observationnelles SPARC
`g_bar, Omega_bar, nu_bar, chi1, chi3, log_chi1, log_chi3`	Variables VLCC calculées
`sector`	Partition morphogénique : G1 / G1.2 / G1.3 (percentiles 33/67 de log χ₁)
`split_s42`	Appartenance train/val/test pour seed=42
`V_MOND`	Prédiction MOND de référence
`T_env_mean, M1_mean, V_H2_mean, V_H2_std, resid_mean`	Consensus 6 seeds
`T_env_H2_sN`	Environnement inertiel T_env — seed N (N ∈ {42,0,1,2,3,4})
`V_H2_sN`	Vitesse prédite V_pred — seed N
`resid_H2_sN`	Résidu V_H2 − V_obs — seed N
`w1_sN, w2_sN, w3_sN`	Poids softmax par attracteur — seed N
`M1_sN, M2_sN, M3_sN`	Mémoire morphogénique τ par attracteur (exclusif VLCC H2)
`split_sN`	Appartenance train/val/test — seed N
`status_sN`	Validité de la seed (VALID / INVALID)
Les colonnes `M1_sN` correspondent directement au gradient de dominance morphogénique
décrit dans la section 4.5 de l'article (Figure 5) :
G1 = 0.815, G1.2 = 0.326, G1.3 = 0.088, p < 10⁻²³.
---
`sparc_vlcc_clustering.csv` — Validation non supervisée (3021 lignes × 16 colonnes)
Résultats du clustering GMM non supervisé (n = 3 composantes) utilisés dans la
section 5 de l'article. Aucun paramètre du modèle H2 n'est utilisé dans ce fichier :
il sert à valider l'existence indépendante des secteurs G1/G1.2/G1.3 dans les données SPARC.
Colonne	Description
`Galaxy, R, Vobs`	Identifiants observationnels
`g_bar, Omega_bar`	Variables baryoniques
`chi1, chi3, log_chi1, log_chi3`	Coordonnées morphogéniques VLCC
`sector`	Partition VLCC de référence (G1 / G1.2 / G1.3)
`cluster_vlcc`	Cluster GMM dans l'espace (χ₁, χ₃) — ARI = 0.488 vs secteurs VLCC
`prob_0_vlcc, prob_1_vlcc, prob_2_vlcc`	Probabilités d'appartenance GMM (espace VLCC)
`cluster_gbar`	Cluster GMM dans l'espace g_bar seul — ARI = 0.383 (référence MOND)
`cluster_vbar`	Cluster GMM dans l'espace V_bar seul — ARI = 0.247
Résultats clés (Tableau 5 de l'article) :
Espace de variables	ARI	NMI
V_bar seul	0.247	0.261
nu_bar seul	0.228	0.264
g_bar seul (MOND)	0.383	0.470
g_bar + nu_bar	0.340	0.350
χ₁ + χ₃ (VLCC)	0.488	0.523
L'espace VLCC révèle mieux la structure latente des données SPARC que tout autre
espace testé, y compris g_bar seul (MOND) : +27 % d'ARI.
---
Description du modèle H2
Équation centrale
```
V²_pred(r) = V²_bar(r) · exp(T_env(r))

T_env(r) = Σᵢ wᵢ(χ₁,χ₃) · Mᵢ(ν_bar) · Fᵢ(g_bar, ν_bar)
```
Étape 1 — Mémoire morphogénique Mᵢ(ν_bar) → τ (t₁)
```
Mᵢ(ν_bar) = 1 / [1 + exp(−αᵢ · log(ν_bar/ν_ref) − βᵢ)]
```
Encode l'extinction progressive de la mémoire τ : Mᵢ → 1 en zone interne (t₁ dominant),
Mᵢ → 0 en zone externe (t₁′ dominant). Directement mesurable dans le CSV (colonnes M1, M2, M3).
Étape 2 — Réponse de cohérence Fᵢ(g_bar, ν_bar) → σ (t₂)
```
Fᵢ(g,ν) = Aᵢ · log(1 + g₀/g_bar)^ngᵢ · log(1 + ν₀/ν_bar)^nνᵢ
```
Couple g_bar et ν_bar pour encoder la cohérence σ.
Variateur softmax wᵢ(χ₁, χ₃)
```
χ₁ = Ω_bar / √(g_bar · Σ_L)
χ₃ = R · f_gaz / (Σ_L · Ω_bar)

wᵢ = softmax dans le plan (log χ₁, log χ₃) — variateur V10 canonique
```
Paramètres k = 22 globaux
Groupe	Nombre	Description
ΘM	1	Largeur du variateur softmax
Centres	6	Coordonnées (χ₁, χ₃) des 3 attracteurs morphogéniques
Mémoire	6	(αᵢ, βᵢ) par bassin G1 / G1.2 / G1.3
Réponse	9	(Aᵢ, ngᵢ, nνᵢ) par bassin
---
Protocole d'optimisation
```
Algorithme    : Nelder-Mead adaptatif (Gao & Han 2012, doi:10.1007/s10589-010-9329-3)
Redémarrages  : 40 par seed
Itérations    : 8000 par redémarrage
Seeds         : [42, 0, 1, 2, 3, 4]
Split         : 15 % test, galaxy-safe (GroupShuffleSplit)
Comparaison   : MOND et H2 évalués sur exactement le même split / jeu test / M/L
```
Reproductibilité : tous les résultats sont entièrement déterministes
étant donné le corpus et les seeds. Aucun appel externe, aucun élément stochastique
hors du générateur contrôlé `numpy.random.default_rng(seed)`.
---
Références du cadre théorique VLCC
Traité VLCC — Édition canonique V1 — Vronsky, F. (2025).  
DOI : https://doi.org/10.5281/zenodo.17946156
Section spéciale VLCC — Lagrangien V10, T_env, Attracteurs morphogéniques — Vronsky, F. (2026).  
DOI : https://doi.org/10.5281/zenodo.20277793
Complétion du traité VLCC — Intronisation de l'état photonicotemporel — Vronsky, F. (2026).  
DOI : https://doi.org/10.5281/zenodo.18703033
---
Références externes
Lelli, F., McGaugh, S.S., Schombert, J.M. (2016). SPARC: Mass Models for 175 Disk Galaxies. AJ, 152, 157. DOI: 10.3847/0004-6256/152/6/157
Milgrom, M. (1983). A modification of the Newtonian dynamics. ApJ, 270, 365–370. DOI: 10.1086/161130
McGaugh, S.S., Lelli, F., Schombert, J.M. (2016). The Radial Acceleration Relation. PRL, 117, 201101. DOI: 10.1103/PhysRevLett.117.201101
Gao, F., Han, L. (2012). Nelder-Mead simplex with adaptive parameters. Comput. Optim. Appl., 51, 259–277. DOI: 10.1007/s10589-010-9329-3
Kass, R.E., Raftery, A.E. (1995). Bayes Factors. JASA, 90, 773–795.
---
Frédérick Vronsky — Chercheur indépendant en cosmologie théorique — Toulouse — Juin 2026
