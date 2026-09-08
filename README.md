# Aide-mémoire des transactions SAP — classées par problème réel

Les listes de transactions SAP sont toujours triées par ordre alphabétique. C'est
inutile : quand un traitement plante à 18h, on ne cherche pas la lettre S, on cherche
« par où je commence ».

Ce dépôt classe les transactions par **symptôme** et par **tâche**. Chaque fiche
donne l'ordre dans lequel regarder, ce que chaque écran répond, et ce qu'il ne
répond pas.

**Pour qui :** consultants et développeurs SAP (ECC et S/4HANA), et toute personne qui
reprend un système qu'elle n'a pas construit et doit y trouver son chemin.

---

## Par où commencer

| Ma situation | Fiche |
|---|---|
| « Ça a dumpé / il y a une erreur et je ne sais pas où » | [Chercher une erreur](scenarios/01-chercher-une-erreur.md) |
| « C'est lent, on ne sait pas pourquoi » | [Performance](scenarios/02-performance.md) |
| « Un job n'a pas tourné / a planté » | [Jobs et batch](scenarios/03-jobs-et-batch.md) |
| « Il n'a pas les droits » | [Autorisations](scenarios/04-autorisations.md) |
| « Je dois développer / comprendre un objet » | [Développement](scenarios/05-developpement.md) |
| « Je dois déboguer, ou regarder ce qu'il y a vraiment dans la table » | [Débogage et édition de table](scenarios/06-debogage-et-edition-de-table.md) |
| « Je dois transporter » | [Transports](scenarios/07-transports.md) |
| « L'édition n'est pas sortie » | [Spool et impression](scenarios/08-spool-et-impression.md) |
| « Qui est connecté, quel serveur, quel paramètre » | [Système et utilisateurs](scenarios/09-systeme-et-utilisateurs.md) |
| « Je dois lire ou charger des données » | [Données](scenarios/10-donnees.md) |

Deux annexes :

- **[Le minimum vital](minimum-vital.md)** — les transactions à connaître par cœur,
  et pourquoi celles-là.
- **[Index alphabétique](INDEX.md)** — pour quand vous avez le code et pas le sens.

---

## Conventions de lecture

- Les codes sont notés en majuscules : `SM58`, `ST22`.
- 🔍 = consultation, sans effet. ✏️ = modifie des données ou relance un traitement.
  ⚠️ = à ne pas faire en production sans savoir ce qu'on fait.
- « ECC / S/4 » : quand une transaction change de nom ou disparaît en S/4HANA,
  c'est indiqué.

## Avertissements

- **Aucune donnée d'aucun client ne figure ici.** Pas de nom d'objet Z, pas de
  nom de système, pas de copie d'écran, pas de volumétrie. Ce dépôt ne contient que
  du savoir standard SAP.
- Les transactions listées sont celles du standard. Selon la version, le
  paramétrage et les rôles, certaines peuvent être absentes, renommées ou
  volontairement retirées — c'est notamment le cas des fonctions d'édition directe
  décrites dans la fiche [Débogage](scenarios/06-debogage-et-edition-de-table.md).
- Une transaction marquée ✏️ ou ⚠️ peut modifier des données ou relancer un
  traitement métier. En production, la question n'est jamais seulement « est-ce que
  j'ai le droit » : c'est « qu'est-ce que ça change, et qui pourra l'expliquer dans
  six mois ».
- Le débogage et l'édition de table sont **tracés et audités**. La fiche qui les
  couvre commence par les règles, pas par la manœuvre. C'est volontaire.

## Contribuer

Une transaction manquante, un enchaînement plus efficace, une correction de version :
les issues et les PR sont bienvenues. Une règle : **rien de spécifique à un client**.

## Licence

MIT — voir [LICENSE](LICENSE).
