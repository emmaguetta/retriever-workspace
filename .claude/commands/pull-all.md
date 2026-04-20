---
description: Pull latest from remote for all 4 workspace repos (workspace included)
---

Pour chacun de ces repos, vérifier que le working tree est clean, puis faire `git pull --ff-only` :

1. `retriever-business/`
2. `retriever-apps/`
3. `retriever-marketing-website/`
4. `Retriever-WEB-EXTRACTION/`
5. `.` (workspace root — retriever-workspace)

**Règles :**
- Si un repo a des modifs non commitées → skip ce repo et le flagger dans le résumé.
- Si `git pull --ff-only` échoue (merge nécessaire) → skip et flagger, ne jamais forcer.
- À la fin, résumer : repo | branche | résultat (updated / up-to-date / skipped + raison).
