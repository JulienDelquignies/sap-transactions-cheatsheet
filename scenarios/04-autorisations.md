# « Il n'a pas les droits »

## Le réflexe : `SU53` — 🔍

**Immédiatement après l'échec**, dans la même session, l'utilisateur lance `SU53`
(ou vous le lancez pour lui : `SU53` → *Autorisations → Autre utilisateur*). L'écran
affiche le dernier contrôle d'autorisation qui a échoué : l'objet, les champs, les
valeurs demandées, et les valeurs que l'utilisateur possède.

Deux limites à connaître, sinon vous allez conclure faux :

- `SU53` ne montre que **le dernier** contrôle échoué de la session. Si l'utilisateur
  a fait autre chose entre-temps, l'information est écrasée.
- Un programme peut échouer sur **plusieurs** objets successivement. Corriger celui
  que montre `SU53` fait apparaître le suivant. C'est normal, pas un signe que le
  correctif n'a pas marché.

## Quand `SU53` ne suffit pas : `STAUTHTRACE` — 🔍

La trace d'autorisations. Elle enregistre **tous** les contrôles, réussis et échoués,
pour un utilisateur donné, y compris pour un utilisateur batch ou RFC — ce que `SU53`
ne sait pas faire.

Méthode :

1. `STAUTHTRACE` → activer la trace, filtrer sur l'utilisateur concerné.
2. Faire reproduire l'action (ou attendre l'exécution du job).
3. Désactiver, évaluer.
4. Filtrer sur `RC ≠ 0` : ce sont les contrôles échoués. `RC=4` = valeurs
   insuffisantes, `RC=12` = objet totalement absent des autorisations.

`ST01` fait la même chose de manière plus générale (traces système), mais pour
l'autorisation `STAUTHTRACE` est plus lisible et supporte le multi-serveur.

## Comprendre et corriger

| Code | Rôle | |
|---|---|---|
| `SU01` | Utilisateur : rôles, profils, verrouillage, mot de passe, données de base | ⚠️ |
| `SU10` | Traitement de masse d'utilisateurs (affecter un rôle à 200 personnes) | ⚠️ |
| `PFCG` | Générateur de rôles — c'est là que se construit le droit | ⚠️ |
| `SUIM` | Système d'information des autorisations : « qui a le droit de faire X ? » | 🔍 |
| `SU24` | Valeurs proposées par transaction — ce qui alimente automatiquement `PFCG` | ⚠️ |
| `SU21` | Objets d'autorisation et leurs champs | 🔍 |
| `SU3` | Ses propres données utilisateur (paramètres, valeurs par défaut) | ✏️ |
| `SU56` | Tampon d'autorisation de l'utilisateur — ce qui est *réellement* actif | 🔍 |
| `SUPC` | Génération de masse des profils après modification de rôles | ⚠️ |
| `PFUD` | Comparaison utilisateur / rôle — la correction du problème ci-dessous | ⚠️ |

## Le piège n°1 : le rôle est là, le droit n'est pas actif

Après une modification dans `PFCG`, il faut **générer le profil** puis faire la
**comparaison utilisateur**. Sans cela, le rôle apparaît bien dans `SU01`, l'onglet
est vert, et l'utilisateur n'a toujours rien.

Symptôme : le rôle est assigné, `SU53` échoue quand même. Vérification : `SU56`
montre le tampon d'autorisation réel — si l'objet n'y est pas, la comparaison n'a
pas été faite.

Correctif : `PFCG` → onglet *Utilisateur* → *Comparaison utilisateur*, ou le job
`PFCG_TIME_DEPENDENCY` (transaction `PFUD`) qui le fait globalement. Ce job doit être
planifié quotidiennement sur tout système sérieux — les affectations à validité
limitée ne se désactivent pas toutes seules sans lui.

## Le piège n°2 : ce n'est pas une autorisation

Trois faux positifs fréquents :

- **Le mandant est fermé** (`SCC4`). L'utilisateur n'a pas « perdu ses droits », le
  système entier est en lecture seule. Message caractéristique : « le mandant n'est
  pas modifiable ».
- **L'objet n'est pas modifiable** dans ce système (`SE03` / paramétrage de
  modifiabilité). Même symptôme côté développement.
- **L'utilisateur est verrouillé ou expiré** (`SU01`) : ce n'est pas un refus
  d'autorisation, c'est un refus de connexion, et le message est différent.

## Méthode propre pour attribuer un droit

Ne jamais partir de l'objet d'autorisation, toujours de la **tâche métier** :

1. Quelle transaction, sur quelles données (société, domaine, type de document) ?
2. Un rôle existant couvre-t-il déjà ce périmètre ? (`SUIM` → *Rôles par
   transaction*.) Réutiliser vaut mieux que créer.
3. Sinon, créer un rôle dérivé plutôt qu'un rôle de plus : même contenu, périmètre
   organisationnel différent. C'est ce qui empêche l'explosion du nombre de rôles.
4. Générer le profil, faire la comparaison utilisateur, **faire retester par
   l'utilisateur**.
5. Documenter la raison métier. Dans deux ans, personne ne saura pourquoi ce rôle
   contient cet objet, et personne n'osera l'enlever.

⚠️ `SAP_ALL` n'est pas une solution de contournement, même « temporairement, le temps
de trouver ». Dans les systèmes audités, cette attribution est tracée et il faudra la
justifier. Utiliser la trace `STAUTHTRACE`, c'est plus rapide de toute façon.
