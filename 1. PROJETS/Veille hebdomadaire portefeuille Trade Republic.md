# Veille hebdomadaire portefeuille Trade Republic

## Objectif

Intégrer un rapport hebdomadaire automatique dans le **briefing Telegram du lundi matin** (`briefing.js morning`, cron 7h30 lun-ven) portant sur le portefeuille Trade Republic constitué le 23/07/2026. Le rapport doit remonter : performance, écarts par rapport aux poids cibles, alertes, suggestions d'arbitrage.

**À implémenter côté instance serveur** (accès à `bot.js` / `briefing.js`, hors périmètre de l'instance Mac).

## Contrainte technique

Pas d'accès API à l'account Trade Republic réel de David (pas d'intégration bancaire). Le suivi doit donc s'appuyer sur les **cours publics** des ETF/indices sous-jacents (recherche web — cours de clôture, performance depuis achat) plutôt que sur le solde exact du compte.

## Portefeuille de référence (achat du 23/07/2026)

| Enveloppe | Support | ISIN | Montant investi | Poids réel |
|---|---|---|---:|---:|
| PEA | Amundi PEA Monde (MSCI World) UCITS ETF | `FR001400U5Q4` | 10 002,58 € | 33,9 % |
| PEA | Amundi PEA Emergent (MSCI Emerging) ESG Transition UCITS ETF Acc | `FR001400ZGO4` | 1 499,55 € | 5,1 % |
| CTO | iShares Core Global Aggregate Bond UCITS ETF EUR Hedged Acc | `IE00BDBRDM35` | 7 501,07 € | 25,4 % |
| CTO | Invesco Physical Gold ETC | `IE00B579F325` | 3 000,83 € | 10,2 % |
| CTO | WisdomTree Broad Commodities UCITS ETF USD Acc | `IE00BKY4W127` | 2 501,00 € | 8,5 % |
| CTO/PEA | S&P 500 EUR (Acc) | *à préciser (ISIN non visible sur le relevé)* | 5 001,15 € | 17,0 % |
| — | Solde non investi | — | ~494 € | ~1,7 % |

**Total investi** : 29 506,18 € / 30 000 €

Ces poids réels (et non le plan initial 52,5/5/25/10/7,5 sans S&P500) servent de **référence de rééquilibrage** puisque c'est ce que David détient effectivement.

## Contexte de décision (pour que les suggestions restent cohérentes)

- Horizon : moyen terme, 5-10 ans.
- Reste du patrimoine : 50k€ assurance-vie, 30k€ Bricks (crowdfunding immobilier, roulé en continu, risque de crédit porté par David), 20-25k€ sur livrets (Livret A + LDDS, sert de coussin de sécurité — ne pas recompter une poche monétaire dans Trade Republic).
- Le S&P500 a été ajouté par préférence personnelle ("j'aime cet ETF"), pas pour diversifier — il chevauche fortement les plus grosses lignes du MSCI World (mega caps US). Ne pas suggérer de le renforcer sans le signaler comme un pari de concentration, pas une diversification.
- Vue macro assumée par David : anticipation de hausse des matières premières (pétrole compris), d'où la ligne commodities.

## Ce que le rapport hebdomadaire doit couvrir

1. **Performance par ligne** depuis le 23/07/2026 (cours du support ou de son indice de référence, en %).
2. **Écart aux poids cibles** ci-dessus — alerte si une ligne dérive de plus de **5 points** par rapport à son poids d'origine (ex. actions qui montent fortement et dépassent 60% du total).
3. **Actualité macro pertinente** de la semaine susceptible d'affecter le portefeuille : décisions de taux (BCE/Fed), inflation, tensions géopolitiques, mouvements significatifs sur l'or/matières premières.
4. **Suggestions d'arbitrage** si un rééquilibrage semble justifié (ex. "les actions ont dérivé à +7pts, envisager de diriger les prochains versements vers les obligations plutôt que vendre").
5. Rester factuel et concis — pas de conseil en investissement réglementé, cadrage éducatif uniquement (cf. échanges du 23/07/2026 sur ce sujet).

## Fréquence

Hebdomadaire, le lundi matin, intégré au message existant plutôt qu'en message séparé.

---

# Bloc crypto (ajouté le 28/08/2026)

## Statut

David est **intégralement cash sur les cryptos** à ce jour. Il a réalisé un **x10 sur un cycle précédent — 10 000 € devenus 100 000 €** — en multipliant les poches. Il connaît le terrain : la veille n'a pas à expliquer les fondamentaux, elle doit remonter des signaux.

## Nature du bloc — différente du bloc ETF

Ce n'est **pas un suivi de position**. Pas de poids cibles, pas d'écart de rééquilibrage, pas de performance depuis achat, puisqu'il n'y a pas de ligne détenue. L'objet est une **veille de marché destinée à préparer une décision d'entrée** et à en repérer la fenêtre.

## Univers suivi

Liste nominative — les poches que David a détenues :

**BTC · ETH · BNB · XRP · SOL · ADA · AVAX · ZEC · CRO**

Le top 10 CoinMarketCap **ne définit pas l'univers**, il sert de détecteur : si un actif absent de cette liste entre dans le top 10, le signaler en une ligne, sans l'ajouter au suivi. Arbitrage retenu le 28/08/2026 — ZEC et CRO sont hors top 10, suivre le top 10 les aurait fait disparaître.

## Ce que David attend réellement

Précisé le 28/08/2026 : **la lecture de phase, pas le relevé de cours.** La question à laquelle le briefing doit répondre chaque lundi est « où en est-on dans le cycle, et est-ce le moment de bouger ou d'attendre ». Les chiffres sont la pièce justificative, pas le contenu.

Ses mots : un *break*, un signal de prise de position ascendante, un *hyper climax run*, une continuation de tendance, ou l'instruction de patienter.

## Taxonomie des états — vocabulaire fixe d'une semaine à l'autre

Le briefing qualifie la phase avec **un seul de ces termes**, pour que deux lundis soient comparables :

| État | Ce qu'il décrit | Lecture pour David, aujourd'hui cash |
|---|---|---|
| **Accumulation / range** | Prix latéral, volumes faibles, volatilité comprimée | Patienter, mais préparer les niveaux |
| **Break haussier** | Sortie de range confirmée en clôture, avec volume | Le signal d'entrée qu'il cherche |
| **Continuation** | Tendance installée, replis rachetés | Entrée possible mais moins favorable qu'au break |
| **Hyper climax run** | Accélération verticale, euphorie, volumes extrêmes | ⚠️ **Signal de fin de cycle, pas d'entrée** |
| **Distribution / retournement** | Sommets plats, divergences, force qui s'épuise | Ne pas entrer |
| **Bear / capitulation** | Baisse installée ou purge violente | Zone d'accumulation à surveiller |

**Le piège à ne jamais laisser passer** : l'*hyper climax run* est le moment où le marché est le plus attirant et le plus dangereux. David étant cash et cherchant une entrée, le briefing doit le nommer explicitement comme un signal de sortie ou d'abstention, jamais comme une opportunité.

## Format hebdomadaire

À intégrer au briefing Telegram du lundi, **après** le bloc ETF, en restant court : le message contient déjà six lignes d'ETF.

1. **L'état de phase en un mot**, pris dans le tableau ci-dessus, pour BTC — c'est BTC qui donne le tempo du reste.
2. **La justification en une phrase** : ce qui a changé par rapport au lundi précédent, ou « inchangé ».
3. **Les niveaux qui comptent** : le seuil dont la rupture invaliderait ou confirmerait la lecture. Deux chiffres, pas plus.
4. **Dominance BTC et capitalisation totale.** La dominance qui baisse pendant que le marché monte signale une rotation vers les altcoins — c'est directement pertinent pour les sept autres poches.
5. **Les sept autres en une ligne.** Seules celles qui franchissent le seuil ci-dessous sont nommées.
6. **Actualité uniquement si structurelle** : décision réglementaire, flux ETF significatifs, incident majeur sur une chaîne. Pas de reprise de bruit médiatique ni de prévision de prix.

## Source suivie — Cédric Froment (YouTube)

David suit cette chaîne, qui publie **le samedi**. Le calendrier tombe bien : la vidéo du samedi précède le briefing du lundi matin, elle peut donc être intégrée fraîche.

**À faire** : récupérer la transcription de la vidéo de la semaine et en tirer trois à cinq lignes — la phase qu'il décrit, les niveaux qu'il cite, ce qu'il attend comme confirmation.

**Contrainte technique à valider côté serveur** : cela suppose de récupérer les sous-titres YouTube automatiquement (yt-dlp ou équivalent) depuis l'instance serveur. Non vérifié à ce jour. Si ça ne passe pas, se rabattre sur un rappel « vidéo du samedi à regarder » avec le lien, plutôt que de rendre le bloc dépendant d'une brique qui échoue.

⚠️ **Toujours attribuer.** Le briefing écrit « Cédric Froment considère que… », jamais « le marché est en… ». C'est son analyse, pas un fait de marché, et la distinction doit rester visible dans le message.

## Seuils propres à la crypto

Le seuil de dérive à 5 points calibré pour les ETF n'a aucun sens ici : une semaine à ±30 % est banale.

- **±20 % sur 7 jours** → l'actif est nommé dans la ligne de synthèse.
- **±40 % sur 7 jours** → alerte explicite.

## Points de vigilance

- **Les neuf actifs ne sont pas de même nature.** BTC et ETH ont un rôle structurel dans l'écosystème. XRP dépend d'un modèle d'adoption bancaire et d'un historique judiciaire américain. CRO est adossé à un exchange, donc porteur d'un risque de contrepartie propre. ZEC est hors top 10 depuis longtemps. Ne pas les aligner dans un même tableau sans distinction : la mise en page suggérerait une comparabilité qui n'existe pas.
- **Un x10 réalisé sur un cycle précédent ne se reproduit pas mécaniquement.** La veille reste descriptive et ne formule jamais d'objectif de performance.
- **Cadrage éducatif uniquement, pas de conseil en investissement** — même règle que le bloc ETF.
- **Une seule source d'analyse est un risque.** Adosser la lecture de phase à Cédric Froment seul revient à déléguer le jugement à une personne. Si sa lecture se révèle fausse sur un cycle, rien dans le briefing ne le corrigera. Deux garde-fous : conserver la lecture chiffrée indépendante (points 1 à 5 du format) même quand elle contredit la vidéo, et signaler explicitement les semaines où les deux divergent — ce sont les plus instructives.
- ⚠️ Le titre de cette note ne couvre plus son contenu : elle ne parle plus seulement de Trade Republic. À renommer si le bloc crypto est confirmé.

→ [[Finances]]
