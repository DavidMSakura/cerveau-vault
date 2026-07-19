# Memory — Sessions de travail

> Une entrée par session. Format : date + ce qu'on a fait + décisions prises + suite prévue.

---

## 2026-07-16 — État du projet technique bot Telegram + veille BOAMP

**Vault mémoire** : `/home/david/Doc` (ce dépôt, Obsidian). **Projet bot** : `/home/david/telegram-claude-bot` (dépôt séparé, non lié au vault).

**Bot Telegram (`bot.js`)**
- Tourne en permanence via **PM2** (process `telegram-claude-bot`, fork mode).
- Accès restreint : ne répond qu'aux messages privés provenant de `TELEGRAM_ALLOWED_USER_ID` (mon ID Telegram) ; tout autre expéditeur est ignoré.
- Appelle Claude via `@anthropic-ai/claude-agent-sdk` (modèle `claude-sonnet-4-6`), avec `cwd` pointé sur `/home/david/Doc` → Claude peut lire/chercher dans le vault mémoire pendant les conversations Telegram.
- Outils MCP exposés : Gmail (lecture/brouillon), Google Calendar (lecture/création), rappels (set/list/delete).
- **Connexion Claude Pro** : via la variable `CLAUDE_CODE_OAUTH_TOKEN` définie dans `.env` (non versionné, secret jamais affiché ni stocké ici).
- **Mémoire de session Telegram** : `.sessions.json` associe chaque `chat_id` à un `session_id` du SDK, ce qui permet de reprendre une conversation Claude sans tout réinjecter. Une session invalide/expirée est automatiquement effacée et recréée.

**Veille BOAMP (`veille-boamp.js`)**
- Interroge l'API BOAMP sur les 7 derniers jours avec mots-clés B2B ciblés (prise de parole, leadership, communication managériale, coaching, cohésion d'équipe, etc.) et exclusions (CDI/CDD, recrutement, hébergement, restauration...).
- Dédoublonne (objet + acheteur), puis filtre via `.boamp-seen.json` (mémoire des annonces déjà traitées/envoyées) pour ne jamais retraiter deux fois la même annonce.
- Notation par Claude sur une grille pondérée /10 (adéquation expertise, faisabilité seul/Steph/partenaires, intérêt commercial, délai, accessibilité Lyon, clarté du besoin, contraintes) avec statut **BLOQUANT** prioritaire (note forcée à 0) si une exigence rend la candidature irréaliste (certification manquante, taille d'entreprise incompatible, etc.).
- Seuil d'envoi : note ≥ **7/10** et statut OK, max 5 opportunités envoyées par exécution.

**Cron BOAMP** : `*/15 6-8 * * 1-5` (lundi-vendredi), protégé par `flock` anti-chevauchement (`.veille-boamp.lock`). Le serveur tourne en UTC sans support `CRON_TZ`, donc le script lui-même vérifie l'heure réelle à Paris (via luxon) et n'agit que dans la fenêtre **08h00-08h14 heure de Paris**, DST géré nativement.

**Autres crons détectés**
- `checker.js` (chaque minute) : envoie les rappels dus (`rappels.json`), ponctuels ou récurrents.
- `briefing.js morning` : 7h30 lun-ven, 9h00 sam-dim.
- `briefing.js evening` : 21h00 tous les jours.

**Prochaines étapes possibles**
1. Vérifier la robustesse PM2 au reboot (`pm2 save` + `pm2 startup`) — pas de fichier `ecosystem.config.js` trouvé.
2. Alléger/rotater les logs (`checker.log` notamment, 2+ Mo, tourne chaque minute).
3. Étendre la veille à **TED / Simap** (marchés européens/suisses) en plus de BOAMP.
4. Ajouter des **sources privées** à la veille (réseaux, organismes de formation partenaires).
5. Ajouter une **veille contenus** (idées vidéo YouTube/Instagram, tendances du secteur).

## 2026-07-18 — Transcriptions YouTube, exploration Prisme One, structuration du vault

**Transcriptions YouTube** : mise en place du process de synthèse (pas de retranscription brute) dans `3. RESSOURCES/Transcriptions YouTube/`. Premier exemple traité : Daniel Pink, 7 principes de persuasion. David va fournir des playlists de sa propre chaîne Mnemonaute à digérer (mémorisation puis feel-good/dev perso).

**Bot Telegram — accès YouTube** : ajout d'un outil `youtube_transcript` (MCP `google-tools`) + autorisation `Write` dans `bot.js`/`mcp-google-server.js`. Fiabilité limitée : Firecrawl ne renvoie la transcription YouTube que ~20% du temps (comportement non déterministe, retry x4 implémenté mais pas garanti). `yt-dlp` testé en alternative mais bloqué par la détection anti-bot YouTube sur cette IP serveur sans cookies d'un compte connecté. David dit qu'il fournira "mieux" bientôt (probablement cookies ou autre solution) — ne pas relancer ce chantier tant qu'il n'a pas donné suite.

**Accès Obsidian** : le vault vit sur le serveur distant (`srv1792680`), pas en local. David n'a jamais mis en place de synchronisation vers son ordinateur/téléphone. Solution proposée mais pas encore actée : Syncthing (gratuit, purpose-built, pas de cloud tiers). En attente de sa décision.

**Exploration Prisme One / Bootcamp IA** : scrapé + interagi (clics sur accordéons/onglets cachés) sur `https://prisme.one/bootcamp-ia` en intégralité (contenu gratuit uniquement — `/plateforme/*` est payant/derrière login). Méthode "Atomic Thinking" d'Eliott Meunier : 3 couches de contexte (fiches process / statique / dynamique), architecture modèle-agnostique, infra + automatisations personnelles. **Découverte clé : le vault de David suit déjà l'ontologie IPCRA (Projets/Casquettes/Ressources/Archives) qu'ils enseignent** — la structure de base existe déjà sans avoir payé la formation.

**Mis en place suite à cette exploration** :
- Nouveau dossier `6. PROCESS/` — fiches process documentant "comment David fait" ses tâches récurrentes (le vrai manque identifié vs. la méthode Prisme One).
- Première fiche : `Conception de programme transformationnel (Sakura & Cercle de Vérité).md` — méthode réelle de David : partir du contexte (souffrances/freins/patterns/autosabotage) → définir la destination (mouvement attendu) → construire les étapes avec un mix d'outils (coaching : PNL/CNV/Karpman/gestion émotions ; sportif haut niveau : visualisation/mindset/entraînement/mantras). Priorité actuelle : révision des programmes existants, pas création ex nihilo. Note : préparation de formations animées n'est PAS une priorité (ingénierie pédagogique rarement de son ressort) ; marketing (ads/emails/Insta) n'a pas encore de méthode fixe — chantier ouvert, à documenter plus tard.
- `1. PROJETS/` peuplé avec 5 fiches : Cercle de Vérité, Sakura, Contrat Crédit Foncier, Vente appartement locatif, Croisière Méditerranée juillet 2026 — toutes avec un champ "Prochaine étape" à préciser par David.
- Nouveau skill `/weekprepare` (`.claude/skills/weekprepare/SKILL.md`) : croise projets ouverts + casquettes + journal récent + memory.md pour sortir 3-5 priorités de la semaine, sans inventer de tâches absentes du vault.

**Suite prévue** : demander à David de compléter les "Prochaine étape" laissées vides dans les fiches projet ; envisager une fiche process sur le marketing une fois qu'une méthode se dégage ; tester `/weekprepare` en conditions réelles.

<!-- Les entrées de session s'ajoutent ici, de la plus récente à la plus ancienne -->
