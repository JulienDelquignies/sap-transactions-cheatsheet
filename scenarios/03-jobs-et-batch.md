# « Un job n'a pas tourné »

## `SM37` — 🔍✏️ la transaction centrale

Tout commence ici. Les réglages de sélection qui font la différence :

- **Nom du job : `*`**. Sinon vous cherchez un nom que vous ne connaissez pas.
- **Utilisateur : `*`**. Le filtre par défaut sur votre propre nom est la première
  cause de « je ne vois aucun job ».
- **Cocher tous les statuts**, en particulier *Annulé* et *Prêt*. Un job « prêt »
  depuis trois heures est un job qui n'a jamais démarré — problème différent d'un
  job planté.

### Lire un job

| Statut | Ce que ça veut dire |
|---|---|
| **Terminé** | le job s'est terminé sans exception. **Pas** « le traitement a réussi » — un programme peut se terminer proprement après avoir tout rejeté. Lire le log. |
| **Annulé** | exception, ou annulation manuelle, ou dump. Le log donne la raison ; s'il renvoie vers un dump, aller dans `ST22`. |
| **Actif** | en cours. Comparer à la durée habituelle (colonne *Durée* sur les exécutions précédentes). |
| **Prêt** | libéré mais aucun processus BTC libre. C'est un problème de capacité, pas de programme. |
| **Planifié** | pas encore libéré — personne ne l'a démarré, ou l'heure n'est pas atteinte. |
| **Suspendu** | volontairement mis en attente. |

Actions utiles :
- *Job → Log* : le journal d'exécution. Toujours la première lecture.
- *Job → Étapes* : quel programme, quelle variante, quel utilisateur d'exécution.
  L'utilisateur d'exécution explique la moitié des « ça marche en manuel mais pas en
  batch » — ce n'est pas le même, donc pas les mêmes autorisations.
- *Job → Spool* : la sortie imprimée, souvent le vrai compte rendu métier.
- ✏️ *Job → Répéter* : relancer une exécution à l'identique. ⚠️ Même avertissement
  que partout : est-ce que ça crée des doublons ?

## Planifier

| Code | Rôle | |
|---|---|---|
| `SM36` | Créer et planifier un job (assistant simple ou mode expert) | ✏️ |
| `SM37` | Surveiller, relancer, supprimer | 🔍✏️ |
| `SM62` | Gérer les événements système utilisés comme déclencheurs | ✏️ |
| `SM64` | Déclencher un événement à la main — pour tester un enchaînement | ✏️ |
| `SM61` | Groupes de serveurs batch — sur quel serveur un job a le droit de tourner | ⚠️ |
| `SM65` | Outil d'analyse du système batch (diagnostic global) | 🔍 |
| `RZ01` | Moniteur graphique de planification | 🔍 |
| `SM35` | Sessions batch input (traitement de masse ancienne génération) | 🔍✏️ |

## Les causes réelles, par fréquence

1. **Le job n'était pas planifié**, ou il l'était dans un autre système / mandant.
   Vérifier avant d'enquêter sur autre chose.
2. **Autorisation manquante pour l'utilisateur d'exécution.** Le job tourne sous un
   utilisateur technique qui n'a pas les mêmes rôles que vous. `SU53` ne marchera pas
   ici (il montre le dernier échec *de votre* session) — utiliser `STAUTHTRACE` en
   filtrant sur l'utilisateur du job.
3. **Variante absente ou vide.** Un transport qui n'a pas emporté la variante, et le
   programme s'exécute avec une sélection vide sans se plaindre.
4. **Aucun processus batch libre.** Statut *Prêt* qui dure. `SM50` : combien de
   processus BTC, combien occupés. C'est du paramétrage (`RZ10`), pas du code.
5. **Le job précédent d'une chaîne a échoué**, et le suivant attend un événement qui
   n'est jamais venu. `SM62` / `SM64`.
6. **Dépassement de temps ou de mémoire.** Aller dans `ST22`.
7. **Fichier d'entrée absent.** `AL11` sur le répertoire attendu.

## `SM35` — sessions batch input

Ancienne technique, toujours vivante pour les reprises et les chargements de masse.

- Statut **Nouveau** : jamais traité.
- Statut **Erreur** : des transactions ont échoué ; la session est réexécutable en
  mode *Traiter/afficher les erreurs seulement*.
- Le mode d'exécution compte : *En arrière-plan* pour la masse, *Affichage des
  erreurs seulement* pour comprendre, *Pas à pas* pour déboguer un seul cas.

⚠️ Rejouer une session batch input rejoue de vraies transactions. Sur une session
partiellement traitée, ne rejouer que les erreurs.

## Bonnes pratiques quand on planifie soi-même

- Un job qui n'écrit rien dans le log est un job qu'on ne pourra pas diagnostiquer.
  Journaliser au minimum : nombre d'enregistrements lus, traités, rejetés.
- Un job doit se terminer en **erreur** quand le traitement a échoué. Un job vert qui
  a tout rejeté ne réveillera jamais personne.
- Prévoir le rejeu dès la conception : le job doit être **idempotent** ou savoir
  reprendre là où il s'est arrêté. Sinon, chaque incident devient un travail manuel.
- Éviter les heures rondes. À 22h00 pile, tous les jobs de tous les projets démarrent
  en même temps.
