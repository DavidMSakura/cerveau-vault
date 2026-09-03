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

## 2026-08-25 — Icône de Doc.app, et clause MyConnecting réglée

**Session courte.** Deux points.

**L'icône de `Doc.app` ne s'affichait pas** alors que le `.icns` était valide et bien installé. Cause : **`osacompile` génère un `Assets.car`** dans `Contents/Resources/` **et une clé `CFBundleIconName`** dans l'Info.plist, et sur macOS récent les deux prennent le pas sur `applet.icns`. Correction : supprimer `Assets.car`, retirer `CFBundleIconName`, garder `CFBundleIconFile = applet`, puis `lsregister -f` et `killall Dock`. **À refaire si l'app est un jour recompilée** — `osacompile` réintroduira les deux à chaque fois.

**La clause de garantie d'illustration du J1 est réglée avec Maori** — David s'en est chargé le 25/08, hors mail (rien dans le fil Gmail). Le bandeau d'avertissement reste sur la slide 9 en attendant la vraie garantie Radiance.

**Non résolu, et non deviné** : la **date du point de cadrage Radiance** reste inconnue. Question posée à David, restée sans réponse — à reposer, car elle commande le planning et la formation est les 16-17/09.

## 2026-08-25 (2) — Les récits fondateurs, le Crédit Foncier remis à plat, et le tableau des idées

### Deux récits capturés, jamais documentés jusqu'ici

**[[Je suis la vague - le protocole d'état de David]]** — championnats du monde 2007 à Yokohama. **Sam**, un ami vivant au Japon, offre à David un **tenugui** portant la Grande Vague de Hokusai (« Sous la grande vague au large de **Kanagawa** » — David disait *Kamigawa*, qui est le plan japonisant de Magic). Flash immédiat : « je suis la vague ». Le rituel complet : sautiller, un grand saut genou levé, arriver le premier sur l'aire, occuper le territoire, le regard. Puis « Hajime », et la vague part comme un barrage qui cède. **Il devient champion du monde ce jour-là.**

> **Le mécanisme, dans ses mots : « je mettais de l'utile pour cacher l'inutile. »**
> Les sensations n'ont **pas** disparu. Ce qui change, c'est leur **attribution** : comme il venait de sauter, il était normal que ses jambes tremblent et que son cœur s'emballe. Le rituel moteur ne sert pas seulement à faire circuler l'adrénaline — il **fournit un alibi physiologique**. La peur est toujours là, elle n'a plus de preuve à exhiber. C'est de la réattribution causale, et c'est bien plus enseignable que « n'aie pas peur ».
> Exception assumée : **avant la finale**, l'enjeu était tel que la peur revenait. À dire en salle — un protocole qui avoue sa limite est cru.

**[[La politique des alliés]]** — la même finale. Face à **Diego**, un Italien qu'il n'a jamais battu. David va voir **Alain Giraud**, un rival qui le bat lui aussi mais qui a été éliminé, et qui bat souvent Diego. Alain donne le pattern : attaque à la jambe, puis feinte jambe → main. David applique **sans fioritures, sans réinventer**. Il gagne.
Trois choses rendent le récit rare : demander à quelqu'un qui vous bat (l'[[Consentement et amortisseur d'ego - concepts originaux du livre|amortisseur d'ego]] en situation), Alain qui donne sans rien y gagner, et la discipline d'appliquer tel quel.

**Les deux s'emboîtent** : l'état amène à 100 % de ses moyens, il ne fabrique pas la compétence manquante.

### ⚠️ Mes quatre erreurs, corrigées par David

1. **« Emotion = Energy in Motion »** proposé comme un apport — il l'emploie déjà à l'oral.
2. **La mise en mouvement du corps face au trac** présentée comme neuve — il l'enseigne depuis toujours, illustrée par le chanbara.
3. **Kahler apporté comme une découverte** — il le connaît mieux que moi et l'utilise dans le livre et dans Sakura. Ce n'était pas une lacune de David mais **du vault** : fiche [[Les quatre mythes de la psychologie (Taibi Kahler)]] créée, avec les quatre croyances et ce que chacune fabrique. C'est **Kahler**, avec un h.
4. **Ma thèse opposait confiance et compétence.** Son recadrage est meilleur : *la confiance l'a mené jusqu'à la limite de ses ressources intérieures, puis il a fallu qu'il utilise sa confiance pour sortir en chercher à l'extérieur.* **Deux étages : la confiance qui porte, la confiance qui ouvre.** Aller demander à un rival qui vous bat demande une confiance considérable. Ce recadrage rend inutile le repositionnement de [[Sakura]] que j'avais proposé.

### Décisions tranchées

- **Upsell 1 du [[Cercle de Vérité]] : 27 €** (« 2 jours pour rester fort »). Ce choix **lève le blocage de production** du 08/08 : le tarif de 39 € alors retenu n'avait aucune vidéo, alors que `Upsell1 27E.mp4` existe.
- **Titre de la vidéo : « C'est pas la confiance qui m'a rendu champion. »** Tranchant en ouverture, nuance en chute, avec la formule finale : *« La confiance ne remplace pas la compétence. Elle donne le courage d'aller la chercher. »* → [[Idée - Confiance vs compétence]]
- **Appel vers [[Sakura]]**, pas le Cercle de Vérité — le lien chanbara/championnat/stratégies est direct.
- **`git push` autorisé** : règle ajoutée à `.claude/settings.local.json`. Le vault n'avait pas été poussé depuis le 24/08 11h25, soit 24 commits — le bot Telegram travaillait sur une version périmée.
- **Les cartes cochées valent « terminé »** : script `~/.local/bin/vault-kanban-tidy.py` créé, branché sur la synchro du dimanche, qui descend les cartes cochées en colonne de fin.

### Crédit Foncier — remis à plat

Fiche [[Contrat Crédit Foncier]] passée de trois lignes à un état complet. **Interlocutrice : Sophie Pons** — dans l'agenda, chercher **« Sophie leadership 🚺 »** autant que « Parcours femmes ». Calendrier de l'automne : **Prise de parole les 14 et 15 septembre**, Communication et gestion de conflits le 12 octobre, **clôture le 8 décembre**.

**Six feuilles de présence** créées ou complétées dans `Formation/CFF - Sophie Pons/Coachings/À envoyer/` — trois n'existaient pas du tout. Deux erreurs trouvées dans `Suivi coachings DM.xlsx` : la séance de Pascaline **reportée du 21/05 au 02/06**, et sa **séance 2 du 27/07** absente. David confirme : 1h30 par séance, le 19 mai pour Laila, et Pascaline est la seule à avoir eu une seconde séance.

**Cinq brouillons Gmail** créés (Laila, Sandra, Pascaline avec ses deux fiches, Christelle, Khom), **sans pièces jointes** — six fichiers Word auraient saturé le contexte pour un geste de cinq secondes. **Vouvoiement retenu**, à changer si David les tutoie.

### Le tableau des idées, et une consigne permanente

`1. PROJETS/Idées de contenu - tableau.md` — Kanban à cinq colonnes (Captées → Étoffées → À produire → Publiées → Écartées), tags `#pub` `#video` `#post`. Règle posée pour éviter la dispersion : ce tableau est le **flux de travail**, le [[Idées de contenu Mnemonaute - backlog|backlog Mnemonaute]] reste le **fonds documenté**.

**Consigne ajoutée à `CLAUDE.md`** : proposer spontanément d'y verser toute formule ou histoire née d'un échange qui ferait une accroche, une pub ou une vidéo. Ses meilleures formules naissent en conversation et se perdaient.

**[[Idée - Réviser sans méthode, le cas de Louis]]** — David préfère partir d'un témoignage. **Louis, 19 ans, brillant, en droit, a raté sa première année à deux dixièmes : 9,80.** Sa phrase : « je n'ai pas de méthode, jusqu'à présent ça fonctionnait, maintenant ça ne fonctionne plus ». **Découverte** : Louis était déjà dans le backlog sans son nom — c'est lui, « un étudiant rencontré par David », qui a inspiré la série « enchaîner les méthodes », priorité 1. Son témoignage en devient la vidéo d'ouverture.

### Autres

**[[Le cycle des émotions (Jérôme Lefeuvre)]]** — fiche créée puis purgée de ce que David maîtrise déjà. Ce qui reste vraiment neuf : la distinction **émotion / sentiment / humeur** (« si ça dure, ce n'est plus une émotion »), **le dégoût** — absent du vault, avec « il y a plus de relations toxiques que de gens toxiques », utile au Cercle de Vérité — et **le chagrin comme station-service**.

**Formulaire F&L 2026** (Jody Cavalié Academy / Liberty Webi) rempli en pilotant Chrome par AppleScript. Prérequis découvert : le réglage se trouve dans le menu **« Présentation »**, pas « Affichage ». Les clics simulés ne passent pas sur les combobox HubSpot — **seule la navigation clavier fonctionne**. Formulaire en 8 étapes, le DOM les contient toutes : remplir une étape masquée ne marche pas.

**Coachings SHERPA** : seule **Véronique** a un rendez-vous à venir (2 septembre, 9h-10h30). **Marina et Dominique n'ont plus rien de planifié** d'ici juin 2027.

### Reste à faire

- **Envoyer les cinq brouillons** en y glissant les pièces jointes, après avoir vérifié le vouvoiement.
- **Préparer les deux journées Prise de parole** des 14 et 15/09, depuis `Prise de parole _ V2.pptx` — échéance **11/09**.
- **Poser la reconduction 2027 avec Sophie** avant la clôture du 8 décembre.
- **Fournir la clé API et le token Trello**, à ranger dans le trousseau macOS.
- **Confirmer l'angle « Gandhi vs Dev Perso »** — mon hypothèse (la citation apocryphe « sois le changement ») attend validation.
- **Mettre à jour `Suivi coachings DM.xlsx`**, faux sur deux points.

## 2026-08-26 — Le guide de Steph, et trois pannes serveur dont deux de mon fait

### Le guide « Ton second cerveau »

Steph veut son propre second cerveau. Produit en une session, révisé quatre fois :

- **Artefact** : https://claude.ai/code/artifact/1285ae55-cafd-47bb-a820-8c1c4e7a12c9
- **Source HTML** : `~/Documents/Second cerveau - guide/guide-source.html`
- **Version texte pour l'assistant** : `~/Documents/Second cerveau - guide/GUIDE.md`

Douze chapitres, 48 étapes, du compte Claude jusqu'au briefing Telegram. Le parti pris structurel : chaque étape porte une étiquette **[TOI]** ou **[L'ASSISTANT]**, et la bascule se produit au chapitre 4 — avant, elle clique ; après, elle donne des consignes. C'est le sujet du guide autant que sa mise en page.

**⚠️ L'épingle de partage ne suit pas les republications.** À chaque modification, il faut la redéplacer manuellement, sinon les destinataires restent sur l'ancienne version. Vérifié deux fois aujourd'hui.

### Les corrections de David sur le guide

- **« On s'en fout des "David a…" »** — j'avais présenté la structure de dossiers comme une préférence personnelle. C'est en réalité **PARA**, de **Tiago Forte** (*Construire un second cerveau*), dont le principe est : **on range par degré d'action, pas par thème**. Un classement thématique s'effondre parce qu'une note appartient à plusieurs thèmes ; PARA pose une question à réponse unique. **Elliot Meunier** ajouté au bon endroit — sur les liens `[[ ]]` et le Zettelkasten, pas sur les dossiers. Les dix mentions personnelles ont ensuite été neutralisées pour permettre la diffusion.
- **Titre générique** : « Ton second cerveau » et non « Le second cerveau de Steph », *« comme ça je peux le partager à d'autres »*.
- **Allègement** : *« aussi épuré que possible… on est sur du "Do as I say", donc "Fais comme je dis, et questionne plus tard" »*. Dix blocs d'explication repliés derrière « Pour en savoir plus ». **Règle de tri appliquée : on replie le *pourquoi*, jamais le *attention*** — les 14 avertissements restent tous ouverts.
- **Modules Obsidian** : David n'en a que deux, `obsidian-git` et `obsidian-kanban`. C'est devenu la recommandation elle-même — étape 5.6 « Deux modules, et pas plus », qui nomme le piège de la surinstallation.

### Le retour de Stéphane et de son IA « Rufio »

Sept points sur neuf retenus. Les trois sérieux :

1. **Le bot Telegram écrivait sans limite** — vraie faille, non vue. Corrigé plus fort que proposé : le bot **n'écrit que dans un seul fichier** (`Depuis Telegram.md`) et **n'exécute aucune commande système**. Restreindre aux `.md` ne protège pas du vrai danger, qui est le shell. Bénéfice en prime : plus de conflit de synchro possible.
2. **Aucun durcissement serveur** — nouvelle étape 9.4 « Fermer les portes », avec l'ordre imposé : vérifier la clé **puis** couper l'auth par mot de passe.
3. **Le jeton Claude du serveur expire aussi** — excellente prise, vérifiée chez lui *et* chez David. Le guide avertissait pour Google et pas pour Claude : incohérence levée.

Retenu aussi : l'**audit de péremption** et la **capture de patterns comportementaux datés**, tous deux ajoutés à la skill `/fin` (passée de quatre à six étapes), et le bloc **« priorités de la semaine »** dans `CLAUDE.md`.

**Refusé, avec argument écrit dans le guide** : deux clés SSH séparées. Le cloisonnement ne paie que si la clé fuit *isolément* ; si elle est volée, le Mac l'est déjà. Le coût — un `~/.ssh/config` à comprendre au pire moment — dépasse le gain. Gardé comme note pour lecteur averti.
**Écarté comme surdimensionné** : la recherche full-text sur `memory.md`. L'assistant *cherche* dans le fichier, il ne le lit pas. Découpage par période le jour où ça dépassera 100 000 caractères.

### Réparations serveur

- **Google OAuth confirmé réparé** — les 69 `invalid_grant` s'arrêtent à 07:30 le 26/08, `tokens.json` réécrit à 09:22. **Premier briefing parti depuis le 30 juillet.**
- **Cron `pull-vault.sh` retiré.** `/home/david/Doc` est un **clone orphelin** : rien ne le lit, ni le bot (qui utilise `/home/david/vault-clone`), ni un Claude Code (absent du serveur). Il était entretenu 288 fois par jour pour personne, avec un journal de **81 418 lignes**. Sauvegardes : `~/crontab.avant-20260826-133956.bak`, `~/pull-vault.log.avant-purge`.
- **Jeton `CLAUDE_CODE_OAUTH_TOKEN` remplacé.** Celui du 18/07 était mort — trois `401 Invalid authentication` au journal. Outil créé : **`~/rotate-secret.sh`** (mode 700), qui demande la valeur en **saisie masquée** pour qu'aucun jeton ne transite par la conversation ni par l'historique shell.
- **Journal du bot créé** : `~/telegram-claude-bot/logs/bot.log`. Il n'en avait aucun avant.

### ⚠️ Mes quatre erreurs

1. **`DRY_RUN` n'existe pas dans `briefing.js`** — seulement dans `veille-boamp.js`. J'ai cru tester à blanc et **j'ai envoyé un vrai briefing** sur le Telegram de David. Vérifier qu'un drapeau existe avant de s'appuyer dessus.
2. **J'ai accusé `pull-vault.sh` de bloquer la synchro en silence.** Faux : zéro échec depuis toujours, parce que rien n'écrit dans `Doc`. Le vrai défaut était ailleurs — le clone ne servait à rien.
3. **La plus coûteuse : j'ai conclu que PM2 n'était pas installé** parce que `which pm2` était revenu vide — il vit sous nvm, absent du PATH en SSH non interactif. J'ai donc écrit un script qui tue le bot et le relance à la main : PM2 a ressuscité le sien, mon script a lancé le sien, **deux instances, `409 Conflict` en boucle**, bot muet. J'avais en plus ajouté un cron `@reboot` redondant qui aurait recréé le conflit à chaque démarrage. Tout corrigé : script basé sur `pm2 restart --update-env`, cron retiré. **Une commande qui échoue ne prouve pas l'absence de la chose.**
4. **Shift+Entrée annoncé pour le retour à la ligne** — c'est **Option+Entrée** sur Terminal.app. `/terminal-setup` active « Option comme touche Meta », et **il faut quitter Terminal.app puis le rouvrir**.

Les erreurs 3 et 4 ont produit deux ajouts au guide : **9.5 « Une fenêtre, deux machines »** (le copier-coller d'un jeton n'est pas une répétition, c'est un transport entre deux ordinateurs — question posée par David lui-même) et **9.7 « Qui relance le bot »** (on ne relance jamais un service à la main quand un gestionnaire existe), plus une ligne de diagnostic `409` au chapitre 10.

### Décisions tranchées

- **ProRealTime écarté** pour le suivi automatique du portefeuille : pas d'API propre, l'accès passe par l'API du courtier, incompatible avec un cron sans navigateur. Yahoo reste la source.
- **Dossier `Secrets` local** (idée de Stéphane) intégré au guide en étape 8.2, avec **trois verrous** : exclu de la sauvegarde, interdit par `CLAUDE.md`, bloqué par une règle de permission. Avec ses limites dites : ce n'est pas un coffre-fort, iCloud peut le synchroniser, et **ce qui *ouvre* quelque chose reste au Trousseau**.
- **Note ADMIN** : recommandations données en trois piles — supprimer les jetons techniques (récupérables sans elle), déplacer les codes de récupération au Trousseau, laisser l'IBAN. **David a nettoyé la note.**
- **Fichier compagnon `GUIDE.md` rendu optionnel** — objection de David : *« ça ferait 2 guides »*. Il devient un accessoire, avec repli explicite (copier le chapitre depuis le navigateur).

### Reste à faire

- **Surveillance automatique des jetons** : proposée, **non tranchée**. Détecter un `401` dans `logs/bot.log` et envoyer une alerte Telegram avec la commande à copier. Vaut pour Claude et Google. ~20 min.
- **L'installation de Steph n'a pas commencé.**
- **Redémarrer Terminal.app** pour activer Option+Entrée.
- Nouvelle idée capturée dans [[Idées de contenu - tableau]] : **« Fais comme je dis, questionne après »** — on n'a pas besoin de comprendre pour agir ; la chute est *« la vraie compétence, ce n'est pas le Terminal, c'est le réflexe de demander »*. Se raccorde aux deux étages de [[Idée - Confiance vs compétence]].

---

## 2026-08-26 (2) — Le CFF remis d'aplomb, la sonde des jetons, et le guide rendu autonome

### Crédit Foncier — trois erreurs à moi, corrigées par David

J'avais transformé les journées Prise de parole des 14-15/09 en chantier de préparation, avec des créneaux à bloquer dans l'agenda. Trois hypothèses fausses, empilées sans vérifier :

1. **Ce n'est pas un parcours de deux jours.** Ce sont **deux journées indépendantes** : un groupe de **7 managers**, un groupe de **6 cadres non managers**. La même journée jouée deux fois devant deux publics.
2. **Le support est déjà conçu et mis en forme par une graphiste.** Il ne reste ni conception ni design. *(Emplacement de la version graphiste non confirmé : rien de plus récent que `Prise de parole _ V2.pptx` du 8 juin dans `~/Documents/Formation/CFF - Sophie Pons/Leadership au féminin/Prise de parole/`.)*
3. **Steph est en lead sur la relation Sophie Pons** et cale la suite. David est **prestataire** : conception, design, animation. Je lui avais mis la reconduction 2027 sur le dos — ce n'est pas son rôle. Le rendez-vous que je visais (04/09 14h) n'est d'ailleurs pas le parcours féminin mais la **tripartite de démarrage du coaching de Jilani Ben-Yahmed**.

Sa réponse, qui vaut règle : *« relax. Typiquement le genre de truc que j'anime tout le temps. confort. »* **Tranché : rien à préparer pour les 14-15/09.**

Deux mémoires système écrites là-dessus (`feedback_animation_est_routine`, `project_cff_repartition_roles_steph`). Fiche [[Contrat Crédit Foncier]] corrigée en conséquence.

**Trouvaille d'agenda au passage** : l'échéance « préparer le 11/09 » que j'avais posée était intenable — David est en **déplacement IGECOM du 9 au 11 septembre** (Toulon, Clermont, Naves) et enchaîne directement sur les 14-15. Une échéance posée sans regarder l'agenda ne vaut rien.

### Décisions tranchées

- **Test Meta 200 € : en sourdine.** *« je t'ai dit que ça arriverait après la validation et tournage des vidéos créatives ads meta »*. Ne plus le remonter comme un retard.
- **Codes de récupération à régénérer : abandonné.** *« on oublie je prends la responsabilité »*. Le point est sorti des vigilances, il ne se rouvre pas.
- **Point de cadrage Radiance** : en cours de calage sur le **28/08**.

### La sonde des jetons — faite, testée

Le contexte : jeton Claude mort le 18/07 découvert le 26/08, briefing Google muet depuis le 30/07 découvert le même jour. Un mois de silence à chaque fois.

**J'avais proposé la veille le mauvais des deux dispositifs.** Le scan de journal (`401` dans `bot.log`) est *réactif* : il n'y a de refus que si quelqu'un a essayé. Bot inutilisé dix jours = journal vide = panne découverte au pire moment. La **sonde active** ne compte sur rien : elle teste elle-même chaque matin. Même coût, meilleur résultat. David : *« oui, vas-y »* — après m'avoir dit *« je n'ai rien compris »* sur ma première explication, trop jargonneuse.

Livré sur le serveur (`187.77.161.25`, `srv1792680`) :

- **`~/telegram-claude-bot/check-tokens.js`** — teste Claude via le SDK avec le modèle le moins cher (le test doit emprunter **le même chemin** que le briefing, sinon il valide une porte qui n'est pas celle qu'on utilise) et Google en forçant `getAccessToken()` (c'est le *refresh* qui casse, pas la lecture). Silence si tout va bien, Telegram nommant le service et la commande sinon. N'affiche jamais un jeton.
- **Cron `0 7 * * *`** (UTC ; le serveur est en `Etc/UTC`) — 30 min avant le briefing de semaine (`30 7`), été comme hiver puisque les deux dérivent ensemble. Journal : `logs/check-tokens.log`. Crontab sauvegardé en `~/crontab.avant-sonde-20260826-205228.bak`.
- **Testé dans les deux sens** : les deux jetons OK en conditions réelles ; avec un faux jeton Claude, détection (`401 OAuth access token has expired`) et alerte réellement partie sur le Telegram de David.
- **L'alerte se répète chaque matin** tant que ce n'est pas réparé — volontaire.

### Le guide de Steph — trois modifications

Artefact republié deux fois sur la même URL, `~/Documents/Second cerveau - guide/guide-source.html` et `GUIDE.md` tenus en parallèle.

**1. Les replis ne ressemblaient pas à des boutons.** Remarque de David, et le défaut était pire que décrit : le bandeau était en petites capitales monospace grises — la mise en forme exacte des titres de section — avec un `+` discret, et sur petit écran le texte « Pour en savoir plus » était **purement masqué** par une media query. Refait : chevron qui pivote vers le bas à l'ouverture, « **Cliquer pour développer** » souligné, casse normale en vert sarcelle, bordure verte, et sur petit écran l'étiquette est **raccourcie** au lieu d'être supprimée. Chapitre 0 réaligné.

> David a précisé ensuite que **le téléphone n'est pas un cas d'usage** — le guide se lit devant le Mac, pendant l'installation. Mon argument « Steph lirait au téléphone » ne tenait pas. La règle petit écran reste, sans coût.

**2. La surveillance des jetons ajoutée en 9.9**, le point « filet de sécurité » : quatrième consigne (vérification quotidienne des deux autorisations), plus deux encadrés — pourquoi tester activement bat relire un journal, et pourquoi l'alerte doit se répéter. Renvoi ajouté depuis 9.5.

**3. Le guide rendu autonome — l'idée de David.** *« Est-il un élément développable du style "copie colle ce qui vient et donne le à ton 2e cerveau" ? avec un raccourci qui copie direct ? »* Il avait raison de tiquer : jusque-là `GUIDE.md` n'était accessible à Steph **que si David le lui envoyait à part**. Le lien seul ne suffisait pas.

Réalisé, mais **par chapitre** plutôt qu'en un bloc — le guide entier fait 69 Ko, pénible à coller et coûteux en contexte, et le guide dit déjà de travailler un chapitre à la fois. Chaque chapitre porte un bouton **« Copier pour l'assistant »** qui met dans le presse-papier : la note de cadrage à l'assistant + le chapitre en markdown + la consigne « conduis-moi ». Un seul collage suffisant. Plus un bouton « Copier le guide entier » en 4.4, avec la bonne voie de dépôt : **coller dans une note Obsidian vide** — une note Obsidian *est* un fichier texte, donc ni TextEdit ni histoire d'extension. Repli prévu si le presse-papier est bloqué : zone de texte pré-sélectionnée. **David a testé sur son Mac : ça marche.**

### Doc.app — deux réparations

**a) Erreur `-1743` au lancement.** `~/Applications/Doc.app` (applet AppleScript, trois lignes qui lancent `claude` dans le vault) n'avait **aucun `CFBundleIdentifier`**, et sa signature ne scellait pas son `Info.plist`. macOS enregistre les autorisations d'automatisation par identifiant de bundle : sans identifiant, pas d'endroit où retenir la réponse, donc refus sans jamais poser la question. Correctif : identifiant `com.davidmarsac.doc`, re-signature ad-hoc, `lsregister -f`, `tccutil reset AppleEvents`. **David a autorisé, Doc se lance.** Détail consigné dans les vigilances de l'état — **même famille de panne que le cron notes iOS** du 31/08.

**b) Une nouvelle fenêtre à chaque clic.** `do script` en ouvre toujours une. Réécrit : à la création, le script pose un **titre personnalisé** sur l'onglet (« Doc — second cerveau ») ; aux clics suivants il le cherche et se contente de réveiller la fenêtre. Traité aussi le démarrage à froid, où Terminal ouvre une fenêtre vide de lui-même — le script la réutilise au lieu d'en ajouter une. Sauvegardes : `~/Applications/Doc.app.bak-20260826-231254` et l'ancien `main.scpt` dans le scratchpad. **Non testé par David au moment d'écrire.**

### Découverte technique de David

**Ctrl+V, pas Cmd+V**, pour coller une image dans Claude Code — repéré dans les astuces affichées en bas de fenêtre. *« c'était la 2e question technique qui restait en suspens »*. La première était Option+Entrée pour le retour à la ligne (session précédente).

### Ce qui reste à faire

- **Confirmer où vit la version graphiste** du support Prise de parole.
- **Tester le nouveau lanceur Doc.app** depuis le Stream Deck.
- **L'installation de Steph n'a toujours pas commencé.**
- **Sakura** : contenu différencié des deux niveaux.
- Supprimer `~/Applications/Doc.app.bak-20260826-231254` après quelques lancements sans souci.

---

## 2026-08-26 (2) — Ménage : deux points de la checklist fermés

Session courte, uniquement du rangement — elle solde deux lignes du « reste à faire » de l'entrée précédente.

**1. Sauvegarde `Doc.app` supprimée.** David a lancé lui-même `rm -rf ~/Applications/Doc.app.bak-20260826-231254`. Le lanceur corrigé (identifiant de bundle, réutilisation de l'onglet Terminal) est **validé au Stream Deck** — les deux réparations de l'entrée précédente tiennent. `~/Applications/` ne contient plus que `Doc.app`. **Tranché**, le sujet est clos.

**2. La version graphiste du support Prise de parole, c'est bien la V2.** Question ouverte depuis le 24/08, **tranchée** : `~/Documents/Formation/CFF - Sophie Pons/Leadership au féminin/Prise de parole/Prise de parole _ V2.pptx` (8 juin) *est* la version graphiste. Il n'y avait rien de plus récent ailleurs, et ma recherche n'a rien remonté d'autre pour le CFF — les autres `V2` trouvés appartiennent à IGECOM et Burger King, dossiers sans rapport. Chemin reporté dans `Etat actuel.md`, sur la ligne des journées du 14-15/09, pour ne plus avoir à le rechercher.

**3. Fichier de verrouillage supprimé.** Un `~$Prise de parole _ V2.pptx` (165 octets, 8 juin) traînait à côté du support — reliquat d'une fermeture ratée de PowerPoint, pas d'une édition en cours (PowerPoint était fermé au moment de la vérification). Supprimé sur demande de David. Sans lui, le support risquait de s'ouvrir en lecture seule le 14/09.

### Ce qui reste à faire

- **Journal du cron notes iOS + Kanban** à vérifier le 01/09, après la première exécution du 31/08.
- **L'installation de Steph n'a toujours pas commencé** (chapitres 0 à 3, non délégables).
- **Cinq brouillons Gmail CFF** : pièces jointes à glisser, vouvoiement à vérifier.
- **Sakura** : contenu différencié des deux niveaux.
- **Cercle de Vérité** : les créas vidéo d'abord, le test Meta 200 € ensuite.

---

## 2026-08-27 — Créas Meta du Cercle de Vérité, veille mail automatisée, et neuf séances CFF au lieu de six

Longue session, trois sujets qui n'avaient rien à voir et qui se sont éclairés l'un l'autre.

### 1. Les créas publicitaires du Cercle de Vérité

**Livrable** : `1. PROJETS/Cercle de Vérité - Créas pubs Meta (lot 1).md` — douze scripts vidéo, douze visuels, la matière preuve sociale, une réserve d'angles. David le soumet à **Anthony, son copywriter**, qui corrige puis étoffe.

**Ma première version était à jeter, et c'est la leçon de la session.** J'ai écrit dix scripts à partir du brief ads et de la masterclass Clovis, sans demander qui d'autre travaillait sur le sujet. Or Anthony avait déjà donné une **structure de script contraignante** : accroche impliquante en 3 secondes avec un « tu », sujet vaste, mise en situation, **échelle de « oui »** (une succession de faits sur lesquels l'avatar est d'accord), son OUI à lui, **révélation de la manipulation à la fin**, déculpabilisation, CTA. Mes scripts posaient le mécanisme dès la première phrase : ils brûlaient la révélation sur une audience qui ne voit pas encore le problème. *« j'attends de toi que tu me poses des questions avant de foncer tête baissée »*. → mémoire `feedback_questions_avant_production`.

**Je n'avais pas non plus trouvé `Creas Ads Meta - 15 angles, hooks, scripts, CTA (08-08-2026).docx`** (dans `~/Documents/Formation/Olivier Clovis Scaling Academy/Contenus/Low Ticket - Le Cercle de Vérité/`), qui contenait déjà les angles, tirés des propres scripts de David. Il ne manquait que la structure et la durée. **À rouvrir systématiquement pour toute question de créa.**

**Ce que David a corrigé dans mes scripts, et qui dit comment il pense** — à chaque fois il remplace une formule qui *nomme* le mécanisme par une scène qui le *montre* : « tu vas dire oui » devient « tu vas **mentir** » · « je disais oui à un service » devient « je **ravalais et je supportais** » · « deux secondes de consultation » devient « **oui, mais à 20h30 je dois être rentré** » · « s'il te plaît ne m'en veux pas » devient « **ça se négocie, insiste un peu et je craque** ». **Règle qui en découle : rien n'est nommé avant le temps 5.**

**Sa correction sur la cigarette change le mécanisme, pas le script.** La faute n'est pas « tu as dit oui » : *tu n'as pas dit non, tu as donné une raison — et une raison, ça se démonte*. La gêne qui monte quand l'autre répond « y'a un tabac juste là » est la preuve que l'excuse n'était pas la vraie raison. C'est vérifiable dans le corps, ça supprime le méchant, et ça donne une règle applicable le soir même. **David a confirmé que le jour 5 du protocole enseigne bien de ne pas se justifier** : la promesse est donc adossée au produit, et c'est un **candidat au positionnement principal**.

**Décisions tranchées le 27/08 :**
- **Durée** : une minute, jusqu'à ~85 s pour les pubs qui démontrent (l'échelle de « oui » ne se compresse pas). Les pubs qui mettent en scène restent serrées.
- **Les dix vidéos du 01/02/2026 partent à la benne** — elles ne peuvent pas respecter cette structure.
- **L'angle « le faux merci » est retiré, sans remplacement.** *« je préfère ne pas inclure le faux merci, point. on aura plein de fois l'occasion de créer de nouvelles pubs »*. Numéro 8 laissé vacant pour ne pas décaler les références de travail avec Anthony.
- **La démonstration des 50 € se joue avec « ça me ferait plaisir »**, pas avec le portefeuille volé (à soumettre quand même à Anthony).
- **Le silence compte comme un oui.** Le mécanisme ne se limite pas au faux OUI prononcé.

**Deux angles nés de l'échange, à porter au crédit de David :**
- **« Qui ne dit mot consent — sauf que toi tu ne consens pas, tu t'écrases »** (angle 12). Entrée la plus large du lot, hook typographique, et seul script qui relie le mécanisme unique au **consentement masculin** sans le expliquer.
- **« Comme tu veux »** (angle 13), et les questions qui cachent l'envie — « ça te dirait de… ? » au lieu de « j'ai envie de…, qu'en penses-tu ? ». Il attrape le mécanisme **en amont du non**. Découverte au passage : c'est **déjà le mail 3 du tunnel**, « Ta nouvelle phrase à bannir : comme tu veux », avec un quatrième témoignage, **Guillaume**, et la meilleure phrase de tout le corpus, prononcée par sa compagne : *« Avec toi, je ne sais jamais si tu es vraiment d'accord ou si tu me dis oui pour ensuite me faire payer quelque chose. »*

**Réserve d'angles pour plus tard** (dans le fichier) : la remarque en apparence innocente qui cache une pique et qu'on ravale (idée de David) · Pierre, le terrain professionnel · les 48 heures après le premier NON.

**Corrections de fiche produit** (`1. PROJETS/Cercle de Vérité.md`) : **upsell 1 à 39 €** et non 27 — le passage à 27 € envisagé le 25/08 n'a pas été retenu, et **aucune vidéo à 39 € n'existe** (prises à 17, 27, 47, 57 € + `Upsell1 neutre 0 prix.mp4`, qui est la solution). Downsell = le même en 2 × 13,50 €. **Hikari = le high-ticket**, soit [[Sakura]] *plus* l'accompagnement premium ; Hikari inclut Sakura.

**Note conceptuelle déposée** (à ne pas traiter tout de suite) : le Cercle pris au pied de la lettre — qui est dedans, qui est dehors, et une fois dedans c'est toujours la vérité. Ça déplace la promesse du « dire NON » défensif vers « arrêter de faire semblant avec ceux qu'on aime », et ça contient une réponse possible à la question ouverte depuis le 08/08 sur **le remplacement des trois besoins**.

### 2. La veille de la boîte mail

*« tu es pas censé les parcourir régulièrement et préparer des réponses dans mes brouillons ? »* — non, rien ne tournait en fond, et je n'étais pas allé voir dans ses mails de la matinée. C'est là qu'était l'invitation du cadrage Radiance.

**Routine cloud créée** : « Veille boîte mail — Doc », `trig_01PHwoiyAPmAaQmr6HUxfDDJ`, cron `0 6,10,15 * * *` UTC = **8h, 12h et 17h heure de Paris**. Elle lit la boîte, prépare des brouillons sans jamais envoyer, et ne range un fil que si son historique porte déjà le libellé. Console : `https://claude.ai/code/routines/trig_01PHwoiyAPmAaQmr6HUxfDDJ`.

⚠️ **Deux limites connues.** Le cron est en UTC : **au passage à l'heure d'hiver fin octobre 2026, les horaires glisseront à 7h/11h/16h** et il faudra décaler. Et le **compte GitHub n'est pas connecté au cloud** (`/web-setup`), donc la routine n'a pas accès au vault et écrit sans connaître les dossiers en cours.

**Découverte utile** : les **dossiers Outlook de David sont ses libellés Gmail**. Ranger depuis Outlook ou poser un libellé depuis Gmail est la même opération. Pas besoin de toucher au stockage local d'Outlook, qui est de toute façon un binaire illisible.

**Deux scripts écrits**, pour combler ce que le connecteur Gmail ne sait pas faire :
- `~/Scripts/mail-pieces-jointes.py` — récupère les pièces jointes d'une recherche Gmail. Le connecteur donne les noms et les identifiants, **jamais les octets**.
- `~/Scripts/brouillon-avec-pieces-jointes.py` — dépose un brouillon avec ses fichiers attachés, par IMAP APPEND. Faire transiter 500 Ko de base64 par la conversation était impossible.

Les deux lisent le **mot de passe d'application Gmail dans le trousseau macOS** (`security find-generic-password -a david.marsac1@gmail.com -s gmail-app-password`). David ne connaissait pas le principe du trousseau ; expliqué. **Le mot de passe a été tapé dans la conversation** : à révoquer sur `myaccount.google.com/apppasswords` avant tout partage de transcription.

Trois pièges rencontrés, réglés dans les scripts : imaplib encode les recherches en ASCII (littéral UTF-8 obligatoire dès qu'il y a un accent) · les clients Windows envoient les accents en caractère autonome, d'où les `pre´sence` illisibles (remis en combinant avant NFC) · une recherche IMAP sur « emargement » ne trouve pas « émargement ».

**n8n : écarté, et c'est tranché.** Un script local suffit, et David n'aurait ni à l'héberger ni à le maintenir. Le seul cas où il gagnerait — classement Mac éteint, fichiers dans un stockage cloud — n'est pas sa situation.

### 3. Crédit Foncier : neuf séances, pas six

**Quatre feuilles d'émargement signées récupérées et classées** dans `~/Documents/Formation/CFF - Sophie Pons/Coachings/Signés/` (Pascaline ×2, Sandra, Laila), fils rangés dans le libellé `Clients/Crédit Foncier Sophie PONS`.

**Brouillon prêt** pour **Nathalie Feugeas** (`nathalie.feugeas@creditfoncier.fr`), Sophie Pons en copie, avec les cinq feuilles. C'est elle qui recueille les émargements — Sophie l'avait écrit dès le 21/04. **Il répond à une demande de Sophie du 29/07 restée sans réponse pendant un mois.**

⚠️ **`Suivi coachings DM.xlsx` est faux sur trois points et ne doit plus servir de référence.** Reconstitution faite à partir des **invitations Teams envoyées par les participantes**, qui font foi — tableau complet des neuf séances dans [[Contrat Crédit Foncier]]. Pascaline était notée au 21/05 alors que l'invitation dit « report du 21/05 au 02/06 », et sa séance 2 du 27/07 manquait. Khom était notée au 29/06 alors que c'est sa **seconde** séance, la première ayant eu lieu le **18/05 en présentiel, salle Braudel**. Et la seconde séance de Laila, le 29/06, manquait aussi.

**Conséquence** : il manque **quatre** feuilles et non deux — Khom ×2, Laila séance 2, Christelle. **Les relances du 26/08 ne demandaient qu'une feuille à chacune : elles sont à refaire pour Khom et Laila.** Sujet de facturation autant que d'administratif.

**L'assiduité n'est pas un sujet, tranché** : treize participantes sur neuf mois, une absence ponctuelle est intégrée au dispositif. Laila sera absente aux journées de septembre, sans conséquence. Consigné dans [[Contrat Crédit Foncier]] pour ne plus le remonter comme un problème.

### 4. Divers

**Cadrage Radiance validé au 4 septembre.** L'invitation Teams de Nathalie Durieux est arrivée le 27/08 à 9h19, avec quatre personnes de Radiance en copie — **toujours pas acceptée, donc absente de l'agenda**. Journée chargée : la tripartite CFF sur Jilani Ben-Yahmed est le même jour de 14h à 15h.

**Faux conflit d'agenda signalé de ma part** : le calendrier scolaire des enfants et les masterminds Clovis ne sont pas des engagements fermes. → mémoire `project_agenda_evenements_indicatifs`. Au passage, David sur Scaling Academy : *« j'essaie d'y aller quand je peux, mais je ne suis encore pas pleinement committed »*.

**Convention d'écriture** : quand David écrit « $ », il veut dire **euro** — son clavier ne sort pas le €. S'il parle de dollars, il écrit USD. → mémoire `user_convention_symbole_euro`.

### Ce qui reste à faire

- **Retours d'Anthony** sur les douze créas, puis tournage.
- **Refaire les relances de Khom Rivière et Laila Ed Damiri**, incomplètes — David n'en a pas voulu le 27/08.
- **Relancer Christelle Erhard** pour sa feuille du 26/06.
- **Accepter l'invitation Teams** du cadrage Radiance du 04/09.
- **Vérifier la première exécution de la routine de veille** le 28/08 au matin.
- **Connecter GitHub au cloud** (`/web-setup`) pour donner le vault à la routine.
- **Décaler le cron de la routine** fin octobre 2026, au passage à l'heure d'hiver.
- **Journal du cron notes iOS + Kanban** à vérifier le 01/09.
- **L'installation de Steph n'a toujours pas commencé.**
- **Sakura** : contenu différencié des deux niveaux.

---

## 2026-08-30 — Louis publiée, deux routines mail au lieu d'une, et les audios manquants du Cercle

Session longue étalée sur le 28, le 29 et le 30 août.

### La vidéo « J'ai rencontré Louis » — de l'idée à la publication

**Note de pilotage unique** : `1. PROJETS/Vidéo - J'ai rencontré Louis.md` ([[Vidéo - J'ai rencontré Louis]]). Elle centralise titre, miniature, description, posts réseaux, checklist et décisions. Le texte du prompteur reste séparé dans [[Mnemonaute Script - J'ai rencontré Louis]] — décision de conception : c'est le document qu'on ouvre seul en plein écran pendant le tournage.

**Les 2,8 Go de médias restent hors du vault**, dans `~/Documents/Mnemonaute/Rentree - Louis pas de methode/`. La note pointe vers le dossier. Ne jamais faire entrer de rushes dans Obsidian.

**Décisions tranchées :**
- **Le test des villes d'Australie est placé AVANT la méthode.** Argument de David : « ils se rendront compte qu'ils sont en difficulté, donc ils seront preneurs de la solution qui arrive ensuite. Sinon on retombe dans le biais de confirmation. » Sa construction en trois temps (affirmer la liste → demander de la produire → révéler que Canberra en était absente) est meilleure que les deux versions que j'avais proposées : elle ajoute un troisième niveau, on ne voit pas ses propres trous.
- **« Il n'a rien foutu » conservé**, ce sont les mots de Louis. David y tient : « c'est du langage courant, j'y suis très attaché. »
- **L'inquiétude de Louis est nommée explicitement** — « il ne sait pas où chercher ». Point d'identification du spectateur.
- **Pas de mention de la boîte de Leitner.**
- **Prénom seul, aucun accord nécessaire** : pas de nom de famille, pas de lieu. Bonjour à Louis dès la partie 1.
- **Titre retenu : « Tu n'as pas un problème de mémoire. Tu as un problème de méthode. »** — David a préféré sa version longue (65 caractères) à ma variante courte, malgré le risque de troncature sur mobile.
- **Miniature : piste A** — « 9,8/20 » + « Le talent ne suffit plus », visage à gauche, doigt pointé. Fichier conforme : `Rentree_tn_YouTube.jpg`, 1920×1080, 702 Ko. L'original PNG faisait 2,4 Mo et **dépassait la limite YouTube de 2 Mo**.
- **La description situe la vidéo, elle ne la raconte pas.** Correction de David : « je voudrais une description qui soit une description, et pas une paraphrase de la vidéo. » Le déroulé de la méthode en a été retiré — ne pas le réintroduire, les chapitres le donnent déjà.
- **Lien du guide en ligne 2 de la description**, pour supprimer la friction. URL confirmée : **https://www.davidmarsac.com/guide-offert** (consignée aussi dans [[Systeme.io - templates lead magnet et tunnel de vente]], où la mention « jamais encore utilisé » était fausse).

**Mon erreur, et David a été net : « c'est in-ter-dit ça. Interdit. »** J'ai écrit 1 900 mots de script après seulement deux questions fermées portant sur des options que j'avais moi-même définies, alors qu'il venait d'annoncer avoir un feedback à donner. [[feedback_questions_avant_production]] a été durci en conséquence : aucun livrable créatif avant que David ait parlé, et son feedback annoncé se recueille en premier.

**Le script final est une fusion**, et son brouillon était meilleur que le mien sur l'essentiel : il montrait les capacités de Louis au lieu de les affirmer, faisait la double lecture du 9,8, nommait l'erreur « je ne suis pas si intelligent que ça », et surtout tenait le passage passion-contre-cours — « si je retiens des centaines d'infos sur un jeu vidéo, le problème n'est pas ma mémoire ». J'ai réinjecté cinq blocs manquants : la formule des bulletins, la promesse d'une matinée, Pomodoro sans mode d'emploi et les soixante vidéos, la citation complète, le CTA.

**Publiée.** 36 vues et 4 likes en 28 minutes sans promo.

**Posts réseaux rédigés sur ses directions**, dans la note hub. Une formule est de lui et vaut d'être gardée : « je vous ai donné une boîte à outils complète, sans mode d'emploi. Cette vidéo est le mode d'emploi. »

**Écarté** : son idée d'ouvrir le post LinkedIn par « Article garanti sans IA ». Le post étant co-rédigé, la mention serait fausse. Remplacée par « Cette vidéo ne sort pas d'un prompt. Elle sort d'une conversation avec un garçon de 19 ans. » — même intention, porte sur le fond, et c'est vrai.

**Reste à faire sur cette vidéo** : générer le fichier `.srt` à partir du script (meilleur levier de référencement disponible, la matière existe déjà), et poser les écrans de fin sur le lot « méthode » du catalogue — pas sur les soixante vidéos, la pertinence thématique prime sur le volume.

### Deux routines mail tournaient en parallèle

La cause des brouillons en double n'était pas un défaut de prompt : **il y avait deux routines cloud**. « Veille boîte mail — Doc » (`trig_01PHwoiyAPmAaQmr6HUxfDDJ`, 8h/12h/17h, créée le 27/08) et **« Tri email + brouillons » (`trig_01UAHKD2jySUGKEVmdfef333`, créée le 03/08/2026, 8h/14h)**, oubliée. Le 28/08 elles ont tiré à 47 secondes d'écart : chacune a bien vérifié les brouillons existants, mais l'autre n'avait pas encore écrit.

**« Tri email + brouillons » a été désactivée** (`enabled: false`). Pas supprimée — impossible depuis l'API, ça se fait sur claude.ai/code/routines. Choisie parce qu'elle n'avait pas le connecteur Agenda, que ses `allowed_tools` incluaient Write, WebFetch, WebSearch et l'exécution de code pour un travail de lecture de mails, et qu'elle s'interdisait tout classement — politique contradictoire avec l'autre sur la même boîte.

**Prompt de la routine survivante réécrit** : compte rendu envoyé par mail à david.marsac1@gmail.com, objet `[Doc] Récap veille mail — JJ/MM HHhMM`, seule exception à l'interdiction d'envoyer ; elle ignore ses propres récaps ; anti-doublon renforcé ; « Nathalie Durieux » corrigé en **Näns Durieux** ; Elisabeth Roche Cornillon, Pascaline Zabardi, Stéphie Rakoto et Anthony Lascostes ajoutés aux interlocuteurs. **Premier récap reçu le 28/08 à 10h15.**

**David valide les classements automatiques**, à condition d'en être informé. Le mail d'Elisabeth classé sous `Formation/Olivier Clovis` est cohérent : il l'a rencontrée par Olivier Clovis.

### Cercle de Vérité — l'état de production du bump et des upsells

⚠️ **Aucun audio n'a jamais été enregistré**, confirmé par David : « les seules audios que j'ai sont les mp3 des vidéos Sakura que j'ai mises à disposition en audio. » Les trois audios prévus à la to-do du tunnel sont donc tous à produire. Détail dans [[Cercle de Vérité]].

- **Upsell 1 — tranché.** Aucune vidéo à 39 €, les prises existent à 17, 27, 47 et 57 €. **On diffuse `Upsell1 neutre 0 prix.mp4`**, le prix n'apparaissant que sur la page. Pas de retournage. Question close.
- **Order bump (15 Scripts Anti-Craquage) — le maillon faible.** Seul le PDF existe. Manquent l'audio de 15 min, la vidéo de 20 min, et **un mockup produit** : les deux upsells ont le leur, pas lui. Ne pas confondre avec `Mockups 10 scripts.png`, qui est le cadeau 1 du front-end.
- **Upsell 2 — le plus complet.** Reste l'audio de l'exercice guidé, dont le script est déjà écrit.
- **Vérification à faire** : passer une commande test dans le tunnel pour voir ce qu'un acheteur reçoit réellement, avant tout budget Meta.

### Sinistre dégât des eaux — LMNP vendu

Dossier **Cardif IARD n° 267 K 15422 J**. Réponse préparée dans `~/Documents/Immo/Vente LMNP/Reponse Cardif - sinistre 267K15422J.md`.

- Assureur du LMNP : **Cardif IARD (BNP Paribas)**, contrat Propriétaire Non Occupant n° 196000012428L81. Pas Allianz — Allianz, c'est l'auto.
- Syndic : **Lamy Lyon Syndic Vaise II**, copropriété « Henry Bertrand », lot 399, 56/1000es. Gestionnaire **David Juquel**, DJUQUEL@lamy-immobilier.fr, 04 72 74 50 20.
- **Assurance de la copropriété : ALLIANZ, courtier BÉLIER ASSURANCES**, police « MRI / 9B » — trouvée dans le carnet d'entretien Lamy, section « Assurance du syndicat ». Numéro non transmis à Cardif car anormalement court.
- David a choisi de renvoyer Cardif vers le syndic pour les données d'assurance manquantes plutôt que de les chercher lui-même.
- ⚠️ **Deux anomalies au constat amiable**, signalées et non résolues : la date du dégât porte **04/2021** alors que le constat est signé le 25/08/2026 (risque de prescription biennale, art. L114-1), et la case « avez-vous subi des dommages ? » est cochée **non** sur les trois exemplaires. David : cette date lui servait à tracer un historique auprès du notaire.
- **Mon erreur** : j'ai affirmé que Cardif imposait de répondre par l'Espace Client. Le pied de page le recommande, mais `gestion-sinistres@gestion-cardif-iard.fr` est une vraie boîte.

### Veille crypto ajoutée à la veille placements

Bloc ajouté à [[Veille hebdomadaire portefeuille Trade Republic]]. **David est intégralement cash sur les cryptos**, après avoir réalisé un **x10 sur un cycle précédent — 10 000 € devenus 100 000 €**. Ce n'est donc pas un suivi de position mais une veille de fenêtre d'entrée.

Univers nominatif : **BTC · ETH · BNB · XRP · SOL · ADA · AVAX · ZEC · CRO**. Le top 10 CoinMarketCap ne définit pas l'univers, il sert de détecteur — ZEC et CRO en sont absents.

Ce qu'il attend : **la lecture de phase, pas le relevé de cours**. Taxonomie fixe de six états, et un garde-fou : l'*hyper climax run* doit être nommé comme un signal de sortie, jamais comme une entrée. Source suivie : **Cédric Froment sur YouTube, qui publie le samedi** — le briefing du lundi peut l'intégrer frais. Son analyse doit toujours être attribuée, et la lecture chiffrée indépendante maintenue même quand elle le contredit.

Comme le bloc ETF, **c'est une spécification à implémenter côté serveur** dans `briefing.js`, hors périmètre de l'instance Mac.

### Divers

- **Architecture des boîtes mail** clarifiée, consignée en mémoire longue. Seules `mnemonaute1@gmail.com` et `dmarsac@sherpa-consulting.com` sont candidates à une connexion ; les deux Hotmail sortent. Demandé « no rush », rien n'a été configuré.
- **Arnaque déjouée** : faux partenariat Maono reçu le 08/07 sur la chaîne, expéditeur réel `adrian.yarlow560@wp.pl` — webmail gratuit polonais. Schéma classique de vol de compte YouTube. A donné une idée de contenu, ajoutée au tableau : « On m'a flatté pour entrer chez moi ».
- **Cadrage Radiance** : vendredi **4 septembre 11h15–11h45**, Teams, organisé par Näns Durieux avec quatre personnes de Radiance. Aucune collision avec la tripartite CFF de 14h. David a répondu à Maori le 28/08 à 10h17.
- **Vols Canaries, 12→19 décembre** : ces deux dates sont des **samedis**, et le direct Lyon–Ténérife (easyJet) ne vole pas ce jour-là. Décaler au **vendredi 11 → vendredi 18** évite aussi la vague tarifaire de Noël. Volotea facture le bagage cabine 9–10 € contre 50 € chez easyJet ; une soute unique partagée fait gagner 60 €. David a déjà fait Ténérife l'an dernier — **Gran Canaria** est la piste retenue à explorer, le Roque Nublo (1 950 m) étant accessible en décembre contrairement au Teide.

### Réglé juste après la clôture (30/08)

- **Feuilles d'émargement CFF envoyées** — les quatre manquantes (Khom ×2, Laila séance 2, Christelle) sont parties.
- **Nathalie Feugeas a eu sa réponse** sur le distanciel/présentiel des séances.
- **Réponse envoyée à Cardif** sur le sinistre 267 K 15422 J. Les deux anomalies du constat — date 04/2021 et case « dommages » cochée non — n'ont pas été corrigées ; David a choisi de ne pas y toucher. Si l'assureur les soulève plus tard, l'historique est ici.
- **Vol Canaries réservé** (île et dates non communiquées).
- **Vidéo Louis en ligne.**

**Reporté à la session suivante** : les **écrans de fin** YouTube et **Motion**.

---

## 2026-08-31 — Miniatures Gandhi, séance Elisabeth, et la routine mail réduite aux brouillons

### Higgsfield : installé, inutile, abandonné

MCP branché (`https://mcp.higgsfield.ai/mcp`, scope user), visible après redémarrage de session. **Compte à zéro crédit, plan gratuit, aucune génération possible.** Et surtout : **David n'en a pas besoin** — il a déjà un générateur d'images qui rend mieux. C'était mon idée, elle partait du principe qu'il n'avait pas d'outil. **Ne pas relancer.** Mémoire auto : `reference_higgsfield_mcp`.

⚠️ **La configuration d'un MCP ajouté en cours de session n'apparaît qu'après redémarrage** (`/exit` puis `claude --continue`). J'avais aussi affirmé l'URL du endpoint sans l'avoir vérifiée — elle s'est trouvée juste, mais c'était de la chance.

### Miniatures — trois candidates pour la vidéo Gandhi

Dans `~/Documents/Mnemonaute/Gandhi vs. Dev perso/B-roll/07 - Miniatures/Retenues/` :

- **`A-il-na-jamais-dit-ca.png`** — la meilleure. Seule à dire ce que la vidéo raconte, seule à respecter la charte jaune/blanc de la miniature 9,8/20. Défaut assumé : c'est un upscale 1280×720, molle en plein écran. **David a tranché : elle part telle quelle.**
- `E - Gandhi ecrase - FINAL.jpg` et `F - Dev perso toxique - FINAL.jpg`, en 1920×1080.

⚠️ **Les PNG faisaient 2,9 Mo, la limite YouTube est de 2 Mo** — les trois auraient été refusées à l'upload. Exportées en JPEG qualité 95.

**Découverte de David, meilleure que ma proposition** : utiliser sa **vraie photo** (celle de la miniature 9,8/20) en image de référence, plutôt qu'une génération IA. Référencer une IA, c'est photocopier une photocopie — son visage dérivait vers plus rouge, plus buriné, plus vieux. Le crop propre est dans `07 - Miniatures/Prompts IA/REF - visage David (photo reelle, sans texte).jpg`. **C'est ce réglage qui a transformé la créa « toxique »**, pas le prompt.

**Test & Compare accepte trois miniatures au maximum** — il en a exactement trois. Mais **il ne teste que les miniatures, pas les titres** : un seul titre pour les trois variantes.

### Séance Elisabeth du 31/08 — versée dans [[Coaching Elisabeth Roche - suivi des séances]]

Récupérée par **Wispr Flow** (transcript intégral, 1 h 02) — la note iOS n'était pas encore synchronisée.

- **Recadrage tenu** : ce que David appelle discipline (préparer ses affaires la veille) est de l'**engagement** et de la **motivation**. La discipline est en amont, non réfléchie. Il complète seul : il l'a apprise dans les arts martiaux, par le sensei, à la place du père.
- **Le système de soutien est une case fermée** — Codev mensuel avec Steph, groupe des magiciens, sa mère, communauté Jean & Jody. « Par rapport à la moyenne, tu en as pas mal. »
- **Le vrai déblocage : la légèreté.** Il l'a déjà en formation (posture Jim Carrey), il se la refuse en coaching **par peur d'induire**. Réponse d'Elisabeth : *« ne pense pas que tu as un grand pouvoir. Tu as juste le pouvoir de révélation. »*
- **Perso : le mariage avec Steph est rouvert.** Blocage identifié = le financement, pas le sentiment.
- ⚠️ **Séance du 07/09 : format questions-réponses inversé, 2 h, 5 à 8 questions à préparer.** Point de vigilance signalé : Elisabeth vient d'expliquer que le coach n'enseigne pas, et propose une séance où elle « a la réponse ». C'est du conseil, pas du coaching — à aborder en le sachant.

### La routine mail, réduite trois fois de suite

Routine `trig_01PHwoiyAPmAaQmr6HUxfDDJ`, modifiable par l'outil **RemoteTrigger** depuis Claude Code.

Trois versions successives dans la même session : mail unique le matin → Telegram à 8h et 12h → **plus rien du tout**.

**État final, tranché** : *« oublions les revues email et triages automatiques. J'attends que sur ma boîte tu prépares des brouillons pour les emails qui te paraissent importants et ce sera déjà bien. »* Elle lit à 8h, 12h et 17h, prépare des brouillons pour les seuls messages importants, **n'envoie rien et ne range rien**. Le tri par libellés est abandonné.

**C'est Doc qui signale les brouillons en ouverture de session** — une routine cloud ne peut pas pousser de message dans une session Claude Code. Règle écrite en mémoire auto : `feedback_brouillons_mail_annonce_en_session`.

**WhatsApp : écarté, et c'est tranché.** La voie officielle impose des templates Meta pré-approuvés et payants pour tout message automatique hors fenêtre de 24 h ; la voie officieuse (whatsapp-web.js, Baileys) fait bannir les numéros. Telegram faisait la même chose gratuitement — et a été abandonné avec le reste.

⚠️ **Le token du bot Telegram est passé dans la conversation** (`8940003285:AAH…`). À régénérer via BotFather avant tout partage de transcription. Il traîne aussi en clair dans la note iCloud « ADMIN ».

### Deux corrections de David

**Sur les absolus** : *« attention aux "jamais". Toute règle est faite pour être enfreinte un jour. »* J'avais brandi le « jamais une thèse » de son propre document comme une règle opposable. C'est un réglage par défaut, pas une loi — et le document des créas est plein d'absolus qui mériteraient d'être requalifiés en partis pris.

**Sur la charrue et les bœufs** : j'ai lancé un rendu alors qu'il avait rejeté un texte sans en valider un autre. *« Tu refais l'image alors qu'on n'a pas encore calé le texte ? »* Cinq rendus de la même carte, dont trois pour rien. **Ne produire qu'une fois le texte arrêté.**

---

## 2026-09-01 — Le système visuel du Cercle de Vérité, et le lot de créas remis d'aplomb

### Le lot de créas, repris angle par angle

Dans [[Cercle de Vérité - Créas pubs Meta (lot 1)]].

**Un quatorzième angle ajouté** : « Le développement personnel comme suradaptation », né du script Gandhi. ⚠️ **Audience tiède, pas froide** — il parle à quelqu'un qui consomme déjà du développement personnel. À trancher avec Anthony : dans le lot froid, ou en retargeting ?

**Renumérotation, tranchée** : *« on comble les trous, une idée dégagée, on ne garde pas une place vacante pour son fantôme : pragmatisme. »* Le 8 vacant (« le faux merci », retiré le 27/08) disparaît, les angles vont de **1 à 13 sans saut**. Trois renvois internes et le tableau des images suivis.

**Les étiquettes de structure retirées** des treize scripts — 91 marqueurs `[ACCROCHE]`, `[SITUATION]`… La structure reste énoncée en tête du document, les scripts se lisent d'une traite.

**Cinq chutes réécrites**, toutes pour la même raison : elles renvoyaient à un numéro de jour du protocole, ce qui ne veut rien dire pour une audience froide.

- **Angle 5** (le dimanche) : *« Sept jours pour choisir ton dimanche, au lieu de le subir. »* — formulation de David, son « ton » vaut mieux que ma version.
- **Angle 8** (le corps) : *« Sept jours pour que ton corps cesse de trahir tes décisions. »* — sa version, meilleure que la mienne : elle boucle avec la première ligne du script.
- **Angle 11** : le hook devient une question — *« et toi, tu t'es tu combien de fois cette semaine ? »* L'ancienne accroche grillait la révélation du temps 5. **« Cette semaine » est la durée exacte du protocole : l'écho est volontaire et volontairement non souligné.**
- **Angle 12** entièrement réécrit : cinq changements de personne en sept lignes, dont un « elle » ambigu au moment clé. Désormais **une seule traversée vers « tu », au temps 5, sans retour**.

**Trois refus de David qui disent sa grille** :

- **« Gagner » dans une relation** : *« comme si on pouvait gagner dans une relation. »* Le mot venait de « joue » — toute la métaphore du jeu a sauté.
- **La déception** : *« on ne peut être déçu que par soi-même et ses attentes. »* Juste pour une image nue ; dans le script, la phrase est ironique et tient.
- **« Aligné tête-cœur-corps »** : contredisait la ligne juste au-dessus — *« les arts martiaux, pas le développement personnel »*.

**Et un remplacement qui vaut d'être retenu** : « te faire plaisir » devient **« te connaître »**. Transactionnel contre existentiel — la vraie douleur n'est pas de mal dîner, c'est d'être seul au milieu des siens.

### Le système visuel — construit de zéro, consigné dans le document

**Palette choisie par David** sur Coolors : `#00296b` `#003f88` `#00509d` `#fdc500` `#ffd500`. Le jaune n'est pas neuf — c'est déjà le bandeau de la miniature 9,8/20, donc un code de chaîne existant.

**Gabarit « papier déchiré »**, 1080 × 1350 : bleu en haut (le problème), jaune en bas (la sortie), déchirure avec liseré de fibres `#f4f1ea`. Le liseré doit avoir une **épaisseur variable** et un **tracé indépendant** du bord jaune — deux courbes identiques donnent une bordure, deux courbes différentes donnent du papier.

**Trois procédés typographiques**, et la règle qui les gouverne : **un effet seulement quand le mot fait quelque chose.**

1. **La dissolution** sur « Tu disparais » — dégradé jusqu'à **6 %** (choix de David ; à 210 px la fin du mot part, c'est assumé). Le « Tu » reste plein : ce n'est pas lui qui s'efface, c'est ce qu'il montre de lui.
2. **L'insertion** — le mot manquant rendu à la phrase, en **« Are You Okay » noir**, avec un **V pointe en bas**. Cinq réglages trouvés en tâtonnant, tous dans le document.
3. **Le blanc** — un trait vide à l'endroit de la réponse absente.

**Six mains comparées** sur la même carte : Bradley Hand est une écriture neutre, Apple Chancery de la calligraphie (« soigné » là où il faut « corrigé »), **AYO dit le marqueur**. Le noir l'emporte : à 210 px le blanc s'estompe sur le jaune, et **l'encre étrangère à la palette est précisément ce qui signale « ajouté après coup »**.

**Six cartes rendues** dans `~/Documents/Mnemonaute/Cercle de Verite - Visuels/` — hors du vault. À retenir : `S-G-ayo-1.png` et `S-E-fade-F2.png`.

**Trois registres**, et c'est ce qui manquait : l'accusation, la compassion (*« Sais-tu encore ce que tu veux ? »*), et **le praticien — reste à écrire**.

⚠️ **Ne pas écrire « 50 % de mes coachés » sans avoir compté.** Une statistique inventée, la semaine d'une vidéo sur une citation apocryphe, c'est la faute que David dénonce.

⚠️ **Citation Roman Frayssinet (*La Clique*) : reconstituée de mémoire, à vérifier avant tout usage.** Et **toujours la sourcer** — l'idée de la reprendre sans attribution a été écartée : signée d'un humoriste qui n'a rien à vendre, elle prouve que le sujet existe en dehors de David. Jamais l'extrait vidéo (Canal+). Retargeting seulement.

**Question de fond restée ouverte** : ce format lit parfaitement mais **n'arrête pas le pouce**, et ressemble à ce que publie chaque coach. Piste à explorer — photo qui arrête le scroll plus bandeau plein qui garantit la lecture, avec les scènes déjà écrites dans les scripts. **À poser à Anthony.**

### Coaching stratégique Clovis — [[Scaling Academy - coaching stratégique (Olivier Clovis)]]

Fiche créée dans `3. RESSOURCES/`, **pas** dans le dossier des notes iOS, qui est régénéré à chaque synchronisation.

Session du 01/09 animée par **Baptiste** : douleur ou cible en premier, épurer au maximum, **pas de prix**. ⚠️ **Contradiction directe avec ce que j'ai consigné dans le document des créas** (garder le prix sur les créas payantes, à 27 € il tue l'objection). **À arbitrer avec Anthony.**

Le plus actionnable : un **projet Claude « media buyer 20 ans d'expérience »** qui analyse des captures du Meta manager. Et la **Facebook Ad Library** pour voir ce qui tourne chez les concurrents — le conseil qui précède tous les autres.

**Accroche de David née en séance**, de la question *« à qui je m'adresse, et quelle est la souffrance ? »* : « Tu es un homme hypersensible, tu te suradaptes et tu t'es perdu en route ? »

⚠️ **Bug trouvé : la synchronisation des notes iOS ne tournait que le dimanche.** Le fichier `~/Library/LaunchAgents/com.davidmarsac.vault-notes-sync.plist` a `Weekday: 0`. Lancée à la main. **Décision à prendre : passer en quotidien ?**

### Radiance — réponse de Simon Chatelain

**Charte graphique : question close** — leur support est en refonte, le design de David est validé. **Déroulé validé** aussi. Mais **Simon n'a pas les réponses aux questions de fond, c'est le client qui les a**, au cadrage du **jeudi 4 septembre 11h15–11h45**.

⚠️ **Trente minutes à sept personnes : les questions doivent partir en amont.** Les trois sont rédigées → [[Radiance - questions pour le cadrage du 4 septembre]]. À dire à Simon au passage : **Maori a bien répondu le 25/08**, il l'ignore.

⚠️ Le mail de Simon **n'est pas dans la boîte `david.marsac1`** — la routine ne couvre donc pas ce canal.

### Deux découvertes techniques

**Claude Code a rattrapé une partie de ce que David et Stéphane ont monté** : `/schedule` pour les routines cloud, **Remote Control** (`/remote-control` publie la session sur claude.ai et le téléphone), et la messagerie entre sessions. Le changelog complet est en local : `~/.claude/cache/changelog.md`. ⚠️ **Les MCP configurés dans Claude Code ne peuvent pas être attachés aux routines cloud.**

**Ce que Remote Control ne remplace pas** : le bot Telegram tourne sur un serveur indépendamment du Mac. Remote Control exige la machine allumée et la session ouverte.

**`/web-setup` réglerait deux choses d'un coup** : les sessions cloud verraient le vault, et la routine mail aussi. Le vault est déjà sur GitHub en dépôt privé, donc pas d'exposition supplémentaire.

**Bruno Pellen a envoyé des supports par WeTransfer le 01/09 à 9h30** — « Supports formation Prise de Parole en Public », 76 Mo, **lien expirant le 4 septembre**. David gère lui-même.

---

## 2026-09-02 — L'app Matrice codée de bout en bout, et trois ressources versées au vault

### Coaching Véronique — premier verbatim réel du dossier

Séance du 02/09 versée dans [[Coaching Véronique (Banque Savoie) - suivi des séances]], fiche créée : les séances de janvier et février n'existaient qu'en photos de notes manuscrites, inexploitables.

⚠️ **Position à surveiller** : Véronique prépare son départ, seuls le CoDir et son chef Angelo le savent. **Dominique et Marina, que David coache aussi, l'ignorent.** Rien à faire tant qu'elle n'a pas annoncé, mais à anticiper si l'une d'elles aborde son avenir dans l'équipe.

**Le contraste qu'elle ne voit pas** : avec sa collaboratrice à 120 km elle n'a **pas laissé le choix** du point hebdo — résultat, 100 % de l'effectif renouvelé en un an. Avec ses sept directeurs de groupe elle a laissé le choix : quatre ont pris, et les trois qui ne se manifestent jamais sont ceux dont elle dit « il y a du job ». **Elle a laissé le choix à ceux qui en avaient le moins la capacité.**

**Deux ouvertures commerciales nommées par elle** : coaching de la relève, et formation management pour les N-1 qui ne sont pas à l'attendu en posture. À faire remonter, mais pas par David en séance.

### Les trois blueprints de Baptiste, versés intégralement

Dans `3. RESSOURCES/Outils & Frameworks/` : [[Blueprint Instagram - optimiser son profil (Baptiste Noel)]] · [[Blueprint VSL - produire une vidéo de vente (Baptiste Noel)]] · [[Séquences de stories Instagram - les 7 frameworks (Baptiste Noel)]]. Reliés entre eux et à [[Scaling Academy - coaching stratégique (Olivier Clovis)]].

**Trois choses qui en sortent** : le Linktree tombe sous la règle du lien unique — une seule page, vers le Cercle · la rubrique « Ils l'ont fait » est le trou, et les six questions de témoignage sont prêtes · la méthode de tournage par blocs, avec claquement de mains pour repérer les ratés au montage, vaut pour toutes ses vidéos.

⚠️ **La séquence 1 de Baptiste ne transfère pas** : elle est écrite pour une niche où la preuve est monétaire (« client 1 a fait X € »). La transformation du Cercle n'est pas chiffrable en euros. L'appliquer à la lettre pousserait à inventer des chiffres.

### Correction de David : le guide qui devient barrière

*« Les recommandations ne sont pas des règles absolues, ce sont des guides, mais parfois les guides deviennent des barrières. »*

J'avais écrit « le blueprint tranche le retour que tu me demandais ». **Un blueprint ne tranche rien.** Deuxième fois de la journée qu'il me reprend sur ce registre, après les « jamais ». Cas concret : la règle du lien unique suppose une seule offre d'entrée ; David en a deux, plus une audience YouTube qui vient pour la mémoire. Le principe tient — *comprendre en trois secondes* — la règle non.

### Obsidian — un vrai trou trouvé, et un plugin refusé

⚠️ **« Toujours mettre à jour les liens internes » est désactivé.** Son `app.json` ne contient que `promptDelete`. Renommer une note casse silencieusement tous les `[[liens]]` qui pointent dessus. **À cocher dans Paramètres → Fichiers et liens.** Deux vidéos indépendantes le signalent comme le réglage critique.

**Dossier `_Modèles` créé** à la racine, le template du journal déplacé dedans. Deux réglages à repointer par David : le dossier des modèles, et l'emplacement du modèle des notes quotidiennes.

**Calendar refusé, et c'est la bonne décision** : le journal est configuré depuis le 24/08 et compte **zéro note quotidienne**. Calendar afficherait une grille vide. Installer un plugin pour encourager une habitude est le piège classique.

**Le journal n'a pas de fonction** — événements, réflexions, idées, décisions, priorités : tout est déjà capté ailleurs et mieux. **La seule chose que rien ne capte, c'est son état** — énergie, et si l'un des cinq signaux d'alerte s'est déclenché. **Décision en attente : un seul travail, ou suppression.**

### L'app Matrice — spécifiée, codée, compilée, lancée

`~/Documents/Apps/Matrice/` — **douze fichiers Swift**, compile sans erreur ni avertissement. Spécification dans [[App Matrice Eisenhower - spécification]].

**Pourquoi construire plutôt qu'utiliser Focus Matrix** : ses données sont enfermées (CloudKit privé, aucun conteneur dans iCloud Drive), et son seul pont est le Calendrier — or **un événement de calendrier n'a pas de champ pour le quadrant**. Même en datant tout, l'information qui compte ne passerait pas.

**Ce qui a été construit** : grille 2×2 avec axes étiquetés et aplats pastel · tap pour ouvrir un quadrant, **double tap pour ouvrir en saisie** · glisser-déposer entre cases · case à cocher qui archive · catégories colorées (Mnemonaute, Offres digitales, Clients, Perso, Magie) · échéance facultative · archives avec la durée de séjour · seuils réglables à la roulette.

**Seuils par quadrant, décidés par David** : urgent-et-important **2 j**, urgent-non-important **5 j**, important-non-urgent **15 j**, ni-l'un-ni-l'autre **30 j** — et ce dernier ne change pas de teinte, **il pose la question** de conserver ou non. Stockés dans le fichier JSON, pas dans les réglages iOS, pour que Doc les voie.

⚠️ **Aucun déplacement automatique vers l'urgent** quand une échéance approche — c'est le réglage que Focus Matrix a, et il dégrade la matrice en liste de tâches. **L'urgence est un jugement.**

**Reste à la charge de David** : créer le conteneur iCloud dans le portail développeur. Tant qu'il n'existe pas, l'app stocke en local, marche parfaitement pour lui, mais **Doc ne voit rien**.

### Quatre erreurs de ma part, toutes corrigées par David

**J'ai résisté à coder l'app** sur trois hypothèses qu'il a démontées : que ce serait long, que ça créerait une habitude nouvelle, que le jeu n'en valait pas la chandelle. *« Pourquoi résistes-tu à coder une application qui me semble très simple ? »*

**Je lui ai dit de donner la spécification à Antigravity** alors que je pouvais écrire le code moi-même. *« Il me semble que tu fais son taff. »* Il avait raison.

**Je n'ai posé aucune question avant d'écrire la spécification.** *« M'as-tu posé assez de questions sur l'UI, les fonctionnalités ? »* Non. Reproche déjà consigné dans `feedback_questions_avant_production`, et refait.

**La première interface était « très très vilaine »** — quatre cartes flottantes avec des titres minuscules, un tableau de bord au lieu d'une matrice. Refaite sur le modèle de ses captures d'écran.

**Et j'ai signalé deux fois comme un défaut** un popup « DIMANCHE À 18H » sur sa page de vente : c'est un composant partagé entre plusieurs tunnels, avec une automatisation qui met la bonne date du dimanche suivant. Une bonne idée d'implémentation, pas un reliquat.

### Le dossier Apps remonté

`~/Documents/Magie/Apps/` → **`~/Documents/Apps/`**. Neuf projets vérifiés, aucun chemin absolu, cinq dépôts git intacts. Une coquille vide supprimée sur demande : `Weather/MeteoMagic.xcodeproj`, sans `project.pbxproj` depuis le 4 mars 2026.

### Ce que trois vidéos apportent

**Nick Milo et le MindMappeur** convergent sur le même réglage — les liens internes. Milo recommande les **Maps of Content** plutôt que les tags : David en a déjà sans le nom (`_Index — Banque Savoie`, `Projets - ce qui a une fin`). Milo met aussi en garde contre le sur-dossierisage — *« la structure doit se mériter »* — ce qui frotte avec le PARA monté d'emblée. Et il garde **une séparation nette entre ses notes et celles générées par l'IA** ; dans le vault de David, rien ne distingue ce qu'il a pensé de ce que j'ai rédigé. Piste sérieuse pour son sentiment de ne pas s'approprier le vault.

**Julien Sanson** reprend l'architecture d'**André Karpathy** : `CLAUDE.md`, `index.md`, `log.md`, `raw/`, `wiki/`. David a l'équivalent de tout **sauf l'index** — aucune carte de l'ensemble du vault. ⚠️ **Ne pas reprendre son « pas de dossiers pour économiser les jetons »** : sa contrainte est que son agent parcourt l'arborescence, alors que Doc cherche avec `grep`. Le PARA ne coûte rien.

**Dave Jeltema — douze choses à faire après avoir publié.** Son arithmétique (« 362 fois plus de vues ») est un argument de vente, la liste est bonne. Six points applicables : **programmer 24 h à l'avance** pour que YouTube transcrive avant la mise en ligne · **chapitres écrits à la main** (le script de Gandhi les contient déjà) · **renommer le fichier vidéo** avec le titre — celui de Louis s'appelle `Rentre-eC'est-la-rentre-e-20260828-160450.MP4` · **mots-clés de chaîne** dans Studio, une fois pour toutes · **commentaire épinglé** écrit d'avance, la catchphrase est faite pour ça · **onglet Communauté**, cinquante mille abonnés et zéro usage.
⚠️ Il déconseille l'A/B de miniatures au lancement — mais sa mise en garde vise les chaînes à quelques centaines de vues. **1 600 vues en 17 h met David au-dessus du seuil**, la recommandation d'A/B tient.

### Deux limites techniques établies

**Google Docs** : lecture possible via l'export `.docx`, qui contient le texte, les couleurs **et les commentaires**. Mais **aucun connecteur Drive** — impossible d'écrire en retour. Le chemin sûr : Fichier → Télécharger → Word, puis donner le chemin.

**Notion** : `WebFetch` ne récupère que la coquille, c'est une application. Il faut coller le contenu ou l'exporter.

### Ce qui reste à faire

- [ ] **Trancher le sort du journal** : un seul travail (l'état, deux lignes par jour), ou suppression.
- [ ] **Créer l'`index.md`** du vault — carte de l'ensemble, utile à Doc autant qu'à David.
- [ ] **Créer le conteneur iCloud** de l'app dans le portail développeur.
- [ ] Cocher « toujours mettre à jour les liens internes » et repointer les deux réglages de modèles.
- [ ] Fiche dédiée pour la liste de contrôle Jeltema — **proposée, pas encore validée**.

---

## 2026-09-03 — La doctrine d'Anthony, et le pont iCloud ouvert

### Le document d'Anthony, lu en entier

`~/Downloads/Tunnel De Vente David Marsac.docx`, exporté par David depuis Google Docs. **Méthode : le `.docx` contient le texte, les couleurs et surtout `word/comments.xml`** — les douze commentaires d'**Anthony Lascostes** du 02/09. Aucune couleur dans l'export ; le repérage passe par les commentaires seuls.

⚠️ **Aucun connecteur Google Drive** : lecture possible via l'export `.docx`, **écriture impossible**. Le chemin sûr reste Fichier → Télécharger → Word.

### La doctrine, tranchée et validée par David

> **« Parler de toi de bout en bout, sans inclure l'avatar. Il doit juste se reconnaître dans des situations que tu décris. L'inclure à la fin pour le CTA. »**

Répétée sur un second script. Et deux fois : **« trop violent pour l'avatar »**.

**Le gabarit de l'idée 1 « Les 50 euros »** — la seule sans commentaire, donc le modèle : une capacité annoncée à la première personne · la scène entière racontée par David · la raison énoncée froidement · l'escalade émotionnelle dans la scène · puis la bascule *moi aussi… je pensais être gentil, jusqu'à comprendre que c'était de la manipulation… aujourd'hui j'aide ceux qui sont comme moi… si tu te reconnais là-dedans → bénéfices → CTA*.

**Trois règles fermes** : **jamais de prix** · le CTA est le même partout et **amène à la masterclass** · l'avatar n'est jamais désigné avant la dernière ligne.

⚠️ **Ça invalide la voix des treize angles de [[Cercle de Vérité - Créas pubs Meta (lot 1)]]** — tous en « tu » et accusatoires. Les scènes et les mécanismes tiennent ; **la voix est à refaire**. C'est le sujet à poser à Anthony.

**Et ça tranche l'arbitrage du prix** que j'avais laissé ouvert : pas de prix. Baptiste disait pareil le 01/09. J'avais argumenté le contraire.

### Les sept scripts réécrits

[[Cercle de Vérité - Créas réécrites à la voix de David (retours Anthony)]] — tout en « je », avec les bénéfices par pub en fin de document.

**Les bénéfices sortent du contenu réel du produit**, trouvé dans `~/Documents/Formation/Olivier Clovis Scaling Academy/LOW TICKET 1.docx` et `Structure Low Ticket Marsac.docx` — **pas du script de la VSL, qui n'existe plus sur le Mac** (il ne reste que le verrou Word de `Script Mini VSL.docx`). La VSL vidéo, elle, existe : `Video/VSL Cercle de verite low ticket.mp4`.

**Ce que le produit promet, mot pour mot** : *« Dans 7 jours, tu auras dit ton premier NON clair, assumé, sans culpabilité — et tu sentiras enfin ce que ça fait d'être un homme libre. »* Mécanisme : **le consentement masculin**. Sept jours : diagnostic des trois zones · la dette émotionnelle et les trois peurs (rejet, déception, conflit) · le triangle de Karpman · le NON sacré et son script *« je comprends que… ET en même temps, je choisis de… »* · l'ancrage corporel, le NON du Samouraï · la culpabilité d'après · le passage à l'action.

### Deux corrections de David

**« Refiler » — deuxième fois qu'il me le dit.** Sorti des quatre endroits où il traînait, y compris deux que je n'avais pas nettoyés dans le lot 1. Remplacé par **« faire porter la décision à l'autre »**, qui est plus juste : « refiler » décrit un geste désinvolte, alors qu'il s'agit d'un transfert de charge.

**Le bénéfice 7 était trop négatif** — trois menaces à la suite. Réécrit vers ce qu'on gagne, avec **« sans rien avoir ravalé »**, qui reprend son propre mot du script du champion du monde.

**Et une erreur d'attribution de ma part** : j'avais mis le « pas ouf » d'Anthony sur l'angle du champion du monde. Il porte sur **l'astreinte émotionnelle**.

### Le pont iCloud est ouvert

`~/Library/Mobile Documents/iCloud~com~davidmarsac~matrice/Documents/matrice.json` — **Doc lit la matrice.**

**Le blocage était une case non cochée** : le fichier d'autorisations déclarait `icloud-services: CloudDocuments` mais les tableaux de conteneurs étaient **vides**. L'app disait « je veux iCloud » sans dire lequel.

⚠️ **`com.davidmarsac.matrice` n'existait pas dans les Identifiers** — Xcode signait avec le profil générique `XC Wildcard *` tant que l'app n'avait aucune capacité. C'est en ajoutant iCloud que Xcode a créé l'App ID tout seul.

Marche à suivre complète et corrigée : `~/Documents/Apps/Matrice/Activer iCloud - marche à suivre.md`.

**Première lecture, le 03/09 à 9h43** : six tâches en urgent-et-important, une seule en important-non-urgent, zéro en urgent-non-important, deux en ni-l'un-ni-l'autre. **Six sur neuf dans la case rouge : le profil de quelqu'un qui subit.** Et le quadrant 2 quasi vide, alors que c'est là que dorment les deux livres, l'installation du second cerveau de Steph et les trois audios du Cercle.

### L'app, cinq ajouts

- **Bandeau** : la couleur ne remonte plus derrière la barre d'état.
- **Double tap réparé.** SwiftUI ne sait pas le distinguer d'un simple à travers une zone défilante ; fait à la main, le simple attend 0,35 s et le double l'annule.
- **Glissements** : vers la droite *Fait*, vers la gauche *Retirer*. Nouveau champ `abandonnee` — **rien ne s'efface**, mais les archives distinguent ce qui a été accompli de ce qui a été abandonné.
- **Rappel local** le matin de l'échéance à 9 h, annulé quand la tâche sort. **Il ne déplace jamais la tâche** — l'urgence reste un jugement.
- **« Ajouter à mon agenda »** via **EventKit**, avec choix du calendrier de destination dans les réglages. ⚠️ **Écarté : l'OAuth Google dans l'app** — plus de travail que tout le reste réuni. EventKit écrit sur l'iPhone, Google synchronise, et Doc lit avec le connecteur Agenda qu'il a déjà.

**Icône** : quatre quadrants, cible blanche, flèche bleu marine — l'esprit de Focus Matrix, les couleurs de l'app. Quatre pistes dans `~/Documents/Apps/Matrice/_icones/`, retenue : `F-cible-ambre`. ⚠️ **Le quatrième quadrant est ambre dans l'icône, gris dans l'app** — délibéré : le gris signifie « inerte » dans la matrice, mais tire une icône vers le bas.

### Un bug que j'ai créé et corrigé

⚠️ **L'équipe de signature sautait à chaque fois.** Mon générateur réécrivait `project.pbxproj` de zéro à chaque ajout de fichier, avec `CODE_SIGN_STYLE = Automatic` mais **sans `DEVELOPMENT_TEAM`**. Identifiant d'équipe de David : **`A9V535MUQ8`**.

**Règle à tenir** : ne plus jamais régénérer le fichier projet de zéro. Y insérer les nouveaux fichiers, comme fait pour le catalogue d'icônes.

### Coaching collectif Clovis du 03/09 — et ça alourdit la reprise du lot

Note [[Olivier 3 septembre]], transcription Wispr d'1 h 08, versée dans [[Scaling Academy - coaching stratégique (Olivier Clovis)]].

⚠️ **« Meta pénalise le marketing frontal sur la douleur. »** Contrainte de plateforme, pas question de goût — et **elle converge avec le « trop violent » d'Anthony écrit la veille sur le même corpus**. Deux sources indépendantes, l'une éditoriale, l'autre algorithmique.

**Les treize angles sont donc à reprendre pour deux motifs, pas un** : la voix en « tu », et la douleur frontale.

**Le niveau de conscience détermine qui l'on attire** : *« si on ne parle que des problèmes, on attire les gens de niveau 1 ou 2 »*. Pour viser plus haut — l'offre, l'expertise, les solutions, le détail de la méthode, le pourquoi. En séance, **Sylvie a fait ses premières ventes high ticket après dix mois infructueux, uniquement en retirant la difficulté de son storytelling.**

**Sur les visuels statiques** : utiles en retargeting seulement, sous deux formes — **visage + accroche**, ou **détail de la méthode**. Et *« éviter les clics de curiosité »*. L'exemple donné, *« arrête de travailler sur ton mental si tu te sens mal, travaille sur ton corps »*, **est exactement le mécanisme de l'angle du corps** — David a la matière. Ça tranche aussi la piste photo contre typo restée ouverte : les cartes typographiques disent la douleur, pas la méthode.

**Trois épinglés Insta** — qui je suis, méthode, témoignages : **identique à Baptiste**. Deux sources, même structure.

**Témoignages** : en tête-à-tête sur un pic émotionnel, **interview naturelle plutôt que script**, structure avant/pendant/après, vidéo > audio > écrit, et **rangés par situation de départ**. *Nuance avec les six questions écrites de Baptiste : les questions cadrent, l'entretien capte l'émotion.*

### Ce qui reste à faire

- [ ] **Refaire la voix des treize angles du lot 1** selon la doctrine. À poser à Anthony : vaut-elle pour tout le lot ?
- [ ] **Valider le bloc `[bénéfices]`** — bénéfices de la vidéo ou du programme ? un jeu par pub ou un seul ?
- [ ] Tester le nouveau build : glissements, rappels, bouton agenda, icône.
- [ ] **Créer un calendrier « Matrice »** dédié, et le désigner dans les réglages de l'app.
- [ ] Toujours ouvert depuis le 02/09 : le sort du journal, et l'`index.md` du vault.

**Radiance, tranché le 03/09 au soir** : les trois questions **ne partent pas en amont**, David les pose en direct le 04/09. Sa raison : *« si MyConnecting est à l'aise de m'envoyer ainsi, je me fais confiance pour être à propos. S'ils étaient plus exigeants ou inquiets, ils seraient dans le sur-contrôle et auraient veillé à caler tout ça en amont. »* La légèreté du client est une information sur le formalisme attendu, et une liste envoyée avant la première réunion a un coût relationnel. **J'avais insisté pour l'envoi ; son raisonnement vaut mieux que mon objection.** Ce qui survit : ouvrir le fichier pendant l'appel et poser les questions tôt.
