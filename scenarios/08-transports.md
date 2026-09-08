# Transports

## Les transactions

| Code | Rôle | |
|---|---|---|
| `SE09` | Ordres de transport de type Workbench (développements) | ⚠️ |
| `SE10` | Ordres de type Customizing. En pratique, les deux affichent tout | ⚠️ |
| `SE01` | Organizer étendu — recherche d'ordres, fusion, objets | ⚠️ |
| `STMS` | Système de gestion des transports : les files d'import, l'historique | ⚠️ |
| `SE03` | Outils de l'organizer : recherche d'objets dans les ordres, modifiabilité | ⚠️ |
| `SCC1` | Copie d'un ordre entre mandants du **même** système | ⚠️ |
| `SCC4` | Paramétrage des mandants (modifiabilité, protection) | ⚠️ |
| `SPAM` / `SAINT` | Support packages / installation de composants | ⚠️ |
| `SPAU` / `SPDD` | Ajustement des modifications après montée de version | ⚠️ |

## L'ordre des opérations

1. **Créer** l'ordre — ou plutôt s'en faire proposer un au premier enregistrement
   d'objet. Un ordre = un lot cohérent, pas un fourre-tout de la semaine.
2. **Vérifier le contenu** avant de libérer : `SE09` → développer l'arborescence.
   Chercher ce qui ne devrait pas être là (un objet d'un autre sujet, une table de
   paramétrage modifiée par erreur).
3. **Libérer** les tâches puis l'ordre. L'ordre libéré n'est plus modifiable — c'est
   le point de non-retour.
4. **Importer** via `STMS` → *Vue d'ensemble des imports* → système cible → file
   d'attente → sélectionner l'ordre → importer.
5. **Vérifier le code retour**.

## Les codes retour d'import

| Code | Signification | Faut-il s'inquiéter |
|---|---|---|
| `0` | Import réussi | Non |
| `4` | Avertissement | **Lire le log.** Souvent bénin (objet déjà existant, texte non traduit), parfois pas (un objet non importé faute de dépendance) |
| `8` | Erreur d'import : au moins un objet n'est pas passé | Oui. L'ordre est partiellement importé, c'est le pire état |
| `12` | Annulation | Oui |
| `16` | Erreur grave (souvent au niveau système) | Oui |

Le log détaillé est dans `STMS` → historique d'import → double-clic sur l'ordre.
Un code `4` non lu est la première cause de « ça ne marche pas en production alors que
c'était bon en recette ».

## Les pièges qui coûtent une soirée

### 1. L'ordre des transports

Si l'ordre A crée un élément de données et l'ordre B l'utilise, importer B avant A
donne un code `8`. Le système ne réordonne pas tout seul. Quand plusieurs ordres
partent ensemble, **importer dans l'ordre de libération**, et utiliser l'option
*Import de tous les ordres de la file* plutôt que de les sélectionner un par un.

### 2. Le paramétrage dépendant du mandant

Un développement (Workbench) est indépendant du mandant. Un paramétrage (Customizing)
ne l'est pas. Une table `Z` créée avec le mauvais réglage de dépendance mandant crée
des surprises durables : le paramétrage transporté n'apparaît pas dans le mandant
attendu.

`SCC1` copie un ordre d'un mandant à l'autre **dans le même système** — utile quand
on a paramétré dans le mauvais mandant.

### 3. Les objets qui ne se transportent pas tout seuls

À vérifier explicitement, ils sont oubliés une fois sur deux :

- **Variantes de programmes** (transportables via `SE38` → *Utilitaires → Variantes*,
  ou le programme `RSTRANSP`).
- **Systèmes logiques et modèles de distribution** ALE (`BD54`, `BD64`).
- **Profils partenaires IDoc** (`WE20`) — souvent volontairement non transportés,
  car spécifiques à chaque environnement. À reparamétrer manuellement, en le sachant.
- **Destinations RFC** (`SM59`) — jamais transportées, et c'est voulu : les mots de
  passe et les hôtes diffèrent par environnement.
- **Jobs planifiés** — se recréent dans chaque système.
- **Numéros de plage** (`SNRO`) — les objets se transportent, pas forcément les états.
- **Rôles et autorisations** — transportables, mais avec des règles propres.

### 4. La modifiabilité

Un import qui « ne prend pas » sur un système où les objets ne sont pas modifiables :
`SE03` → *Positionner options de modification système*, et `SCC4` pour le mandant.
En production, ces réglages doivent rester fermés ; s'ils sont ouverts, c'est un
constat d'audit, pas une facilité.

### 5. L'ordre libéré qu'on veut modifier

Impossible. Les options : créer un ordre correctif (la bonne réponse), ou —
⚠️ uniquement en environnement de développement et en sachant ce qu'on fait —
utiliser la fonction de rattachement d'objets à un ordre déjà libéré, ce qui est
généralement une mauvaise idée. Prendre l'habitude de vérifier avant de libérer coûte
moins cher.

## Hygiène

- **Un ordre, un sujet.** Le jour où il faut retirer une fonctionnalité de la
  livraison, un ordre fourre-tout devient un problème insoluble.
- **Description explicite.** `Corrections` n'aide personne. `Interface fournisseurs :
  gestion du rejet en cas de TVA absente` permet de décider six mois plus tard s'il
  faut le rejouer.
- **Ne jamais libérer un vendredi soir** un ordre qui n'a pas été importé en recette
  au moins une fois avec le même contenu.
- **Documenter les actions manuelles post-import** (paramétrage non transportable,
  jobs à planifier, données à initialiser) dans le même document que l'ordre. C'est
  ce qui manque le plus souvent au moment de la mise en production.
