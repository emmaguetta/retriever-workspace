---
description: Show which workspace repo the current working directory belongs to
---

Depuis la CWD actuelle, déterminer dans quel repo on est :

1. Lire le remote via `git remote get-url origin` (depuis la CWD).
2. Afficher :
   - Repo name (ex. `retriever-business`)
   - Remote URL
   - Branche actuelle
   - Chemin relatif par rapport à la racine du workspace (`/Claude for GTMBuilder/`)

Si la CWD est la racine du workspace, le mentionner explicitement : "Tu es à la racine du workspace (retriever-workspace), pas dans un sous-repo. Pour bosser sur du code, `cd` dans un des 4 sous-repos."
