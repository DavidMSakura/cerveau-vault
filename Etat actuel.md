# État actuel

> Injecté au démarrage. Ne contient que ce qui est **vivant maintenant** ; le résolu part dans `memory.md`.
> Mise à jour : **26/08/2026 (soir, 2)**

## ⏰ Échéances

| Quand | Quoi | État |
|---|---|---|
| **ven. 28/08** | Point de cadrage Radiance | en cours de calage sur cette date |
| **dim. 31/08 18h** | 1ère exécution auto du cron notes iOS + rangement Kanban | vérifier le journal le lendemain : permission Notes.app en tâche de fond non garantie. **En cas d'échec, regarder d'abord l'autorisation d'automatisation macOS** — voir la note de vigilance plus bas |
| **ven. 04/09 14h** | Tripartite Teams CFF — coaching **Jilani Ben-Yahmed** (avec Sophie Pons et François) | autre dossier que le parcours féminin |
| **09 → 11/09** | IGECOM : Toulon, Clermont, Naves | déplacement, aucune prépa possible |
| **14-15/09** | Journées Prise de parole, Crédit Foncier | **deux journées indépendantes** : 7 managers, puis 6 cadres non managers. Support déjà conçu et mis en forme. **Rien à préparer** — format que David anime en routine |
| **16-17/09** | Formation DDA Radiance, Dijon, via MyConnecting | supports envoyés au client |
| **08/12** | Clôture du parcours CFF | la reconduction 2027 est du ressort de **Steph**, pas de David |

## 🔨 Chantiers actifs

**Second cerveau de Steph.** Guide terminé et partagé — artefact `1285ae55…`, source et version texte dans `~/Documents/Second cerveau - guide/`. **L'installation n'a pas commencé.** Steph démarre par les chapitres 0 à 3, qui ne se délèguent pas.

**Crédit Foncier.** Le contrat le plus rentable. **Steph est en lead sur la relation Sophie Pons et cale la suite** — David est prestataire en conception, design et animation. La reconduction 2027 n'est pas son sujet. Agenda : « Sophie leadership 🚺 ». **Cinq brouillons Gmail prêts** : y glisser les pièces jointes (`Coachings/À envoyer/`), vérifier le vouvoiement. `Suivi coachings DM.xlsx` faux sur deux points.

**MyConnecting / Radiance.** Livrables partis. Deux trous que seul le client peut boucher : la conséquence factuelle d'une formule au minimum sur un deux-roues, et la répartition des six informations dues.

**Cercle de Vérité.** Produit prêt, upsell 1 à 27 €. **Test Meta 200 € : en sourdine, ne plus le relancer** — il vient après la validation et le tournage des créas vidéo (tranché le 26/08). Reste ouvert depuis le 08/08 : par quoi remplacer les 3 besoins que la sur-adaptation satisfait ?

**Sakura.** Format arrêté (6 séances d'1 h sur 3 mois, 2 500 €). Reste le **contenu différencié** des deux niveaux, les pages, la mécanique de passage.

**Contenu.** Quatre idées dans `Idées de contenu - tableau` : « C'est pas la confiance qui m'a rendu champion » (prête à tourner), « le cas de Louis », « Gandhi vs Dev Perso » (**angle à confirmer**), et « Fais comme je dis, questionne après » (nouvelle, née du guide de Steph).

**Trello.** Coordination avec Steph, cartes autoportantes. **En attente de la clé API et du token** (`trello.com/power-ups/admin`) — à ranger dans le trousseau. · **Coaching Elisabeth Roche** : hebdomadaire.

## ⏸️ En attente d'une décision de David

- **Surveillance automatique des jetons** : David veut la faire. Reste à trancher entre le **scan de journal** (`401` / `invalid_grant` dans `logs/bot.log`, réactif — ne voit rien si le bot dort) et la **sonde active quotidienne** à 7h, avant le briefing (recommandée : prévient avant la panne). ~20 min dans les deux cas. Ne renouvelle jamais le jeton, alerte seulement.
- **Les deux livres.** *Libérez votre mémoire* en jachère depuis le 16/08. *Le Triangle Dramatique* en rédaction. Aucun n'avance.
- **La tension marketing.** Elisabeth dit « sois toi-même » ; la stratégie dit ads Meta → funnel.
- **Marina et Dominique** n'ont plus aucun coaching planifié d'ici juin 2027.

## ⚠️ Points de vigilance

- **Autorisations d'automatisation macOS (erreur `-1743`).** Réglé pour `Doc.app` le 26/08 : une app AppleScript **sans `CFBundleIdentifier`** ne peut pas se voir accorder le droit de piloter une autre app — macOS n'a nulle part où retenir la réponse, et refuse sans jamais poser la question. Correctif : ajouter l'identifiant, re-signer (`codesign --force --sign - --identifier ...`), ré-enregistrer (`lsregister -f`), puis `tccutil reset AppleEvents <id>`. **Même famille de panne que le cron notes iOS** qui doit piloter Notes.app.
- **Épingle de partage.** L'artefact partagé ne suit pas les republications : après chaque modification du guide, il faut redéplacer l'épingle, sinon les destinataires restent sur l'ancienne version.
- **Administratif MyConnecting**, en pause volontaire (08/08) : URSSAF, RIB, Qualiopi. **Ne pas relancer de moi-même.**
- **Signaux comportementaux** (voir CLAUDE.md) : procrastination admin, Magic Arena, dispersion, absence de Pomodoro, le petit singe.

## 🧠 Comment marche ta mémoire

`Etat actuel.md` = le présent · `memory.md` = 21 sessions d'historique, à ouvrir au besoin · `Tableau de bord` et `Idées de contenu - tableau` = les deux Kanban · `/fin` met le tout à jour.
