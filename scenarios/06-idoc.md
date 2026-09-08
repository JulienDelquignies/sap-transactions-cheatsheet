# « Un IDoc est en erreur »

L'IDoc reste, après trente ans, le format d'échange le plus répandu du monde SAP.
Il a l'avantage d'être entièrement traçable : chaque message est stocké, avec son
contenu et son historique de statuts.

## Les transactions

| Code | Rôle | |
|---|---|---|
| `WE02` | Liste des IDoc — l'écran de travail standard | 🔍 |
| `WE05` | Idem, présentation en arbre. Question de goût | 🔍 |
| `WE09` | Recherche des IDoc **par contenu de segment** — le seul moyen de retrouver « l'IDoc de la commande 4711 » | 🔍 |
| `BD87` | Moniteur de statuts : voir les erreurs regroupées et **retraiter** | ✏️ |
| `WE19` | Outil de test : rejouer un IDoc existant, ou en fabriquer un | ⚠️ |
| `WE20` | Profils partenaires — qui envoie quoi à qui, avec quel type de message | ⚠️ |
| `WE21` | Ports (fichier, RFC, XML) | ⚠️ |
| `WE30` / `WE31` | Types d'IDoc / segments — la structure | 🔍 |
| `WE60` | Documentation d'un type d'IDoc, champ par champ | 🔍 |
| `WE82` | Lien type de message ↔ type d'IDoc ↔ version | 🔍 |
| `BD54` | Systèmes logiques | ⚠️ |
| `BD64` | Modèle de distribution ALE | ⚠️ |
| `SALE` | Point d'entrée du paramétrage ALE complet | ⚠️ |
| `BD10` / `BD12` | Envoi manuel d'articles / de clients (exemples de programmes d'envoi) | ✏️ |
| `WE47` | Description des codes statut | 🔍 |

## Lire un statut

Le statut est la clé. Ceux qu'on rencontre vraiment :

### Entrant (l'IDoc arrive chez nous)

| Statut | Signification | Action |
|---|---|---|
| `50` | Reçu, en attente de traitement | Normal si récent. Sinon, le traitement n'est pas déclenché : voir le profil partenaire `WE20` (traitement immédiat ou par job ?) |
| `51` | **Erreur applicative** | Le contenu a été refusé par le métier. Corriger la donnée en amont ou le paramétrage, puis retraiter via `BD87` |
| `53` | Traité avec succès | Rien à faire |
| `56` | Erreur de profil partenaire | `WE20` : le partenaire ou le type de message n'est pas déclaré |
| `62` | Transmis à l'application | Étape intermédiaire normale |
| `63` | Erreur de transmission à l'application | Souvent une autorisation ou un module fonction absent |
| `64` | Prêt à être transmis à l'application | En attente du job de traitement (`RBDAPP01`) |
| `65` | Erreur ALE | Paramétrage ALE / modèle de distribution |
| `69` | Modifié / marqué comme traité manuellement | Décision humaine, pas un traitement |

### Sortant (nous envoyons)

| Statut | Signification | Action |
|---|---|---|
| `01` | Généré | Étape initiale |
| `03` | Transmis au système externe | Le port l'a pris en charge — ça ne prouve pas la réception |
| `02` | **Erreur de transmission** | Le port ou la destination RFC. `SM59`, `WE21` |
| `26` | Erreur de syntaxe à la sortie | La structure produite ne respecte pas le type d'IDoc |
| `29` | Erreur du service ALE | Modèle de distribution, `BD64` |
| `30` | Prêt à partir, pas encore envoyé | Le job d'envoi (`RSEOUT00`) n'a pas tourné |
| `12` | Envoi confirmé | Accusé de réception reçu |

`WE47` donne la liste complète et à jour pour votre version.

## Retraiter proprement

`BD87` est l'écran de retraitement : sélection par date / statut / type de message,
regroupement par erreur, puis *Traiter*.

⚠️ Trois précautions, dans cet ordre :

1. **Comprendre l'erreur avant de retraiter.** Un IDoc en `51` retraité sans
   correction repasse en `51`. Vingt fois si on insiste.
2. **Vérifier que le document n'a pas déjà été créé manuellement.** Le scénario
   classique : l'IDoc plante, quelqu'un saisit le document à la main pour débloquer
   la situation, puis on retraite l'IDoc trois jours plus tard et on crée un doublon.
3. **Retraiter par lot, pas tout d'un coup.** Un `BD87` sur 40 000 IDoc en production
   monopolise les processus de travail et fait tomber le reste.

## `WE19`, l'outil à double tranchant

`WE19` permet de reprendre un IDoc existant, d'en modifier le contenu, et de le
rejouer. C'est l'outil idéal pour tester un développement d'entrée sans dépendre du
système émetteur.

⚠️ En production, `WE19` crée de **vrais documents**. Un test « pour voir » avec une
commande réelle crée une vraie commande. Si le système le permet, faites-le en
recette. Si vous devez le faire en production, sachez à l'avance quel document sera
créé et comment l'annuler.

Mode *Traitement en entrée en arrière-plan* pour un test réaliste ; mode *Traitement
en entrée avec débogage* pour poser un point d'arrêt dans le module de traitement.

## Retrouver un IDoc

C'est souvent le vrai problème : on a un numéro de commande client, pas un numéro
d'IDoc.

- `WE09` — recherche par valeur dans un segment. Indiquer le type de segment, le
  champ, la valeur. C'est lent sur de gros volumes mais c'est le seul moyen fiable.
- `WE02` avec une plage de dates serrée et le type de message, si on connaît le
  contexte.
- Depuis le document métier : beaucoup de transactions d'affichage proposent
  *Environnement → Liens ALE* ou un bouton *Flux de documents* qui remonte à l'IDoc.

## Ce qui va mal, et pourquoi

| Symptôme | Cause quasi certaine |
|---|---|
| Les IDoc arrivent en `64` et restent là | Le profil partenaire (`WE20`) est en « traitement par job » et le job `RBDAPP01` n'est pas planifié |
| Les IDoc sortants restent en `30` | Idem à l'envoi : `RSEOUT00` non planifié |
| Statut `56` sur tous les IDoc d'un partenaire | Profil partenaire absent ou type de message non déclaré dans `WE20` |
| Erreurs de segment inconnu après un transport | Version du type d'IDoc différente entre émetteur et récepteur (`WE82`) |
| Ça marchait en recette, pas en production | Systèmes logiques (`BD54`) ou modèle de distribution (`BD64`) non transportés |

## Conseil de conception

Un flux IDoc sans **job de retraitement automatique des erreurs transitoires** et
sans **alerte sur les statuts en erreur** finit toujours par accumuler des milliers
d'IDoc en `51` que plus personne ne regarde. Prévoir la supervision dès la
conception ; c'est le sujet de la [méthodologie interfaces](https://github.com/JulienDelquignies/sap-interface-methodology).
