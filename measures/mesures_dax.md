# 🧮 Mesures DAX

Le modèle contient **123 mesures** dans la table `_Mesure`. Ce document liste les **mesures de base** de chaque domaine, avec leur formule. Les mesures dérivées suivent toutes le même schéma :

| Suffixe | Rôle | Exemple |
|---|---|---|
| `LY` | Même période, année précédente | `Montant total paye LY` |
| `Var %` | Variation relative vs N-1 | `Montant total paye Var %` |
| `PT` | Écart en points vs N-1, pour les taux | `Taux de recouvrement PT` |
| `var Delta` / `pt Delta` | Texte avec flèche ▲ ▼ ▶ et variation, « Pas de N-1 » si vide | `Montant total paye var Delta` |
| `var Couleur` / `pt Couleur` | Code couleur selon le sens métier | `Montant total paye var Couleur` |
| `Texte` | Valeur formatée pour les cartes | `Taux moyen pret Texte` |

Les codes couleur sont dans le dossier *Technique* : `_Couleur bon` (#1B7F52), `_Couleur mauvais` (#C4362E) et `_couleur neutre` (#4F6A75).

> Pour un indicateur « plus c'est bas, mieux c'est » (retards), la logique de couleur est inversée : une baisse est verte (voir `Montant en retard var Couleur`).

---

## 📐 Schéma N-1

Le N-1 est borné à la dernière date du fait concerné, ce qui évite de comparer une année partielle à une année complète. Aucune mesure n'utilise `TODAY()`.

```dax
Montant total paye LY =
VAR _bornes = EDATE ( [_Derniere date remboursement], -12 )
RETURN
    CALCULATE (
        [Montant total paye],
        SAMEPERIODLASTYEAR ( dim_temps[DT_TEMPS] ),
        dim_temps[DT_TEMPS] <= _bornes
    )
```

```dax
Montant total paye Var % =
VAR n   = [Montant total paye]
VAR n_1 = [Montant total paye LY]
RETURN
    IF ( NOT ISBLANK ( n ) && NOT ISBLANK ( n_1 ), DIVIDE ( n - n_1, n_1 ) )
```

Les dates de borne sont portées par trois mesures techniques : `_Derniere date pret`, `_Derniere date remboursement` et `_Derniere date echeancier`.

---

## 💶 Prêts

| Mesure | Formule |
|---|---|
| `Nb prets` | `DISTINCTCOUNT(fait_pret[ID_PRET])` |
| `Nb clients emprunteurs` | `DISTINCTCOUNT(fait_pret[ID_CLIENT])` |
| `Montant total prete` | `SUM(fait_pret[MONTANT_PRET])` |
| `Montant moyen pret` | `DIVIDE([Montant total prete], [Nb prets])` |
| `Duree moyenne pret` | `AVERAGE(fait_pret[DUREE_MOIS])` |
| `Taux moyen pret` | Taux pondéré par le montant : `DIVIDE(SUMX(fait_pret, MONTANT_PRET * TAUX_INTERET), [Montant total prete])` |

## 📅 Échéancier

| Mesure | Formule |
|---|---|
| `Nb echeances` | `DISTINCTCOUNT(fait_echeancier[ID_ECHEANCE])` |
| `Nb echeances echues` | `Nb echeances` pour les statuts `PART`, `REGL`, `RET` |
| `Part echues %` | `DIVIDE([Nb echeances echues], [Nb echeances])` |
| `Montant total prevu` | `SUM(fait_echeancier[MONTANT_PREVU])` |
| `Interet prevus` | `SUM(fait_echeancier[INTERET_PREVU])` |
| `Montant total prevu echu` | `Montant total prevu` pour les statuts `PART`, `REGL`, `RET` (`KEEPFILTERS` : respecte un filtre de statut posé par l'utilisateur) |

## 💳 Remboursements

Toutes les mesures de paiement **excluent les copies** avec `fait_remboursement[EST_COPIE] = FALSE()` (voir [Limites connues](../README.md#-limites-connues)).

| Mesure | Formule |
|---|---|
| `Montant total paye` | `CALCULATE(SUM(fait_remboursement[MONTANT_PAYE]), EST_COPIE = FALSE())` |
| `Capital total paye` | idem sur `CAPITAL_PAYE` |
| `Interet total paye` | idem sur `INTERET_PAYE` |
| `Nb remboursements` | `CALCULATE(DISTINCTCOUNT(fait_remboursement[ID_REMB]), EST_COPIE = FALSE())` |
| `Montant moyen rembourse` | `DIVIDE([Montant total paye], [Nb remboursements])` |
| `Nb mensualites estimees` | Somme pondérée par la périodicité (`MENS` 1, `TRIM` 3, `SEM` 6, `ANN` 12, autre 1), hors copies |

## ⏱️ Recouvrement et retards

| Mesure | Formule |
|---|---|
| `Taux de recouvrement %` | `DIVIDE([Montant total paye], [Montant total prevu echu])` |
| `Taux de paiement dans les delais` | Montant prévu des échéances `REGL` / `Montant total prevu echu` |
| `Retard echeance echues %` | Nombre d'échéances `RET` / nombre d'échéances `PART`, `REGL`, `RET` |
| `Montant en retard` | `Montant total prevu` pour le statut `RET` |
| `Montant restant a rembourser` | `[Montant total prevu] - [Montant total paye]` |

`Montant en retard` désigne des échéances payées avec retard, pas des impayés. `Montant restant a rembourser` ne porte que sur les 4 premières échéances de chaque prêt, celles de l'échéancier chargé.

## 👥 Clients

| Mesure | Formule |
|---|---|
| `Nb clients actifs` | `CALCULATE(COUNTROWS(dim_client), dim_client[ACTIF] = 1)` |
| `Part emprunteurs %` | Clients emprunteurs actifs / clients actifs |
| `Pret moyen client` | `DIVIDE([Montant total prete], [Nb clients emprunteurs])` |
| `Nb prets par client` | `DIVIDE([Nb prets], [Nb clients emprunteurs])` |

`ACTIF` est un instantané sans historique : `Nb clients actifs LY` vaut volontairement `BLANK()` et ses mesures dérivées affichent « Pas de N-1 ».

---

## 🗂️ Dossiers d'affichage

```
_Mesure
├── Dashboard
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Prets
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Echeancier
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Remboursement
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
├── Clients
│   ├── 1 Base
│   ├── 2 Comparaison N-1
│   └── 3 Affichage carte
└── Technique              (codes couleur)
```