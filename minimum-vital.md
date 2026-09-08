# Le minimum vital : 15 transactions

Si vous n'en retenez que quinze, celles-ci. Elles couvrent, à elles seules, la grande
majorité des situations où on ne sait pas par où commencer.

| # | Code | Ce qu'elle répond | Pourquoi celle-là |
|---|---|---|---|
| 1 | `SE16N` | « Qu'y a-t-il vraiment dans cette table ? » | Le seul moyen de vérifier un fait plutôt que de croire un écran |
| 2 | `SE11` | « À quoi ressemble cette table / ce champ ? » | Structure, clés, index, et le bouton *Où utilisé* |
| 3 | `SE38` | Lancer, lire, déboguer un programme | Le point d'entrée de tout le code |
| 4 | `SE93` | « Quel programme se cache derrière cette transaction ? » | Fait le lien entre l'écran de l'utilisateur et le code |
| 5 | `ST22` | « Ça a dumpé ? » | Le premier journal, celui qui donne la réponse une fois sur deux |
| 6 | `SM21` | « Que s'est-il passé sur le serveur à cette heure-là ? » | Ce que `ST22` ne montre pas |
| 7 | `SLG1` | « Le métier a-t-il rejeté quelque chose ? » | Le journal que personne ne regarde, et qui contient la vraie erreur |
| 8 | `SM37` | « Le job a-t-il tourné, et qu'a-t-il dit ? » | Presque tout ce qui compte tourne en batch |
| 9 | `SM58` | « Mon appel asynchrone est-il passé ? » | Le premier écran de toute interface bloquée |
| 10 | `SMQ2` | « Ma file d'attente est-elle bloquée ? » | Une seule entrée en erreur bloque tout le reste |
| 11 | `SM59` | « Ce système parle-t-il encore à l'autre ? » | Le test de connexion tranche en dix secondes |
| 12 | `WE02` | « Où est passé mon IDoc ? » | L'échange le plus répandu du monde SAP |
| 13 | `SU53` | « Est-ce un problème de droits ? » | À lancer immédiatement après l'échec |
| 14 | `ST05` | « Que fait vraiment ce programme ? » | La seule mesure objective en performance |
| 15 | `SE09` | « Qu'y a-t-il dans ce transport ? » | À regarder avant de libérer, pas après l'import |

## Les trois raccourcis qui valent une transaction

- **`/n` + code** : ouvrir une transaction en abandonnant l'écran courant.
  **`/o` + code** : l'ouvrir dans une nouvelle session. **`/h`** : activer le
  débogueur pour l'action suivante.
- **F1 sur un champ → Informations techniques** : le nom réel de la table et du champ
  derrière l'écran. Le réflexe le plus rentable de tout SAP.
- **Système → Statut** : le programme, la transaction, le mandant, la version, le
  serveur. Répond à « où suis-je exactement » avant de faire une bêtise dans le
  mauvais système.

## Deux habitudes qui font la différence

**Avant d'agir, savoir où on est.** *Système → Statut*. Le nombre d'incidents causés
par une action juste faite dans le mauvais mandant est considérable.

**Après un échec, noter l'heure exacte.** Tous les journaux se cherchent par plage
horaire. Une fenêtre de dix minutes fait la différence entre quatre lignes à lire et
quatre mille.
