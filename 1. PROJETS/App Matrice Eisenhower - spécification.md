---
statut: à coder
outil: XCode / Antigravity
décidé: 2026-09-02
tags:
  - app
  - productivite
---

# App Matrice Eisenhower — spécification

Document destiné à être donné à **Antigravity** pour générer l'app. Il décrit ce qu'elle doit faire, pas comment le coder.

**Ce qu'elle remplace** : Focus Matrix, dont les données sont enfermées (synchronisation CloudKit privée, pont uniquement vers le Calendrier, et un calendrier n'a pas de champ « urgent » ni « important »).

**Ce qu'elle apporte que Focus Matrix ne peut pas** : un fichier lisible et modifiable par Doc, donc une matrice branchée sur le vault.

---

## 1. Ce que fait l'app, en une phrase

Une matrice d'Eisenhower sur iPhone, dont le contenu vit dans un fichier iCloud que David et Doc peuvent tous les deux lire et modifier.

## 2. L'écran

**Un seul écran principal : la grille 2×2**, plein écran, chaque case occupant un quart.

```
┌──────────────────┬──────────────────┐
│ URGENT           │ IMPORTANT        │
│ ET IMPORTANT     │ non urgent       │
│                  │                  │
│  • ...           │  • ...           │
│  • ...           │  • ...           │
├──────────────────┼──────────────────┤
│ URGENT           │ NI L'UN          │
│ non important    │ NI L'AUTRE       │
│                  │                  │
│  • ...           │  • ...           │
└──────────────────┴──────────────────┘
                              ( + )
```

**Dans chaque case** : le titre de la case, puis la liste des tâches non faites. Si la liste dépasse la hauteur, la case défile toute seule.

**Le bouton `+`** ouvre une saisie : un champ de texte, et le choix de la case. Valider ferme et revient à la grille.

**Toucher une tâche** ouvre son détail : modifier le texte, changer de case, la marquer faite, la supprimer.

**Changer de case** doit aussi être possible par glisser-déposer d'une case à l'autre. C'est le geste qui rend une matrice utile — on reclasse plus souvent qu'on ne crée.

**Une tâche marquée faite disparaît de la grille** mais reste dans le fichier, avec sa date. C'est ce qui permettra plus tard de regarder ce qui a réellement avancé.

**Signalement de l'âge** : une tâche présente depuis plus de 30 jours s'affiche avec un point ou une teinte différente. Rien d'autre — pas de notification, pas de culpabilisation.

## 3. Le fichier

Un unique fichier **`matrice.json`**, dans le conteneur iCloud de l'app, visible sur le Mac dans `~/Library/Mobile Documents/`.

```json
{
  "version": 1,
  "modifie_le": "2026-09-02T14:30:00Z",
  "taches": [
    {
      "id": "b7f3…",
      "texte": "Préparer les 5 questions pour Elisabeth",
      "quadrant": "important",
      "creee_le": "2026-09-02T14:30:00Z",
      "modifiee_le": "2026-09-02T14:30:00Z",
      "faite": false,
      "faite_le": null,
      "source": "doc"
    }
  ]
}
```

**Les quatre valeurs de `quadrant`**, jamais autre chose : `urgent_important` · `important` · `urgent` · `aucun`.

**`id`** est un UUID généré à la création et qui ne change jamais. C'est lui qui rend la synchronisation possible.

**`source`** vaut `david` ou `doc`. Une tâche créée par Doc s'affiche avec une petite marque jusqu'à ce que David la touche une première fois — pour qu'il ne découvre pas des lignes apparues de nulle part.

## 4. La synchronisation et les conflits

Les deux côtés écrivent dans le même fichier. La règle, simple et suffisante :

- **On compare tâche par tâche, jamais fichier par fichier.** Chaque tâche a son `id` et son `modifiee_le`.
- **Pour une même `id`, la version dont `modifiee_le` est la plus récente gagne.**
- **Une tâche présente d'un seul côté est conservée**, jamais supprimée par l'autre.
- **Une suppression n'efface pas la ligne** : elle passe `faite` à `true` avec un marqueur. Rien ne disparaît jamais vraiment, ce qui rend tout conflit réparable.

Côté Mac, un script reprend le fichier iCloud, le recopie dans le vault et le commite — même mécanique que `vault-notes-sync.py`, qui fonctionne déjà pour les notes iOS.

## 5. Ce que Doc en fait

**En lecture** : signaler ce qui dort — « le livre mémoire est en important-non-urgent depuis 47 jours » · alerter quand le quadrant urgent-et-important déborde, ce qui est le signe qu'on subit · croiser la matrice avec les échéances et les chantiers du vault.

**En écriture** : poser directement dans la bonne case ce qui sort d'une session — les questions à préparer pour un coaching, une relance, une décision en attente. Sans jamais toucher aux lignes de David.

## 6. Ce qui n'est PAS dans la version 1

À écarter explicitement, pour ne pas alourdir : les rappels et notifications · les créneaux hebdomadaires sur le quadrant 2 · les projets et les notes attachées aux tâches · les sous-tâches · l'iPad et le Mac · la synchronisation avec le Calendrier ou les Rappels · les statistiques.

Chacun de ces points peut s'ajouter plus tard. **Le quadrant 2 est traité comme les trois autres** — décision de David : on observe un mois avant d'ajouter quoi que ce soit.

## 7. Les hypothèses que j'ai prises seul

À corriger si l'une est fausse :

- **iPhone uniquement**, portrait, iOS récent.
- **Pas de compte, pas de serveur.** iCloud du compte Apple de David, c'est tout.
- **Français** dans l'interface.
- Une tâche faite **reste dans le fichier** plutôt que d'être effacée — c'est ce qui permettra de mesurer plus tard.
- Le nom de l'app reste à choisir.

---

## Pour mémoire — pourquoi cette app plutôt qu'un outil existant

Focus Matrix propose une synchronisation vers le Calendrier Google, déjà activée sur `david.marsac1@gmail.com`. Elle ne sert à rien ici, pour deux raisons : David ne met jamais de date, et **un événement de calendrier n'a pas de champ pour le quadrant**. Même en datant tout, l'information qui compte ne passerait pas.

Les Rappels Apple auraient pu servir — quatre listes valent quatre quadrants, et Doc y accède depuis le Mac. Écarté au profit d'une app propre, parce que **le format des données est alors sous contrôle total** : la date d'entrée, l'identifiant stable, la source. C'est la différence entre une liste de courses et quelque chose qui se branche sur le second cerveau.

→ [[Tableau de bord]]
