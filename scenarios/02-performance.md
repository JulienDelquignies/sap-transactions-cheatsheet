# « C'est lent »

Le mot « lent » recouvre cinq problèmes différents. La première étape n'est jamais
d'ouvrir un moniteur, c'est de choisir lequel.

## Qualifier avant de mesurer

| Ce que dit l'utilisateur | Ce que ça veut dire | Où aller |
|---|---|---|
| « Tout le système est lent » | ressources serveur ou base saturées | `SM66`, `ST06`, `ST04` |
| « Cette transaction est lente » | un programme, une requête | `ST05` puis `SAT` |
| « C'est lent depuis lundi » | un changement — transport, volume, statistiques | `ST03N` en comparaison, `STMS` |
| « Ça bloque puis ça repart » | verrou ou attente d'un système tiers | `SM12`, `SM50` (statut) |
| « Le job dure de plus en plus » | volume croissant, ou index manquant | `SM37` (durées), `ST05` |

## Vue d'ensemble

### `SM66` — 🔍 processus de travail, tous serveurs

L'écran à ouvrir en premier quand « tout est lent ». Il montre, en direct, ce que
fait chaque processus sur chaque instance. Ce qu'on cherche :

- Beaucoup de processus en statut **`Stopped`** avec le motif `CPIC` / `RFC` :
  le système attend un partenaire externe. Le problème n'est pas chez vous.
- Beaucoup de processus sur la **même table** : contention, ou un traitement de masse
  lancé par quelqu'un.
- **Zéro processus DIA libre** : plus personne ne peut travailler. Regarder la colonne
  utilisateur — c'est souvent une seule personne ou un seul job qui les monopolise.

`SM50` fait la même chose pour l'instance locale uniquement, avec plus d'actions
possibles (⚠️ *Annuler sans core* pour tuer un processus — dernier recours, en
sachant que ça peut laisser des verrous derrière).

### `ST03N` — 🔍 statistiques de charge

La vue historique. Utile pour deux questions précises :

- « Quelles sont les transactions les plus coûteuses ? » → *Profil de charge →
  Transaction profile*, trier par temps de réponse total.
- « Est-ce que c'était mieux avant ? » → comparer la même journée de la semaine
  précédente. C'est la seule preuve objective d'une dégradation.

Les temps qui comptent : **temps de réponse = temps CPU + temps base + temps
d'attente**. Si le temps base domine, c'est la base. Si le temps d'attente domine,
c'est la saturation. Si le CPU domine, c'est du code.

### `ST06` / `ST06N` — 🔍 système d'exploitation

CPU, mémoire, disque, réseau du serveur. À regarder avant de conclure quoi que ce
soit sur SAP : un serveur qui swappe rend tout lent, et aucune optimisation ABAP n'y
changera rien.

### `ST04` — 🔍 base de données

Ratios de cache, requêtes coûteuses, attentes. Sur HANA, l'équivalent utile est plutôt
`HDBADMIN` ou le SAP HANA Cockpit ; `ST04` reste un point d'entrée.

### `ST02` — 🔍 buffers

Les colonnes rouges (« swaps ») indiquent des buffers trop petits : le système
recharge en permanence ce qu'il devrait garder en mémoire. Des milliers de swaps sur
le buffer de programmes ou de tables génériques dégradent tout le système, et se
corrigent par du paramétrage (`RZ10`), pas par du code.

## Descendre au programme

### `ST05` — 🔍 la trace, l'outil décisif

Activer la trace SQL, lancer le traitement, désactiver, afficher.

Ce qu'on y cherche, dans l'ordre :

1. **La même requête répétée des milliers de fois** — un `SELECT` dans une boucle.
   C'est la cause n°1 des lenteurs ABAP, et la plus facile à corriger.
2. **Une requête longue** — le temps est dans la colonne durée. Cliquer dessus →
   *Explain* pour voir si un index est utilisé.
3. **Un accès sans clé** — la liste des champs de la clause `WHERE` ne correspond à
   aucun index. `SE11` sur la table → *Index* pour vérifier.

Astuce : la fonction *Résumer trace* (Summarize) transforme 40 000 lignes illisibles
en un classement par requête. Toujours commencer par là.

### `SAT` — 🔍 analyse d'exécution ABAP

Successeur de `SE30`. Répond à « où le temps est-il passé dans **mon** code », par
opposition à `ST05` qui répond « qu'est-ce qui a été demandé à la base ».

À utiliser quand `ST05` ne montre rien d'anormal : le temps est alors dans du calcul,
des boucles imbriquées ou du tri.

### `SE30`

L'ancien nom de `SAT`. Existe encore sur les systèmes plus anciens.

## Les erreurs classiques de diagnostic

- **Optimiser sans mesurer.** Le point lent n'est presque jamais celui qu'on croit.
  `ST05` d'abord, toujours.
- **Confondre lent et bloqué.** Un traitement en attente de verrou (`SM12`) n'est pas
  lent, il attend. Optimiser le code ne changera rien.
- **Mesurer en production aux heures creuses.** Le problème est souvent la
  concurrence, pas le programme.
- **Oublier les statistiques de la base.** Après un chargement de masse, un plan
  d'exécution peut devenir absurde tant que les statistiques ne sont pas rafraîchies.
- **Regarder un seul serveur d'application.** `SM50` ment par omission dès qu'il y a
  plusieurs instances. Utiliser `SM66`.

## Récapitulatif

| Code | Portée | |
|---|---|---|
| `SM50` | Processus, instance locale | 🔍⚠️ |
| `SM66` | Processus, tout le système | 🔍 |
| `ST03N` | Statistiques de charge, historique | 🔍 |
| `ST02` | Buffers mémoire | 🔍 |
| `ST04` | Base de données | 🔍 |
| `ST06` | Système d'exploitation | 🔍 |
| `ST05` | Trace SQL / RFC / buffer / HTTP | 🔍 |
| `SAT` | Analyse d'exécution ABAP | 🔍 |
| `DB02` | Espace disque, croissance des tables, index manquants | 🔍 |
| `DB12` | Sauvegardes de la base | 🔍 |
| `RZ20` | Moniteur d'alertes CCMS | 🔍 |
| `SM12` | Verrous | ⚠️ |
| `RZ10` / `RZ11` | Profils système / paramètres individuels | ⚠️ |
