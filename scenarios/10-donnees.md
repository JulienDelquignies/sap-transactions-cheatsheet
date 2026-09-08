# Lire, extraire et charger des données

## Lire

| Code | Rôle | |
|---|---|---|
| `SE16N` | Contenu d'une table, avec sélection sur n'importe quel champ | 🔍⚠️ |
| `SE16H` | Variante HANA de `SE16N` : agrégations, jointures, group by | 🔍⚠️ |
| `SE11` | La **structure** : champs, clés, index, relations, aide à la recherche | 🔍 |
| `SQVI` | QuickViewer — une jointure de deux ou trois tables sans développement | 🔍 |
| `SQ01` / `SQ02` / `SQ03` | InfoSet Query : requêtes, InfoSets, groupes d'utilisateurs | ⚠️ |
| `ST05` | Trace SQL : voir quelles tables une transaction lit réellement | 🔍 |
| `DB02` | Taille et croissance des tables | 🔍 |

### Trouver la bonne table

Le problème n'est jamais « comment lire une table », c'est « laquelle ».

1. **F1 sur le champ à l'écran → Informations techniques.** Donne la table et le
   champ. C'est la méthode la plus rapide et la plus fiable.
2. **`ST05`** : activer la trace, faire l'action à l'écran, arrêter, résumer. La liste
   des tables réellement lues, dans l'ordre. Imbattable quand F1 renvoie une structure
   plutôt qu'une table.
3. **`SE84`** → recherche par nom ou par description dans le dictionnaire.
4. Sur S/4HANA, beaucoup de « tables » historiques sont devenues des **vues de
   compatibilité** (`BSEG`, `BSIS`, `BSID`… au-dessus de `ACDOCA`). Lire la vue
   fonctionne, mais pour de l'extraction en volume, viser la table sous-jacente ou une
   vue CDS.

### `SQVI` plutôt qu'un développement

Pour un besoin ponctuel de type « la liste des X avec le libellé de Y », `SQVI`
produit le résultat en dix minutes sans ordre de transport. Limites : mono-utilisateur,
non transportable, et les jointures complexes deviennent vite illisibles. Au-delà,
passer par `SQ01`/`SQ02` (partageable, transportable) ou un vrai rapport.

## Maintenir du paramétrage

| Code | Rôle | |
|---|---|---|
| `SM30` | Maintenance d'une vue / table via sa vue de maintenance | ⚠️ |
| `SM31` | Ancien nom, redirige généralement vers `SM30` | ⚠️ |
| `SE54` | Générateur de vue de maintenance et de dialogue de maintenance | ⚠️ |
| `SM34` | Maintenance de clusters de vues (plusieurs tables liées) | ⚠️ |
| `SPRO` | L'IMG — le paramétrage fonctionnel dans son arborescence documentée | ⚠️ |

⚠️ `SM30` sur une table de paramétrage en production génère un ordre de transport si
la table est configurée pour. Si elle ne l'est pas, la modification reste locale et ne
sera **jamais** rejouée dans les autres environnements — c'est une source classique
d'écarts entre recette et production. Vérifier la classe de livraison de la table
(`SE11` → *Attributs de livraison et de maintenance*) avant de conclure.

## Charger

| Outil | Quand |
|---|---|
| `LSMW` | Reprise de données classique : lecture de fichier, mapping, génération de batch input / d'appels BAPI. Ancien, complet, toujours utilisé. Absent ou déconseillé sur S/4 récent |
| **Migration Cockpit** (`LTMC`, puis `LTMOM` pour l'objet) | Le successeur sur S/4HANA. Modèles XLSX fournis par SAP, mapping guidé |
| `SCAT` / `eCATT` (`SECATT`) | Rejouer des transactions à partir d'un jeu de données. Utile pour les tests autant que pour la reprise |
| `SM35` (batch input) | Le mécanisme sous-jacent de beaucoup de chargements. Voir la [fiche jobs](03-jobs-et-batch.md) |
| BAPI / API OData | La bonne réponse dès qu'un chargement doit être répété ou automatisé |

### La règle qui évite les catastrophes

Un chargement doit être **rejouable sans doublon**. Concrètement : avant d'insérer,
le programme vérifie si l'enregistrement existe déjà, avec une clé métier stable. Sans
cela, la première interruption à mi-parcours transforme la reprise en cauchemar, parce
que personne ne saura dire quelles lignes sont passées.

Corollaire : **toujours charger d'abord un échantillon de 10 lignes**, vérifier le
résultat dans l'application (pas dans la table), puis les 100 suivantes, puis le reste.
Le chargement de 200 000 lignes qui plante à la ligne 3 est une bonne journée. Celui
qui réussit avec un mauvais mapping est une mauvaise année.

## Extraire

Pour sortir de la donnée d'un système SAP proprement :

- `SE16N` → *Liste → Exporter* → tableur / fichier local. Correct pour quelques
  milliers de lignes.
- Un rapport avec sortie ALV : l'export est standard, et il respecte les autorisations.
- `AL11` + un programme d'extraction vers le serveur, si le volume est important : le
  téléchargement local sur des centaines de milliers de lignes échoue ou fige le poste.
- Une vue CDS exposée en OData, si le besoin est récurrent et consommé par un outil
  externe.

⚠️ **Toute extraction est une sortie de données du système.** Sur des données
personnelles ou financières, la question n'est pas technique : qui a le droit de
recevoir ce fichier, où va-t-il être stocké, et combien de temps. Un export vers un
poste de travail échappe ensuite à toute traçabilité SAP. C'est le point sur lequel
les audits sont les plus stricts, et à juste titre.
