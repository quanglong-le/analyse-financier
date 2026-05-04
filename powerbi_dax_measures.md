# Power BI — Documentation Dashboard LuxMode Retail 2025

**Auteur** : LE Quang Long — Numéro étudiant 2400145  
**Université Paris Panthéon-Assas — Février 2026**

---

## 1. Architecture du modèle de données

### Source de données

Importer le fichier `donnees_simulation.csv` dans Power BI Desktop via :  
`Accueil → Obtenir des données → Texte/CSV`

### Table principale : `Transactions`

| Colonne | Type Power BI | Description |
|---|---|---|
| date_piece | Date | Date de la pièce comptable |
| mois | Nombre entier | Numéro du mois (1–12) |
| type_piece | Texte | FA / FV / SA |
| centre_cout | Texte | CC1000 – CC4000 |
| produit | Texte | P01 – P04 |
| compte_gl | Texte | Compte du grand livre |
| libelle | Texte | Description |
| debit | Décimal | Montant débit |
| credit | Décimal | Montant crédit |
| montant_net | Décimal | Montant net |

### Table de dimensions : `Dim_Mois` (créer manuellement)

```
Saisir données → créer tableau :
NumMois | LibelleMois | Trimestre
1       | Janvier     | T1
2       | Février     | T1
...
12      | Décembre    | T4
```

Relation : `Transactions[mois]` → `Dim_Mois[NumMois]` (Many-to-One)

### Table de dimensions : `Dim_Produit` (créer manuellement)

```
CodeProduit | LibelleProduit      | Categorie
P01         | Hauts               | Vêtements
P02         | Pantalons & Jeans   | Vêtements
P03         | Robes & Jupes       | Vêtements
P04         | Accessoires         | Accessoires
```

Relation : `Transactions[produit]` → `Dim_Produit[CodeProduit]` (Many-to-One)

### Table de dimensions : `Dim_CentreCout` (créer manuellement)

```
Code   | Libelle
CC1000 | Magasin
CC2000 | Logistique
CC3000 | Marketing
CC4000 | Administration
```

---

## 2. Mesures DAX

### 2.1 Mesures de base

```dax
// Chiffre d'affaires
CA =
CALCULATE(
    SUMX(Transactions, Transactions[montant_net]),
    Transactions[compte_gl] = "701000"
)

// Coût des marchandises vendues
CMV =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] = "601000"
)

// Marge brute
Marge_Brute = [CA] - [CMV]

// Taux de marge brute
Taux_Marge_Brute =
DIVIDE([Marge_Brute], [CA], 0)

// Total charges de personnel
Charges_Personnel =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] IN { "641000", "641001" }
)

// Loyer et charges locatives
Charges_Loyer =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] = "613000"
)

// Charges marketing
Charges_Marketing =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] = "623000"
)

// Charges logistique
Charges_Logistique =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] = "624000"
)

// Frais divers
Frais_Divers =
CALCULATE(
    SUMX(Transactions, ABS(Transactions[montant_net])),
    Transactions[compte_gl] = "627000"
)

// Total charges fixes
Charges_Fixes = [Charges_Personnel] + [Charges_Loyer]

// Total charges variables
Charges_Variables = [Charges_Marketing] + [Charges_Logistique] + [Frais_Divers]

// Résultat d'exploitation (EBIT)
EBIT = [Marge_Brute] - [Charges_Fixes] - [Charges_Variables]

// Taux d'EBIT
Taux_EBIT =
DIVIDE([EBIT], [CA], 0)
```

### 2.2 Mesures de comparaison et tendance

```dax
// CA mois précédent
CA_Mois_Precedent =
CALCULATE(
    [CA],
    DATEADD(Transactions[date_piece], -1, MONTH)
)

// Variation CA vs mois précédent
Variation_CA_MoM =
DIVIDE(
    [CA] - [CA_Mois_Precedent],
    [CA_Mois_Precedent],
    BLANK()
)

// CA cumulé (YTD)
CA_YTD =
CALCULATE(
    [CA],
    DATESYTD(Transactions[date_piece])
)

// Meilleur mois (CA max)
CA_Max_Mois =
MAXX(
    VALUES(Dim_Mois[LibelleMois]),
    [CA]
)

// Pire mois (CA min)
CA_Min_Mois =
MINX(
    VALUES(Dim_Mois[LibelleMois]),
    [CA]
)
```

### 2.3 Mesures budget vs réel

> Note : Pour la comparaison budget vs réel, créer une table `Budget_2025` avec les colonnes `mois` et `CA_Budget`, puis relier à `Dim_Mois`.

```dax
// CA Budget mensuel (exemple — à adapter selon table Budget)
CA_Budget =
CALCULATE(
    SUM(Budget_2025[CA_Budget]),
    USERELATIONSHIP(Budget_2025[mois], Dim_Mois[NumMois])
)

// Écart budget vs réel (€)
Ecart_Budget =
[CA] - [CA_Budget]

// Écart budget vs réel (%)
Ecart_Budget_Pct =
DIVIDE([Ecart_Budget], [CA_Budget], BLANK())
```

### 2.4 Mesures par produit

```dax
// Part de chaque produit dans le CA total
Part_CA_Produit =
DIVIDE(
    [CA],
    CALCULATE([CA], ALL(Dim_Produit)),
    0
)

// Rang produit par CA
Rang_Produit =
RANKX(
    ALL(Dim_Produit[LibelleProduit]),
    [CA],
    ,
    DESC,
    DENSE
)
```

### 2.5 Mesures break-even

```dax
// Seuil de rentabilité mensuel
Seuil_Rentabilite =
DIVIDE(
    [Charges_Fixes],
    1 - DIVIDE([CMV] + [Charges_Variables], [CA], 0),
    BLANK()
)

// Marge de sécurité (€)
Marge_Securite = [CA] - [Seuil_Rentabilite]

// Marge de sécurité (%)
Marge_Securite_Pct =
DIVIDE([Marge_Securite], [CA], 0)
```

---

## 3. Structure du rapport Power BI

### Page 1 — Vue d'ensemble (Overview)

**Disposition :**

```
┌─────────────────────────────────────────────────────────────┐
│  Slicers : [Mois ▼]  [Produit ▼]  [Centre de coût ▼]        │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  KPI Card    │  KPI Card    │  KPI Card    │  KPI Card      │
│  CA Total    │  Marge Brute │  EBIT        │  Taux EBIT     │
│  1 243 762 € │  42,2 %      │  -2 053 €    │  -0,2 %        │
├──────────────┴──────────────┴──────────────┴────────────────┤
│  Graphique courbe : CA mensuel + Marge brute (axe double)   │
├─────────────────────────────┬───────────────────────────────┤
│  Histogramme empilé :       │  Graphique donut :            │
│  Charges fixes vs variables │  Répartition CA par produit   │
│  par mois                   │                               │
└─────────────────────────────┴───────────────────────────────┘
```

**Visuels à créer :**
- 4 × KPI Cards (CA YTD, Marge Brute %, EBIT, Taux EBIT)
- 1 × Graphique en courbes — CA et Marge brute mensuels
- 1 × Histogramme empilé — Charges fixes / variables par mois
- 1 × Graphique en anneau — Répartition CA par produit

### Page 2 — Analyse détaillée

**Disposition :**

```
┌─────────────────────────────────────────────────────────────┐
│  Slicer : [Trimestre ▼]                                     │
├─────────────────────────────┬───────────────────────────────┤
│  Tableau récapitulatif :    │  Graphique barres horizontales│
│  CA / CMV / MB / EBIT       │  EBIT par mois (couleur       │
│  par mois (avec drill-down) │  rouge si négatif)            │
├─────────────────────────────┴───────────────────────────────┤
│  Matrice : CA par produit × mois (heatmap conditionnelle)   │
├─────────────────────────────┬───────────────────────────────┤
│  Graphique courbe :         │  Jauge :                      │
│  Taux de croissance CA MoM  │  CA YTD vs seuil rentabilité  │
└─────────────────────────────┴───────────────────────────────┘
```

**Visuels à créer :**
- 1 × Table/Matrice avec drill-down mois → semaine
- 1 × Graphique barres avec formatage conditionnel (vert/rouge)
- 1 × Matrice heatmap (mise en forme conditionnelle par couleur)
- 1 × Graphique en courbes taux de croissance
- 1 × Jauge break-even

### Page 3 — Centres de coûts

**Disposition :**

```
┌─────────────────────────────────────────────────────────────┐
│  Slicer : [Centre de coût ▼]  [Mois ▼]                      │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  KPI CC1000  │  KPI CC2000  │  KPI CC3000  │  KPI CC4000    │
│  Magasin     │  Logistique  │  Marketing   │  Admin         │
├──────────────┴──────────────┴──────────────┴────────────────┤
│  Histogramme : Total charges par centre de coût × mois      │
├─────────────────────────────┬───────────────────────────────┤
│  Tableau : Détail           │  Treemap : proportion de       │
│  transactions par CC        │  chaque CC dans charges totales│
└─────────────────────────────┴───────────────────────────────┘
```

---

## 4. Formatage et mise en forme

### Palette de couleurs recommandée (sobre, professionnel)

| Élément | Couleur | Code HEX |
|---|---|---|
| En-têtes | Gris foncé | #2F2F2F |
| KPI positif | Vert neutre | #217346 |
| KPI négatif | Rouge | #C00000 |
| Barres CA | Bleu acier | #4472C4 |
| Barres charges | Gris moyen | #808080 |
| Fond rapport | Blanc | #FFFFFF |
| Texte principal | Noir | #000000 |

### Paramètres de page
- Taille : 16:9 (1280 × 720 px)
- Police : Segoe UI 10pt pour les étiquettes
- Padding interne des visuels : 8 px

### KPI Cards — formatage
- Valeur principale : taille 28pt, gras
- Étiquette : taille 11pt, normal
- Indicateur de tendance : flèche avec couleur conditionnelle

---

## 5. Fonctionnalités interactives

### Slicers à configurer
- Slicer **Mois** : style liste, sélection multiple activée
- Slicer **Produit** : style liste déroulante
- Slicer **Centre de coût** : style boutons
- Slicer **Trimestre** : style liste, lié à `Dim_Mois[Trimestre]`

### Drill-down
Sur le graphique CA mensuel, activer le drill-down :
`Mois → Semaine → Jour`  
(nécessite une hiérarchie de dates dans Power BI)

### Interactions entre visuels
- Désactiver le filtre croisé entre le donut produit et l'histogramme charges  
  (`Format → Modifier les interactions → Aucun`)
- Conserver le filtre croisé actif entre slicers et tous les KPI cards

### Bookmarks conseillés
- "Vue annuelle" : aucun slicer mois actif
- "T4 Focus" : slicer trimestre = T4
- "Produits rentables" : filtré sur P01 et P03


---

*Document produit dans le cadre du projet académique — Université Paris Panthéon-Assas — Février 2026*
