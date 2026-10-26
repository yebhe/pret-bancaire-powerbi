# 📁 Données

Les fichiers CSV sont **inclus** dans ce dépôt, dans le dossier [`banque_p3_last_csv`](banque_p3_last_csv). Ce sont des données de démonstration, fictives, qui ne représentent aucun établissement bancaire réel.

## Origine

Jeu de données fourni par **Martinien Raoul S. ADDA** dans le cadre d'un challenge Power BI : [publication LinkedIn](https://www.linkedin.com/posts/martinien-adda_challenge-power-bi-concevez-un-tableau-share-7484093212773650433-34oX).

## Fichiers

9 fichiers CSV : 3 faits et 6 dimensions.

| Fichier | Contenu | Lignes |
|---|---|---:|
| `fait_pret.csv` | Prêts octroyés | 250 000 |
| `fait_echeancier.csv` | Échéances théoriques | 1 000 000 |
| `fait_remboursement.csv` | Règlements d'échéances | 1 000 000 |
| `dim_client.csv` | Clients et localisation | 10 000 |
| `dim_temps.csv` | Calendrier | 3 121 |
| `dim_type_pret.csv` | Types de prêt | 5 |
| `dim_periodicite_remb.csv` | Périodicités de remboursement | 4 |
| `dim_statut_echeance.csv` | Statuts d'échéance | 5 |
| `dim_mode_paiement.csv` | Modes de paiement | 5 |

## Brancher les données au rapport

1. Ouvrir le `.pbix` dans **Power BI Desktop**.
2. **Accueil → Transformer les données → Gérer les paramètres**.
3. Renseigner `Chemin_source` avec le chemin **complet** du dossier `banque_p3_last_csv`, **sans antislash final**.
   Exemple : `C:\projets\banque-portefeuille-prets\data\banque_p3_last_csv`
4. Fermer l'éditeur et cliquer sur **Actualiser**.

## Points d'attention

- Les montants utilisent le point décimal (locale en-US).
- `fait_remboursement.csv` contient 250 000 lignes en double, neutralisées dans le modèle (voir le [README principal](../README.md#-limites-connues)).