# Débogage et édition de table

Deux outils que tout le monde utilise et que personne ne documente : le débogueur
ABAP, et l'édition directe du contenu d'une table.

Ce sont des **outils de diagnostic**, pas des raccourcis. Cette fiche donne la
manœuvre et, tout aussi important, les règles qui vont avec.

---

## ⚠️ À lire avant le reste

Ces manipulations ont trois propriétés qu'il faut avoir en tête à chaque fois :

**1. Elles sont tracées et auditées.**
Le débogage en production, la modification de variable en session de débogage et
l'édition directe de table sont journalisés sur tout système correctement paramétré,
et ces journaux sont regardés. Les autorisations correspondantes (`S_DEVELOP` avec
l'activité de débogage-modification, `S_TABU_DIS` / `S_TABU_NAM` en modification) font
partie des points systématiquement examinés en audit. Utilisez-les en le sachant, pas
en espérant que personne ne verra.

**2. Elles court-circuitent la logique applicative.**
Un `UPDATE` direct sur une table ne déclenche aucun contrôle de cohérence, aucune
mise à jour des tables liées, aucun document de modification, aucun enregistrement
dans les fichiers d'historique. Une écriture comptable, un statut de document ou une
quantité en stock modifiés à la main créent une incohérence qui ne se voit pas tout de
suite et qui se découvre à la clôture, dans un état, ou dans un rapprochement.

**3. Leur place est le développement et la recette. Pas la production.**
En production, la règle raisonnable est : **jamais sans autorisation écrite**, avec
une trace de ce qui a été fait, par qui, et pourquoi. Si vous vous surprenez à en
avoir régulièrement besoin en production, le vrai problème est ailleurs — un
correctif manquant, un contrôle absent, un processus qui ne prévoit pas un cas réel.

**La bonne question n'est jamais « comment je force cette valeur ».**
C'est : *pourquoi le système refuse-t-il ?*, et *quelle est la voie applicative pour
obtenir le même résultat ?* Il existe presque toujours une transaction, une
annulation, une contre-passation ou une correction prévue. Elle est plus lente et elle
laisse le système cohérent.

---

## Le débogueur ABAP

### Entrer dedans

| Comment | Quand |
|---|---|
| `/h` dans la zone de commande, puis l'action | La méthode standard. Le débogueur s'ouvre à l'action suivante |
| `SE38` → *Exécuter → Déboguer* | Pour démarrer un programme directement en pas à pas |
| Point d'arrêt de session posé dans le code (`SE80`/`SE38`, double-clic sur la marge) | Pour s'arrêter à un endroit précis |
| `SM50` → sélectionner le processus → *Programme/Mode → Programme → Déboguer* | **Pour attraper un job en arrière-plan qui tourne déjà**. La seule méthode |
| `SM37` → job actif → *Job → Capturer le job actif* | Variante pour le batch |

### Les points d'arrêt qui font gagner du temps

**Point d'arrêt sur instruction ABAP** (dans le débogueur : *Points d'arrêt →
Instruction ABAP*, ou bouton dédié) :

| Instruction | Ce que ça permet |
|---|---|
| `MESSAGE` | **Le plus utile de tous.** S'arrêter exactement là où le message d'erreur est émis, quand on ne connaît que son texte |
| `CALL FUNCTION` | Intercepter tous les appels de module fonction |
| `SELECT` / `UPDATE` / `INSERT` / `MODIFY` / `DELETE` | Voir chaque accès base — utile pour trouver qui écrit dans une table |
| `RAISE` / `RAISE EXCEPTION` | Attraper l'exception à sa source, avant que l'appelant ne la masque |
| `SUBMIT` / `CALL TRANSACTION` | Suivre un enchaînement de programmes |

**Point d'arrêt sur module fonction** : indispensable pour intercepter un appel RFC
entrant — le programme n'existe pas côté appelant, seul le nom du module est connu.

**Point d'arrêt sur méthode / classe** : `CLASS=...` `METHOD=...`, pour du code objet.

**Watchpoint** — s'arrêter quand une **variable change de valeur**. C'est le seul
moyen raisonnable de répondre à « qui écrase mon champ ? ». Dans le débogueur :
*Watchpoints → Créer*, saisir le nom de la variable. On peut y ajouter une condition
(`s'arrêter seulement si la valeur devient 'X'`), ce qui évite de s'arrêter trois
cents fois.

### Différence session / externe

| Type | Portée | Survit à la déconnexion |
|---|---|---|
| **Point d'arrêt de session** | Votre session uniquement | Non |
| **Point d'arrêt externe** | Déclenché par un appel HTTP / RFC / d'un autre utilisateur | Oui, pendant une durée limitée |

Le point d'arrêt **externe** est ce qu'il faut pour déboguer un service web, un appel
OData ou un appel RFC entrant : le code s'exécute dans une session qui n'est pas la
vôtre. Il se pose depuis `SE80`/`SE38` (*Utilitaires → Points d'arrêt → Externe*) et
il faut renseigner l'utilisateur concerné.

⚠️ Un point d'arrêt externe oublié bloque des sessions réelles. Retirez-les en partant.

### Inspecter une table interne

C'est ce qu'on vient chercher neuf fois sur dix.

- Dans l'onglet *Variables*, saisir le nom de la table interne : le nombre de lignes
  s'affiche. Double-clic pour ouvrir le contenu.
- Bouton **Table** (ou onglet dédié selon la version du débogueur) pour la vue
  tabulaire complète : on navigue, on trie, on filtre.
- Pour une table volumineuse, utiliser le **filtre** plutôt que de faire défiler :
  poser une condition sur une colonne clé.
- *Services de l'outil de débogage → Sauvegarder les données* pour exporter le
  contenu et le comparer tranquillement en dehors de la session.

Pour les structures profondes et les objets, l'onglet *Variables* du nouveau
débogueur permet de déplier l'arborescence ; sur une référence objet, double-clic
ouvre l'instance et ses attributs.

### Modifier une variable en session de débogage

Dans l'onglet *Variables*, double-clic sur la valeur, saisir la nouvelle, valider
(icône crayon / *Modifier*). On peut ainsi forcer un indicateur, sauter une condition,
ou faire passer un contrôle.

**À quoi ça sert légitimement :**

- **Reproduire un cas qu'on ne sait pas provoquer.** Le client a un rejet sur une
  combinaison rare ; on force la variable pour parcourir la même branche de code et
  comprendre le comportement.
- **Isoler la cause.** Le traitement échoue : en changeant une valeur, on vérifie si
  c'est bien ce champ qui déclenche le refus, plutôt que de le supposer.
- **Tester un correctif avant de l'écrire.** On simule le comportement corrigé pour
  confirmer l'hypothèse, puis on écrit le vrai code.

**Ce que ce n'est pas :**

Ce n'est **pas** un moyen de contourner un contrôle pour faire passer un document en
production. Le contrôle est là pour une raison ; le forcer produit un document que le
reste du système considère comme impossible. Et une donnée modifiée par le débogueur
n'est pas tracée comme une modification de donnée : personne, plus tard, ne pourra
expliquer d'où elle vient.

⚠️ La modification de variable en débogage nécessite une autorisation distincte de la
simple lecture (`S_DEVELOP`, activité de modification en débogage). Sur les systèmes
sensibles, elle est délibérément retirée en production. C'est un paramétrage sain, pas
une contrariété à contourner.

### Autres possibilités utiles

- **Modifier le compteur d'instruction** (*Aller à → Positionner instruction
  suivante*) : sauter une portion de code. À manier avec précaution — on peut laisser
  des variables non initialisées.
- **Layouts du débogueur** : on peut sauvegarder une disposition (les variables qu'on
  regarde toujours). Sur un programme qu'on déboguera plusieurs fois, c'est rentable.
- **Débogage en arrière-plan** via `SM50` : voir plus haut. C'est la seule façon de
  comprendre un job qui se comporte différemment du même programme lancé en dialogue —
  différence d'utilisateur, de variante, ou de volume.

---

## L'édition de table

### `SE16N` et le mode édition

`SE16N` affiche le contenu d'une table. Il existe un mode qui rend les colonnes
modifiables : dans `SE16N`, on saisit `&sap_edit` dans la **zone de commande** (en
haut à gauche, celle où l'on tape les codes transaction), puis on lance la sélection.
Les résultats deviennent éditables et un bouton de sauvegarde apparaît.

**Ce qu'il faut savoir avant même d'essayer :**

- **Sur les systèmes récents, ce n'est plus disponible.** SAP a désactivé ce
  comportement par une note de sécurité, précisément parce qu'il permettait
  d'écrire en base sans passer par les contrôles. Selon la version et les correctifs
  appliqués, la commande est sans effet, refusée, ou tracée.
- **Là où c'est encore possible, c'est journalisé.** Les modifications passent par un
  enregistrement dédié, et l'accès en modification est soumis à une autorisation
  spécifique (`S_TABU_DIS` / `S_TABU_NAM` en activité modification, plus les objets
  liés à `SE16N`). Ce n'est pas discret.
- **Aucun contrôle applicatif n'est exécuté.** Pas de vérification de cohérence, pas
  de mise à jour des tables dépendantes, pas de document de modification métier.

### Les autres voies

| Voie | Ce que c'est | Quand |
|---|---|---|
| `SM30` / `SM31` | Maintenance via la **vue de maintenance** de la table | La voie propre pour du paramétrage : elle exécute les contrôles définis, et génère un ordre de transport si la table est configurée pour |
| `SE11` → *Contenu* → mode modification | Éditeur du dictionnaire (souvent restreint) | Développement uniquement |
| `SE16` en debug + `&sap_edit` | Anciennes variantes du même principe | Historique — mêmes réserves, en pire |
| Un programme dédié (`UPDATE` explicite) | Un correctif écrit, transporté, tracé | **La bonne réponse pour une correction de masse** : relu, testé, réversible, et il reste une trace de ce qui a été fait |

### La règle de décision

Avant toute modification directe de contenu de table, trois questions dans l'ordre :

1. **Existe-t-il une transaction applicative qui fait cela ?** Une annulation, une
   contre-passation, une transaction de correction, un point d'entrée de maintenance.
   Si oui, c'est la réponse — même si elle est plus longue.
2. **Suis-je en train de corriger une donnée, ou de masquer un défaut ?** Si la même
   correction revient toutes les semaines, ce n'est pas de la maintenance, c'est un
   symptôme.
3. **Est-ce que je saurais expliquer cette modification dans six mois, à quelqu'un qui
   la découvre dans un état incohérent ?** Si la réponse est non, ne le faites pas.
   Si la réponse est oui, écrivez-le quelque part **avant**, pas après.

Et, en production, une quatrième : **ai-je une autorisation écrite ?** Un message,
un ticket, un mail. Pas parce que c'est une formalité, mais parce que le jour où la
donnée pose problème, c'est ce qui distingue une intervention assumée d'une faute.

### Si vous devez le faire quand même

- **Extraire d'abord l'état avant.** `SE16N` → export des lignes concernées avant
  toute modification. C'est votre seul retour arrière.
- **Une ligne d'abord.** Modifier un enregistrement, vérifier le résultat **dans
  l'application**, pas dans la table. Puis les autres.
- **Noter la clé exacte** des enregistrements touchés, l'horodatage, et la raison.
- **Prévenir** la personne qui exploitera le système, pas seulement celle qui a
  demandé.

---

## Autour, au quotidien

Les transactions qui accompagnent ce travail. Chacune a sa fiche.

| Code | À quoi ça sert ici | Fiche |
|---|---|---|
| `ST22` | Lire le dump quand le programme s'est arrêté. L'exception, la ligne source, la pile d'appel | [Chercher une erreur](01-chercher-une-erreur.md) |
| `SM37` | Le job a-t-il tourné, sous quel utilisateur, avec quelle variante, qu'a dit son log | [Jobs](03-jobs-et-batch.md) |
| `SM58` | Les appels RFC asynchrones en attente ou en erreur — et le message d'erreur associé | [Chercher une erreur](01-chercher-une-erreur.md) |
| `SM50` | Voir un processus en cours, et y attacher le débogueur | [Performance](02-performance.md) |
| `ST05` | Ce que le programme demande vraiment à la base | [Performance](02-performance.md) |
| `SU53` | Vérifier que ce n'est pas simplement une autorisation manquante | [Autorisations](04-autorisations.md) |
| `SE16N` | Lire l'état réel de la donnée, avant et après | [Données](10-donnees.md) |
| `SE93` | Retrouver le programme derrière la transaction, pour savoir où poser le point d'arrêt | [Développement](05-developpement.md) |

## Les deux réflexes qui valent tout le reste

**Avant de déboguer :** *Système → Statut*. Le mandant, le système, le programme.
Le nombre d'interventions faites dans le mauvais environnement est considérable, et
elles commencent toutes par « je pensais être en recette ».

**Avant de modifier quoi que ce soit :** notez l'état avant. Une capture, un export,
un numéro de document. C'est trente secondes, et c'est la différence entre une
correction et un incident.
