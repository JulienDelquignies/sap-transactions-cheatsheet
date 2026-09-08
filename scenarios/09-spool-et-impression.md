# « L'édition n'est pas sortie »

## Les transactions

| Code | Rôle | |
|---|---|---|
| `SP01` | Demandes d'impression (spool) — l'écran de travail | 🔍✏️ |
| `SP02` | Ses propres demandes d'impression | 🔍 |
| `SPAD` | Administration de l'impression : imprimantes, types d'appareil, serveurs de sortie | ⚠️ |
| `SP12` | Administration du TemSe (le stockage temporaire séquentiel) | ⚠️ |
| `SPIC` | Contrôle de cohérence du spool | 🔍 |
| `SU3` | Valeurs par défaut de l'utilisateur, dont son imprimante | ✏️ |

## Diagnostic en trois questions

### 1. La demande de spool existe-t-elle ?

`SP01`, en mettant **l'utilisateur à `*`** et une plage de dates large. Si rien
n'apparaît, l'édition n'a jamais été générée : le problème est dans le programme,
pas dans l'impression. Retour à `SM37` (le job a-t-il tourné ?) ou `ST22`.

### 2. La demande existe — quel est son statut ?

Colonne *Statut de la demande de sortie* :

| Statut | Signification | Action |
|---|---|---|
| `-` (vide) | Spool créé, aucune sortie demandée | Normal si l'utilisateur a choisi « ne pas imprimer immédiatement ». Imprimer depuis `SP01` |
| `En attente` | En file, pas encore pris en charge | Attendre, ou vérifier que le serveur de sortie tourne (`SPAD`) |
| `En traitement` | En cours d'envoi vers l'imprimante | |
| `Terminé` | SAP a envoyé le document au système d'impression | **Ce n'est pas « c'est sorti »** — voir ci-dessous |
| `Erreur` | Échec | Double-clic → le log donne la raison |
| `<F>` / erreur de traitement | Problème de formatage | Type d'appareil incompatible avec le formulaire |

### 3. Statut « Terminé » et rien ne sort de l'imprimante

C'est le cas le plus fréquent, et le plus mal compris. « Terminé » signifie que SAP a
remis le document au **système de spool du système d'exploitation** (ou au serveur
d'impression). Ce qui se passe après échappe complètement à SAP.

À vérifier, dans l'ordre :

1. **La bonne imprimante ?** `SP01` → afficher les attributs de la demande. L'imprimante
   par défaut de l'utilisateur (`SU3`) est souvent celle de son ancien bureau.
2. **L'imprimante est-elle en ligne ?** Test depuis `SPAD` → l'appareil de sortie →
   *Contrôle de sortie*.
3. **La file du système d'exploitation.** Côté serveur (`lpq`, `lpstat`, ou la file
   Windows). C'est là que les documents s'accumulent silencieusement.
4. **Le serveur d'impression a-t-il été redémarré ?** Les documents remis pendant
   l'indisponibilité sont perdus sans que SAP le sache.

Pour repartir : ✏️ `SP01` → sélectionner la demande → *Imprimer sans modification*
(réimprime tel quel) ou *Imprimer avec modification* (permet de changer d'imprimante,
de nombre d'exemplaires, de format).

## Le formulaire est illisible / mal formaté

Le couple **formulaire ↔ type d'appareil** est la cause quasi systématique.

- Un Smart Form ou un SAPscript conçu pour un type d'appareil donné rendra
  différemment sur un autre.
- Pour du PDF, viser un type d'appareil `PDF*` / `PDF1`.
- Les polices doivent exister dans le type d'appareil : `SPAD` → *Polices* →
  jeux de caractères.
- Les caractères accentués ou spéciaux qui deviennent `#` sont un problème de jeu de
  caractères, pas de formulaire.

## Purge et volumétrie

Le spool est une ressource limitée : les numéros de demande sont bornés (par défaut
32 000, extensible par le paramètre `rspo/spool_id/max_number`), et le TemSe (`SP12`)
stocke le contenu réel.

Symptôme d'un spool saturé : plus aucune édition ne se crée, avec un message
« nombre maximal de demandes atteint ». Ce n'est pas un problème d'imprimante, c'est
un problème d'exploitation.

Les jobs standard à planifier (et qui manquent souvent) :
- `RSPO0041` / `RSPO1041` — suppression des vieilles demandes de spool.
- `RSPO1043` — contrôle de cohérence du spool.
- `RSBTCDEL2` — purge des logs de jobs, qui alimentent aussi le TemSe.

⚠️ Ces jobs suppriment. Vérifier la durée de rétention avant de planifier : dans
certains contextes, une édition est une pièce justificative avec une obligation de
conservation. Dans ce cas, l'archivage doit être en place **avant** la purge.

## Archivage

Si le besoin est de retrouver une édition six mois plus tard, le spool n'est pas la
réponse. Le lien spool → système d'archivage (ArchiveLink) se paramètre dans `SPAD`
et `OAC0` / `OAC3`. Une politique de purge sans archivage, sur des documents à valeur
probante, est un risque, pas un gain de place.
