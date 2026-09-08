# « Mon interface est bloquée »

Le message n'est pas arrivé, ou il est arrivé et rien ne s'est passé. C'est le
scénario le plus fréquent et celui où on perd le plus de temps, parce qu'on regarde
au mauvais endroit.

## D'abord : quelle technologie ?

Avant d'ouvrir quoi que ce soit, répondre à cette question. Les écrans n'ont rien à
voir entre eux.

| Comment ça entre / sort | Où regarder en premier |
|---|---|
| RFC transactionnel (appel asynchrone, `IN BACKGROUND TASK`) | `SM58` |
| RFC en file d'attente (qRFC, ordre garanti) | `SMQ2` (entrant), `SMQ1` (sortant) |
| bgRFC (le remplaçant moderne de tRFC/qRFC) | `SBGRFCMON` |
| IDoc | `WE02` / `BD87` — voir la [fiche IDoc](06-idoc.md) |
| Web service / SOAP entrant ou sortant | `SRT_MONI`, puis `SRT_UTIL` |
| OData / Fiori / SAP Gateway | `/IWFND/ERROR_LOG` |
| SAP PI / PO au milieu | `SXMB_MONI` (côté SAP), puis l'outil PI |
| SAP AIF (Application Interface Framework) | `/AIF/ERR` |
| Fichier déposé / lu sur le serveur | `AL11`, puis le job qui le lit (`SM37`) |

Si vous ne savez pas laquelle : `SM59` liste les destinations RFC et vous dit au
moins avec quoi ce système parle.

## L'enchaînement standard

### 1. `SM58` — 🔍 les tRFC en attente ou en erreur

Le premier réflexe pour tout ce qui est asynchrone. La colonne **Statut** est ce qui
compte :

- **Vide / « Transaction en cours »** : le message est parti, pas encore traité.
  Si ça dure, c'est un problème de ressources côté cible, pas d'interface.
- **« Système cible non joignable » (`CPIC` / `communication failure`)** : le réseau
  ou la destination. Testez la destination dans `SM59` → *Test de connexion*.
- **Un message d'erreur applicatif** : le message est arrivé, le code métier l'a
  refusé. Le blocage n'est plus technique — allez lire le log applicatif (`SLG1`).

Sélection : mettre l'utilisateur à `*` et une plage de dates large. Le filtre par
défaut sur l'utilisateur courant est la première cause de « je ne vois rien ».

✏️ Relancer : *Éditer → Exécuter LUW*. **Attention** : relancer un tRFC ré-exécute
la fonction côté cible. Si le premier appel avait déjà créé un document avant de
planter, vous en créez un deuxième.

### 2. `SMQ1` / `SMQ2` — 🔍 les files qRFC

`SMQ2` = entrant (ce que ce système doit traiter), `SMQ1` = sortant (ce qu'il doit
envoyer). Le point clé du qRFC : **une file est ordonnée**. Un message en erreur en
tête de file bloque tout ce qui suit, même si les suivants sont parfaitement valides.

Statuts utiles :

| Statut | Signification |
|---|---|
| `READY` | prêt, attend un processus |
| `RUNNING` | en cours |
| `SYSFAIL` | erreur système — la file est bloquée, il faut agir |
| `CPICERR` | problème de communication |
| `STOP` | file arrêtée manuellement ou par le système |
| `NOSEND` | enregistré mais pas encore éligible à l'envoi |

Double-clic sur la file → double-clic sur l'entrée → le message d'erreur complet.

✏️ Débloquer : activer la file (*Débloquer / Activer*). ⚠️ Supprimer une entrée
d'une file la fait disparaître définitivement : le message est perdu, pas rejoué.
Ne le faites qu'après avoir noté ce qu'elle contenait et vérifié que la source peut
renvoyer.

`SMQR` (enregistrement des files entrantes) et `SMQS` (planificateur sortant)
expliquent parfois pourquoi une file ne part jamais : elle n'est simplement pas
enregistrée.

### 3. `SBGRFCMON` — 🔍 bgRFC

Sur les systèmes récents, tRFC et qRFC sont remplacés par bgRFC. Le moniteur est
`SBGRFCMON`, la configuration `SBGRFCCONF`. Le piège classique : bgRFC a besoin d'un
**superviseur de destination** configuré ; sans lui, les unités s'accumulent sans
jamais être traitées, et rien dans `SM58` ne le montre.

### 4. `/AIF/ERR` — 🔍✏️ le moniteur AIF

Si l'interface est passée par AIF, c'est ici que se trouve la vraie information :
message par message, avec le contenu, le log d'erreur et le point exact du mapping
qui a échoué.

- `/AIF/ERR` — moniteur des messages et gestion des erreurs. C'est l'écran de travail.
- `/AIF/IFMON` — vue d'ensemble par interface, avec les compteurs. Utile pour
  répondre à « est-ce que ça va globalement ? ».
- `/AIF/CUST` — le paramétrage (structures, mappings, contrôles, actions).
- `/AIF/VMAP` — les tables de correspondance de valeurs.

L'intérêt d'AIF, et la raison pour laquelle on l'utilise : ✏️ on peut **corriger la
donnée dans le message puis rejouer**, sans redemander l'envoi à l'émetteur. C'est
aussi le risque : un rejeu reste un traitement métier. Vérifiez le statut
d'un document avant de rejouer, pas après.

> Les codes de l'espace de noms `/AIF/` varient selon la version d'AIF installée.
> Si l'un d'eux n'existe pas chez vous, cherchez `/AIF/*` dans `SE93`.

### 5. `SXMB_MONI` — 🔍 si PI / PO est dans la chaîne

Le moniteur des messages XML côté SAP. Cherchez le statut : traité avec succès,
en erreur système, en attente. Un message « traité » côté PI et invisible côté
métier signifie que le problème est *après* PI — retour à `SM58` ou `/AIF/ERR` sur le
système cible.

### 6. `SLG1` — 🔍 le log applicatif

La transaction la plus sous-utilisée du lot. Beaucoup de développements et de
modules standard écrivent leurs erreurs dans le log applicatif, où personne ne va
regarder. Sélection par objet / sous-objet et par date. Si vous ne connaissez pas
l'objet, laissez-le vide et filtrez sur l'heure de l'incident.

### 7. `SM21` et `ST22` — 🔍 le filet de sécurité

Si rien n'a rien donné : `SM21` (log système) sur la fenêtre horaire de l'incident,
et `ST22` (dumps ABAP). Un dump côté serveur d'application explique beaucoup de
« l'interface ne répond plus » qui ressemblaient à du réseau.

## Les autres transactions du domaine

| Code | Rôle | |
|---|---|---|
| `SM59` | Destinations RFC : créer, tester, voir les paramètres de connexion | 🔍✏️ |
| `AL11` | Répertoires du serveur SAP — vérifier qu'un fichier est bien arrivé | 🔍 |
| `SRT_MONI` | Moniteur des messages de services web | 🔍 |
| `SRT_UTIL` | Utilitaire / traces des services web — plus verbeux que `SRT_MONI` | 🔍 |
| `SOAMANAGER` | Configuration des services web et des consommateurs (interface web) | ✏️ |
| `/IWFND/ERROR_LOG` | Log d'erreurs SAP Gateway (OData, Fiori) | 🔍 |
| `/IWFND/MAINT_SERVICE` | Activation et maintenance des services OData | ✏️ |
| `SMICM` | Moniteur ICM — le serveur HTTP interne. Voir les ports, les traces | 🔍 |
| `STRUST` | Certificats SSL — la cause de la moitié des « connexion refusée » en HTTPS | ⚠️ |
| `SM12` | Verrous — un verrou orphelin bloque un traitement sans erreur visible | ⚠️ |
| `SM13` | Mises à jour en erreur — le traitement a réussi puis la mise à jour a échoué | 🔍✏️ |

## Le piège n°1

`SM13`. Un traitement peut se terminer « avec succès » côté appelant et échouer
ensuite dans la tâche de mise à jour asynchrone. Résultat : aucune erreur nulle part
sauf dans `SM13`, et le document n'existe pas. Quand tout a l'air vert et que la
donnée n'est pas là, regardez `SM13` avant de conclure au problème réseau.
