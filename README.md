# insurance-claims-statistical-modeling
> Projet de statistique des assurances — Master 2 Actuariat

Modélisation complète de la sinistralité d'un portefeuille de 5 352 ménages assurés, couvrant la tarification a priori/a posteriori, la modélisation de la fréquence et de la sévérité des sinistres, ainsi que l'analyse de la durée de souscription.

---

## Objectifs

- Estimer la **prime pure** à partir des caractéristiques socio-démographiques des assurés
- Modéliser la **fréquence** et la **sévérité** des sinistres séparément
- Analyser la **durée de fidélité** des assurés au contrat

---

## Données

| Caractéristique | Détail |
|---|---|
| Observations | 5 352 ménages |
| Variables | 27 (socio-démo, sinistres, polices, durée) |
| Source | Base simulée à des fins pédagogiques |
| Période de censure | Variable `censure` (données de survie) |

Les variables incluent : catégorie socio-professionnelle (PCS), revenu par unité de consommation (RUC), composition du ménage, région, type d'habitat, possession d'un véhicule, montants de sinistres (Sinistre0/1/2/3), primes (Police1/2/3) et durée de souscription.

---

## Modèles implémentés

### 1. Tarification a priori — GLM Gamma
Modélisation de `Sinistre0` (prime agrégée, strictement positive) par un GLM à loi Gamma avec lien log.

```r
glm(Sinistre0 ~ pcs + cs + agecat + RUC + region + Acompm + nbpers + Anat + Bauto,
    family = Gamma(link = "log"), data = dat)
```

**Résultats clés :** RUC et composition du ménage (Acompm) sont les principaux déterminants de la prime. Erreurs robustes utilisées pour corriger l'hétéroscédasticité.

---

### 2. Modélisation de la sévérité — Tobit / Hurdle
Comparaison de trois approches sur `Sinistre1` (76 % de zéros) :

| Modèle | AIC | Conclusion |
|---|---|---|
| Tobit standard | 14 246 | Hypothèse unique restrictive |
| Tobit généralisé (type 2) | 16 711 | rho non significatif, pas d'amélioration |
| **Hurdle Logit + Gamma** | **11 736** | **Meilleur ajustement — retenu** |

Le modèle hurdle modélise séparément :
- La **probabilité de sinistre** (régression logistique sur PCS et âge)
- Le **montant conditionnel** (GLM Gamma sur région et taille du ménage)

---

### 3. Modélisation de la fréquence — Binomiale Négative
`NSin` présente une surdispersion marquée (variance = 14,53 >> moyenne = 4,25).

| Modèle | AIC |
|---|---|
| Poisson | 29 127 |
| **Binomiale Négative** | **25 949** |

Variables significatives : PCS, âge, taille du ménage, région 5 et région 9.

---

### 4. Analyse de survie — Kaplan-Meier & Cox
Modélisation de la durée de souscription en tenant compte de la censure à droite.

**Kaplan-Meier stratifié** — différences significatives selon :
- Type d'habitat (p = 0,005)
- Possession d'un véhicule (p < 0,001)
- Catégorie d'âge (p < 0,001)

**Modèle de Cox** — concordance = 0,77 :
- Les ménages plus âgés et à revenus élevés résilient moins
- La taille du ménage augmente fortement le risque de résiliation

---

## Stack technique

![R](https://img.shields.io/badge/R-4.x-276DC3?logo=r&logoColor=white)

| Package | Usage |
|---|---|
| `glm` (base R) | GLM Gamma, Logit, Binomiale Négative |
| `MASS` | `glm.nb()` — Binomiale Négative |
| `AER` | Modèle Tobit |
| `sampleSelection` | Tobit généralisé (type 2) |
| `survival` | Kaplan-Meier, modèle de Cox |
| `survminer` | Visualisation des courbes de survie |
| `sandwich` / `lmtest` | Erreurs robustes (hétéroscédasticité) |
| `ggplot2` | Visualisations |

---

## Structure du dépôt

```
├── README.md
├── rapport/
│   ├── projet_assurances.Rmd       # Code source reproductible
│   └── projet_assurances.pdf       # Rapport final
├── data/
│   └── README.md                   # Description des variables (données non publiées)
└── outputs/
    └── figures/                    # Graphiques exportés
```

---

## Résultats synthétiques

- La **composition du ménage** et le **revenu (RUC)** sont les déterminants les plus robustes de la prime
- Le modèle **hurdle** est supérieur au Tobit pour modéliser des sinistres avec inflation de zéros
- Les assurés **plus âgés** et **sans véhicule** sont les plus fidèles (risque de résiliation plus faible)
- La **surdispersion** du nombre de sinistres est correctement capturée par la binomiale négative

---

## Auteur

**Ridge Louigarde**  
Master 2 — Statistique & Actuariat  
2026
