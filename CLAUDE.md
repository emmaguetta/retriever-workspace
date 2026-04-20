# Retriever Workspace — CLAUDE.md

## C'est quoi

Workspace multi-repos qui regroupe les 4 repos de l'écosystème Retriever. Chaque sous-dossier est un **repo Git indépendant** avec son propre remote GitHub. Ce CLAUDE.md donne le contexte global ; chaque sous-repo peut avoir son propre CLAUDE.md pour le contexte local (Claude Code les cascade automatiquement).

## Les 4 repos

| Dossier | Remote | Stack | Rôle |
| --- | --- | --- | --- |
| `GTMbuilder/` | `emmaguetta/GTMbuilder` | Python (MLX, DuckDB, crawl4ai) | ICP search engine — pipeline data, extraction curseurs LLM, embeddings |
| `retriever-apps/` | `yonsy7/retriever-apps` | backend + frontend | Apps Retriever (produit) |
| `retriever-marketing-website/` | `yonsy7/retriever-marketing-website` | Next.js + Tailwind | Site marketing / landing |
| `Retriever-WEB-EXTRACTION/` | `yonsy7/Retriever-WEB-EXTRACTION` | Python (scraping) | Extraction web, scraping, pipeline data |

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
- **CLAUDE.md de GTMbuilder** contient un working directory obsolète (`/Users/yoannaka/...`) — à corriger au passage quand on travaille dessus.
