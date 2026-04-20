# Retriever Workspace — CLAUDE.md

## C'est quoi

Workspace multi-repos qui regroupe les 4 repos de l'écosystème Retriever. Chaque sous-dossier est un **repo Git indépendant** avec son propre remote GitHub. Ce CLAUDE.md donne le contexte global ; chaque sous-repo peut avoir son propre CLAUDE.md pour le contexte local (Claude Code les cascade automatiquement).

## Vue d'ensemble

| Dossier | Remote | Nature |
| --- | --- | --- |
| `retriever-business/` | `emmaguetta/retriever-business` | **Business & docs only** (pas de code) |
| `retriever-apps/` | `yonsy7/retriever-apps` | **Produit user-facing** (API + dashboard) |
| `retriever-marketing-website/` | `yonsy7/retriever-marketing-website` | **Site marketing** (landing Next.js) |
| `Retriever-WEB-EXTRACTION/` | `yonsy7/Retriever-WEB-EXTRACTION` | **R&D + pipeline data** (extraction, scraping, expériences) |

## Qui consomme qui ?

- `retriever-apps/` = produit livré → c'est ce que voient les users/devs
- `Retriever-WEB-EXTRACTION/` = laboratoire R&D → alimente la logique d'extraction de retriever-apps
- `retriever-marketing-website/` = acquisition → amène les leads vers retriever-apps
- `retriever-business/` = pilotage → docs stratégiques, sales, product qui informent les 3 autres

---

## `retriever-business/` — docs business

Repo de référence business : stratégie, sales, marketing, product, coûts, discovery, compétition, to-do. **Pas de code.**

Tooling Claude Code : skills dédiés (`add-competitor`, `add-product-feature`, `update-crm`, `update-todo`, `discovery-call-notes`, `master-add-discovery-call`) — chaque skill demande validation avant modif.

```
retriever-business/
├── docs/
│   ├── sales/
│   │   └── crm.tsv                          ← CRM prospects (TSV, 1 ligne/prospect)
│   ├── strategy/
│   │   ├── strategy.md                      ← vision, roadmap, décisions clés
│   │   └── insights.md                      ← insights marché / utilisateurs
│   ├── product/
│   │   ├── product features.md              ← features (blocs narratifs)
│   │   ├── product features.tsv             ← idem en tableau
│   │   └── targeted queries.md              ← exemples de requêtes cibles
│   ├── competition/
│   │   ├── competitors.md                   ← fiches compétiteurs (narratif)
│   │   ├── features-comparison.md           ← tableau comparatif Markdown
│   │   ├── features-comparison.tsv          ← idem en TSV
│   │   ├── exa_ai.md / gojiberry.md         ← fiches individuelles
│   │   └── data sources.md
│   ├── discovery/
│   │   └── discovery_calls/                 ← un .md par call (date-nom-société)
│   ├── marketing/
│   │   └── linkedin/idees_post.md
│   ├── couts/
│   │   └── hypothese-couts.md
│   └── to do/
│       └── to do.md                         ← backlog P0/P1/P2 + semaine
└── memory/
    ├── MEMORY.md                            ← index mémoire Claude Code
    └── *.md                                 ← fichiers mémoire individuels
```

---

## `retriever-apps/` — le produit

**Le produit user-facing.** Auth, API keys/MCP tokens, recherche de companies (credit-based), listes, profils entreprises, billing Stripe.

**URLs :** dashboard → https://app.retriever.run · API → https://api.retriever.run · marketing → https://retriever.run

```
retriever-apps/
├── backend/
│   ├── .env.example                         ← LIRE EN PREMIER (vars requises)
│   ├── truth-store/                         ← LIRE EN SECOND (brief, endpoints, data-model, flows)
│   │   ├── brief.md                         ← vision produit + contraintes
│   │   ├── endpoints.md                     ← tous les endpoints documentés
│   │   ├── data-model.md                    ← schéma BDD expliqué
│   │   ├── flows.md                         ← flux auth, billing, search
│   │   ├── tech-stack.md
│   │   └── use-cases.md
│   ├── src/app/
│   │   ├── server.ts                        ← entry point Fastify (port 3000)
│   │   ├── config.ts                        ← validation env vars
│   │   └── middleware/                      ← auth.middleware, api-key.middleware
│   ├── src/features/
│   │   ├── auth/                            ← register, login, refresh, reset password
│   │   ├── search/                          ← /v1/search?q= (+ déduction crédits)
│   │   ├── billing/                         ← Stripe checkout, plans, credits.service
│   │   ├── lists/                           ← sauvegarder/gérer des listes de résultats
│   │   ├── companies/                       ← lookup par domaine
│   │   ├── mcp/                             ← serveur MCP HTTP (/mcp), tools search/lists/companies/usage
│   │   ├── mcp-tokens/                      ← tokens mcp-... pour Claude Code
│   │   ├── api-keys/                        ← gestion API keys
│   │   ├── user/                            ← profil utilisateur
│   │   ├── usage/                           ← suivi crédits consommés
│   │   └── admin/                           ← panel admin (stats, gestion users)
│   ├── src/shared/
│   │   ├── db/prisma.ts                     ← client Prisma singleton
│   │   ├── search-api/custom.provider.ts    ← appels au moteur de recherche externe
│   │   ├── payment/stripe.provider.ts       ← Stripe
│   │   └── email/resend.provider.ts         ← Resend
│   ├── infra/prisma/schema.prisma           ← schéma BDD
│   └── docker-compose.yml                   ← Postgres local (port 5433)
└── frontend/
    ├── src/config/env.js                    ← URLs API/app/marketing (+ fallback localhost:3003 ⚠️)
    ├── src/router.jsx                       ← toutes les routes
    ├── src/pages/                           ← SearchPage, ListsPage, CompanyPage, SettingsPage...
    ├── src/api/                             ← clients API (auth, search, lists, companies, user...)
    ├── src/hooks/                           ← React Query hooks (useSearch, useLists, useAuth...)
    └── src/context/AuthContext.jsx
```

**Lancer en local :**
```bash
# Backend
cd retriever-apps/backend
cp .env.example .env   # éditer: JWT_ACCESS_SECRET, API_KEY_ENCRYPTION_SECRET (32 chars),
                       # SEARCH_API_URL + SEARCH_API_KEY
npm install
npm run dev:local      # PGlite embarqué + migrations auto → :3000

# Frontend (autre terminal)
cd retriever-apps/frontend
npm install
VITE_API_BASE_URL=http://localhost:3000 npm run dev   # → :5173
```

⚠️ **Gotcha port :** sans `VITE_API_BASE_URL`, le frontend tape `localhost:3003` (pas 3000) → tous les appels API échouent.

Ce qui marche/casse sans services optionnels : ✅ search + lists (si SEARCH_API_URL tourne) · ❌ register/reset pwd (sans Resend) · ❌ billing (sans Stripe) · ❌ admin (sans ADMIN_SECRET).

---

## `retriever-marketing-website/` — landing

Site marketing Next.js qui présente le produit. Stack : Next.js 16.2.3 + React 19 + TypeScript + Tailwind 3.4 + Vercel Analytics. État : v0.1.0, pre-launch.

```
retriever-marketing-website/
├── app/
│   ├── page.tsx                             ← landing principale (scroll animations, parallax hero)
│   ├── mcp/page.tsx                         ← page produit MCP dédiée
│   ├── layout.tsx                           ← layout global + metadata SEO
│   ├── globals.css                          ← styles globaux
│   ├── robots.ts / sitemap.ts               ← SEO
├── tailwind.config.ts                       ← theme, couleurs, fonts
├── next.config.ts
└── public/fonts/                            ← UrbaneRounded (tous les poids woff/woff2)
```

Note : `index.html`, `landing-page-inspiration.html`, `mcp.html` et `convert.js/py` — traces d'itérations design, pas dans le build Next.js.

---

## `Retriever-WEB-EXTRACTION/` — R&D + pipelines

**Repo multi-projets.** Plusieurs sous-projets indépendants — pas de point d'entrée unique. Lire `plan.md` + `1.md` pour la vision globale.

```
Retriever-WEB-EXTRACTION/
│
├── icp-matching-mvp/                        ★ MATURE — framework eval schémas ICP
│   ├── index.md                             ← vue d'ensemble (lire en premier)
│   ├── criteria-v1.md                       ← critères ICP retenus
│   ├── canonical-schema.md                  ← schéma extraction expliqué
│   ├── company-canonical-v1.schema.json     ← schéma JSON (19KB)
│   ├── company-types.md
│   └── decisions/criteria-registry-v1.md
│
├── website-processing/                      ★ MATURE — pipeline crawl + extraction LLM
│   ├── scripts/extract-company.py           ← script principal (crawl → JSON structuré)
│   ├── scripts/generate-example-extractor-prompt.py
│   ├── schemas/company-common-criteria-v1.schema.json
│   ├── schemas/company-page-selection-v1.schema.json
│   └── examples/                            ← exemples d'extraction (Clay, Payfit, Pigment...)
│
├── MVP/                                     ⚙ WIP — pipeline RAG local
│   ├── data/sample/                         ← scripts pipeline (create_sample, crawl, embed, filter_llm...)
│   │   └── .env (gitignored)                ← CEREBRAS_API_KEY
│   ├── data/pdl-company/                    ← analyses dataset PDL 35M entreprises
│   ├── data/linkedin-company-hugging-face/  ← analyses dataset LinkedIn 3.9M
│   ├── experiments/semantic_filtering/llm_as_a_judge/  ← Qwen MLX, LLM-as-judge 85% accuracy
│   ├── docs/tech/                           ← crawling.md, data-storage.md, llm-inference.md
│   ├── planification/                       ← what-to-extract.md, llm-strategy.md
│   └── PLAN-V0-Building.md
│
├── linkedin-automation/                     ⚠ PARTIEL — extension Chrome MV3
│   ├── extension/manifest.json              ← charger en Chrome (dev mode)
│   ├── extension/content.js                 ← injection page LinkedIn
│   ├── extension/background.js
│   ├── extension/popup.html / popup.js
│   └── STATUS.md                            ← ce qui marche (feed ✅) / casse (messaging ❌)
│
├── free-api/                                ✦ EXPLORATOIRE — clients LLM gratuits
│   ├── chat-gateway.mjs                     ← gateway multi-provider
│   ├── gemini-* / grok-* / kimi-* / zai-* / perplexity-* / inception-*
│   └── *-INVESTIGATION.md                   ← notes d'investigation par provider
│
├── data/
│   └── companies.france.parquet             ← 103MB, gitignored (analyse DuckDB)
├── tech/proxies/                            ← listes Webshare 100+500 (⚠️ credentials dedans)
├── 1.md                                     ← doc pipeline benchmark ICP (FR, détaillé)
├── plan.md                                  ← vision & 10 phases du moteur ICP
└── .gitmodules                              ← submodule Andrew-Girgis/linkedin-company-scraper
```

**Submodule :** `MVP/experiments/linkedin_scrapping/Andrew-Girgis-linkedin-company-scraper` → pinné commit `a696cae`.

---

## Règles de navigation

- **Avant de modifier du code, toujours `cd` dans le bon sous-repo.** Remotes différents, branches différentes, historiques séparés.
- Ne jamais faire `git add .` depuis la racine — les sous-repos sont gitignorés au niveau workspace.
- Les `CLAUDE.md` locaux cascadent automatiquement quand on bosse depuis un sous-repo.

## Workflow push

| Modifs dans | Commande |
| --- | --- |
| Un sous-repo | `cd <sous-repo> && git push` |
| Workspace (ce CLAUDE.md, commandes) | `git push` depuis la racine → `RetrieverGTM/retriever-workspace` |

## Points d'attention

- **iCloud :** workspace dans iCloud Drive. Si bizarreries `.git/`, attendre fin de sync.
- **Secrets exposés :** clé Cerebras (MVP/.env) et creds Webshare (proxies) dans l'historique git. À rotate côté fournisseur.
