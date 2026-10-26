# 🧮 Mesures DAX

Le modèle contient environ 150 mesures dans la table `_Mesures`. Ce document liste les **mesures de base** de chaque domaine. Les mesures dérivées suivent toutes le même schéma :

| Suffixe | Rôle | Exemple |
|---|---|---|
| `LY` | Même période, année précédente (`SAMEPERIODLASTYEAR`) | `CA net LY` |
| `Var %` / `Var pt` | Variation relative ou en points vs N-1 | `CA net Var %` |
| `Delta` | Texte avec flèche ▲▼► et variation | `CA net Delta` |
| `Texte` | Valeur formatée (K / M) pour les cartes | `CA net Texte` |
| `Couleur` | Code couleur selon le sens métier | `CA net var Couleur` |

Les codes couleur sont dans le dossier *Technique* : `_Couleur bon` (#2E7D32), `_Couleur mauvais` (#C62828), `_Couleur neutre` (#6B7280), `_Couleur alerte` (#E88A00).

> Pour un indicateur « plus c'est bas, mieux c'est » (coûts, défauts), la logique de couleur est inversée : une baisse est verte.

---

## 💶 CA et Ventes

| Mesure | Définition |
|---|---|
| `CA net` | `SUM(fact_sales[net_revenue])` |
| `CA brut` | `SUM(fact_sales[gross_revenue])` |
| `Remises` | `SUM(fact_sales[discount_amount])` |
| `Cout ventes` | `SUM(fact_sales[total_cost])` |
| `Profit` | `SUM(fact_sales[profit])` |
| `Qté vendues` | `SUM(fact_sales[quantity_sold])` |
| `Nb Commandes` | `DISTINCTCOUNT(fact_sales[order_number])` |
| `Panier moyen` | `DIVIDE([CA net], [Nb Commandes])` |
| `Taux de marge %` | `DIVIDE([Profit], [CA net])` |
| `Valeur cascade` | Valeur par étape (CA brut, remises, coûts, profit) pour le graphique en cascade |
| `Titre Fiche Produit` | Nom du produit sélectionné, sinon sa catégorie |

## 🏭 Production

| Mesure | Définition |
|---|---|
| `Qté produites` | `SUM(fact_production[quantity_produced])` |
| `Unités defectueuses` | `SUM(fact_production[defective_units])` |
| `Taux de defaut` | `DIVIDE(unités défectueuses, quantité produite)` |
| `Nb Lots` | `DISTINCTCOUNT(fact_production[batch_number])` |
| `Cout production` | `SUMX(fact_production, quantité × coût unitaire du produit)` |
| `Ratio prod/vente` | `DIVIDE([Qté produites], [Qté vendues])` |
| `taux de remplissage %` | Production / capacité de la période (voir ci-dessous) |

**Taux de remplissage.** La capacité annuelle est proportionnelle au nombre de mois présents dans le contexte de filtre :

```dax
taux de remplissage % =
VAR _Mois =
    COUNTROWS ( SUMMARIZE ( dim_date, dim_date[year], dim_date[month] ) )
VAR _CapaciteAnnuelle =
    SUM ( dim_facility[annual_capacity] )
VAR _CapacitePeriode =
    _CapaciteAnnuelle / 12 * _Mois
RETURN
    DIVIDE ( [Qté produites], _CapacitePeriode )
```

## 📦 Stock / Stockage

| Mesure | Définition |
|---|---|
| `Stock` | Niveau de stock au dernier snapshot de la période (`LASTNONBLANK`) |
| `Valeur du stock` | Stock × coût unitaire du produit, au dernier snapshot |
| `Stock_valeur_moyen` | Valeur du stock moyenne sur les snapshots de la période |
| `Stock sous seuil` | Nb de produits dont le stock est inférieur au point de commande |
| `Stock critique` | Nb de produits dont le stock est inférieur ou égal au stock de sécurité |
| `Nb Produits Snapshot` | Nb de couples produit × site dans le dernier snapshot |
| `Statut critique` / `Statut sous seuil` / `Statut ok` | Répartition exclusive des produits (critique, sous seuil hors critique, ok) |
| `Rotation du stock` | `DIVIDE(coût des ventes, valeur moyenne du stock)` |
| `Jours de stock` | `DIVIDE(nb de jours, rotation du stock)` |
| `Nb sites stockage` | `DISTINCTCOUNT(dim_facility[facility_id])` |

## 🧾 Approvisionnement

| Mesure | Définition |
|---|---|
| `Qté commandées` | `SUM(fact_procurement[order_quantity])` |
| `Cout commandes` | `SUM(fact_procurement[total_cost])` |
| `Nb Achats` | `DISTINCTCOUNT(fact_procurement[procurement_id])` |
| `Cout moyen par Achat` | `DIVIDE([Cout commandes], [Nb Achats])` |
| `Score qualité moyen` | `AVERAGE(fact_procurement[quality_score])` |
| `Délais moyen approvis` | Moyenne de (clé date de livraison − clé date de commande) sur les lignes livrées |

Les clés de date (`date_key`) sont des entiers séquentiels (1 à 731) : leur différence est donc un nombre de jours.

## 🚚 Expédition / Livraison

| Mesure | Définition |
|---|---|
| `Nb expeditions` | `DISTINCTCOUNT(fact_shipment[shipment_id])` |
| `Qté expediées` | `SUM(fact_shipment[quantity])` |
| `Cout expeditions` | `SUM(fact_shipment[shipping_cost])` |
| `Coût moyen par expedition` | `DIVIDE([Cout expeditions], [Nb expeditions])` |
| `Taux livraison %` | Part des expéditions au statut *Delivered* |
| `Nb retards` | Expéditions au statut *Delayed* |
| `Delais moyen de livraison` | Moyenne de (clé date de livraison − clé date d'expédition) |
| `Cout expédition YTD` | Cumul annuel du coût d'expédition (`TOTALYTD`) |

Le `Taux Livraison Couleur` utilise des seuils : ≥ 90 % vert, ≥ 80 % orange, sinon rouge.

---

## 🗂️ Dossiers d'affichage

```
_Mesures
├── Dashboard
│   ├── CA
│   │   ├── 1 Base
│   │   ├── 2 Comparaison N-1
│   │   └── 3 Affichage carte
│   ├── Commandes
│   │   ├── 1 Base
│   │   ├── 2 Comparaison N-1
│   │   └── 3 Affichage carte
│   ├── Expedition
│   │   ├── 1 Base
│   │   ├── 2 Comparaison N-1
│   │   └── 3 Affichage carte
│   ├── Production
│   │   ├── 1 Base
│   │   ├── 2 Comparaison N-1
│   │   └── 3 Affichage carte
│   ├── Profit
│   │   ├── 1 Base
│   │   └── 2 Comparaison N-1
│   └── Stock
│       ├── 1 Base
│       ├── 2 Comparaison N-1
│       └── 3 Affichage carte
├── Approvis
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Livraison
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Production
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Stockage
│   ├── 1 Base
│   └── 3 Affichage carte
├── Ventes
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
└── Technique              (codes couleur)
```