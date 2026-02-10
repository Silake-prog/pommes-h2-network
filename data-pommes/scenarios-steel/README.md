# Scénarios acier – Pommes H₂ Network

Ce dossier regroupe les **scénarios de production d’acier** utilisés pour alimenter le framework **POMMES**, avec un focus explicite sur :
- la **demande d’hydrogène**,
- la **réallocation progressive des routes de production**,
- la **cohérence comptable** des volumes d’acier.

Les scénarios sont construits à partir des **données observées en année de base (2019)** issues du fichier :

Toutes les hypothèses sont **paramétrées, traçables et explicites** via le fichier `steel_config.yaml`.

---

## 1. Point de départ commun à tous les scénarios

Ce document décrit la construction des quatre scénarios technologiques de production d’acier utilisés dans le module industrie du projet POMMES–H₂.

L’objectif est de transformer une situation observée en année de base (2019) en trajectoires cohérentes jusqu’en 2050, en conservant :
	•	la production totale d’acier,
	•	les capacités existantes,
	•	la cohérence comptable entre routes technologiques,
	•	une montée progressive des technologies bas-carbone.

### 1.1 Données de base (année 2019)

Pour chaque pays c, on dispose en 2019 de :
	•	S(c,2019) : production totale d’acier
	•	BF(c,2019) : production BF-BOF
	•	DRI_CH4(c,2019) : DRI au gaz naturel
	•	DRI_H2(c,2019) : DRI à l’hydrogène

La production EAF (acier secondaire, scrap) est reconstruite comme résidu :
EAF_scrap(c,2019) = S(c,2019) − [ BF(c,2019) + DRI_CH4(c,2019) + DRI_H2(c,2019) ]
Condition de validité :
EAF(c,2019) >= 0
---

### 1.2 Projection de la production totale d’acier

2. Production totale d’acier (commune à tous les scénarios)

La production totale d’acier est projetée via un CAGR constant par pays :

S(c,t) = S(c,2019) * (1 + CAGR(c))^(t − 2019)

Si CAGR(c) = 0, alors :
S(c,t) = S(c,2019)

- Le CAGR est paramétrable par pays (par défaut nul).
- Cette trajectoire est **identique pour tous les scénarios**.
- Les scénarios modifient **la répartition entre routes**, jamais le volume total.

---

### 1.3 Part croissante de l’acier recyclé (EAF-scrap)

Hypothèse transversale à **tous les scénarios** :

- Objectif : **50 % d’acier secondaire en 2050**,
- **Exception** : si un pays est déjà au-dessus de 50 % en 2019, sa part est conservée (aucune baisse forcée),
- Évolution **linéaire** entre 2019 et 2050 (rampe linéaire entre recycling.ramp_start et recycling.ramp_end (par défaut 2019→2050))

La rampe est paramétrée dans steel_config.yaml :
- recycling.ramp_start (défaut : 2019)
- recycling.ramp_end   (défaut : 2050)

Principe général

Pour chaque pays c, on définit la part d’acier secondaire :

share_EAF(c,2019) = EAF(c,2019) / S(c,2019)

Objectif européen :

share_EAF_target = 0.50 en 2050

Règle d’application
	•	Si share_EAF(c,2019) >= 0.50
→ la part EAF est maintenue constante
	•	Si share_EAF(c,2019) < 0.50
→ montée linéaire vers 50 % en 2050

Formellement :

share_EAF(c,t) =
  max(
    share_EAF(c,2019),
    share_EAF(c,2019) + (0.50 − share_EAF(c,2019)) * (t − 2019) / (2050 − 2019)
  )

La production EAF devient :

EAF(c,t) = share_EAF(c,t) * S(c,t)

---

### 1.4 Masse primaire disponible

La production primaire (routes BF + DRI) est définie par :

PRIMARY(c,t) = S(c,t) − EAF(c,t)
Contrainte :
PRIMARY(c,t) >= 0

Cette production primaire est la seule répartie entre :
	•	DRI-EAF
	•	BF-BOF

Toutes les dynamiques technologiques des scénarios (DRI, BF-BOF, CCUS, etc.)
s’appliquent **exclusivement à cette masse primaire**.

---

## 2. Structure commune aux scénarios

5. Définition générique de la part DRI

Dans tous les scénarios, on définit une cible de part DRI :

share_DRI_target(c,t)

La production DRI est alors ancrée sur l’existant :

DRI_target(c,t) = PRIMARY(c,t) * share_DRI_target(c,t)

DRI(c,t) = max(
  DRI(c,2019),
  DRI_target(c,t)
)

Attention : l’ancrage empêche la désinstallation du DRI existant, même si la montée du scrap réduit la masse primaire disponible.

La production BF-BOF est résiduelle :

BF(c,t) = PRIMARY(c,t) − DRI(c,t)

Mix énergétique du DRI

Le DRI est partagé entre CH4 et H2 selon une rampe temporelle :
	•	phase initiale : 100 % CH4
	•	transition progressive vers H2

share_H2_DRI(t) = ramp(t; t_start + N_CH4_only , t_start + N_CH4_only + N_H2_ramp)
share_CH4_DRI(t) = 1 − share_H2_DRI(t)

Avec : 

DRI_H2(c,t)  = DRI(c,t) * share_H2_DRI(t)
DRI_CH4(c,t) = DRI(c,t) * share_CH4_DRI(t)

---

## 3. Description des scénarios

---

## 3.1 Scénario `full-dri-eaf`

**Objectif**  
Transition maximale vers la filière **DRI–EAF**, avec disparition progressive du BF-BOF.

**Hypothèses principales**


7.1 Full DRI-EAF

Logique
Décarbonation maximale de la production primaire via DRI, tout en conservant :
	•	la montée du scrap (EAF),
	•	l’existant DRI en 2019.

share_DRI_target(t) → 1.0 en 2050

donc : 

BF(c,2050) ≈ 0

**Mix énergétique du DRI**
- Phase initiale 100 % gaz naturel,
- Introduction progressive de l’hydrogène après une période de transition,
- Substitution CH₄ → H₂ par rampe linéaire :


---

## 3.2 Scénario `full-bf-bof-ccus`

**Objectif**  
Maintien dominant du BF-BOF, avec décarbonation via **CCUS**, biomasse et injection d’hydrogène.

**Hypothèses principales**
share_DRI_target(c,2050) = valeur plafonnée (ex: 0.35)
- Le reste du primaire est produit via BF-BOF.

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
DRI_H2(c,t) = DRI_H2(c,2019)
DRI_CH4(c,t) = DRI(c,t) − DRI_H2(c,2019)

---

## 3.4 Scénario `intermediary`

**Objectif**  
Trajectoire intermédiaire, sans rupture technologique majeure.

**Hypothèses principales**
- La part DRI augmente progressivement mais reste plafonnée,
- Substitution CH₄ → H₂ plus lente que dans `full-dri-eaf`,
- BF-BOF reste significatif sur toute la période.

---


---

## 5. Garanties de cohérence

Tous les scénarios respectent strictement :

S(c,t) =
  EAF(c,t)
+ DRI_CH4(c,t)
+ DRI_H2(c,t)
+ BF(c,t)
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