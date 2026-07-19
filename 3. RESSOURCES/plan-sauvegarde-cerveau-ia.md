# Plan de sauvegarde vers GitHub — cerveau-ia

## 1. Dossier local de sauvegarde proposé

Ne pas transformer directement Doc/ ou telegram-claude-bot/ en repos Git (pour ne pas mélanger un .git avec un vault Obsidian actif ou un bot en prod). À la place :

```
/home/david/backup-cerveau-ia/          <- nouveau repo git local, dédié à la sauvegarde
├── .git/
├── .gitignore
├── Doc/                                <- copie/rsync du contenu de /home/david/Doc
└── telegram-claude-bot/                <- copie/rsync du contenu de /home/david/telegram-claude-bot
```

Synchronisation via rsync (avec exclusions) depuis les dossiers sources vers ce repo, à chaque sauvegarde — pas de lien symbolique, pas de `git init` dans les dossiers de travail eux-mêmes.

## 2. Fichiers inclus

**Doc/** : tout le vault Obsidian (.md, .obsidian/, .claude/settings.json, CLAUDE.md, memory.md, etc.) — exclusion de `.claude/settings.local.json` par principe (local à la machine).

**telegram-claude-bot/** : bot.js, veille-boamp.js, briefing.js, checker.js, auth-google.js, mcp-google-server.js, package.json, package-lock.json, rappels.json (pas sensible), .gitignore.

## 3. Fichiers exclus (dans le .gitignore du repo de sauvegarde)

```
# Secrets / tokens / clés
**/.env
**/credentials.json
**/tokens.json
**/*.pem
**/*.key
**/id_ed25519*
.ssh/

# Sessions / mémoire d'exécution sensible
**/.sessions.json
**/.boamp-seen.json

# Dépendances
**/node_modules/

# Logs volumineux
**/*.log
**/logs/
**/.veille-boamp.lock

# Fichiers locaux à la machine
Doc/.claude/settings.local.json
```

Confirmation explicite des points demandés :
- .env (Doc et bot) -> exclu
- tokens.json, credentials.json (identifiants Google OAuth) -> exclus
- node_modules/ -> exclu
- checker.log (2+ Mo), briefing.log, logs/veille-boamp.log -> exclus (règle *.log + logs/)
- .sessions.json -> exclu
- .boamp-seen.json -> exclu
- ~/.ssh -> jamais copié dans le dossier de sauvegarde, et exclu par précaution si jamais il traînait

## 4. Commandes Git prévues

```bash
mkdir -p /home/david/backup-cerveau-ia
cd /home/david/backup-cerveau-ia
git init
git remote add origin git@github.com:DavidMSakura/cerveau-ia.git

# écrire le .gitignore ci-dessus

mkdir -p Doc telegram-claude-bot
rsync -a --delete \
  --exclude='.env' --exclude='.obsidian/workspace.json' \
  --exclude='.claude/settings.local.json' \
  /home/david/Doc/ ./Doc/

rsync -a --delete \
  --exclude='.env' --exclude='node_modules/' --exclude='*.log' \
  --exclude='logs/' --exclude='.sessions.json' --exclude='.boamp-seen.json' \
  --exclude='.veille-boamp.lock' --exclude='credentials.json' --exclude='tokens.json' \
  /home/david/telegram-claude-bot/ ./telegram-claude-bot/

git add -A
git status   # vérification manuelle avant commit
git commit -m "Sauvegarde initiale Doc + telegram-claude-bot"
git push -u origin main
```

(Pas de cron créé à cette étape — juste la sauvegarde initiale manuelle.)

## 5. Comment éviter d'envoyer les secrets

- Exclusion à la source : le rsync exclut .env, credentials.json, tokens.json avant même qu'ils touchent le dossier de sauvegarde — ils ne sont jamais copiés, donc jamais dans l'historique Git.
- .gitignore en filet de sécurité : même si un fichier sensible se glissait par erreur dans le dossier de sauvegarde, le .gitignore l'empêcherait d'être stagé.
- Vérification manuelle avant push : après le git add -A, vérifier git status et la liste exacte des fichiers stagés (pas leur contenu) avant de committer/pusher.
- ~/.ssh jamais dans le champ : ni Doc/ ni telegram-claude-bot/ ne contiennent ~/.ssh.
- Aucun cat/affichage de .env, tokens.json ou credentials.json ne sera fait à aucun moment.
