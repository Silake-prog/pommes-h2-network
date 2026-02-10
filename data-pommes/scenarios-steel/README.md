# Scénarios acier – Pommes H₂ Network

Ce dossier regroupe les **scénarios de production d’acier** utilisés pour alimenter le framework **POMMES**, avec un focus explicite sur :
- la **demande d’hydrogène**,
- la **réallocation progressive des routes de production**,
- la **cohérence comptable** des volumes d’acier.

Les scénarios sont construits à partir des **données observées en année de base (2019)** issues du fichier :

Toutes les hypothèses sont **paramétrées, traçables et explicites** via le fichier `steel_config.yaml`.

---

## 1. Point de départ commun à tous les scénarios

### 1.1 Données de base (année 2019)

Pour chaque couple `(country, base_scenario)` :

- Production totale d’acier :
\[
S_{c,2019} = \texttt{steel\_production}_{2019}
\]

- Routes observées :
  - Haut-fourneau / convertisseur à oxygène (BF-BOF) :
\[
BF_{c,2019} = \texttt{blastfurnace\_bof\_production}_{2019}
\]
  - DRI au gaz naturel :
\[
DRI^{CH4}_{c,2019}
\]
  - DRI à l’hydrogène :
\[
DRI^{H2}_{c,2019}
\]

- Acier secondaire (EAF-scrap), **reconstruit par bilan de masse** :
\[
EAF_{c,2019} =
S_{c,2019}
- \left(
BF_{c,2019}
+ DRI^{CH4}_{c,2019}
+ DRI^{H2}_{c,2019}
\right)
\]

Cette reconstruction est **strictement comptable** : aucune hypothèse technologique supplémentaire n’est introduite.

---

### 1.2 Projection de la production totale d’acier

La production totale d’acier est projetée par pays à l’aide d’un **taux de croissance annuel composé (CAGR)** constant :

\[
S_{c,t} =
S_{c,2019}
\times (1 + \text{CAGR}_c)^{(t - 2019)}
\]

- Le CAGR est paramétrable par pays (par défaut nul).
- Cette trajectoire est **identique pour tous les scénarios**.
- Les scénarios modifient **la répartition entre routes**, jamais le volume total.

---

### 1.3 Part croissante de l’acier recyclé (EAF-scrap)

Hypothèse transversale à **tous les scénarios** :

- Objectif : **50 % d’acier secondaire en 2050**,
- **Exception** : si un pays est déjà au-dessus de 50 % en 2019, sa part est conservée (aucune baisse forcée),
- Évolution **linéaire** entre 2019 et 2050.

Soit :
- Part initiale :
\[
s_{c,2019} = \frac{EAF_{c,2019}}{S_{c,2019}}
\]
- Cible :
\[
s^{target} = 0.5
\]

Part en 2050 :
\[
s_{c,2050} = \max(s_{c,2019}, s^{target})
\]

Trajectoire temporelle :
\[
s_{c,t} =
s_{c,2019}
+ \lambda_t \cdot (s_{c,2050} - s_{c,2019})
\]

où \(\lambda_t \in [0,1]\) est une rampe linéaire entre 2019 et 2050.

Production EAF :
\[
EAF_{c,t} = s_{c,t} \cdot S_{c,t}
\]

---

### 1.4 Masse primaire disponible

La production primaire (routes BF + DRI) est définie par :

\[
P_{c,t} = S_{c,t} - EAF_{c,t}
\]

Toutes les dynamiques technologiques des scénarios (DRI, BF-BOF, CCUS, etc.)
s’appliquent **exclusivement à cette masse primaire**.

---

## 2. Structure commune aux scénarios

Pour chaque scénario :

\[
P_{c,t} = BF_{c,t} + DRI_{c,t}
\]

avec :
\[
DRI_{c,t} = DRI^{CH4}_{c,t} + DRI^{H2}_{c,t}
\]

### Ancrage sur l’existant

Dans tous les scénarios, on impose :

\[
DRI_{c,t} \ge DRI_{c,2019}
\]

Il n’y a **jamais de désinstallation rétroactive** de capacités existantes.

---

## 3. Description des scénarios

---

## 3.1 Scénario `full-dri-eaf`

**Objectif**  
Transition maximale vers la filière **DRI–EAF**, avec disparition progressive du BF-BOF.

**Hypothèses principales**
- La totalité de la production primaire devient DRI à l’horizon 2050 :
\[
DRI_{c,2050} = P_{c,2050}
\]
- Le BF-BOF devient strictement résiduel :
\[
BF_{c,t} = P_{c,t} - DRI_{c,t}
\]

**Mix énergétique du DRI**
- Phase initiale 100 % gaz naturel,
- Introduction progressive de l’hydrogène après une période de transition,
- Substitution CH₄ → H₂ par rampe linéaire :

\[
DRI^{H2}_{c,t} = \alpha_t \cdot DRI_{c,t}
\]

---

## 3.2 Scénario `full-bf-bof-ccus`

**Objectif**  
Maintien dominant du BF-BOF, avec décarbonation via **CCUS**, biomasse et injection d’hydrogène.

**Hypothèses principales**
- La part DRI est **plafonnée** :
\[
DRI_{c,2050} = \theta \cdot P_{c,2050}, \quad \theta < 1
\]
- Le reste du primaire est produit via BF-BOF.

**BF-BOF**
- Substitution partielle du charbon par :
  - biomasse (≤ 50 %),
  - hydrogène (≤ 20 %),
- Mise en place progressive du CCUS avec un taux de capture croissant :
\[
\text{capture\_rate}_t \in [0, \text{max}]
\]

> Le CCUS agit **uniquement sur les émissions**, jamais sur le volume d’acier produit.

---

## 3.3 Scénario `import-dri`

**Objectif**  
Externalisation de la production de DRI hors Europe, avec transformation locale en EAF.

**Hypothèses principales**
- La totalité du primaire devient DRI à l’horizon 2050,
- La croissance du DRI **n’induit aucune demande d’hydrogène domestique**,
- Le DRI importé est comptabilisé comme DRI non-H₂ (proxy gaz).

Formellement :
\[
DRI^{H2}_{c,t} = DRI^{H2}_{c,2019}
\]
\[
DRI^{CH4}_{c,t} = DRI_{c,t} - DRI^{H2}_{c,2019}
\]

---

## 3.4 Scénario `intermediary`

**Objectif**  
Trajectoire intermédiaire, sans rupture technologique majeure.

**Hypothèses principales**
- La part DRI augmente progressivement mais reste plafonnée,
- Substitution CH₄ → H₂ plus lente que dans `full-dri-eaf`,
- BF-BOF reste significatif sur toute la période.

---

## 4. Demande énergétique associée

Deux colonnes de demande sont calculées pour chaque scénario :

- Demande en hydrogène :
\[
H2\_demand_{c,t} = 0.077 \times DRI^{H2}_{c,t}
\]

- Demande en gaz naturel (proxy CH₄) :
\[
CH4\_demand_{c,t} = 0.2281 \times DRI^{CH4}_{c,t}
\]

Unités : **kg de gaz par tonne d’acier produite via la route concernée**.  
Les facteurs sont **paramétrés dans le YAML**.

---

## 5. Garanties de cohérence

Tous les scénarios respectent strictement :

\[
S_{c,t} =
EAF_{c,t}
+ BF_{c,t}
+ DRI^{CH4}_{c,t}
+ DRI^{H2}_{c,t}
\]

avec :
- continuité annuelle stricte (2019–2050),
- non-négativité de toutes les productions,
- traçabilité complète des hypothèses et paramètres.

---

## 6. Philosophie générale

Ces scénarios :
- ne cherchent **ni l’optimisation**, ni la prévision,
- n’introduisent **aucune technologie non paramétrée**,
- fournissent à POMMES un **ensemble cohérent et lisible de trajectoires** pour analyser :
  - la demande d’hydrogène,
  - les besoins d’infrastructures,
  - les effets systémiques de la transition industrielle.

---