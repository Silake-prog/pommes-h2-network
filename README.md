# README — Scénarisation sectorielle de la demande d’hydrogène (EU, 2019–2050)

Ce dépôt regroupe une chaîne de scénarisation **paramétrique**, structurée en *modules* sectoriels (acier, ammoniac, raffineries, eSAF, maritime-méthanol, oléfines-CONCAWE) produisant des sorties CSV homogènes (par pays et par année).  
L’objectif est de quantifier des **ordres de grandeur** de demande d’hydrogène (H₂) induite par des trajectoires de production, de substitution technologique et d’émergence de nouvelles filières industrielles, en s’appuyant sur des données de référence (2019) et des hypothèses explicites.

---

## 1) Sources de données primaires et construction des bases

### Sources utilisées
Deux sources principales sont mobilisées de manière complémentaire :

- **TYNDP (année de référence 2019)**  
  Utilisé pour établir une photographie cohérente des **niveaux de production industriels** et servir de point d’ancrage des trajectoires (volumes initiaux par pays/secteur lorsque disponibles).

- **IDEES 2023 (JRC)**  
  Utilisé pour caractériser les **consommations énergétiques** détaillées (secteur, vecteur, usages finaux) afin d’étayer la structure énergétique initiale et certaines intensités.

### Hypothèses générales de modélisation
- Les bases de données reproduisent une photographie “fidèle” de la production et/ou de la consommation **à l’année de référence** (souvent **2019**, parfois **2025** pour les modules adossés à CONCAWE).
- Les trajectoires sont **exogènes** : pas de boucle prix, pas d’équilibre général, pas d’optimisation intertemporelle.
- Les dynamiques temporelles sont volontairement **simples et traçables** : croissance composée, rampes linéaires, plateaux.
- Les paramètres structurants (intensités spécifiques, rendements, facteurs stœchiométriques) sont **déclarés** et **documentés** dans les YAML.

---

## 2) Philosophie “hydrogène” : quels mécanismes sont modélisés ?

L’hydrogène intervient selon des mécanismes qui peuvent coexister au sein d’un même secteur :

1. **Substitution d’un intrant / agent réactionnel carboné dans un procédé existant**  
   Exemple : décarbonation progressive d’une filière industrielle sans changer la nature du produit final (acier bas carbone via DRI-H₂, etc.).

2. **Structuration de nouveaux débouchés sectoriels et reconfiguration des chaînes de valeur**  
   Exemple : montée en puissance de filières “nouvelle architecture” (DRI–EAF–H₂ dans l’acier ; e-fuels ; e-molécules), qui modifient les flux énergétiques, la localisation des sites et les besoins en H₂.

3. **Déviation d’hydrogène vers des synthèses moléculaires nouvelles**  
   Exemple : production de carburants de synthèse (eSAF) ou de méthanol maritime (PtL/Power-to-Methanol), qui constitue une demande H₂ additionnelle par rapport aux usages historiques.

---

## 3) Organisation modulaire (vue d’ensemble)

Chaque module suit le même pattern :
1) lecture d’un **YAML** (paramètres, chemins, colonnes, contrôles),  
2) lecture d’une ou plusieurs **sources CSV**,  
3) reconstruction d’un **proxy d’activité** (production / volume substitué),  
4) génération de **scénarios** (constant / croissance / décroissance),  
5) calcul d’une **demande H₂ associée**,  
6) écriture des **sorties** dans une arborescence stable.

**Point critique : anti-double comptage**  
Les sorties sont “taggées” (scope / scope_tag) pour distinguer :
- **H₂ de raffinage** déjà inclus dans CONCAWE (unités hydrotraitement/hydrocracking),
- **H₂ additionnel** calculé par les modules eSAF / maritime / oléfines non-raffinerie.

---

## 4) Modules et hypothèses principales

### A) Module “Acier” (EU steel scenarios)
**But :** projeter la production d’acier et la répartition des routes (BF-BOF, DRI, EAF-scrap), puis calculer les demandes de H₂ et CH₄ associées.

**Hypothèses clés**
- Horizon : 2019–2050 (annuel).
- Production totale : trajectoire exogène (CAGR par pays).
- Mix technologique : rampes linéaires (EAF scrap, DRI) et ancrage sur l’existant 2019.
- DRI : split CH₄ vs H₂ par rampe de pénétration.
- Facteurs stœchiométriques : coefficients constants (H₂/acier, CH₄/acier).

**Sorties typiques**
- volumes par route (BF, DRI_H₂, DRI_CH₄, EAF),
- demandes H₂/CH₄ associées,
- contrôles de bilan matière.

**À faire**
- enrichir les intensités (efficacité énergétique, progrès technologique),
- intégrer contraintes de capacités / disponibilité ferraille / import DRI.

---

### B) Module “Ammoniac” (EU ammonia scenarios)
**But :** projeter une demande NH₃ “core” (ancrée) et une demande maritime additionnelle optionnelle, puis calculer H₂ réseau vs H₂ onsite (SMR), CH₄ et électricité Haber–Bosch.

**Hypothèses clés**
- Horizon : 2019–2050.
- Demande core : constante (par défaut).
- Demande maritime : trajectoire exogène + répartition spatiale par poids.
- Part domestique vs import : trajectoire en plateaux quinquennaux.
- Décarbonation : part NH₃ produite via H₂ bas-carbone vs route CH₄.
- Stœchiométrie H₂/NH₃ constante ; SMR via rendement global.

**Sorties typiques**
- NH₃ total/dom/imp, NH₃_H₂ vs NH₃_CH₄,
- H₂ réseau vs onsite, CH₄, électricité.

**À faire**
- connecter la demande maritime NH₃ à un module maritime dédié (si choix NH₃ comme carburant),
- affiner rendements et hypothèses d’électrolyse / import.

---

### C) Module “Raffineries” (CONCAWE)
**But :** reconstruire les feeds par unité (hydrocrackers/hydrotreater…) à partir de trajectoires CONCAWE et d’un proxy d’activité de raffinage, puis calculer la demande H₂ de raffinage.

**Sources**
- **CONCAWE** : capacités utilisées par unité, par scénario, disponibles sur un sous-ensemble d’années (ex. 2024/2030/2040/2050).
- Fichier de production totale “refinery_output” (pays, année, scénario), utilisé comme **proxy d’activité**.

**Hypothèses clés**
- Interpolation linéaire des capacités entre années CONCAWE.
- Part technologique = capacité unité / somme capacités.
- Feed unité = part × production totale raffinée.
- H₂ unité = feed × consommation spécifique (wt%) × (1 + inefficacité globale).
- Bilan matière : somme des feeds = production totale raffinée.

**Paramètres**
- Consommations spécifiques par unité (wt%),
- inefficacité globale (ex. 0.14).

**À faire**
- clarifier / verrouiller définitivement les unités d’entrée (t/an vs kt/an) et homogénéiser l’ensemble des pipelines,
- éventuellement intégrer des rendements produits (slates) si on souhaite aller au-delà de la logique “feeds unitaires”.

---

### D) Module “eSAF” (Power-to-Liquid aviation)
**But :** substituer la disponibilité fossile kérosène par une demande aviation exogène, et calculer l’eSAF requis + la demande H₂ associée.

**Source amont**
- Sortie CONCAWE (refinery_eu_demand.csv), utilisée pour reconstruire un proxy de kérosène fossile.

**Hypothèses clés**
- Reconstruction kérosène fossile : méthode “prefer_kero_hydrotreater” (anti-double comptage interne).
- Demande aviation totale : trajectoire exogène à partir de 2025 (constant / +3% / -1%).
- Substitution “stricte” : fossile ajusté borné par la demande totale ; eSAF = résiduel.
- Intensité H₂ eSAF : constante (ex. 0.50 tH₂/tSAF).

**À faire**
- documenter les sources technico-éco de l’intensité H₂ (et éventuellement dépendance au CO₂),
- introduire des scénarios d’amélioration de rendement PtL.

---

### E) Module “Maritime — méthanol (CH₃OH)”
**But :** construire un proxy de fuels marins à partir d’une unité raffinerie, définir une demande maritime totale, puis substituer linéairement vers le méthanol à équivalence énergétique et en déduire le H₂ requis.

**Proxy choisi**
- **Diesel Hydrotreater** comme proxy de la production MGO/pool diesel maritime.

**Hypothèses clés**
- Proxy marine fossile = θ × feed(Diesel Hydrotreater), avec θ = 0.10.
- Demande maritime totale : scénarios constant / croissance / décroissance (à partir de 2025).
- Pénétration méthanol : rampe linéaire de 2025 à 2050.
- Équivalence énergétique via PCI (diesel vs méthanol).
- Stœchiométrie MeOH : ratio massique H₂/MeOH (théorique) corrigé par un rendement global PtL (ex. 0.85).

**À faire**
- valider/justifier θ=0.10 (calibration par statistiques fuel marine),
- sourcer précisément les PCI et le rendement PtL retenus,
- vérifier la cohérence anti-double comptage avec un module “production de méthanol” si celui-ci existe déjà ailleurs (scope/tags).

---

### F) Module “Oléfines (FCC proxy)”
**But :** reconstruire un proxy “oléfines raffinerie/FCC” à partir de CONCAWE, définir une demande totale d’oléfines (3 scénarios), puis décomposer strictement en “raffinerie” vs “non-raffinerie”, avec ventilation non-raffinerie (e-oléfines, bio-e, bio) et demande H₂ associée.

**Proxy choisi**
- **FCC Gasoline Hydrotreater** comme indicateur de dynamique FCC.
- Proxy oléfines raffinerie = α × feed(proxy), avec α = 0.10.

**Hypothèses clés**
- Demande totale d’oléfines : trajectoire exogène à partir de 2025 (constant / +3% / -1%).
- Décomposition stricte : oléfines raffinerie bornées par le total ; résiduel = non-raffinerie.
- Non-raffinerie : 70% e-oléfines / 10% bio-e-oléfines / 20% bio-oléfines.
- Demandes H₂ : intensités constantes par segment (raffinerie proxy vs e/bio-e vs bio).

**À faire**
- consolider une interprétation claire de “non-raffinerie” (ce que cela représente réellement : import, steam cracker, nouvelle chimie…),
- documenter les intensités H₂ (sources + fourchettes),
- (option) créer le module “fcc-cracker” si l’on souhaite séparer explicitement la part steam cracker d’un total d’oléfines exogène.

---

## 5) Interdépendances et agrégation H₂ : règles pratiques

### Dépendance amont commune
Les modules **eSAF**, **maritime méthanol**, **oléfines-CONCAWE** lisent tous une source CONCAWE (refinery_eu_demand.csv) et dérivent des proxys à partir de feeds d’unités.

### Anti-double comptage : principe
- Le H₂ “raffinerie” provient directement des unités CONCAWE (hydrotraitement/hydrocracking).
- Les modules eSAF/maritime/oléfines ajoutent un H₂ **additionnel** lié à la production de nouvelles molécules ou aux segments “non-raffinerie”.
- Les sorties doivent être agrégées par **scope tags** disjoints.

### Recommandation
Unifier la nomenclature des colonnes H₂ en sortie (ex. `h2_total_t_per_yr`, `h2_refinery_t_per_yr`, `h2_additional_t_per_yr`) et imposer un contrôle automatique dans le pipeline (Snakemake).

---

## 6) Choix des paramètres : comment ils ont été fixés

Les paramètres ont été fixés selon trois logiques :
1) **Directement issus des sources** (ex. consommations spécifiques par unité CONCAWE).
2) **Ordres de grandeur réalistes** lorsque la source détaillée manque (ex. θ maritime=0.10, coefficient FCC→oléfines=0.10), en privilégiant la calibrabilité.
3) **Stœchiométrie / thermodynamique** pour les molécules de synthèse (H₂/MeOH théorique, correction par rendement global).

Chaque valeur doit être :
- déclarée dans le YAML,
- commentée (rôle + unité),
- idéalement accompagnée d’une référence bibliographique (à compléter).
- La plupart des paramètres méritent d'être discutés, a minima. 

---

## 7) Ce qu’il reste à faire (priorités)

1) **Homogénéisation des unités**
   - verrouiller définitivement l’unité des fichiers amont (t/an vs kt/an) et supprimer les ambiguïtés.

2) **Eventuellement vérifier la rédaction des fonctions**
   - Repasser sur les fonctions en cas de doute sur les sorties de résultats. Les rendre plus lisibles, si besoin. 

2) **Sourcing systématique**
   - ajouter des références explicites (rapport, paper, datasheet) pour : intensités H₂ eSAF, rendement PtL, PCI, intensités H₂ oléfines.

3) **Calibrations**
   - calibrer θ maritime et α FCC→oléfines sur des statistiques (bunkers/marine gasoil ; oléfines EU).
   - proposer des plages (min/central/max) plutôt qu’une valeur unique.

4) **Pipeline long-terme (Snakemake)**
   - verrouiller les chemins, versions de données et sorties attendues,
   - rendre reproductible l’ensemble (une commande, mêmes résultats).

5) **Contrôles automatiques renforcés**
   - bilans matières systématiques,
   - non-négativité,
   - couverture temporelle,
   - cohérence des scopes (anti-double comptage).

---

## 8) Comment exécuter (à adapter à votre arborescence)

- Chaque module : un YAML + un notebook (ou script) qui lit le YAML et produit des CSV.
- Objectif cible : exécution via Snakemake pour garantir la reproductibilité.

> TODO : compléter cette section avec les commandes exactes (`snakemake -j ...`) et la liste des règles une fois le pipeline stabilisé.

---