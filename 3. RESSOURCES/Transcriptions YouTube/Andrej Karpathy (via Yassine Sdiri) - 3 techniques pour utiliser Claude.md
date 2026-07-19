---
titre: "La nouvelle méthode pour utiliser Claude IA sans prompt"
auteur: Yassine Sdiri (synthèse d'une conférence d'Andrej Karpathy)
url: https://youtu.be/oU2slLP15VA
durée: 11:41
date_ajout: 2026-07-19
---

# Andrej Karpathy — 3 techniques pour utiliser Claude (au-delà du prompt)

**Qui parle** : Andrej Karpathy — cofondateur d'OpenAI, ex-directeur IA de Tesla, inventeur du terme "vibe coding", récemment arrivé chez Anthropic pour travailler sur l'entraînement de Claude. La vidéo est une synthèse par Yassine Sdiri d'une de ses conférences.

## Synthèse

**Constat de départ** : le prompt (la formulation de la demande) a un impact marginal. Ce qui fait vraiment la différence, c'est le **contexte** que Claude a sous les yeux au moment de répondre — qui vous êtes, ce que vous cherchez, vos contraintes, vos exemples. La plupart des gens ne préparent jamais ce contexte, puis blâment le prompt ou l'outil quand la réponse est moyenne.

**Image clé** : Claude n'est pas un cerveau humain, c'est un **stagiaire avec une mémoire et une culture monstrueuses, mais un jugement à zéro**. Il peut sortir une stratégie marketing brillante en 30 secondes et se planter sur un comptage de lettres. Le pousser, l'engueuler ou insister ne sert à rien — la seule chose qui marche, c'est de bien l'encadrer.

1. **Donner une mémoire maîtrisée** — Par défaut Claude est amnésique (ou pire, la mémoire auto est une boîte noire que vous ne contrôlez pas). Les pros construisent une mémoire qu'ils maîtrisent eux-mêmes : dans Claude, ça passe par les **Projects** — un espace dédié où l'on écrit une fois pour toutes qui on est, ce qu'on fait, son ton, ses règles, et où l'on dépose ses meilleurs documents/exemples de référence. Plus on nourrit cette mémoire, plus les réponses deviennent précises — et ça devient un avantage que personne ne peut copier.

2. **Faire des briefs, pas des questions vagues** — Remplacer "écris-moi X" par un vrai cahier des charges répondant à 4 questions : (1) quel est le vrai objectif/résultat visé, pas juste la tâche ? (2) qu'est-ce que Claude doit savoir pour ne pas avoir à deviner ? (3) quelles sont les contraintes (ton, format, longueur, interdits) ? (4) tout d'un coup, ou étape par étape avec validation au fur et à mesure ? Astuce : on peut demander directement à Claude de construire ce brief avec vous — "pose-moi des questions pour cerner mon objectif, liste les décisions importantes, fais-les-moi valider avant de te lancer."

3. **Rendre le résultat vérifiable (la plus puissante)** — Claude devient meilleur quand il peut vérifier son propre travail. Niveau 1 : donner des critères précis et lui demander de se relire point par point avant de rendre. Niveau 2 (plus fort) : laisser le réel trancher — remonter les vrais résultats (clics, ventes, conversions) et lui demander d'analyser ce qui a marché pour améliorer la suite. Ça crée une boucle vertueuse, mais il faut **rester le pilote** : ne pas disparaître de la boucle en espérant un miracle automatique.

**Message de clôture** : Claude peut écrire, chercher, exécuter à votre place — mais pas décider à votre place ce qui compte, ce qui est vrai, ce qui mérite d'être construit. L'objectif n'est pas de devenir expert en prompt engineering, mais le meilleur pilote possible de son IA.

## Application possible

- **Technique 1 (mémoire maîtrisée), déjà en place** : c'est exactement ce que fait ce vault — `CLAUDE.md` + `memory.md` jouent le rôle du "Project" décrit par Karpathy. Le levier n'est pas de le refaire, mais de continuer à l'enrichir (casquettes, journal) plutôt que de laisser des templates vides.
- **Technique 2 (briefs)** : à réutiliser pour les prochaines demandes de contenu (scripts CDV, séquences email) — donner objectif + contraintes + mode étape par étape plutôt qu'une consigne vague, ou carrément laisser Claude poser les questions de cadrage avant de rédiger.
- **Technique 3 (boucle de vérification par le réel)** : directement applicable au diagnostic [[1. PROJETS/Cercle de Vérité]] / [[1. PROJETS/Sakura]] — une fois les pubs Meta lancées, remonter les vraies performances (CPA, taux de conversion CDV → upsell → Sakura) pour ajuster les briefs plutôt que de deviner ce qui marche.
