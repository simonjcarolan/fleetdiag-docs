# Grille de choix : instruction, CLAUDE.md, Skill, sous-agent, hook, MCP

| Mécanisme | En une phrase | Quand il est chargé / agit | Coût principal | Exemple dans `fleetdiag` |
|---|---|---|---|---|
| **Instruction** (prompt) | Le contexte d'une tâche | Une fois, dans la conversation | Se perd avec la compaction, à répéter | Le Context Brief de FD-142 |
| **CLAUDE.md** | Règles et repères **stables** du projet | À chaque session | Occupe du contexte en permanence → rester court | `CLAUDE.md` |
| **Règle ciblée** | Règles valables pour **une partie** du code | Quand un fichier correspondant (`paths:`) est lu ou modifié | Peut ne pas être chargée si aucun fichier ne correspond | `.claude/rules/*.md` |
| **Skill** | Connaissance ou **procédure réutilisable** | À la demande (`/nom`) ou quand la description correspond | Maintenance de la procédure | `/context-brief`, `/revue-diff`, `/compte-rendu` (manuelles) ; `octet-statut-uds` (automatique, avec fichier de référence et script) |
| **Sous-agent** | **Mission spécialisée dans un contexte séparé**, avec ses propres outils et permissions | Délégué par la session principale | Coordination, transmission, latence | `explorateur` (lecture seule), `relecteur` (pas d'Edit) |
| **Hook** | **Contrôle ou action déterministe** à une étape précise | Automatiquement (avant/après outil, à l'arrêt…) | Rigidité, faux positifs, à tester | `.claude/hooks/*.py` |
| **MCP** | Connexion structurée à des **ressources ou outils externes** | Outils appelés par l'agent | Surface d'attaque, droits du connecteur | Serveur `jira` en lecture seule |

## Activité : quel mécanisme pour quel besoin ?

| # | Besoin | Mécanisme choisi | Justification (contexte, coût, contrôle) |
|---|---|---|---|
| 1 | Conserver les conventions du projet | | |
| 2 | Analyser plusieurs modules sans saturer le contexte principal | | |
| 3 | Lancer automatiquement le formatter après une modification | | |
| 4 | Rechercher des informations dans Jira | | |
| 5 | Réutiliser une procédure de revue | | |
| 6 | Effectuer une analyse de sécurité en lecture seule | | |

**Et un cas où il serait inutile de créer un agent :** ______________________

## Quatre types de contrôle — à ne pas confondre

| Type | Exemple | Garanti ? |
|---|---|---|
| Instruction donnée au modèle | « Ne lis pas `secrets.env` » dans CLAUDE.md | Non : le modèle peut l'ignorer ou être manipulé |
| Limitation technique de permission | `"deny": ["Read(./config/secrets.env)"]` | Oui pour l'outil visé (pas pour `cat` via Bash) |
| Contrôle automatique | Hook `guard_bash.py`, hook `stop_gate.py` | Oui, pour les cas que le hook sait reconnaître |
| Validation humaine | Point d'arrêt avant implémentation, décision `ask` | Oui, si l'humain lit vraiment |

Pour aller plus loin (anatomie des Skills, sous-agents, hooks, harnais) : support 08.
