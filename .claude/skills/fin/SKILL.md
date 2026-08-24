---
name: fin
description: Clôture la session — résume ce qu'on a fait, met à jour memory.md et retaille Etat actuel.md. Utiliser quand David dit qu'on s'arrête, ou avant de fermer une session de travail.
---

# Clôture de session

Trois étapes, dans cet ordre. Ne rien sauter, et ne rien inventer : si tu n'es pas sûr de ce qui a été décidé, demande plutôt que de deviner.

## 1. Résumer la session à David

En cinq lignes maximum, avant d'écrire quoi que ce soit : ce qu'on a fait, ce qui a été décidé, ce qui reste ouvert. C'est sa dernière chance de te corriger avant que ça parte dans la mémoire.

## 2. Écrire dans `memory.md`

Ajouter une entrée en fin de fichier, au format des précédentes : `## AAAA-MM-JJ — Titre court`. S'il y a déjà une entrée du jour, numéroter — `## 2026-08-24 (2) — …`.

Ce qui doit y figurer :

- **Ce qu'on a produit**, avec les chemins de fichiers réels.
- **Les décisions de David**, en le citant quand la formulation compte. Marquer explicitement celles qui sont **tranchées**, pour ne pas les rouvrir dans trois mois.
- **Mes erreurs**, quand il en a corrigé une. Elles valent plus cher que les réussites : elles disent comment il pense.
- **Les découvertes** qui éclairent un autre sujet du vault, avec les liens `[[wiki]]`.
- **Ce qui reste à faire**, et par qui.

Convertir toute date relative en date absolue. Pas de « la semaine prochaine » : la date.

## 3. Retailler `Etat actuel.md`

C'est le fichier injecté au démarrage de chaque session. Il ne contient que ce qui est **vivant maintenant**, et il doit rester sous 4 000 caractères.

- **Retirer** ce qui est résolu — c'est déjà dans `memory.md`, ça n'a plus à occuper le contexte.
- **Ajouter** ce qui vient d'apparaître.
- **Mettre à jour** les échéances : une date dépassée doit soit sauter, soit devenir une alerte.
- **Mettre à jour la date** en en-tête.

## 4. Vérifier avant de rendre la main

- Aucun lien `[[wiki]]` cassé dans ce qui vient d'être écrit.
- `git status` propre, ou les fichiers ajoutés et commités si David le demande.
- Les points de vigilance qui traînent depuis longtemps sans bouger : le signaler une fois, sans insister.

## Ce qu'il ne faut pas faire

- Écrire une entrée pour une session où il ne s'est rien passé. Un échange de deux messages ne mérite pas une trace.
- Recopier dans `Etat actuel.md` ce qui est déjà dans `memory.md`. L'un est le présent, l'autre l'historique.
- Faire disparaître un point de vigilance parce qu'il traîne. Il sort quand il est réglé, pas quand il est vieux.
