# Système, serveurs, utilisateurs, paramètres

## Qui est là, où, et fait quoi

| Code | Rôle | |
|---|---|---|
| `SM04` | Utilisateurs connectés sur **l'instance locale** | 🔍⚠️ |
| `AL08` | Utilisateurs connectés sur **toutes** les instances | 🔍 |
| `SM51` | Liste des serveurs d'application et leurs services | 🔍 |
| `SM50` / `SM66` | Processus de travail, local / global | 🔍⚠️ |
| `SM12` | Verrous en cours | ⚠️ |
| `SM13` | Mises à jour en erreur | 🔍✏️ |
| `SM21` | Log système | 🔍 |
| `SM02` | Messages système (le bandeau « le système sera arrêté à 20h ») | ✏️ |
| `SM59` | Destinations RFC | 🔍⚠️ |
| `SMGW` | Moniteur du gateway (connexions RFC, programmes enregistrés) | 🔍 |
| `SMICM` | Moniteur ICM (HTTP/HTTPS, ports, traces) | 🔍 |

`SM04` répond à « pourquoi ce document est-il verrouillé ? » : trouver l'utilisateur,
lui demander de sortir. ⚠️ La fonction *Fin de session* déconnecte quelqu'un et lui
fait perdre sa saisie en cours. C'est un dernier recours, et on prévient avant.

## Configuration

| Code | Rôle | |
|---|---|---|
| `RZ10` | Profils système — modification des paramètres, nécessite un redémarrage | ⚠️ |
| `RZ11` | Documentation et valeur courante d'**un** paramètre. Sans risque | 🔍 |
| `RZ20` | Moniteur d'alertes CCMS | 🔍 |
| `RZ03` / `RZ04` / `SM63` | Modes d'exploitation (répartition jour / nuit des processus) | ⚠️ |
| `SCC4` | Mandants : modifiabilité, protection, rôle (production / test) | ⚠️ |
| `SE06` / `SE03` | Modifiabilité du système et des espaces de noms | ⚠️ |
| `SICK` | Contrôle d'installation — à lancer après toute mise à jour système | 🔍 |
| `SLICENSE` | Licences | 🔍 |
| `STRUST` | Certificats et magasins de confiance SSL | ⚠️ |
| `SECSTORE` | Magasin sécurisé (contrôle de cohérence après copie système) | ⚠️ |
| `AL11` | Répertoires du serveur SAP | 🔍 |
| `OS01` / `ST06` | Système d'exploitation, réseau | 🔍 |

`RZ11` est sous-utilisé et très utile : il donne la valeur **actuellement active**
d'un paramètre, sa valeur par défaut, sa documentation et s'il est modifiable
dynamiquement. Avant de discuter d'un timeout ou d'une taille de buffer, vérifier la
valeur réelle plutôt que la valeur supposée.

## Mandants et copies

| Code | Rôle | |
|---|---|---|
| `SCC4` | Créer / paramétrer un mandant | ⚠️ |
| `SCCL` | Copie de mandant locale (même système) | ⚠️ |
| `SCC9` | Copie de mandant à distance | ⚠️ |
| `SCC3` | Log des copies de mandant | 🔍 |
| `SCC1` | Copie d'un ordre de transport entre mandants | ⚠️ |
| `SCC5` | Suppression d'un mandant | ⚠️ |

## Après une copie système ou une restauration

La liste des actions oubliées, qui produisent des symptômes déroutants :

1. `SM59` — les destinations RFC pointent encore vers l'ancien environnement. Une
   recette qui écrit en production, c'est là que ça commence.
2. `SM37` — les jobs copiés se remettent à tourner dans le nouveau système. Un job
   d'envoi de facture qui tourne sur une copie de test envoie de vraies factures.
3. `SCOT` — la configuration mail. Même risque : envoi réel depuis un environnement
   de test.
4. `WE20` / `WE21` — profils partenaires et ports IDoc.
5. `SECSTORE` — les entrées du magasin sécurisé sont chiffrées avec une clé liée au
   système ; après copie, elles doivent être régénérées.
6. `STRUST` — certificats liés au nom d'hôte.
7. `SLICENSE` — nouvelle clé de licence.
8. `SICK` — contrôle d'installation.

Le point 1, 2 et 3 sont les trois qui, ensemble, causent la quasi-totalité des
incidents « le système de test a écrit chez le client ». Ils se traitent avant
d'ouvrir le système aux utilisateurs, pas après.

## Divers utile

| Code | Rôle |
|---|---|
| `SM35` | Sessions batch input |
| `SCOT` | Configuration et moniteur des envois (mail, fax) |
| `SOST` | Statut des envois — « le mail est-il parti ? » |
| `SBWP` | Business Workplace (boîte de réception SAP, workflow) |
| `SWI1` | Instances de workflow — retrouver un workflow bloqué |
| `SWU3` | Contrôle du paramétrage workflow (le premier écran à ouvrir si le workflow ne démarre pas) |
| `STAD` | Enregistrements statistiques détaillés par transaction / utilisateur |
| `SU3` | Ses propres paramètres utilisateur |
| `SE38` + `RSUSR003` | Contrôle des mots de passe des utilisateurs standard (`SAP*`, `DDIC`) |
