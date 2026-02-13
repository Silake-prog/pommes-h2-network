Voici un README structuré, académique et traçable, prêt à être intégré dans Git (Markdown standard, équations lisibles, pas de dépendance LaTeX externe).

⸻

Olefins – EU H₂ Demand Scenarios

1. Objectif

Ce module construit deux trajectoires prospectives de demande d’hydrogène pour le périmètre pétrochimie – oléfines (steam cracking) à l’échelle UE :
	•	bau-eu : trajectoire tendancielle (prolongation structure moléculaire).
	•	decarbo-eu : trajectoire de décarbonation (électrification partielle, circularité, efficacité).

Le modèle est conçu pour :
	•	Être cohérent avec les scénarios refinery (via un couplage macro d’activité),
	•	Éviter toute double comptabilité avec le module raffinage,
	•	Être traçable, paramétrable et audit-able via config_olefins.yaml.

⸻

2. Périmètre

Inclus
	•	Steam cracking et production d’oléfines (proxy d’activité pétrochimique).
	•	Demande d’H₂ associée aux procédés pétrochimiques.

Exclus
	•	Hydrotreating, hydrocracking, désulfuration, etc. (périmètre raffinage).
	•	Ammoniac et méthanol (modules dédiés).
	•	Plastiques en aval (la demande est ici captée indirectement via l’activité oléfines).

⸻

3. Structure du modèle

3.1 Variable d’activité

Le fichier olefins.csv fournit :

country | year | scenario? | olefins_production

⚠️ Important :
olefins_production est un proxy énergétique (issu d’IDEES – naphta converti en MWh), pas une production physique en tonnes d’oléfines.

On l’interprète donc comme :

olefins_activity(t)

et non comme une production physique directe.

⸻

3.2 Formulation de la demande H₂

Pour chaque pays c, année t et scénario s :

H2_demand(c,t,s) = activity(c,t,s) × intensity(c,t,s)

où :
	•	activity(c,t,s) = activité oléfines
	•	intensity(c,t,s) = intensité H₂

⸻

4. Couplage avec le raffinage

Si linkage.enabled = true, l’activité oléfines suit l’évolution relative du raffinage :

index_refinery(t) = refinery_output(t) / refinery_output(base_year)

activity(c,t,s) = activity_2019(c) × index_refinery(c,t,s)

Cela garantit :
	•	Cohérence macro des volumes fossiles,
	•	Absence de divergence structurelle entre raffinage et pétrochimie,
	•	Réalisme dynamique des scénarios.

Le mapping est défini dans :

scenario_mapping:
  refinery_to_olefins:
    more-molecule → bau-eu
    max-electron  → decarbo-eu


⸻

5. Intensité H₂

5.1 Valeur de base

intensity_2019_tH2_per_activity_unit

⚠️ Cette intensité est un paramètre de calibration sur proxy,
et non un coefficient physico-chimique direct (l’activité n’est pas en tonnes).

5.2 Trajectoire

L’intensité évolue linéairement :

intensity(t) = intensity_2019 × [1 + r(t) × (factor_2050 - 1)]

avec :

r(t) = rampe linéaire entre start et end

Paramètres typiques :

Scénario	Facteur 2050
bau-eu	0.95
decarbo-eu	0.80

Interprétation :
	•	BAU : amélioration marginale d’efficacité.
	•	DECARBO : gains structurels plus importants (électrification, circularité).

⸻

6. Description des scénarios

6.1 bau-eu

Hypothèses :
	•	Maintien de la pétrochimie fossile.
	•	Activité corrélée au raffinage (tendance moléculaire).
	•	Baisse modérée de l’intensité H₂ (-5% en 2050).

Logique :

Continuité industrielle, adaptation incrémentale.

⸻

6.2 decarbo-eu

Hypothèses :
	•	Déclin structurel des volumes fossiles.
	•	Couplage au scénario refinery max-electron.
	•	Baisse plus forte d’intensité H₂ (-20% en 2050).

Logique :

Substitution partielle, efficacité accrue, circularité des plastiques.

⸻

7. Contrôles de cohérence

Le runner applique :
	•	Non-négativité des activités et demandes,
	•	Couverture complète 2019–2050,
	•	Tag explicite scope = petrochem_olefins,
	•	Vérification de cohérence avec l’index refinery.

⸻

8. Anti double comptabilité

La séparation est explicite :

Module	Scope
Refinery	refining_h2
Olefins	petrochem_olefins

Agrégation correcte :

total_h2 = sum_by_scope(...)

On ne réutilise jamais refinery_output_total comme base pour olefins.

⸻

9. Limites du modèle

9.1 Proxy énergétique

L’activité est dérivée d’un proxy naphta IDEES converti en MWh, puis ajusté.
Ce n’est pas une production physique.

Conséquence :
	•	L’intensité H₂ est une calibration,
	•	L’interprétation thermodynamique directe est limitée.

⸻

9.2 Absence de désagrégation technologique

Le modèle ne distingue pas :
	•	Steam cracking conventionnel,
	•	Cracking électrifié,
	•	Pyrolyse plasma,
	•	CCS associé.

Il s’agit d’un modèle macro.

⸻

9.3 Hypothèse de corrélation raffinage–pétrochimie

Le lien via l’index refinery suppose que :

Les volumes pétrochimiques suivent les tendances globales des flux fossiles.

Ce n’est pas garanti à long terme si la chimie se découple des carburants.

⸻

10. Améliorations futures
	•	Calibration de intensity_2019 sur données observées H₂ pétrochimie.
	•	Conversion proxy → tonnes physiques d’oléfines.
	•	Désagrégation technologique (steam cracker électrique).
	•	Intégration de scénarios de circularité plastique explicites.

⸻

11. Fichiers générés

data-pommes/scenarios-olefins/
 ├── olefins_base_2019.csv
 ├── olefins_refinery_index.csv
 ├── metadata.json
 ├── bau-eu/olefins.csv
 └── decarbo-eu/olefins.csv


⸻

12. Conclusion

Le module fournit :
	•	Une scénarisation cohérente avec le raffinage,
	•	Une séparation stricte des périmètres,
	•	Une architecture paramétrable,
	•	Une traçabilité complète via YAML + metadata.

Il constitue un cadre macro robuste, mais reste dépendant d’un proxy d’activité nécessitant une calibration explicite pour toute interprétation physique fine.

⸻

Si vous le souhaitez, je peux maintenant :
	•	Ajouter une section “Justification économique des trajectoires”,
	•	Ou rédiger un README global harmonisé pour ammonia + steel + refinery + olefins.