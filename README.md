# Retriever Workspace

Dossier de travail qui regroupe les 4 repos de l'écosystème Retriever pour bosser avec Claude Code.

## Structure

```
Claude for GTMBuilder/
├── retriever-business/              → emmaguetta/retriever-business
├── retriever-apps/                  → yonsy7/retriever-apps
├── retriever-marketing-website/     → yonsy7/retriever-marketing-website
└── Retriever-WEB-EXTRACTION/        → yonsy7/Retriever-WEB-EXTRACTION
```

Chaque sous-dossier est un repo Git indépendant. Ce repo-ci ne contient que le scaffolding workspace (CLAUDE.md, commandes Claude Code partagées).

## Setup sur une nouvelle machine

```bash
git clone <url-de-ce-repo> "Claude for GTMBuilder"
cd "Claude for GTMBuilder"

git clone https://github.com/emmaguetta/retriever-business.git
git clone https://github.com/yonsy7/retriever-apps.git
git clone https://github.com/yonsy7/retriever-marketing-website.git
git clone https://github.com/yonsy7/Retriever-WEB-EXTRACTION.git
```

Voir `CLAUDE.md` pour les règles de navigation et le workflow de push.
