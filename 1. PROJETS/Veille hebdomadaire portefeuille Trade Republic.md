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
