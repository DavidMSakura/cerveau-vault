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

## 2026-07-19 — Migration d'architecture : vault local (Mac) + serveur en lecture/écriture pour le bot uniquement

**Problème identifié** : le vault vivait uniquement sur le serveur, édité via une session Claude Code SSH. Pas de vault local sur le Mac de David, pas d'Obsidian synchronisé. Sur recommandation d'amis ayant une architecture similaire, bascule vers : Claude Code + Obsidian en local sur le Mac (source d'édition principale), serveur réservé à ce qui tourne 24/7 (bot Telegram, crons).

**Mis en place** :
- Nouveau repo GitHub dédié au vault : `github.com/DavidMSakura/cerveau-vault` (séparé de `cerveau-ia`, qui reste pour la sauvegarde brute du bot).
- **Mac de David** : Node déjà présent, Claude Code installé (`sudo npm install -g @anthropic-ai/claude-code`), clé SSH perso créée et ajoutée à son compte GitHub, vault cloné dans `~/Documents/IA/Vault-Mnemonaute`, ouvert dans Obsidian avec le plugin **Git** (nom affiché juste "Git", auteur Vinzent03) réglé en auto-pull/auto-backup toutes les 5 min.
- **Serveur** : `telegram-claude-bot/bot.js` modifié — `VAULT_DIR` pointe vers un clone dédié `/home/david/vault-clone` (remote SSH via une deploy key propre, alias `github-cerveau-vault`) au lieu de l'ancien `/home/david/Doc`. Le bot fait un `git pull --rebase --autostash` avant chaque requête et un `git add/commit/push` après si des fichiers ont changé. Ajout de `permissionMode: 'bypassPermissions'` dans les options du SDK — sans ça, l'outil `Write` restait bloqué en attente d'une confirmation interactive impossible à gérer via Telegram.
- Ancien dossier serveur `/home/david/Doc` archivé en `/home/david/Doc.archive-2026-07-19` (plus utilisé, conservé par précaution).
- **Test bout-en-bout réussi** : message Telegram → écriture bot dans le clone serveur → push GitHub → apparition dans Obsidian sur le Mac après le pull auto (5 min). Flux confirmé fonctionnel dans les deux sens.

**Nouvelle répartition des rôles** :
- **Claude Code sur le Mac** (`~/Documents/IA/Vault-Mnemonaute`) : usage quotidien, tout ce qui touche au contenu du vault (projets, process, réflexion, contenu).
- **Claude Code sur le serveur** (session SSH, ex-`/home/david/Doc`) : uniquement l'administration technique (bot, crons, veille BOAMP, infra). Rarement utilisé désormais.
- **Ce fichier `memory.md`**, étant dans le vault synchronisé, est lu par les deux instances au démarrage (consigne `CLAUDE.md`) — c'est le point de mémoire partagé entre elles. La mémoire "auto" spécifique à chaque session Claude Code (dossier `~/.claude/projects/...`) reste, elle, privée à chaque machine et ne se synchronise pas.

**Suite prévue** : David va utiliser principalement l'instance Mac désormais. Vérifier dans quelques temps que la sync Obsidian Git (5 min) tient bien dans la durée sans conflit, notamment si édition simultanée Mac + bot rapprochée dans le temps.

## 2026-07-19 (2) — Diagnostic rentabilité Sakura / Cercle de Vérité, fiches projet enrichies

**Contexte** : David voulait que Claude prenne connaissance en profondeur des projets [[1. PROJETS/Sakura]] (mid/high-ticket) et [[1. PROJETS/Cercle de Vérité]] (low-ticket), en explorant deux dossiers locaux hors vault : `~/Documents/Formation/Liberty Webi/` (accompagnement passé Jean & Jodi, vente Sakura par webinaire) et `~/Documents/Formation/Olivier Clovis Scaling Academy/` (accompagnement en cours, launch-funnel.com, construction du tunnel low-ticket).

**Diagnostic chiffré établi avec David** : le tunnel Sakura par webinaire (vente à froid, 497€ early bird → 997€ tarif plein) a coûté ~9100€ de pub Meta pour ~3000€ de CA encaissé (tout collecté, aucun impayé) sur 13 élèves — soit ~-6100€ net, CAC ~700€/vente proche ou supérieur au prix de l'offre. David a coupé les pubs en novembre 2025 pour stopper la fuite de trésorerie ; le projet Sakura est en pause côté acquisition depuis.

**Repositionnement stratégique en cours** (accompagné par Scaling Academy) : inverser l'entonnoir — Meta ads → [[1. PROJETS/Cercle de Vérité]] (27€ + bump 17€ + upsell1 17€ + upsell2 27€, panier moyen visé 50-60€) → upgrade vers Sakura, plutôt que vente à froid directe sur l'offre chère. Pages de vente Cercle de Vérité déjà en ligne (davidmarsac.com/cercle-de-verite), mais contenu vidéo pas finalisé et **aucune pub Meta encore créée** — c'est le chantier actuel.

**Mis en place** :
- `1. PROJETS/Sakura.md` et `1. PROJETS/Cercle de Vérité.md` enrichis avec ces chiffres réels, le statut actuel, et une "Prochaine étape" concrète (au lieu du champ vide laissé le 18/07).
- Nouvelle fiche `3. RESSOURCES/Accompagnements marketing (Liberty Webi & Scaling Academy).md` documentant les deux accompagnements, leurs méthodes, résultats, et les chemins locaux vers le matériel (`~/Documents/Formation/...`, hors vault donc non synchronisé Git/Obsidian).

**Suite prévue** : une fois le Cercle de Vérité finalisé et testé en pub, définir avec David le taux d'upgrade CDV → Sakura nécessaire pour rentabiliser le tunnel avant de relancer du budget Meta significatif — éviter de reproduire l'erreur du tunnel Sakura direct.

## 2026-07-19 (3) — Intégration du dossier ~/Documents/Formation dans le vault (outils/frameworks)

**Contexte** : David a demandé d'intégrer tout le contenu de `~/Documents/Formation` (hors vault, jamais synchronisé) comme "outils du quotidien". Dossier énorme et hétérogène (Sherpa seul fait 839 Mo) mélangeant frameworks réutilisables et archives client/admin (contrats, factures, feuilles de présence par organisme : Bruno Pellen, Deepmark, IGECOM, Sherpa, CFF-Sophie Pons, IAE Lyon, Motivalance...).

**Périmètre retenu avec David** : outils/frameworks réutilisables uniquement (pas l'administratif par organisme). Dossier Steph : extraire les outils de coaching (PCM, Drivers, CoDev, EQI), pas les factures/fichiers personnels.

**9 fiches créées dans `3. RESSOURCES/Outils & Frameworks/`** :
- Boîte à outils formation — frameworks courts (Matrice Eisenhower, SMARTEE, DIVAS, Peur, Ne dites pas-mais dites, Burger Quizz, Cartes Forces-Valeurs, Les séquences/WorkLab)
- DISC & Forces Motrices (Assessments24x7/Progressence) — outil certifié de David
- Outils de coaching (Steph) — Drivers de Kahler (**source directe du contenu Cercle de Vérité**), 4 mythes de Kahler/CoDev, Questiologie, chronotypes (Breus), syndrome de l'imposteur (Valérie Young), test hypersensibilité (Elaine Aron)
- Coaching AGORA — protocole de coaching en 7 étapes (script complet) + niveaux logiques de Robert Dilts
- High Impact Presenting (Grant Aylward) — structure RIDAC + grille d'entretien préparatoire PRO/PERSO
- Fast&Slow (Kohlmann) — création de promesse/offre, entretien exploratoire, script de traitement des objections en vente
- Systeme.io — templates lead magnet et tunnel de vente, **contient le détail complet et récent des 6 modules S.A.K.U.R.A.** et un funnel par appel (Calendly) différent du webinaire
- Content Creator — storyboard vidéo pub (Attention Getter / Gap / Bridge)
- (+ nouvelle fiche accompagnements marketing créée en session (2) le même jour)

**Découverte importante non résolue** : le questionnaire copywriting Systeme.io décrit un funnel Sakura par **appel de vente (Calendly)**, différent du funnel webinaire documenté précédemment (qui a généré le déficit de -6100€). Il n'est pas clair si ce funnel Calendly a été testé, remplace le webinaire, ou est une version parallèle jamais lancée — **question posée à David dans `1. PROJETS/Sakura.md`, réponse en attente**.

**Suite prévue** : le reste du dossier Formation (frameworks noyés dans les dossiers clients — Bruno Pellen/méthode Bushido, CFF Sophie Pons/leadership féminin, IGECOM, Sherpa, Motivalance) n'a pas encore été traité — proposé à David de continuer ou de s'arrêter là.

## 2026-07-20/21 — Rapatriement des notes iOS (Notes.app / iCloud)

**Contexte** : David a demandé d'intégrer ses notes iOS (synchronisées via iCloud sur ce Mac, ~40 dossiers, jamais reliées au vault) au même titre que le dossier Formation. Périmètre validé : business/outils uniquement, mais y compris le dossier catch-all "Notes" à trier moi-même en signalant tout contenu manifestement privé.

**Méthode technique** : export via AppleScript (`osascript`, propriété `body` puis nettoyage HTML — attention, l'encodage de sortie du `write` d'AppleScript est **MacRoman**, pas UTF-8, à décoder explicitement sinon les accents sont corrompus). Les corps de notes contiennent des images en base64 énormes (143 Mo bruts pour ~250 notes) ; un script Python les strip pour ne garder que le texte (~0,1 Mo net).

**⚠️ Alerte sécurité signalée à David** : la note "ADMIN" du dossier iCloud **Level-Up Attitude** contient en clair des secrets actifs — token OAuth Claude complet, token du bot Telegram, IBAN Qonto + clé de récupération, codes de récupération Hotmail/davidmarsac.com, clé de licence MacWhisper, ID/secret client Google Cloud. **Rien de tout cela n'a été copié où que ce soit.** Recommandation donnée : migrer vers un gestionnaire de mots de passe, envisager de régénérer le token Claude et le token Telegram par précaution.

**Constat majeur** : la quasi-totalité des dossiers "clients" (Bruno Pellen, CCf Sophie Pons, Deepmark, IGECOM, SHERPA, Motivalance, Banque Savoie, Coaching(s), Codev, Module 0-9, Olivier Clovis...) ne contiennent quasiment que des **notes manuscrites capturées en photos** — texte non extractible sans OCR. Seuls quelques nuggets de texte réel ont pu être récupérés.

**3 fiches créées** dans `3. RESSOURCES/` :
- `Outils & Frameworks/SHERPA - méthode MENTOR (feedback managérial).md` — trame de débriefing manager après entretien de vente
- `Outils & Frameworks/Prompts IA business - bibliothèque de prompts.md` — 7 prompts ChatGPT/Claude prêts à l'emploi
- `Contenu & idées capturées (notes iOS).md` — pegs mnémotechniques, brainstorm "zone de confort", notes de conférence Thomas d'Ansembourg, liste TedX, liste de lecture, **script archivé de la première partie de Fabien Olicard**

**Contenu personnel volontairement exclu du vault** (dossier "Notes" catch-all, mélangé au contenu business) : deck Magic the Gathering, idées cadeaux Steph, watchlist films/séries, notaire/immobilier/locataires, idée d'appli perso (magie), une liste de couple très intime, un message à Audrey sur Emilie, une lettre personnelle d'un tiers — rien de tout cela n'a été copié dans le vault.

**Découverte à faire confirmer par David** : deux notes évoquent un **projet de livre** (CNV/triangle de Karpman, et/ou "la zone de confort, cette prison aux barreaux dorés") non documenté ailleurs dans le vault — pas encore créé de fiche projet, en attente de confirmation.

**Suite prévue** : proposer à David l'OCR des notes manuscrites (photos) si le volume de contenu perdu (débriefs clients, séances de coaching) lui semble valoir l'effort d'outillage supplémentaire.

## 2026-07-21 — Découverte confirmée : le projet livre existe bel et bien

David a confirmé et fourni le manuscrit (`~/Downloads/Le Triangle Dramatique - _Quand ça fait du bien de faire du mal_.docx`) évoqué la veille via les notes iOS. Livre co-écrit avec **Stéphanie Kapp**, sur le triangle de Karpman/CNV/consentement.

**Créé** :
- `1. PROJETS/Livre - Le Triangle Dramatique.md` — sommaire complet, statut par section (Consentement très rédigé ; Triangle de Karpman, Émotions et Conclusion encore à l'état de plan)
- `3. RESSOURCES/Outils & Frameworks/Consentement et amortisseur d'ego - concepts originaux du livre.md` — les frameworks originaux extraits (amortisseur d'ego, méthode des 3 feux tricolores, acronyme DESIR, biais de consentement) réutilisables en formation indépendamment du livre

**Suite prévue** : demander à David quelle section du livre prioriser en rédaction, et l'échéance/piste de publication visée.

<!-- Les entrées de session s'ajoutent ici, de la plus récente à la plus ancienne -->

## 2026-07-22 — Architecture Mac / GitHub / serveur

Architecture cible validée :
- Le vault Obsidian principal vit maintenant en local sur le Mac dans :
  /Users/DavidM/Documents/IA/Vault-Mnemonaute
- Ce vault local est un dépôt Git relié à :
  git@github.com:DavidMSakura/cerveau-vault.git
- Le serveur utilise désormais un clone de ce même repo dans :
  /home/david/Doc
- Le bot Telegram Doc lit le vault serveur /home/david/Doc.
- Le Mac pousse les changements vers GitHub.
- Le serveur récupère les changements depuis GitHub.
- GitHub devient le pont entre Obsidian local, Claude Code local et le bot Telegram serveur.
- L’ancien /home/david/Doc serveur a été sauvegardé avant remplacement.
- Ne pas confondre :
  - /home/david/Doc = clone serveur du vault GitHub
  - /home/david/telegram-claude-bot = code du bot Telegram et automatisations
  - /Users/DavidM/Documents/IA/Vault-Mnemonaute = vault Obsidian local principal sur Mac

## 2026-08-08 — Dossier administratif MyConnecting + refonte de la plaquette commerciale

**Dossier MyConnecting** : balayage complet du Mac pour les 11 pièces demandées par MyConnecting, copiées dans `~/Documents/Formation/MyConnecting/_Admin` avec un récapitulatif `00 - INFOS A TRANSMETTRE (recap).md`. Trouvés : CNI recto/verso (valide jusqu'au 24/06/2035, photos redressées à 180°), RIB Qonto, CV, photo d'identité, 3 diplômes (BTS Banque A 2007, BTS Banque B 2008, ITB CFPB 2011) + certifications DISC et Coach AGORA, récépissé NDA, Kbis à jour.

**Points administratifs identifiés** :
- **NDA à utiliser : 84691738269** (Level-Up Attitude, juillet 2020) — à ne pas confondre avec 84691719569, qui est celui de David en nom propre (avril 2020). C'est le 738269 qui figure sur tous les BPF depuis 2021.
- **SIRET incertain** : 88306507000012 date du récépissé de 2020, quand le siège était à Craponne. Le siège est désormais 4 rue du Prieuré, 69130 Écully (Kbis à jour au 30/06/2026), ce qui implique probablement un nouveau NIC. À vérifier sur annuaire-entreprises.data.gouv.fr avec le SIREN 883 065 070.
- **Attestation de vigilance URSSAF inutilisable** : celle du 01/12/2025 est périmée (validité 6 mois) et établie au nom de David en nom propre, pas de Level-Up Attitude. À retélécharger au nom de la société.
- **Qualiopi : aucun certificat sur le Mac**, seulement une fiche explicative générique. Conclusion la plus probable : non certifié.
- La société est passée de SASU à **SARL à associé unique** (Kbis 2026 vs 2024).

**Refonte de la plaquette commerciale** : l'ancienne (`Plaquette CV David Marsac.pptx`, 2023) vendait le positionnement quitté — 8 références bancaires sur 9, titre « Formateur – Coach certifié », diplômes en tête, et aucune mention du titre de champion du monde, des 50K abonnés YouTube ni de la première partie de Fabien Olicard.

Nouvelle plaquette 2 pages A4 créée en HTML imprimable : `~/Documents/Formation/_Level-Up Attitude/Plaquette David Marsac 2026.html` (ouvrir, Cmd+P, enregistrer en PDF). Positionnement retenu avec David : **« Prise de parole en public · Confiance en soi · Management »**, avec le chanbara comme preuve centrale. Accroche validée par lui : « Double champion du monde de sabre japonais. J'enseigne ce que le combat m'a appris : atteindre des objectifs ambitieux, surtout quand la pression monte. » Structure : page 1 = accroche, 3 preuves chiffrées, 3 offres, méthode ; page 2 = mission signature (Crédit Foncier), références groupées, parcours, certifications. Direction artistique tirée de son univers : bleu de Prusse (la vague de Hokusai qu'il cite comme son symbole) et un unique trait vermillon en bord de page (la coupe de sabre).

**Décision actée** : la clause de confidentialité signée avec High Impact (Grant Aylward) en 2025 s'applique — les clients finaux de Grant (Ferrero, DS Smith) ne sont pas nommés dans la plaquette, formulation neutre retenue. David a confirmé que Grant préfère qu'on l'applique. Question tranchée, ne pas la rouvrir.

**Plaquette — version finale** : livrée en deux formats côte à côte, `Plaquette David Marsac 2026.html` (imprimable A4 via Cmd+P) et `Plaquette David Marsac 2026.pptx` (A4 portrait, 2 diapos, éditable). Attention : le pptx utilise Avenir Next, qui n'existe que sur macOS — exporter en PDF avant tout envoi à un tiers sous Windows.

**Principe éditorial retenu** : la plaquette part en même temps que le CV, donc tout ce que le CV fait mieux en a été retiré — bloc « Parcours » chronologique supprimé (doublon intégral), diplômes supprimés (BTS Banque, ITB, Licence LEA), remplacés par un renvoi « voir CV joint ». Ne restent que les **habilitations** qui autorisent un outil (Coach AGORA, DISC & Forces Motrices), parce qu'elles servent d'argument commercial et pas de biographie. La place libérée a servi à un bloc « Modalités d'intervention » (formats, langues, lieux) et à la mention du livre en cours.

**Un seul verbatim, volontairement** : David trouve l'exercice d'auto-citation inconfortable. Celui retenu (participante prise de parole, « un avant et un après… j'ai changé de métier… j'étais pétrifiée ») raconte une transformation concrète et vaut mieux que plusieurs compliments empilés. À noter : aucun organisme n'a demandé de verbatims, c'était une proposition commerciale de ma part — David a eu raison de vérifier avant d'accepter.

**Suite prévue** : vérifier le SIRET et retélécharger l'attestation URSSAF au nom de la société. Le CV, lui, reste à réécrire — celui du dossier s'arrête en 2018 et porte l'adresse de Craponne.

## 2026-08-08 (2) — SIRET vérifié, CV 2026 réécrit

**SIRET tranché** : le bon est **88306507000038** (siège Écully, actif depuis le 01/09/2025), vérifié via l'API officielle `recherche-entreprises.api.gouv.fr`. L'ancien **88306507000012 est fermé** — c'était le siège de Craponne. Ne plus l'utiliser (conventions, factures, BPF).

**Attestation URSSAF — point non résolu** : celle du 08/08/2026 est à jour côté date, mais reste établie au nom de **MR MARSAC DAVID en nom propre** (n° 8248539D01, adresse Tassin), pas de Level-Up Attitude. C'est l'attestation du gérant au régime des indépendants. Si la SARL n'a jamais eu de salarié, elle n'a probablement pas de compte employeur URSSAF et donc pas d'attestation de vigilance à son nom — à expliquer au demandeur le cas échéant.

**CV 2026 réécrit** — `~/Documents/Formation/_Level-Up Attitude/CV David Marsac 2026.html` (+ `.pdf`, 2 pages A4). Le fichier `04 - CV David Marsac 2026.pdf` qui existait n'était que l'ancien CV avec l'adresse corrigée : il s'arrêtait en 2019 et dupliquait le bloc formation. Le nouveau reprend la direction artistique de la plaquette (bleu de Prusse, trait vermillon, mêmes polices) et se pose en **complément** d'elle : la plaquette vend, le CV prouve. Page 1 = accroche + parcours chronologique (Level-Up Attitude, Allianz, Fiducial, BNP Paribas regroupé 2004-2015) ; page 2 = diplômes, compétences, encart chanbara, livre/YouTube/scène.

**Ajouts demandés par David en séance** : le **codéveloppement** (il anime et participe à un groupe fermé, une séance par mois) et l'**IA** (pratique quotidienne, conception d'assistants et d'agents, automatisation de la veille et du contenu — systèmes développés et exploités en propre). Les deux figurent au CV.

**Pipeline de génération PDF** : le HTML est monté depuis un template avec la photo injectée en base64, puis rendu par Chrome headless (`--headless --print-to-pdf --no-pdf-header-footer`) — pas besoin de passer par Cmd+P. Attention, contrairement à la plaquette, le CV embarque son propre reset CSS et un `<!doctype html>` : sans ça les marges par défaut du navigateur cassent la mise en page hors contexte Artifact.

**Note de cadrage** : David a signalé en fin de session que le dossier de pièces justificatives MyConnecting n'est plus le sujet actif — ne pas relancer les points RIB/URSSAF/Qualiopi de sa propre initiative.

## 2026-08-08 (3) — Intégration de l'entretien Tony Robbins dans le vault

**Source** : `https://youtu.be/I_w81rptxkc` — *Tony Robbins: No One Is Ready For What's Coming (The truth about AI)*, The Diary of a CEO (Steven Bartlett), 15/01/2026, 2 h. Le titre parle d'IA mais l'IA n'occupe que le premier quart : le vrai sujet est comment un humain change.

**Récupération de la transcription — méthode qui marche enfin** : `yt-dlp` installé dans un venv **sur le Mac** récupère les sous-titres auto (`--write-auto-subs --sub-langs en --sub-format json3`, avec `--extractor-args "youtube:player_client=android,web"` car le client web par défaut échoue). Le json3 se nettoie en paragraphes horodatés avec dédoublonnage des sous-titres roulants. C'est la solution au problème resté ouvert depuis le 18/07 (Firecrawl ~20 % de réussite, yt-dlp bloqué depuis l'IP du serveur) — **le blocage venait de l'IP serveur, pas de l'outil**. Depuis le Mac, ça passe.

**5 notes créées / enrichies** :
- `3. RESSOURCES/Transcriptions YouTube/Tony Robbins - No One Is Ready For What's Coming (Diary of a CEO).md` — synthèse complète en 10 sections, avec une section « Limites » (il affirme beaucoup, source peu ; biais de survivant sur la partie argent).
- `3. RESSOURCES/Outils & Frameworks/Les 6 besoins humains (Tony Robbins).md` — les 6 besoins, la règle d'addiction (un comportement qui satisfait 3 besoins sur 6 devient une addiction), le piège significativité→amour, les problèmes comme drogue, et **l'exercice des 2 besoins dominants** utilisable tel quel.
- `3. RESSOURCES/Outils & Frameworks/Strategy Story State et le levier (Tony Robbins).md` — la séquence inversée (State → Story → Strategy), le levier (« le changement n'est jamais une question de capacité »), le conditionnement, et le protocole d'état de Robbins avant scène.
- `3. RESSOURCES/Outils & Frameworks/Saisons de la vie et les 3 niveaux de patterns (Tony Robbins).md` — les 4 saisons, les 3 niveaux de patterns, créer vs gérer, la mécanique d'apprentissage (immersion, chunking, micro-learning, modélisation).
- `2. CASQUETTES/Société.md` — section « Matière Tony Robbins pour les formations », ventilée par offre du catalogue.

**Tags posés** (demande explicite de David) : `tony-robbins`, `coaching`, plus `developpement-personnel`, `mindset`, `outil-formation`, `prise-de-parole`, `sakura`, `cercle-de-verite`, `ia`, `apprentissage`, `transcription`. Le vault n'utilisait quasiment pas de tags avant (seules les casquettes en avaient un) — c'est le premier ensemble structuré.

**Deuxième volet demandé — brancher ça sur les futures sessions de conception** :
- `6. PROCESS/Conception de programme transformationnel...` : nouvelle section **3 bis, une grille en 6 points** à dérouler systématiquement (ordre State/Story/Strategy · quels 3 besoins le problème satisfait et par quoi on les remplace · où est le levier · où sont l'immersion et le conditionnement · calibrage du chunking · niveau d'influence visé). C'est le mécanisme durable : cette fiche est relue à chaque session Sakura/CDV.
- `1. PROJETS/Cercle de Vérité.md` : le piège significativité→amour décrit le client cible avec précision. **Test le plus dur à passer avant de tourner les vidéos** : la sur-adaptation satisfait au moins 3 besoins (connexion, significativité, certitude d'être accepté) — par quoi les remplace-t-on ? Un programme qui demande juste d'arrêter échoue.
- `1. PROJETS/Sakura.md` : critère de réussite du parcours autonome = l'échelle d'influence de Robbins, niveau 3-4 (« changer l'état d'un groupe sans être là ») — pas « du contenu vendu en ligne ». Et **objection sérieuse au découpage hebdomadaire** : Robbins défend l'immersion (l'état de pic verrouille le souvenir). Piste notée : concentrer sur 1-2 temps forts immersifs, réserver l'étalement au conditionnement.

**Repère personnel noté dans les fiches** : à 48 ans, David est en **début d'automne** dans les saisons de Robbins — la saison de la récolte et du levier, pas celle du travail en force. Cohérent avec son objectif de dissocier temps et revenus.

**À faire remonter à la prochaine session sur les offres** : la question ouverte la plus utile est celle du remplacement des 3 besoins dans le Cercle de Vérité — elle n'est pas tranchée.

## 2026-08-08 (4) — Ads Meta du Cercle de Vérité + nouvelle architecture Sakura

**Contexte** : David travaille avec un prestataire/outil qui produit ses ads Meta et lui a envoyé un questionnaire en 7 points sur l'offre low-ticket. Réponses rédigées à partir des sources réelles (page de vente en ligne, bible de production, mécanisme unique, compilation des scripts).

**Matière du Cercle de Vérité — où elle est** : tout est dans `~/Documents/Formation/Olivier Clovis Scaling Academy/`. Les deux documents à ouvrir en priorité pour toute question pub sont `Contenus/Low Ticket - Le Cercle de Vérité/🔥 MÉCANISME UNIQUE Nice Guy.docx` (le positionnement) et `.../Parcours/COMPILATION — Scripts Cercle de Vérité (base créas pubs).docx` — 26 000 mots compilés le 06/08/2026, contenant un index publicitaire déjà prêt (mécanisme, douleurs, punchlines, séquences, objections). C'est cette compilation qu'il faut fournir comme matière brute, pas les scripts jour par jour.

**Règle éditoriale non négociable des créas** (extraite de la compilation, à rappeler à chaque fois) : l'accusation « tu manipules » n'est supportable que dans une bouche qui se l'applique d'abord à elle-même. **David doit porter l'accusation sur lui-même avant de la retourner vers le spectateur** — « je manipulais » avant « tu manipules ». Sans ça le crochet devient une agression et les signalements Meta grimpent.

**Avatar — ne pas confondre avec celui du webinaire Sakura** : le webinaire visait 50 % hommes / 50 % femmes. Le Cercle de Vérité est **exclusivement masculin, 35-60 ans**, hypersensibles, en reconstruction (rupture *ou* séparation *ou* reconversion *ou* épreuve — la rupture n'est qu'un cas parmi d'autres, ne pas réduire le ciblage à « post-rupture »). Conscient du symptôme (« je n'arrive pas à dire non »), **pas du mécanisme** — c'est ce qui fait la force du crochet.

**Correction importante à mon analyse** : j'avais conclu de l'inventaire local que les 7 vidéos quotidiennes n'étaient pas tournées et recommandé de repousser les ads. **Faux — elles sont tournées**, simplement hébergées côté Systeme.io et absentes du Mac. Ne plus déduire l'état de production du contenu local.

**Statut réel du Cercle de Vérité** : produit complet et prêt. **Premier test Meta prévu avec un budget limité à 200€** — test d'apprentissage (CPM, CTR, coût par achat, taux de prise bump/upsells), pas un lancement. Le prix de l'upsell 1 n'est toujours pas arrêté (versions 17/27/47/57€ toutes tournées).

**Sakura — nouvelle architecture cible (pas encore implémentée)** : Sakura devient une offre à **deux niveaux** — un **mid-ticket à ~997€** avec coachings collectifs uniquement, et un **high-ticket à ~2500€** avec accompagnement total et coaching individuel premium. Le « 2500€ » qui figurait dans le questionnaire du prestataire n'était donc pas une erreur mais la cible de David. Rien n'est construit : ni contenus différenciés, ni pages, ni mécanique de passage. **Le calcul du taux d'upgrade nécessaire depuis le Cercle de Vérité est à refaire une fois les deux niveaux définis** — il change radicalement selon le niveau visé.

## 2026-08-16 — Scraping de la chaîne Mnemonaute et intégration du corpus mémoire dans le vault

**Contexte** : le vault ne contenait aucune fiche mnémotechnique — le cœur de métier de David était le seul domaine absent, alors que dix ans de contenu existaient sur YouTube sans être exploités ailleurs que dans les vidéos.

**Récupération technique** : `yt-dlp` réinstallé dans un venv durable `~/.local/venv-ytdlp/` (le venv du 08/08 vivait dans un scratchpad de session, perdu depuis). Les 58 vidéos de la chaîne récupérées en une passe avec `--write-auto-subs --sub-langs fr --sub-format json3 --write-info-json` et `--extractor-args "youtube:player_client=android,web;lang=fr"` — le `lang=fr` est indispensable, sans lui yt-dlp renvoie les titres auto-traduits en anglais. Script de conversion json3 → markdown conservé en `~/Documents/Mnemonaute/build_scripts.py`, métadonnées en `catalogue.json`.

**Livré hors vault** : `~/Documents/Mnemonaute/Scripts/` — 58 verbatims (un par épisode, en-tête avec lien/date/durée/vues/mots), **130 947 mots**, plus un `_LISEZ-MOI.md`. Format choisi par David : verbatim nettoyé, pas de réécriture.

**⚠️ Limite importante à connaître** : coupure de qualité nette au niveau de l'année 2022. Les 7 vidéos de 2025-2026 sont ponctuées et propres ; **les 50 vidéos de 2016 à 2022 viennent de l'ancien moteur ASR de YouTube — aucune ponctuation, aucune majuscule, erreurs de reconnaissance lourdes** (« mes mots notres » = Mnemonaute, « Dominique Aubri » = Dominic O'Brien). Exploitables pour retrouver le fond, pas pour être citées. Une seule vidéo est sans sous-titres du tout : l'épisode 2 de 2016, qui est justement le fondateur sur le palais de mémoire. Piste non lancée : MacWhisper (installé sur le Mac) donnerait bien mieux, mais le Mac est en Intel donc plusieurs heures de calcul — proposé à David, pas décidé.

**Livré dans le vault** — nouveau dossier `3. RESSOURCES/Mémoire & Apprentissage/` avec 14 fiches, structurées **par technique et non par vidéo** (choix de David) : les fondamentaux (ALI de Dominic O'Brien + les 5 secrets), l'arbre de décision, la table de rappel 1→53 (celle de David et celle de Fabien Olicard), le Grand Système, le palais de mémoire, les prénoms, apprendre par cœur, la révision/courbe d'Ebbinghaus, Pomodoro & Flowtime, le mind mapping, la lecture rapide, le calcul mental, les trous de mémoire à l'oral, la prise de parole sans notes, le versant développement personnel, plus le catalogue des 58 vidéos et un `_INDEX`.

**Trois découvertes importantes** :

1. **Un deuxième projet de livre existe** — `1. PROJETS/Livre - Libérez votre mémoire.md` créé. Dossier `~/Documents/Mnemonaute/1. Libérez votre mémoire - Le Livre/` : chemin de fer complet en 9 chapitres, avant-propos rédigé (~3 400 mots), chapitre 1 amorcé, le reste non écrit. **La préface de Fabien Olicard est marquée comme accordée** dans la note d'intention. Le sous-titre du livre — « une mémoire illimitée pour gagner en confiance en soi » — annonçait déjà le pivot que la chaîne a fait en 2025. Ne pas confondre avec [[Livre - Le Triangle Dramatique]] : ce sont deux livres différents, et aucun des deux n'avance. **Questions posées à David, en attente** : projet vivant ou abandonné ? préface toujours valable ? lequel des deux passe en priorité ?

2. **Le chapitre 9 du livre décrit déjà l'idée que David a eue avec l'étudiant** (enchaîner plusieurs méthodes) : « association de 3 méthodes — écrire puis synthétiser, mindmap, palais mental ». Jamais filmé.

3. **Plusieurs promesses faites à l'antenne n'ont jamais été tenues**, et ce sont les meilleures idées de contenu disponibles : la vidéo détaillant la mémorisation de la première partie de Fabien Olicard (annoncée en mars 2020), le mind map de toutes les techniques (annoncé en 2017, ferait un lead magnet évident), la table de rappel au-delà de 53, le calcul mental avancé, la vidéo sur le syndrome du bon élève.

**`1. PROJETS/Idées de contenu Mnemonaute - backlog.md`** créé : backlog argumenté (série « enchaîner les méthodes » en priorité 1, dettes éditoriales, recyclage, angles neufs), chaque idée avec le manque comblé, la cible et le lien funnel.

**Constat stratégique posé dans le backlog et dans la casquette Contenu & Médias** : les 5 vidéos les plus vues (1,18 M sur 1,89 M de vues cumulées) sont toutes des applications concrètes — Rubik's cube (573 k), multiplications (193 k et 184 k), lecture rapide (122 k). Les vidéos de mindset pur publiées depuis 2025 font **~900 vues**. Le pivot mémoire → confiance ne prend pas auprès de l'audience historique tant qu'on abandonne la promesse concrète ; il faut la garder en surface (titre, miniature, 8 premières secondes) et placer le mindset dedans. Et **une seule vidéo sur 58 renvoie vers un lead magnet du funnel** — celle des 10 leçons, dont le point 11 (« ne t'efface pas, ne dis pas toujours oui ») est exactement l'avatar du Cercle de Vérité. Ajouter ce CTA aux 5 vidéos les plus vues coûte une après-midi.

**Suite prévue** : trancher le sort des deux livres ; décider si on relance MacWhisper sur les anciennes vidéos ; attaquer le backlog par la vidéo « Réviser un partiel en 5 jours » ou par la dette Fabien Olicard.

## 2026-08-17 — Complétion du corpus Mnemonaute et réorganisation

**Point de départ** : David a fait remarquer que si j'avais su déchiffrer les références déformées dans les verbatims, je devais pouvoir faire les corrections. Deux mises au point ont été nécessaires. D'une part **cette session ne portait pas la conversation du 16/08** — j'ai reconstitué en relisant `memory.md` et les fiches. D'autre part **je n'avais pas de table de correspondances** : j'avais déchiffré les noms au fil de la lecture, pas constitué un inventaire.

**Erreur de ma part corrigée** : « Dominique Aubri » n'existe nulle part dans le corpus — je l'avais inventée comme exemple dans le `_LISEZ-MOI`. Les vraies déformations sont « dominique o'brian » (1 fois) et « dominique système » (4 fois), toutes dans la FAQ #1 du 06/07/2018. Le `_LISEZ-MOI` a été corrigé avec des exemples réellement présents.

**⚠️ MacWhisper est définitivement écarté** : David l'a testé, environ **une minute de calcul pour une seule phrase** sur son Mac Intel. Inenvisageable sur 130 947 mots. Je l'avais proposé deux fois sans le savoir. Le `_LISEZ-MOI` le recommandait encore : corrigé. **Ne plus jamais le proposer** — la correction des verbatims se fait à la lecture.

**Trois décisions de David** : (1) le Rubik's cube et le code de la route restent dans le corpus technique — « on transforme une technique en quelque chose d'utilisable et de pragmatique » ; (2) une seule fiche géographie pour les trois cas, c'est la même technique appliquée à des usages différents ; (3) le développement personnel sort de `Mémoire & Apprentissage` — ça ne relève pas de la mémoire, ça va dans les outils de coaching.

**Audit réalisé** : croisement des 58 scripts avec les 14 fiches existantes. **L'angle mort principal était le Rubik's cube** — 681 196 vues sur deux vidéos, soit plus d'un tiers de l'audience cumulée de la chaîne, et aucune fiche : il n'apparaissait que dans une phrase de comparaison statistique.

**Neuf fiches créées** dans `3. RESSOURCES/Mémoire & Apprentissage/` : Rubik's cube sans algorithme · Mémoriser la géographie · Vocabulaire et langues étrangères · Code de la route et permis moto (enchaîner les techniques) · Retenir les dates · Retenir les itinéraires et les indications · Le Dominic System · Hygiène de la mémoire · Mythes et idées reçues sur la mémoire · Erreurs, oublis et limites de la méthode. Le dossier passe de 14 à 24 fiches et de ~19 000 à **32 200 mots**. La fiche Réviser a reçu une section Anki qui manquait. `Développement personnel - la matière Mnemonaute` déplacé dans `Outils & Frameworks/`.

**Découverte importante** : la vidéo **Code de la route et permis moto (2018)** est déjà la démonstration complète de l'idée « enchaîner les méthodes » portée en priorité 1 du backlog et annoncée au chapitre 9 du livre. Sur un seul examen David utilise quatre techniques selon la nature de ce qu'il a à retenir — mind map pour les 12 fiches, une table de rappel improvisée sur la carte du jeu Risk pour les 8 familles de risques, la table des symboles pour les 5 modifications interdites, un palais de mémoire pour les motifs de refus d'assurance. **La série du backlog n'est donc pas à créer de zéro : il y a un cas complet à re-monter**, avec un public de recherche permanent (les gens qui passent le permis).

**Lecture des 107 692 vues de la deuxième vidéo Rubik** : ce n'est pas une perte de 80 %, c'est le seul endroit du catalogue où l'audience a montré une **intention de progression**. Les deux vidéos multiplications (193 k et 184 k) sont deux captations de recherche indépendantes ; le couple Rubik est un vrai escalier — les gens sont revenus après 42 minutes de tutoriel. Un taux de 19 % sur une suite est élevé. **À vérifier dans YouTube Studio** (part du trafic de la vidéo 2 venant des suggestions de la vidéo 1) avant d'en faire une stratégie. L'escalier s'arrête là : la méthode Fridrich est promise et jamais faite.

**Dettes éditoriales nouvellement identifiées** (à verser au backlog) : la masterclass « dates de la Seconde Guerre / guerre froide » proposée en 2016 avec appel à commentaires ; la vidéo « retenir un numéro de téléphone » explicitement promise fin 2016 ; la liste d'associations du Dominic System annoncée en 2018 ; la compilation des images proposées par la communauté, jamais publiée alors qu'il dit en avoir intégré plusieurs ; l'Europe des 28 devenue obsolète depuis le Brexit (recyclage évident) ; le format FAQ arrêté à l'épisode 3 alors qu'il répond à des recherches à fort volume.

**Reste à faire** : éclater `Développement personnel - la matière Mnemonaute` en fiches séparées dans `Outils & Frameworks/` — méditation, pleine conscience, accords toltèques, matrice Eisenhower, auto-évaluation, la bonne question, les 3 phrases. Aujourd'hui les huit vidéos sont compressées en 1 634 mots, c'est un résumé et pas une ressource exploitable, alors que c'est exactement la matière de Sakura et du Cercle de Vérité.

**Deuxième volet du 17/08 — éclatement du développement personnel.** Quatre fiches créées dans `3. RESSOURCES/Outils & Frameworks/` : *Surmonter ses peurs, décider et agir (2019)* · *Méditation et pleine conscience (2019)* · *Accords toltèques et grille d'auto-évaluation (2022)* · *Poser de bonnes questions et soutenir quelqu'un*. La fiche mère `Développement personnel - la matière Mnemonaute` devient un index vers ces quatre-là. Seule la matrice Eisenhower reste sans fiche dédiée : elle est déjà dans `Boîte à outils formation - frameworks courts`.

**Trouvailles de ce volet** :
- **L'anecdote du vestiaire de chanbara (Bordeaux)** est la meilleure histoire de scène de tout le corpus, et elle est enterrée dans une vidéo de 2019 à 7 400 vues. Deux jeunes se préparent, l'un dit « je tombe contre David, il est ceinture noire, je vais me faire éclater » — David se présente, puis décide consciemment ce jour-là que **« le mec dangereux, c'est moi »**, enlève la boule au ventre et la donne symboliquement à ses adversaires. Son subconscient ne la lui a plus jamais imposée. Utilisable en formation prise de parole, dans Sakura, et en vidéo courte.
- **La physiologie de la peur** y est expliquée en détail (doigts froids, ventre noué, « petit pipi de la peur ») avec un renversement fort : **on confond la cause et la conséquence** — ce n'est pas parce que le corps réagit qu'on a peur, c'est parce que le cerveau a identifié une situation inconfortable. Module de formation prêt à l'emploi.
- **David parle de son propre rapport au jeu vidéo** dans cette vidéo de 2019 : « par défaut j'allumais, je jouais une ou deux heures, et je me disais : ça m'a apporté quoi ? J'ai perdu une ou deux heures de ma vie. » Sa règle : le problème n'est pas le jeu, c'est le **défaut**. À rapprocher du signal d'alerte Magic Arena.
- **La méditation qu'il enseigne est entièrement laïque et opérationnelle**, héritée des arts martiaux : un sas d'entrée et de sortie applicable à une réunion comme à un entraînement, avec une posture décrite au détail près. L'anecdote des parents d'élèves qui craignaient « l'embrigadement spirituel » est la réponse toute faite à l'objection en entreprise.
- **La grille d'auto-évaluation en quatre questions** (2022) est le livrable le plus directement vendable du versant coaching. Et son idée à contre-courant : **il est plus difficile d'apprendre de ses réussites que de ses échecs**, parce qu'on ressasse les échecs et qu'on passe vite sur les réussites.
- Il annonce en 2019 une **émission spéciale sur le fait de noter tout ce dont on se souvient avant de réviser** (le *testing effect*) : jamais faite.
- La question signature **« c'était quand la dernière fois que tu as fait quelque chose pour la première fois ? »** est déjà en clôture des vidéos de janvier 2022 — soit trois ans avant la vidéo sur la zone de confort.
- Les deux meilleures vidéos du versant coaching (*Qu'est-ce qu'une bonne question* et *Les 3 phrases*) font **670 et 980 vues**. C'est le constat du pivot en chiffres : le meilleur contenu actuel n'atteint pas l'audience historique.

**État final du corpus** : `Mémoire & Apprentissage` compte 24 fiches et 32 200 mots (contre 14 fiches et ~19 000 mots avant la session). Les 58 vidéos sont désormais toutes exploitées, à l'exception des deux vidéos promotionnelles Memory Box (unboxing et jeu concours), qui n'ont pas de contenu technique.

## 2026-08-23 — Reprise du chantier MyConnecting : correction du cas Vernet, déroulé pédagogique et supports PowerPoint

**Priorité redonnée par David** : terminer MyConnecting. La formation **Radiance Mutuelle (agence de Dijon), les 16 et 17 septembre 2026**, était le vrai travail en cours — pas le dossier administratif, qui reste ouvert sur trois points (URSSAF au nom propre, RIB adresse Craponne, Qualiopi introuvable) mais dont il n'a pas demandé la reprise.

**Contacts MyConnecting** : **Maori Chunnee** (m.chunnee@groupemyconnecting.com), équipe Humain & RSE, gère l'onboarding et le cadrage ; **Simon Chatelain** (s.chatelain@) est le commercial qui a placé la mission. Tutoiement des deux côtés.

**Corrections de fond apportées par David au cas fil rouge Marc Vernet**, toutes justes et toutes intégrées en v4 du déroulé :
- **La décennale était mal posée.** Elle pèse sur le constructeur (art. 1792) : sur les lots que Marc réalise lui-même il n'y a pas d'attestation à réclamer, il n'y a personne derrière. Le meilleur angle de conseil est ailleurs — la **dommages-ouvrage** (L.242-1), obligatoire pour le particulier qui fait construire et presque jamais souscrite. Trois questions distinctes désormais dans la fiche, plus l'avertissement que la question « plomberie/électricité relève-t-elle de la décennale » est trop fine pour la salle.
- **L'urbanisme était présenté à l'envers.** Chenôve est en zone U d'un PLU métropolitain : à 25 m² une **déclaration préalable suffit** (seuil DP à 20 m², relevé à 40 m² en zone U). Et le lien avec l'assurance n'est pas l'autorisation mais la PJ et la surface à redéclarer.
- **25 m² ne peuvent pas contenir salon, cuisine, chambre et salle d'eau.** L'extension devient une **suite parentale** (chambre + salle d'eau), l'ancienne chambre du RDC étant réunie au salon. Ça justifie d'un coup la plomberie faite maison, le remeublement, et donne les 120 m² à opposer à la surface déclarée. La cuisine équipée disparaît du poste mobilier.
- **Le cas B tombait sur la complémentaire collective obligatoire** : un salarié du privé ne quitte pas librement sa mutuelle d'entreprise. Le client devient **Jean-Claude, 64 ans, jeune retraité** sorti du collectif à la liquidation de sa pension — validé par David. Sa variante (un salarié à quelques mois de la retraite) est notée en réserve, jugée trop longue pour douze minutes.
- Même problème sur **Marc Vernet lui-même** : il est désormais **salarié couvert par le contrat collectif Radiance de son entreprise**. David n'a pas voulu en faire un TNS — « s'ils ne sont pas à l'aise avec le pro, ça risque d'être clivant ou de générer : je le passe à un collègue spécialisé ».
- Les **trois cas courts sont étoffés** (prénoms, âge, métier, revenus, propriétaire ou locataire) pour que les participants n'aient pas à inventer en jouant.

**Deux apports de David sur la conclusion d'entretien, désormais au cœur du dispositif** :
1. **Le tremplin**, dans la présentation du J1 : « nous déciderons ensemble de ce que nous mettrons en place aujourd'hui ou lors de nos prochains rendez-vous. » C'est la partie culottée, et elle est délibérée — elle pose qu'il y aura une décision, rend le second rendez-vous normal, et installe le « nous » dont la conclusion aura besoin.
2. **Le triptyque de clôture** : « avez-vous la sensation que ce que j'ai partagé vous permettra d'atteindre votre objectif ? » · « qu'est-ce qui vous fait penser ça ? » · « quelle serait la suite logique pour vous à partir de maintenant ? » Logé dans le **E d'ACTE** (J2, 14h50, porté de 40 à 50 min en reprenant 10 min sur PACC) plutôt que dans une séquence nouvelle, pour ne pas mordre sur les trios.

Les deux forment l'**arc des deux jours** : la conclusion n'est pas un moment, c'est l'encaissement d'une promesse faite à la première minute. David vise explicitement le remplacement de « bon, je vous l'envoie par mail, si ça vous convient revenez vers moi », qu'il a trop vu.

**Trois livrables produits**, tous dans `~/Documents/Formation/MyConnecting/` :
- `Deroule pedagogique DDA Radiance.html` — passé en **v4**, c'est la source de vérité du contenu.
- `Deroule pedagogique DDA Radiance.pptx` — **15 pages, format IGECOM** (tableau 5 colonnes horaires / titre / objectifs / contenus / modalités), avec les objectifs tagués selon la **taxonomie de Bloom** et une page de progression Comprendre → Analyser → Appliquer → Créer sur les deux jours.
- `Support DDA Radiance - JOUR 1.pptx` (**44 slides**) et `- JOUR 2.pptx` (**51 slides**), avec animations d'apparition au clic.

**Charte graphique — la découverte utile** : le support `2026_Support Assertivité & Leadership_DDA.pptx` (reçu par WeTransfer le 21/07, dans le sous-dossier `wetransfer_...`) embarque le **vrai thème « MyConnecting CORPO »** — violet `#4E006E`, orange `#FF4B00`, canard `#005064`, Montserrat, 100 layouts chartés avec logos et bandeaux dégradés. Les trois livrables sont construits **directement sur ce fichier** (slides purgées, masters conservés), ce qui donne une charte exacte sans avoir à la reconstituer. Montserrat est installée sur le Mac. À réutiliser pour toute production MyConnecting future.

**Outillage** (scratchpad de session, à recréer si besoin) : `deck.py` — moteur de composants (intertitres, manifestes, consignes, cartes, étapes, encarts, tableaux, split) plus injection XML `<p:timing>` pour les animations, que python-pptx ne sait pas faire nativement. `check.py` — contrôleur de débordement qui mesure chaque zone de texte avec la vraie Montserrat via PIL, et la hauteur réelle des tableaux. Indispensable : c'est lui qui a rattrapé les débordements, et le tableau du déroulé est étiré pour remplir la page.

**Mail de cadrage** : brouillon créé dans Gmail (non envoyé), en réponse au fil Radiance, à Maori avec Simon en copie. Il relance sur la date **et** transmet les sept questions de cadrage en amont. Point d'alerte relevé au passage : David avait proposé le 31 août après-midi, le 1er septembre matin et le 2 septembre après-midi le 13 août, **et n'a jamais eu de confirmation** — la semaine proposée commence dans huit jours.

**Reste à faire** : ouvrir les trois PowerPoint pour valider le rendu à l'écran ; faire remplacer le texte de garantie du test d'entrée (J1, 9h45) par une vraie garantie Radiance ; obtenir les réponses au cadrage, notamment la conséquence factuelle d'une formule au minimum sur un deux-roues et la répartition des six informations dues, qui sont les deux seuls trous restants dans le contenu.

## 2026-08-23 (2) — Nettoyage du vault : index renommés, fantôme supprimé, identité Doc rétablie

**Les sept `_INDEX.md` sont renommés** — ils formaient autant de gros nœuds indistincts dans le graphe Obsidian. Schéma retenu par David : nom + définition courte. `Projets - ce qui a une fin` · `Casquettes - ce qui n'a pas de fin` · `Ressources - ce que je veux garder` · `Corpus Mnemonaute - toutes les techniques` · `Archives - ce qui est terminé` · `Journal - mode d'emploi` · `Process - comment je fais, moi`. Renommage fait en `git mv`, donc l'historique est conservé. Toutes les références ont suivi : le lien de `Contenu & Médias`, la skill `weekprepare` (qui excluait `_INDEX.md` du parcours des projets), et les quatre liens vers des dossiers dans `0. INBOX`. **Zéro lien cassé** après coup, vérifié.

**Le dossier `~/Documents/IA/Doc` est supprimé.** C'était le vault d'avant la migration du 19/07 : 15 notes, figé au 18 juillet, sans dépôt git. Vérification faite note par note — aucune n'était absente du vault actuel, et les quatre qui différaient étaient toutes des versions plus pauvres. Seule ligne unique de son `CLAUDE.md` : la croisière de juillet, qui a déjà sa note. Archive de secours faite avant suppression (scratchpad de session, éphémère). Il a aussi été retiré de la liste des coffres Obsidian, qui n'en compte plus qu'un.

**⚠️ « Doc » n'avait jamais été écrit nulle part.** Ni dans l'ancien CLAUDE.md, ni dans le nouveau — c'était une convention orale, ce qui explique qu'elle n'ait pas survécu à la migration. **C'est réparé** : une section « Qui tu es — Doc » ouvre désormais `CLAUDE.md`, avec la référence à *Retour vers le futur* et le positionnement voulu par David — un associé qui challenge, pas un exécutant. Le nom est donc stable pour toutes les sessions à venir.

**Graphe Obsidian nettoyé** : `CLAUDE.md` et `memory.md` masqués par un filtre (`-file:CLAUDE -file:memory`) plutôt que renommés, puisque Claude Code lit `CLAUDE.md` par son nom. `hideUnresolved` passé à `true`. Et `Bienvenue.md`, la note d'accueil par défaut d'Obsidian jamais supprimée, est effacée — elle générait à elle seule le nœud fantôme « créez un lien ».

**Point non résolu** : David dit voir un nœud nommé « Titre » dans son graphe. Introuvable — aucun fichier, aucun lien, aucun alias, aucun titre H1 de ce nom. L'hypothèse la plus probable est qu'il s'agissait de « créez un lien » ou de « Bienvenue », tous deux supprimés depuis. À reposer s'il le voit encore.

## 2026-08-24 — Supports MyConnecting v3, et automatisation des notes iOS

**Volet 1 — MyConnecting.** David a relu les trois livrables et corrigé beaucoup. Trois de ses retours ont mis au jour de vraies erreurs de ma part :

1. **Le A d'ACTE était faux.** J'avais écrit « recevoir l'objection sans la contester — *vous avez raison* ». Or valider l'objection interdit de la traiter. La formule juste est « je comprends que le prix compte pour vous ». Corrigé partout.
2. **Le point dur d'ACTE est le C, pas le T.** Le A est acquis (« je comprends » est la phrase 4×4 passe-partout), le T arrive trop tôt (syndrome du bon élève qui veut répondre juste avant d'avoir compris à quoi). Ma slide « le T est la lettre dangereuse » a été supprimée et remplacée par *Le point dur, c'est le C*, avec un iceberg en rappel et la formule de David : **« sors ta pelle et creuse »**.
3. **Le triptyque de clôture n'est pas le E d'ACTE.** ACTE traite une objection ; le triptyque conclut, avec ou sans objection préalable, et peut même l'éviter. J'avais fait un raccord logique inexistant.

**Autre correction structurelle de David, la veille** : les **deux tours de découverte** n'avaient pas de sens — une découverte déjà faite ne se refait pas, et j'avais en plus placé la révélation de la fiche complète *entre* les deux tours, ce qui faisait rejouer un client dont le groupe connaissait déjà les couches. Remplacé par **un seul entretien avec un arrêt au milieu** : découverte (25') · **stop** (20') · reprise (25') · révélation (20'), la révélation passant à la fin.

**Deux règles éditoriales posées par David, à appliquer partout** :
- Les **commentaires de formateur ne se projettent pas**. Toutes les petites lignes grises sont passées en **notes du présentateur** (7 sur le J1, 12 sur le J2).
- **Typographie française** : espace insécable avant `; : ! ?` et à l'intérieur des guillemets. Une passe automatique a corrigé ~200 runs sur les trois fichiers.

**La phrase du paperboard a changé sur sa proposition** : « ne prépare pas ce qu'il va **dire** » → « ce qu'il va **vendre** ». Elle règle la contradiction avec une journée passée à préparer son discours, elle vise le vrai défaut (arriver avec sa solution déjà choisie), et vendre/comprendre est presque une rime.

**Sept schémas natifs créés** (formes PowerPoint, éditables, à la charte) : pyramide des trois niveaux, 3C en chevrons, frise des six informations dues (J1) · schéma de la flèche, iceberg des trois couches, courbe de l'oubli tracée, arc du rendez-vous, rotation des trois rôles (J2). **L'arc n'est plus « des deux jours » mais « du rendez-vous »** — David a fait remarquer qu'il fallait parler de début et de fin de rendez-vous, pas d'hier et d'aujourd'hui : c'est la structure de leur entretien qu'on enseigne, pas celle de la formation.

Autres décisions de David : « greffier » devient **scribe** ; les acronymes portent leurs **initiales en enluminure** dans les carrés (P A C C, A C T E, T O P) et non des chiffres ; le mot « placement » est banni (il veut dire épargne en assurance).

**État final** : J1 43 slides, J2 51, déroulé pédagogique 15 pages, déroulé HTML en v6. Zéro débordement, zéro animation orpheline.

---

**Volet 2 — Les notes iOS, enfin automatisées.**

**Réponse à sa question : non, aucun cron ne les parcourait.** Le rapatriement des 20-21 juillet était ponctuel et n'a jamais été reconduit — cinq semaines de notes n'étaient jamais entrées dans le vault. Les crons existants (BOAMP, rappels, briefings) tournent sur le **serveur**, qui n'a aucun accès à Notes.app : seul un processus **local sur le Mac** peut le faire.

**Mis en place** : `~/.local/bin/vault-notes-sync.py`, lancé par un agent launchd `com.davidmarsac.vault-notes-sync` **chaque dimanche à 18h** (juste avant sa préparation de semaine). Le script exporte via AppleScript, écrit dans `3. RESSOURCES/Notes iOS/`, ne réécrit que ce qui a changé (empreintes SHA1 dans `~/.local/share/vault-notes-sync.json`), puis commit et push. Journal dans `~/.local/share/vault-notes-sync.log`.

**Deux garde-fous, et ils comptent** :
- **Liste blanche de dossiers** — 33 dossiers business. Sont exclus en dur : **Level-Up Attitude** (la note ADMIN contient toujours les secrets en clair repérés en juillet), Perso, Notes, Quick Notes, Magie, Ukulele, Whatsapp, Rdv MD.
- **Détecteur de secrets** — toute note contenant un token Anthropic/Telegram/GitHub, un IBAN, une clé d'API, un mot de passe ou des codes de récupération est **ignorée et signalée**, jamais écrite. Le vault part sur GitHub, donc l'écriture est irréversible.

**Première exécution** : 51 notes intégrées, 208 Ko, aucun secret détecté.

**⚠️ Effet de bord à trancher avec David** : le dossier **« Elisabeth Roche coaching »** est dans le périmètre, donc le **verbatim brut de son coaching personnel — dont la séance « le couple » — est désormais poussé sur GitHub**, en plus de la fiche de synthèse. En juillet, ce type de contenu avait été délibérément tenu hors du vault. À retirer du périmètre s'il préfère que seule la fiche subsiste.

**Fiche créée** : `3. RESSOURCES/Coaching Elisabeth Roche - suivi des séances.md` (1 240 mots), accompagnement hebdomadaire démarré le 20/07. Trois séances : le couple (3/08), moi et les autres (17/08), le travail (24/08). Le fil conducteur des trois est le même — **passer du faire à l'être**.

**Découverte qui compte pour le business** : la phrase du 24 août, **« arrêter de n'être qu'utile, et reconnecter à l'authentique »**, avec « être pour recevoir et non faire pour avoir », décrit très exactement le mécanisme du [[Cercle de Vérité]] — l'homme qui existe par son utilité. **David travaille sur lui la chose qu'il vend**, et sa réponse en séance (la connexion à soi, pas une autre stratégie) est une piste pour la question restée ouverte depuis le 08/08 : par quoi remplacer les besoins que la sur-adaptation satisfait ?

**Tension à trancher consciemment** : Elisabeth ouvre la séance du 24 par « plus visible marketing, réseaux, pub — **on s'en fout, sois toi-même** », ce qui frotte avec la stratégie funnel en cours (ads Meta, test 200 €). Les deux peuvent coexister — elle parle du contenu, pas du canal — mais à reposer avant le lancement.

**Tâche en attente donnée par Elisabeth le 24/08** : aller voir les croyances qui l'empêchent, et « qu'est-ce qui me ferait le plus peur ? ». Prochaine séance annoncée plus technique : carré, discipline, ressources.

**Limite technique à vérifier dimanche** : launchd doit obtenir l'autorisation d'automatiser Notes.app en arrière-plan. Si le log du dimanche indique « Notes.app inaccessible », c'est une permission à accorder dans Réglages → Confidentialité → Automatisation.

## 2026-08-24 (2) — Mémoire à deux étages, Kanban, format Sakura arrêté

**Point de départ** : David partage une conversation WhatsApp avec Stéphane et Calix sur leurs systèmes IA, et demande ce qu'on peut copier.

**Ce qui manquait vraiment, et qui est désormais en place** :
- **`Etat actuel.md`** (racine du vault, 3 674 car.) — les échéances datées, les chantiers actifs, les décisions en attente, les points de vigilance. Ne contient que ce qui est vivant.
- **Hook `SessionStart`** dans `.claude/settings.json` — il injecte ce fichier au démarrage via `jq --rawfile`, testé en réel. Le hook `Stop` existant est préservé.
- **Skill `/fin`** (`.claude/skills/fin/SKILL.md`) — résumé, écriture dans `memory.md`, retaille de `Etat actuel.md`.
- **`~/Applications/Doc.app`** — lance Claude Code dans le vault. Icône : le **condensateur de flux** de *Retour vers le futur*, générée en `.icns` (trois branches cyan, fond bleu nuit → violet).
- **`CLAUDE.md` réécrit** : l'instruction demandait de lire `memory.md` **en entier** au démarrage — 65 000 caractères, ~16 000 tokens brûlés avant la première question. Elle dit maintenant de s'appuyer sur l'état injecté et de n'ouvrir `memory.md` qu'au besoin. **C'est le vrai gain : la séparation présent / archive, pas le hook.**

**Correction factuelle à retenir** : Stéphane affirme que `/done` et `/backup` sont des commandes natives de Claude Code. **C'est faux** — ce sont des skills qu'il a créés ou qui viennent d'un plugin. Sa recommandation d'usage, elle, est reprise : lancer `/fin` à la fin de chaque session, terminal comme Telegram.

**Trello — la décision a évolué en cours de session.** D'abord écarté au profit du plugin **Kanban d'Obsidian** (cartes en markdown dans le vault, accessibles nativement, zéro API) — le tableau `1. PROJETS/Tableau de bord.md` a été créé : 24 cartes, 7 colonnes, dont une colonne « ⏳ En attente d'un tiers » qui isole ce que seul Radiance peut débloquer. Puis David signale que **Steph se met à Trello et qu'ils auront des projets communs** : Trello redevient justifié, mais **en parallèle** du vault et pour la seule **coordination** (qui fait quoi, pour quand), la matière restant dans le vault.
- **Steph n'a pas de second cerveau** : Trello sera son seul endroit, donc les cartes doivent être **autoportantes** — compréhensibles sans contexte externe.
- David : « pourquoi je collerai dans Trello, si tu peux le faire ? » — il a raison, et ça annule mon conseil d'attendre un mois : c'est précisément la friction du recopiage qui tue les tableaux partagés. **En attente de sa clé API + token**, à ranger dans le **trousseau macOS** et non dans le vault.
- **API sur le Mac** (curl suffit, j'ai Bash) ; **MCP seulement plus tard pour le bot Telegram**, qui n'a pas de shell libre.

**Sakura — le format du high-ticket est ARRÊTÉ (décision tranchée)** : **6 séances individuelles d'1 h sur 3 mois**, une tous les quinze jours, plus l'accès direct par messages **sur horaires et jours convenables**, pour **2 500 €**. Soit ~9 h → **~280 €/h**.
- Le mot **« illimité » est abandonné**. Il contredisait frontalement l'objectif de dissocier temps et revenus, et ignorait la garde alternée (vendredi soir → mardi soir). La formulation retenue est **celle que David utilise déjà dans ses pages Systeme.io** — « samedi à 23 h je ne répondrai pas ». Il l'avait donc déjà résolu sans le savoir.
- Reste à définir : le contenu différencié des deux niveaux, les pages, la mécanique de passage, le rythme des collectifs du mid-ticket.

**MyConnecting** : David a **envoyé le mail à Maori le 24/08 à 15h15**, Simon en copie, avec la relance de cadrage, les sept questions, et en pièces jointes le déroulé pédagogique et les supports J1 et J2.

**⚠️ Mon erreur, et elle est partie chez le client.** En basculant les commentaires gris vers les notes du présentateur, j'ai déplacé l'avertissement « texte d'illustration à remplacer par une vraie garantie Radiance ». Ce n'était pas un commentaire de formateur mais un **avertissement sur la validité du contenu** : la clause de dégât des eaux que j'ai inventée est donc partie sur la slide 9 du J1 **sans mention visible**. Un bandeau orange a été ajouté depuis. **À signaler à Maori ou à évoquer au cadrage** — un formateur qui signale lui-même sa zone non validée inspire plus confiance qu'un formateur dont on découvre l'approximation en réunion.

**⚠️ Deuxième erreur, réparée** : l'import des notes iOS du matin les avait déversées **sans aucun lien**, créant 52 orphelines. Sur 131 notes, **64 étaient orphelines**. Corrigé :
- **21 index de dossier** créés dans `3. RESSOURCES/Notes iOS/` (`_Index — Sakura`, `_Index — Olivier Clovis`…), chacun listant ses notes et pointant vers les fiches de référence du vault.
- Les 12 orphelines plus anciennes rattachées ([[Finances]] → Trade Republic et la vente d'appartement, [[Famille]] → croisière et séance sur le couple, le Tableau de bord → ses 8 fiches projet).
- `Etat actuel.md` rejoint `CLAUDE.md` et `memory.md` dans les fichiers masqués du graphe.
- **Résultat : 64 → 3 orphelines**, toutes techniques, zéro lien cassé.
- **La cause est traitée** : `~/.local/bin/vault-notes-index.py` est appelé par le script de synchronisation, donc les index se regénèrent chaque dimanche.

**Reste à faire** : signaler la clause d'illustration à Maori ; fournir la clé et le token Trello ; définir le contenu différencié des deux niveaux Sakura ; trancher entre les deux livres ; trancher la tension marketing ; vérifier le journal du cron notes iOS le lundi 25/08 (permission d'automatisation Notes.app en tâche de fond, non garantie).
