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
