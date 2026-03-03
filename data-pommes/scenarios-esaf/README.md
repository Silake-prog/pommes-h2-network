eSAF Module – Substitution interne stricte du kérosène fossile

1. Objectif

Ce module modélise une substitution interne stricte du kérosène fossile par de l’eSAF (Power-to-Liquid) à partir d’un scénario refinery existant (CONCAWE).

Il est couplé aux deux trajectoires refinery :
	•	more-molecule
	•	max-electron

L’objectif est de :
	1.	Reconstruire la production de kérosène fossile à partir des unités refinery.
	2.	Définir une trajectoire exogène de demande aviation.
	3.	Calculer la production d’eSAF nécessaire.
	4.	En déduire la demande d’hydrogène associée.

Toutes les grandeurs sont exprimées en kilotonnes par an (kt/an).

⸻

2. Architecture du modèle

Le module s’appuie sur :

scenarios-refinery/
    refinery-eu-demand_more-molecule/
    refinery-eu-demand_max-electron/

→ reconstruction Jet fossile
→ définition Jet_total(t)
→ substitution stricte
→ calcul Jet_eSAF(t)
→ calcul H2_eSAF(t)

Les résultats sont écrits dans :

scenarios-esaf/
    more-molecule/
        constant-demand/
        growth-3pct/
        decline-1pct/
    max-electron/
        constant-demand/
        growth-3pct/
        decline-1pct/


⸻

3. Reconstruction du kérosène fossile

3.1 Principe

Le kérosène fossile est reconstruit à partir des données refinery :
	•	colonne unit
	•	colonne unit_feed

Deux méthodes sont disponibles :

Méthode recommandée (anti double comptage)

Jet_fossil(t) = feed(Kero Hydrotreater)

Cela évite :
	•	double comptage via hydrocrackers
	•	comptage indirect via coupes intermédiaires

Méthode fallback (si nécessaire)

Jet_fossil(t) = Σ (yield_u × feed_u)

avec :
	•	VGO Hydrocracker
	•	Residue Hydrocracker
	•	rendements paramétrés dans le YAML

⸻

4. Substitution interne stricte

4.1 Année de référence

On fixe :

reference_year = 2025

On définit :

Jet_{reference} = Jet_{fossil}^{refinery}(2025)

⸻

4.2 Trajectoire de demande aviation

Pour chaque scénario :

Jet_{total}(t) = Jet_{reference} \cdot (1 + g)^{t-2025}

où :
	•	g = 0.0 (constant-demand)
	•	g = 0.03 (growth-3pct)
	•	g = -0.01 (decline-1pct)

⸻

4.3 Substitution stricte

On impose :

Jet_{fossil}^{adj}(t) =
\min(Jet_{fossil}^{refinery}(t), Jet_{total}(t))

Jet_{eSAF}(t) =
Jet_{total}(t) - Jet_{fossil}^{adj}(t)

Donc toujours :

Jet_{total} = Jet_{fossil}^{adj} + Jet_{eSAF}

Le bilan de masse aviation est strictement respecté.

⸻

5. Demande d’hydrogène eSAF

Hypothèse :

I_{PtL} = 0.50 \; tH_2 / tSAF

Comme tout est en kt/an :

H2_{eSAF}(t) = Jet_{eSAF}(t) \cdot 0.50

Unité :

kt H₂ / an

Hypothèses simplificatrices :
	•	Intensité constante
	•	Pas d’amélioration technologique
	•	Pas de co-produits
	•	Pas de contraintes de capacité

⸻

6. Unités

Toutes les colonnes exportées sont en :

kilotonnes par an (kt/an)

Colonnes principales :
	•	jet_fossil_kt_per_yr
	•	jet_total_kt_per_yr
	•	jet_esaf_kt_per_yr
	•	h2_esaf_kt_per_yr

La cohérence d’unité est garantie avec les fichiers refinery CONCAWE.

⸻

7. Invariants et contrôles

Le module applique :

7.1 Bilan de masse

Jet_{total} = Jet_{fossil}^{adj} + Jet_{eSAF}

7.2 Non-négativité

Aucun flux négatif autorisé.

7.3 Couverture temporelle complète

Toutes les années doivent être présentes pour chaque pays.

⸻

8. Interprétation économique

Le modèle distingue clairement :
	•	La trajectoire structurelle refinery (offre fossile industrielle)
	•	La demande aviation exogène
	•	L’eSAF comme variable d’ajustement

Dans le scénario :
	•	more-molecule → trajectoire fossile plus persistante
	•	max-electron → contraction plus forte

L’eSAF vient compenser uniquement lorsque la demande aviation dépasse la capacité fossile.
