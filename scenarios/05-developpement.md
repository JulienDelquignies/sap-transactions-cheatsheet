# Développement et exploration d'objets

## Les incontournables

| Code | Rôle | |
|---|---|---|
| `SE38` | Éditeur ABAP : afficher, modifier, exécuter un programme | ⚠️ |
| `SE80` | Object Navigator — tout, au même endroit, avec l'arborescence du package | ⚠️ |
| `SE11` | Dictionnaire ABAP : tables, vues, types, domaines, éléments de données | ⚠️ |
| `SE16N` | Affichage du contenu d'une table (successeur de `SE16`) | 🔍⚠️ |
| `SE37` | Modules fonction | ⚠️ |
| `SE24` | Classes et interfaces ABAP Objects | ⚠️ |
| `SE91` | Classes de messages | ⚠️ |
| `SE93` | Codes transaction — créer, ou **retrouver la transaction qui appelle un programme** | ⚠️ |
| `SE84` | Système d'information du Repository — la recherche transverse | 🔍 |
| `SE18` / `SE19` | BAdI : définition / implémentation | ⚠️ |
| `CMOD` / `SMOD` | User-exits (technique ancienne, toujours présente) | ⚠️ |
| `SE41` / `SE51` | Menu Painter / Screen Painter (Dynpro) | ⚠️ |
| `SE71` | SAPscript (formulaires anciens) | ⚠️ |
| `SMARTFORMS` | Smart Forms | ⚠️ |
| `SFP` | Adobe Forms | ⚠️ |
| `SE09` / `SE10` | Ordres de transport — voir la [fiche transports](07-transports.md) | ⚠️ |
| `SAT` | Analyse de performance d'exécution | 🔍 |
| `SCI` / `ATC` | Contrôles qualité de code (Code Inspector / ABAP Test Cockpit) | 🔍 |
| `SE95` | Assistant de modification — retrouver les modifications du standard | 🔍 |
| `ADT` | Pas une transaction : ABAP Development Tools sous Eclipse. Sur S/4, c'est l'outil de référence | |

## Trouver quelque chose quand on ne sait pas où chercher

C'est 80 % du travail sur un système qu'on ne connaît pas.

### « Quel programme se cache derrière cette transaction ? »

`SE93` → saisir le code transaction → afficher. Donne le programme, l'écran de
départ, et le type (dialogue, rapport, paramètre, variante de transaction).

Variante rapide : depuis n'importe quel écran, *Système → Statut* donne le programme
et l'écran en cours. C'est le réflexe le plus rentable de tout SAP.

### « Où cette table est-elle utilisée ? »

`SE11` → afficher la table → bouton *Où utilisé* (Ctrl+Maj+F3). Cocher *Programmes*,
*Classes*, *Modules fonction*. Sur une table standard très utilisée, le résultat sera
énorme : restreindre au package ou à l'espace de noms client.

### « Où ce champ est-il rempli ? »

`SE11` sur la table → *Où utilisé* ne suffit pas toujours. La méthode fiable est le
point d'arrêt : `SE38` → exécuter en mode débogage → point d'arrêt sur instruction
`UPDATE` / `INSERT` / `MODIFY` sur la table. En dialogue, poser un *watchpoint* sur
le champ.

### « Quel est le champ derrière cet écran ? »

F1 sur le champ → *Informations techniques*. Donne le nom de la table et du champ,
l'élément de données, et le nom de l'écran. Point de départ de toute investigation
fonctionnelle.

### « Quels sont les développements spécifiques de ce système ? »

`SE84` → *Objets du Repository* → filtrer sur l'espace de noms client (`Z*`, `Y*`,
ou l'espace de noms réservé du projet). Trier par package pour comprendre
l'organisation.

### « Y a-t-il un point d'extension ici ? »

Dans l'ordre de préférence moderne :
1. **BAdI** (`SE18` pour chercher la définition, `SE19` pour implémenter).
2. **Enhancement Framework** (points et sections d'extension implicites/explicites —
   accessibles depuis `SE80` en mode extension).
3. **User-exit** (`SMOD` pour trouver le composant, `CMOD` pour l'activer via un
   projet). Technique ancienne mais très présente.
4. **Modification du standard** — dernier recours absolu. Chaque modification est un
   coût à chaque montée de version, pour toujours. `SE95` liste celles qui existent
   déjà : à consulter avant de promettre une estimation de migration.

## `SE16N` : le confort et le danger

`SE16N` affiche n'importe quelle table sans passer par un programme. Indispensable
en analyse.

⚠️ Deux mises en garde sérieuses :

- **Lire une table n'est pas neutre du point de vue des données personnelles.**
  Les tables de personnel, de clients ou de fournisseurs contiennent des données
  protégées. L'accès est tracé sur les systèmes correctement paramétrés, et il doit
  l'être.
- **`SE16N` a historiquement permis la modification directe** de contenu de table via
  une commande cachée. C'est une pratique à proscrire : elle contourne toute la
  logique applicative, les contrôles de cohérence et les documents de modification.
  Sur les systèmes récents, cette possibilité est verrouillée ou tracée. Si vous vous
  surprenez à en avoir besoin, le vrai problème est ailleurs.

Alternatives propres pour lire : `SQVI` (QuickViewer, pour une jointure simple sans
développement), ou une vue CDS / un rapport dédié si le besoin est récurrent.

## Débogage : le minimum utile

> La fiche [Débogage et édition de table](06-debogage-et-edition-de-table.md) traite
> le sujet en entier, avec les règles d'usage. Résumé ici.

- `/h` dans la zone de commande active le débogueur pour l'action suivante.
- **Point d'arrêt sur instruction** : dans le débogueur, *Points d'arrêt → Instruction
  ABAP*, puis par exemple `MESSAGE` pour s'arrêter à l'endroit exact où le message
  d'erreur est émis. C'est la technique la plus rapide pour retrouver l'origine d'un
  message dont on ne connaît que le texte.
- **Point d'arrêt sur module fonction** : pour intercepter un appel RFC entrant, ou
  un appel dont on ne connaît que le nom du module.
- **Watchpoint** : s'arrêter quand une variable change de valeur. Le seul moyen
  raisonnable de trouver « qui écrase mon champ ».
- Débogage en arrière-plan : `SM50` → sélectionner le processus → *Programme/Mode →
  Programme → Déboguer*. Pour attraper un job qui tourne.

⚠️ En production, le débogage est une action à privilèges élevés, et la modification
de variables en cours de débogage (« debug & replace ») est une modification de
données non tracée. À traiter comme telle.
