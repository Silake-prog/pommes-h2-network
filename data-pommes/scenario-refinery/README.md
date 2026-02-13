Voici une version synthétique, rigoureuse et adaptée à un README Git, claire sur les hypothèses et les équations.

⸻

Refinery EU H₂ Demand – Modelling Assumptions

1. Objective

This module estimates hydrogen demand in EU refineries under two CONCAWE scenarios:
	•	more-molecule (main scenario)
	•	max-electron

The goal is to generate a country-level annual H₂ demand dataset (2019–2050) suitable for integration into the POMMES hydrogen network framework.

⸻

2. Data Sources

2.1 IDEES (JRC Energy Balance)

We use:

Oil products (excluding biofuel portion) – O4600XBIO

Unit in IDEES:

kilotonnes per year (kt/year)

This represents total refined petroleum products output.

We convert to tonnes:

refinery_output_t = IDEES_value_kt × 1000


⸻

2.2 CONCAWE Scenarios

For each refining unit:
	•	Utilized capacity (Mton/year)
	•	Specific H₂ consumption:

Spec Cons (wt% on feed)

We use:
	•	max-electron
	•	more-molecule

CONCAWE values are used to determine technology shares, not absolute country volumes.

⸻

3. Core Modelling Assumptions

3.1 Proxy for Feed

CONCAWE provides:

H₂ consumption = wt% × feed

However, IDEES provides refined product output, not crude feed.

We assume:

refined products ≈ refinery feed

Justification:
	•	Refinery mass balance efficiency ≈ 92–98%
	•	Losses are small relative to macro modelling uncertainty

This approximation is acceptable at EU system level.

⸻

3.2 Technology Shares

For each year t:

share_u(t) = \frac{Cap_u(t)}{\sum_v Cap_v(t)}

Where:
	•	Cap_u(t) = CONCAWE utilized capacity (Mton)
	•	Shares are dimensionless
	•	Interpolated annually between 2024–2050

⸻

3.3 Allocation to Countries

For each country c:

Feed_{u,c}(t) = refinery\_output_c(t) \times share_u(t)

Unit: tonnes/year

This assumes:

EU refining structure is representative at country level.

This is a structural simplification for network modelling.

⸻

3.4 Hydrogen Consumption

CONCAWE provides specific consumption:

Spec_u = \text{wt% on feed}

Hydrogen process demand:

H2_{process,u,c}(t) =
Feed_{u,c}(t) \times \frac{Spec_u}{100}

⸻

3.5 System Inefficiencies

CONCAWE indicates:

Total with inefficiencies = +14%

We therefore model:

H2_{network,u,c}(t) =
H2_{process,u,c}(t) \times (1 + 0.14)

Final unit: tonnes H₂ per year.

⸻

4. Level Adjustment (Optional)

To reflect CONCAWE decline in total refining activity:

Level(t) =
\frac{\sum_u Cap_u(t)}{\sum_u Cap_u(2024)}

Applied as:

refinery\_output_c(t) =
refinery\_output_c(2019) \times Level(t)

Before 2024:

Level(t) = 1

This ensures:
	•	Structural + volume consistency with CONCAWE scenarios.

⸻

5. Final Output Structure

For each:

country
year
scenario
unit

We provide:
	•	refinery_output_total (t/year)
	•	unit_share
	•	unit_feed (t/year)
	•	spec_cons_wt
	•	h2_demand (t H₂/year)

Mass balance identity enforced:

\sum_u unit\_feed = refinery\_output\_total

⸻

6. Units Summary

Variable	Unit
IDEES data	kt/year
refinery_output_total	t/year
unit_feed	t/year
h2_demand	t H₂/year
CONCAWE capacity	Mton/year (used only for shares)

No unit inconsistency exists because:
	•	Mton cancel in share computation.
	•	All physical quantities are expressed in tonnes at allocation stage.

⸻

7. Structural Limitations
	1.	Country-level refining structure assumed identical to EU mix.
	2.	Feed approximated by refined product output.
	3.	No explicit modelling of refinery closures by country.
	4.	H₂ intensity assumed constant per unit type over time.

These simplifications are acceptable for network-level hydrogen infrastructure planning, but not for refinery process engineering analysis.

⸻

8. Rationale for Model Choice

The objective is:

Estimate hydrogen network demand sensitivity to refinery decarbonisation pathways.

Therefore:
	•	Structural consistency > micro-process precision
	•	Transparency > complexity
	•	Auditability > hidden assumptions

All parameters are YAML-driven for traceability.

⸻

If you want, I can also provide a short “Modelling Philosophy” section explaining why this is appropriate for POMMES but not for process simulation.