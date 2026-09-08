# « Il y a une erreur et je ne sais pas où »

Quatre journaux différents, qui ne se recouvrent pas. Une erreur peut n'apparaître
que dans un seul. L'ordre ci-dessous va du plus probable au moins probable.

## 1. `ST22` — 🔍 les dumps ABAP

Le programme s'est arrêté brutalement. Sélection par date, utilisateur, ou
`*` partout si vous ne savez pas.

Ce qu'il faut lire, dans l'ordre, et pas la totalité du dump :

1. **Le nom de l'exception** (en haut) — c'est la nature du problème.
2. **« Que s'est-il passé ? » / « Que pouvez-vous faire ? »** — SAP y explique
   souvent la cause exacte.
3. **La ligne source** — le code fautif, avec son contexte.
4. **« Informations sur l'appel »** — la pile d'appel. C'est ce qui vous dit d'où ça
   venait.

Les exceptions les plus fréquentes et ce qu'elles veulent vraiment dire :

| Exception | Cause réelle, presque toujours |
|---|---|
| `CX_SY_OPEN_SQL_DB` / `DBIF_RSQL_*` | requête invalide, ou table lock, ou dépassement de la taille d'un `FOR ALL ENTRIES` |
| `CONVT_NO_NUMBER` | une donnée texte là où le code attendait un nombre — typiquement un fichier d'entrée mal formé |
| `TIME_OUT` | le programme a dépassé `rdisp/max_wprun_time` en dialogue. À faire tourner en batch, ou à optimiser |
| `MESSAGE_TYPE_X` | erreur volontaire du standard : le message SAP associé est la vraie information |
| `CALL_FUNCTION_REMOTE_ERROR` | l'erreur est **sur le système distant**, pas ici |
| `GETWA_NOT_ASSIGNED` | accès à une ligne de table interne inexistante (index hors bornes) |
| `OBJECTS_OBJREF_NOT_ASSIGNED` | référence objet non instanciée |
| `RAISE_EXCEPTION` | exception classique non traitée par l'appelant |
| `SYSTEM_NO_ROLL` / `TSV_TNEW_PAGE_ALLOC_FAILED` | mémoire épuisée — souvent une table interne qui grossit sans limite |

⚠️ `ST22` a une rétention limitée (paramètre `rdisp/max_alt_modes`… non, en pratique
la table `SNAP` est purgée par le job `RSSNAPDL`). Un dump d'il y a trois semaines
peut avoir disparu. Si un dump compte, exportez-le tout de suite : *Dump ABAP →
Sauvegarder/Envoyer*.

## 2. `SM21` — 🔍 le log système

Le journal du serveur d'application lui-même. On y voit ce que `ST22` ne montre pas :
verrous, problèmes de base, arrêts et démarrages, saturations de processus.

Réglages qui changent tout :
- Passer en affichage **« toutes les instances distantes »** — par défaut vous ne
  voyez que le serveur auquel vous êtes connecté, et l'incident était sur un autre.
- Cadrer la fenêtre horaire serrée autour de l'incident, sinon c'est illisible.

## 3. `SLG1` — 🔍 le log applicatif

Là où le code métier écrit ses erreurs *fonctionnelles*. Un traitement peut rejeter
1 200 lignes sur 1 300 sans provoquer un seul dump ni une seule ligne de `SM21`.
Si l'utilisateur dit « ça n'a pas marché » alors que tous les voyants techniques
sont verts, c'est presque toujours ici.

`SLG2` sert à purger les vieux logs (⚠️ en production, uniquement via un job planifié).

## 4. `SM13` — 🔍✏️ les mises à jour terminées en erreur

**Le piège que ce journal résout.** Un traitement peut se terminer « avec succès »
côté appelant et échouer ensuite dans la tâche de mise à jour asynchrone. Résultat :
aucune erreur nulle part sauf dans `SM13`, et le document n'existe pas. Quand tout a
l'air vert et que la donnée n'est pas là, regardez `SM13` avant de conclure au
problème réseau.

## Aller plus loin

| Code | Quand | |
|---|---|---|
| `ST05` | Trace SQL / RFC / buffer / HTTP. Pour voir *ce que le programme fait vraiment* | 🔍 |
| `STAUTHTRACE` | Trace d'autorisations (remplace `ST01` pour ce besoin) | 🔍 |
| `SM58` | Appels RFC asynchrones en attente ou en erreur — le message d'erreur y est lisible | 🔍✏️ |
| `SMQ1` / `SMQ2` | Files d'attente qRFC sortantes / entrantes. Une entrée en erreur bloque tout ce qui suit | ⚠️ |
| `SM59` | Destinations RFC — le *Test de connexion* tranche en dix secondes | 🔍⚠️ |
| `SAT` | Analyse d'exécution ABAP (successeur de `SE30`) — où le temps est passé | 🔍 |
| `SM12` | Verrous en cours. Un verrou orphelin = un traitement figé sans erreur | ⚠️ |
| `SM50` / `SM66` | Processus de travail : voir en direct ce qui tourne et sur quoi ça bloque | 🔍 |
| `SLGD` | Suppression de logs applicatifs (variante de `SLG2`) | ⚠️ |

## Méthode, quand on n'a rien

1. **Obtenir l'heure exacte** et l'utilisateur. « Ce matin » ne suffit pas ; une
   fenêtre de 10 minutes fait la différence entre 4 lignes et 4 000.
2. Passer les quatre journaux dans l'ordre : `ST22`, `SM21`, `SLG1`, `SM13`.
3. Si toujours rien : le traitement n'a peut-être jamais démarré. Vérifier `SM37`
   (le job existe-t-il, a-t-il tourné ?) et `SM58` / `SMQ2` (le message est-il
   seulement arrivé ?).
4. Si le traitement a bien tourné et n'a rien fait : c'est une sélection vide, pas
   une erreur. Rejouer le même programme avec les mêmes paramètres et un `ST05` actif.
5. Si le message d'erreur est connu mais son origine non : point d'arrêt sur
   l'instruction `MESSAGE` dans le débogueur. Voir la fiche
   [Débogage](06-debogage-et-edition-de-table.md).
