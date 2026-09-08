# Aide-mémoire des transactions SAP — classées par problème réel

Les listes de transactions SAP sont toujours triées par ordre alphabétique. C'est
inutile : quand une interface est bloquée à 18h, on ne cherche pas la lettre S, on
cherche « par où je commence ».

Ce dépôt classe les transactions par **symptôme** et par **tâche**. Chaque fiche
donne l'ordre dans lequel regarder, ce que chaque écran répond, et ce qu'il ne
répond pas.

**Pour qui :** consultants et développeurs SAP (ECC et S/4HANA), en particulier
côté intégration / interfaces, et toute personne qui reprend un système qu'elle n'a
pas construit.

---

## Par où commencer

| Ma situation | Fiche |
|---|---|
| « Mon interface ne passe plus, les données ne sont pas arrivées » | [Interface bloquée](scenarios/01-interface-bloquee.md) |
| « Ça a dumpé / il y a une erreur et je ne sais pas où » | [Chercher une erreur](scenarios/02-chercher-une-erreur.md) |
| « C'est lent, on ne sait pas pourquoi » | [Performance](scenarios/03-performance.md) |
| « Un job n'a pas tourné / a planté » | [Jobs et batch](scenarios/04-jobs-et-batch.md) |
| « Il n'a pas les droits » | [Autorisations](scenarios/05-autorisations.md) |
| « Un IDoc est en erreur » | [IDoc](scenarios/06-idoc.md) |
| « Je dois développer / comprendre un objet » | [Développement](scenarios/07-developpement.md) |
| « Je dois transporter » | [Transports](scenarios/08-transports.md) |
| « L'édition n'est pas sortie » | [Spool et impression](scenarios/09-spool-et-impression.md) |
| « Qui est connecté, quel serveur, quel paramètre » | [Système et utilisateurs](scenarios/10-systeme-et-utilisateurs.md) |
| « Je dois lire ou charger des données » | [Données](scenarios/11-donnees.md) |

Deux annexes :

- **[Le minimum vital](minimum-vital.md)** — les 15 transactions à connaître par cœur,
  et pourquoi celles-là.
- **[Index alphabétique](INDEX.md)** — pour quand vous avez le code et pas le sens.

---

## Conventions de lecture

- Les codes sont notés en majuscules : `SM58`, `/AIF/ERR`.
- 🔍 = consultation, sans effet. ✏️ = modifie des données ou relance un traitement.
  ⚠️ = à ne pas faire en production sans savoir ce qu'on fait.
- « ECC / S/4 » : quand une transaction change de nom ou disparaît en S/4HANA,
  c'est indiqué.

## Avertissements

- **Aucune donnée d'aucun client ne figure ici.** Pas de nom d'objet Z, pas de
  nom de système, pas de copie d'écran, pas de volumétrie. Ce dépôt ne contient que
  du savoir standard SAP.
- Les transactions listées sont celles du standard. Selon la version, le
  paramétrage et les rôles, certaines peuvent être absentes ou renommées —
  notamment dans l'espace de noms `/AIF/`, qui varie sensiblement d'une version
  d'Application Interface Framework à l'autre.
- Une transaction marquée ✏️ ou ⚠️ peut relancer un traitement métier. En
  production, la question n'est jamais « est-ce que j'ai le droit », c'est « est-ce
  que ça va créer un doublon ».

## Contribuer

Une transaction manquante, un enchaînement plus efficace, une correction de version :
les issues et les PR sont bienvenues. Une règle : **rien de spécifique à un client**.

## Licence

MIT — voir [LICENSE](LICENSE).
