# Suivi de la Performance Financière — LuxMode Retail

**Auteur** : LE Quang Long  
**Numéro étudiant** : 2400145  
**Établissement** : Université Paris Panthéon-Assas  
**Date** : Février 2026  

---

## Présentation du projet

Ce projet académique simule le suivi de la performance financière d'une entreprise fictive de retail : **LuxMode Retail**, un magasin de vêtements multi-produits basé à Paris. L'analyse couvre 12 mois (janvier–décembre 2025) et mobilise trois outils complémentaires : simulation SAP FI/CO, modélisation Excel, et visualisation Power BI.

**Objectifs pédagogiques :**
- Comprendre la logique de comptabilité analytique dans SAP (centres de coûts, imputations)
- Construire un tableau de bord de contrôle de gestion sous Excel
- Développer un dashboard interactif Power BI avec KPI dynamiques
- Formuler des recommandations financières argumentées

---

## Structure du projet

```
projet_retail/
│
├── README.md                    ← Ce fichier
├── donnees_simulation.csv       ← Jeu de données SAP-like (transactions 2025)
├── analyse_financiere.xlsx      ← Tableau de bord Excel (marges, ratios, break-even)
├── powerbi_dax_measures.md      ← Documentation Power BI : mesures DAX et design
└── rapport_final.md             ← Analyse de performance et recommandations
```

---

## Entreprise fictive : LuxMode Retail

| Paramètre | Valeur |
|---|---|
| Secteur | Retail – Prêt-à-porter |
| Localisation | Paris, France |
| Effectif simulé | 18 salariés |
| Périmètre d'analyse | Janvier – Décembre 2025 |
| Devise | EUR (€) |

**Gamme produits :**

| Code | Catégorie |
|---|---|
| P01 | Hauts (t-shirts, chemises, pulls) |
| P02 | Pantalons & jeans |
| P03 | Robes & jupes |
| P04 | Accessoires (ceintures, sacs, bijoux) |

**Centres de coûts (SAP CO) :**

| Centre | Libellé |
|---|---|
| CC1000 | Magasin (exploitation directe) |
| CC2000 | Logistique & approvisionnement |
| CC3000 | Marketing & communication |
| CC4000 | Administration & RH |

---

## Module 1 — Simulation SAP FI/CO

**Fichier :** `donnees_simulation.csv`

Le fichier contient 240 lignes de transactions simulant les écritures comptables SAP sur 12 mois. Chaque ligne représente une pièce comptable avec les champs suivants :

| Champ | Description |
|---|---|
| `date_piece` | Date de la transaction (format YYYY-MM-DD) |
| `mois` | Mois (1–12) |
| `type_piece` | Type SAP : FA (Facture achat), FV (Facture vente), SA (Écriture analytique) |
| `centre_cout` | Code centre de coût (CC1000–CC4000) |
| `produit` | Code produit (P01–P04) |
| `compte_gl` | Compte du grand livre SAP (ex. 701000 = Ventes, 601000 = Achats) |
| `libelle` | Description de la transaction |
| `debit` | Montant au débit (€) |
| `credit` | Montant au crédit (€) |
| `montant_net` | Montant net (crédit – débit) |

**Comptes GL simulés :**

| Compte | Nature |
|---|---|
| 701000 | Chiffre d'affaires ventes |
| 601000 | Achats marchandises |
| 641000 | Charges de personnel |
| 613000 | Loyer & charges locatives |
| 623000 | Publicité & marketing |
| 624000 | Transport & logistique |
| 627000 | Frais bancaires & divers |

---

## Module 2 — Analyse Excel

**Fichier :** `analyse_financiere.xlsx`

Le classeur contient trois feuilles :

### Feuille 1 : `Donnees`
Données brutes importées, structurées par mois et produit.

### Feuille 2 : `Tableau_de_Bord`
Tableau de bord mensuel avec les indicateurs suivants :

| Indicateur | Formule |
|---|---|
| Chiffre d'affaires (CA) | Somme des ventes mensuelles |
| Coût des marchandises vendues (CMV) | Somme des achats imputés aux ventes |
| Marge brute | CA − CMV |
| Taux de marge brute | Marge brute / CA |
| Charges fixes | Loyer + Salaires + Frais admin |
| Charges variables | Logistique + Marketing variable |
| Résultat d'exploitation (EBIT) | Marge brute − Charges fixes − Charges variables |
| Taux de croissance CA | (CA_n − CA_n-1) / CA_n-1 |

### Feuille 3 : `Analyse_Ratios`
- Ratios de rentabilité (marge nette, ROE simulé)
- Seuil de rentabilité (break-even) mensuel et annuel
- Analyse coûts fixes vs variables
- Comparaison budget vs réel

**Formules Excel utilisées :**
```
SOMME.SI.ENS, XLOOKUP, SI, TAUX.CROISSANCE, MAX, MIN, MOYENNE, NB.SI
```

---

## Module 3 — Power BI Dashboard

**Fichier :** `powerbi_dax_measures.md`

Documentation complète du dashboard Power BI incluant :
- Architecture du modèle de données (tables et relations)
- Toutes les mesures DAX avec explication
- Instructions de construction du rapport (visuels, slicers, drill-down)
- Captures d'écran schématiques de la mise en page

**KPI principaux du dashboard :**
- CA Total (YTD) avec variation vs N-1
- Marge brute en % avec jauge cible
- Résultat d'exploitation mensuel
- Top produits par rentabilité
- Heatmap charges par centre de coût

---

## Module 4 — Rapport d'analyse

**Fichier :** `rapport_final.md`

Synthèse analytique incluant :
- Diagnostic de la performance 2025
- Identification des produits et périodes critiques
- Recommandations opérationnelles chiffrées

---

## Prérequis techniques

| Outil | Version recommandée | Usage |
|---|---|---|
| Microsoft Excel | 2019 / Microsoft 365 | Analyse financière |
| Power BI Desktop | Dernière version (gratuit) | Dashboard interactif |
| Python 3.x | 3.10+ | Génération des données (optionnel) |
| SAP S/4HANA | Simulation uniquement | Référence comptable |

---

## Instructions d'utilisation

1. **Données** : Ouvrir `donnees_simulation.csv` — données prêtes à importer dans Excel ou Power BI
2. **Excel** : Ouvrir `analyse_financiere.xlsx` — toutes les formules sont dynamiques
3. **Power BI** : Suivre `powerbi_dax_measures.md` pour reconstruire le dashboard depuis le CSV
4. **Rapport** : Lire `rapport_final.md` pour l'interprétation et les recommandations

---

## Remarques méthodologiques

- Les données financières sont entièrement fictives et générées à des fins pédagogiques uniquement
- Les montants sont calibrés pour refléter une PME retail française réaliste (CA annuel ~1,3 M€)
- La saisonnalité (soldes d'été, fêtes de fin d'année) est intégrée dans les données
- Les centres de coûts suivent la nomenclature standard SAP CO (plan comptable français PCG)

---

*Projet réalisé dans le cadre du cursus Finance-Contrôle de Gestion — Université Paris Panthéon-Assas — Février 2026*
