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

---

### `retriever-business/` — docs business

Repo de référence business : stratégie, sales, marketing, product, coûts, discovery, compétition, to-do. Pas de code.

- **Structure :** `docs/` avec sous-dossiers `sales/`, `marketing/`, `strategy/`, `product/`, `competition/`, `discovery/`, `couts/`, `to do/`
- **Tooling Claude Code :** skills dédiés (`add-competitor`, `add-product-feature`, `update-crm`, `update-todo`, `discovery-call-notes`, `master-add-discovery-call`) + memory dans `memory/`
- **Format :** compétiteurs et features maintenus en double (bloc narratif Markdown + ligne TSV pour tableaux)

---

### `retriever-apps/` — le produit

**Le produit user-facing de Retriever.** Les utilisateurs s'authentifient, gèrent API keys / MCP tokens, lancent des recherches de companies (credit-based), sauvegardent des listes, consultent des profils entreprises.

**URLs déployées :**
- **Dashboard (front)** : https://app.retriever.run
- **API (back)** : https://api.retriever.run (docs Swagger : `/docs`)
- **Marketing (landing)** : https://retriever.run (→ pointe vers `retriever-marketing-website/`)

**Structure :**
- **`backend/`** : API Node.js/Fastify, Prisma + PostgreSQL, Stripe (billing credits), Sentry, Resend. 11 modules (auth, search, billing, lists, mcp, admin, usage, etc.). Entry point : `backend/src/app/server.ts`
- **`frontend/`** : SPA React 19 + Vite + React Router v7 + React Query + Tailwind. Routes principales : `/`, `/searches`, `/lists`, `/company/:domain`, `/settings`, `/api`
- **Endpoints clés :** `/v1/search?q=...` (recherche avec déduction de crédits), `/mcp` (serveur MCP HTTP pour Claude Code, tokens `mcp-...`)
- **Billing :** Stripe Checkout, packs de crédits (1K/$9, 10K/$49, 100K/$199)

**Lancer en local :**

Prérequis : Node 20+ (pas de `engines` dans package.json mais la stack l'exige). Pas de Docker obligatoire : le backend peut utiliser PGlite embarqué. `npm` (pas de lockfile pnpm/yarn).

```bash
# ── Backend ──
cd retriever-apps/backend
cp .env.example .env
# Éditer .env : JWT_ACCESS_SECRET et API_KEY_ENCRYPTION_SECRET (32 chars min),
# SEARCH_API_URL + SEARCH_API_KEY (vers un moteur de recherche qui tourne).
# Stripe/Resend/Sentry optionnels pour dev local.
npm install
npm run dev:local        # PGlite embarqué + migrations auto → écoute sur :3000

# Alternative avec Postgres Docker :
# docker compose up -d postgres   # expose 5433 (user/password, DB search_api)
# npm run db:migrate && npm run dev

# ── Frontend (dans un autre terminal) ──
cd retriever-apps/frontend
npm install
VITE_API_BASE_URL=http://localhost:3000 npm run dev    # UI sur :5173
```

⚠️ **Gotcha port :** le backend écoute sur **3000** (via `.env`), mais le frontend tape **`localhost:3003`** par défaut quand on n'est pas sur `app.retriever.run`. D'où le `VITE_API_BASE_URL=http://localhost:3000` — sans ça, la UI charge mais chaque appel API échoue.

**Ce qui marche / casse en minimal env :**
- ✅ Search, lists, company lookup (si `SEARCH_API_URL` tourne)
- ❌ Register / password reset (silencieux sans `RESEND_API_KEY`)
- ❌ Billing / achat de crédits (sans clés Stripe)
- ❌ Admin endpoints (requiert `ADMIN_SECRET` + JWT avec `isAdmin: true`)

**Pas de seed :** créer un user via `POST /v1/auth/register` ou bootstrap admin manuellement.

**État :** production-ready (graceful shutdown, Sentry, logs Pino structurés, transactions sûres sur les crédits).

---

### `retriever-marketing-website/` — landing

Site marketing Next.js qui présente le produit.

- **Stack :** Next.js 16.2.3 (App Router) + React 19 + TypeScript + Tailwind 3.4 + Vercel Analytics
- **Entry points :** `app/page.tsx` (landing principale avec animations scroll, parallax hero, navbar shrinking), `app/layout.tsx`, `app/mcp/` (page produit MCP dédiée), `app/robots.ts` + `app/sitemap.ts`
- **Assets :** `public/` (fonts, logos SVG)
- **État :** v0.1.0, early-stage, pre-launch. Déploiement Vercel-ready
- **Note :** des HTML de référence (`index.html`, `landing-page-inspiration.html`, `mcp.html`) et scripts de conversion (`convert.js/py`) — traces d'itérations design

---

### `Retriever-WEB-EXTRACTION/` — R&D + pipelines

**Repo fourre-tout pour tout ce qui est extraction web, scraping, expériences LLM, et pipelines data.** Plusieurs sous-projets indépendants cohabitent — pas un seul point d'entrée.

**Sous-projets actifs :**
- **`icp-matching-mvp/`** *(mature)* — Framework d'évaluation de schémas ICP. Le schéma canonique (`company-canonical-v1.schema.json`) est testé sur un benchmark retrieval (BM25 + vectoriel) pour valider quel schéma d'extraction maximise la précision de recherche B2B sémantique.
- **`website-processing/`** *(mature)* — Pipeline Python de crawl + extraction LLM structurée vers JSON. Script principal : `extract-company.py`. Utilise un endpoint Codex custom pour les appels LLM.
- **`MVP/`** *(WIP)* — Pipeline RAG local sur dataset d'entreprises françaises. Phases : sample PDL → Parquet/DuckDB → crawl (crawl4ai) → embeddings → filtering LLM → vector index → RAG eval. Inclut `experiments/` (Qwen via MLX sur Apple Silicon, LLM-as-judge à 85% accuracy, scraping LinkedIn). Entry : `MVP/data/sample/create_sample.py`. **Contient `.env` avec clé Cerebras — gitignoré localement.**
- **`linkedin-automation/`** *(partiellement fonctionnel)* — Extension Chrome MV3. Like/comment/reply sur le feed marchent ; messaging et connection requests cassés (blocage React Fiber). Load via `extension/manifest.json`.
- **`free-api/`** *(exploratoire)* — Clients Node.js pour tester des LLMs gratuits (Gemini, Grok, Kimi, ZAI, Perplexity) via Playwright/Puppeteer.

**Autres :**
- **`data/`** — `companies.france.parquet` (~103MB), dataset entreprises françaises pour analyses DuckDB
- **`tech/proxies/`** — Listes de proxies Webshare (100 + 500). **Contient credentials** (user/pass).
- **`scripts/`** — quasi vide
- **`tmp-rayobrowse/` · `tmp-schema-assets/` · `tmp-steel-browser/`** — stubs vides, résidus d'expés abandonnées
- **`1.md`, `plan.md`** — docs stratégiques (français) sur le pipeline ICP et la vision

**Submodule :** `MVP/experiments/linkedin_scrapping/Andrew-Girgis-linkedin-company-scraper` → `Andrew-Girgis/linkedin-company-scraper` (pinné commit `a696cae`)

---

## Qui consomme qui ?

- `retriever-apps/` = produit livré → c'est ce que voient les users/devs
- `Retriever-WEB-EXTRACTION/` = laboratoire R&D → alimente la logique d'extraction utilisée (ou inspirée) par retriever-apps
- `retriever-marketing-website/` = acquisition → amène les leads vers retriever-apps
- `retriever-business/` = pilotage → docs stratégiques, sales, product qui informent les 3 autres

---

## Règles de navigation

- **Avant de modifier du code, toujours `cd` dans le bon sous-repo.** Les 4 repos sont indépendants : remotes différents, branches différentes, historiques séparés.
- Ne jamais faire `git add .` depuis la racine en pensant commiter dans un sous-repo — les sous-repos sont gitignorés au niveau workspace.
- Les `CLAUDE.md` locaux (dans chaque sous-repo) cascadent automatiquement quand on bosse depuis ce sous-repo.

## Workflow push

| Où sont les modifs | Commande |
| --- | --- |
| Dans un sous-repo | `cd <sous-repo> && git push` → push vers le remote du sous-repo |
| Workspace-level (ce CLAUDE.md, commandes, docs cross-repo) | `git push` depuis la racine → push vers le remote workspace |

Le `.gitignore` à la racine exclut les 4 sous-repos, donc aucun risque de mélanger les commits.

## Points d'attention

- **iCloud :** workspace stocké dans iCloud Drive. Si bizarreries sur `.git/`, vérifier qu'iCloud n'est pas en cours de sync. En cas de problèmes récurrents, envisager de déplacer vers `~/dev/`.
- **Secrets exposés :** clé Cerebras (`.env` de MVP) et creds Webshare (proxies) sont dans l'historique de `retriever-business` et `Retriever-WEB-EXTRACTION`. À rotate côté fournisseur si tu veux être propre.
