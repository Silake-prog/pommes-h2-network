

# Scénarios de demande d’ammoniac – Réseau Hydrogène Europe *** 


Ce module construit des scénarios prospectifs de demande d’ammoniac (NH₃) pour l’Europe sur la période 2019–2050, afin d’alimenter le modèle de réseau hydrogène.

Les objectifs sont :
	•	Ancrer toutes les trajectoires sur la consommation observée en 2019.
	•	Modéliser la substitution technologique :
	•	CH4 → H2 → NH3
	•	H2 → NH3
	•	Introduire une croissance de la demande maritime (carburant NH₃).
	•	Contrôler la part de production domestique vs importations.
	•	Déduire explicitement les demandes en hydrogène et méthane.

⸻

## 1. Horizon temporel

Les scénarios sont construits sur une base annuelle :

t ∈ {2019, ..., 2050}

L’année 2019 est l’année de référence.

⸻

## 2. Données de base

Le fichier brut contient :

country, base_scenario, year, nh3_consumption_t

Seule l’année 2019 est utilisée comme ancrage.

Pour chaque pays c :

NH3_core_c(t) = NH3_2019_c   ∀ t

La demande structurelle est donc constante (hors effet maritime).

⸻

## 3. Demande maritime en ammoniac

Un module optionnel ajoute une demande supplémentaire liée au secteur maritime.

3.1 Demande maritime totale européenne

Si activé :

NH3_maritime_EU(t) =
    NH3_maritime_2019 × (1 + g)^(t − 2019)

où :
	•	g = taux de croissance annuel (CAGR)
	•	NH3_maritime_2019 = demande maritime initiale

Sinon :

NH3_maritime_EU(t) = 0


⸻

3.2 Allocation spatiale

Deux modes d’allocation sont disponibles :

(A) Proportionnelle à la consommation 2019

w_c = NH3_2019_c / Σ NH3_2019_c

(B) Répartition uniforme

w_c = 1 / N_pays

La demande maritime par pays devient :

NH3_maritime_c(t) = w_c × NH3_maritime_EU(t)


⸻

## 4. Demande totale d’ammoniac

Pour chaque pays :

NH3_total_c(t) =
    NH3_core_c(t)
  + NH3_maritime_c(t)

Cette demande totale doit respecter :

NH3_total_c(t) =
    NH3_domestic_c(t)
  + NH3_imported_c(t)


⸻

## 5. Production domestique vs importations

Chaque scénario définit une trajectoire de part domestique :

s_domestic(t) ∈ [0,1]

Production domestique :

NH3_domestic_c(t) =
    s_domestic(t) × NH3_total_c(t)

Importations :

NH3_imported_c(t) =
    NH3_total_c(t) − NH3_domestic_c(t)

Cas des pays sans accès maritime (AT, HU, SK)

Dans les scénarios concernés :

NH3_imported_c(t) = 0
NH3_domestic_c(t) = NH3_total_c(t)

L’adéquation est donc toujours garantie :

NH3_total = NH3_domestic + NH3_imported


⸻

## 6. Substitution technologique (décarbonation)

La production domestique est divisée en deux routes :
	1.	CH4 → H2 → NH3
	2.	H2 → NH3

Chaque scénario définit une trajectoire :

s_H2(t) ∈ [0,1]

Alors :

NH3_domestic_H2_route_c(t) =
    s_H2(t) × NH3_domestic_c(t)

NH3_domestic_CH4_route_c(t) =
    NH3_domestic_c(t)
  − NH3_domestic_H2_route_c(t)

Ce découpage garantit :

NH3_domestic =
    NH3_CH4_route
  + NH3_H2_route

Les trajectoires évoluent par paliers de 5 ans.

⸻

## 7. Demande en hydrogène

Hypothèse technique :

0.18 t H2 / t NH3

7.1 Hydrogène issu du réseau

Uniquement pour la route directe :

H2_network_c(t) =
    0.18 × NH3_domestic_H2_route_c(t)

7.2 Hydrogène produit sur site (SMR)

Pour la route méthane :

H2_onsite_c(t) =
    0.18 × NH3_domestic_CH4_route_c(t)


⸻

## 8. Demande en méthane

Stoéchiométrie minimale :

2.0 t CH4 / t H2

Corrigée par un rendement η :

CH4_per_H2 = 2.0 / η

La demande en méthane devient :

CH4_c(t) =
    H2_onsite_c(t) × CH4_per_H2


⸻

## 9. Électricité Haber–Bosch

Consommation spécifique :

0.98522 MWh / t NH3

Appliquée à la production domestique :

HB_electricity_c(t) =
    0.98522 × NH3_domestic_c(t)


⸻

## 10. Description des scénarios

10.1 full-eu-nh3
	•	Production 100 % domestique.
	•	Aucune importation.
	•	Substitution complète vers H2 → NH3 en 2050.

s_domestic(t) = 1
s_H2(2050) = 1


⸻

10.2 import-eu-nh3
	•	Production domestique diminue jusqu’à 50 % en 2050.
	•	Le reste est importé.
	•	Pays enclavés : import interdit.
	•	Décarbonation complète du domestique en 2050.

s_domestic(2050) = 0.5
s_H2(2050) = 1


⸻

10.3 intermediary-eu-nh3
	•	Production domestique diminue jusqu’à 85 % en 2050.
	•	Le reste est importé.
	•	Pays enclavés : import interdit.
	•	Décarbonation complète en 2050.

s_domestic(2050) = 0.85
s_H2(2050) = 1


⸻

## 11. Contraintes d’intégrité

Pour tout pays et toute année :
	1.	Équilibre matière :

NH3_total =
    NH3_domestic
  + NH3_imported

	2.	Cohérence technologique :

NH3_domestic =
    NH3_CH4_route
  + NH3_H2_route

	3.	Non-négativité :

Tous les flux ≥ 0


⸻

## 12. Sorties

Chaque scénario génère :
	•	ammonia.csv
	•	config_effective.yaml

Les colonnes principales incluent :
	•	NH₃ total, domestique, importé
	•	Répartition technologique
	•	Demande H₂ réseau
	•	Production H₂ sur site
	•	Demande CH₄
	•	Électricité Haber–Bosch
	•	Composante maritime

⸻

13. Philosophie de modélisation
	•	Ancrage strict 2019.
	•	Trajectoires par paliers (5 ans).
	•	Séparation claire entre :
	•	Structure de demande
	•	Commerce
	•	Technologie
	•	Croissance maritime
	•	Conversion physico-chimique

Le système est :
	•	Conservatif (masse respectée),
	•	Paramétrable,
	•	Audit-able,
	•	Compatible avec un modèle de réseau hydrogène multi-pays.