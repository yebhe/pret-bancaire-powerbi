# 📁 Données

Les fichiers CSV ne sont **pas inclus** dans ce dépôt. Ils proviennent d'un jeu de données public disponible sur Kaggle.

## Fichiers attendus

Placez ces **10 fichiers** dans un même dossier local :

| Fichier | Contenu |
|---|---|
| `fact_sales.csv` | Ventes |
| `fact_shipment.csv` | Expéditions |
| `fact_production.csv` | Production |
| `fact_procurement.csv` | Achats fournisseurs |
| `fact_inventory.csv` | Stock |
| `dim_date.csv` | Calendrier |
| `dim_product.csv` | Produits |
| `dim_supplier.csv` | Fournisseurs |
| `dim_facility.csv` | Sites |
| `dim_customer.csv` | Clients |

## Brancher les données au rapport

1. Ouvrir le `.pbix` dans Power BI Desktop.
2. **Accueil → Transformer les données → Gérer les paramètres**.
3. Renseigner `Chemin_source` avec le chemin du dossier, **sans antislash final**.
   Exemple : `C:\data\supply_chain`
4. Fermer l'éditeur et cliquer sur **Actualiser**.