# Mémo : Skills, sous-agents, hooks et harnais

Aide-mémoire à garder près du clavier. Chaque point renvoie à un fichier du
dépôt `fleetdiag` que vous pouvez ouvrir. Carte complète du harnais :
`.claude/README.md` ; détail des hooks : `.claude/hooks/README.md`.

## 1. Le harnais en une phrase

Le **harnais** est la couche qui entoure le modèle : il lui fournit ses outils
et décide de ce qu'il voit. Dans un dépôt, on le configure avec `CLAUDE.md`,
`.claude/` (règles, Skills, sous-agents, hooks, permissions) et `.mcp.json`.

| Probabiliste : le modèle **lit** | Déterministe : le harnais **applique** |
|---|---|
| `CLAUDE.md`, `.claude/rules/`, corps des Skills, prompt des sous-agents | Permissions allow / ask / deny, hooks, outils autorisés d'un sous-agent, outils exposés par un serveur MCP |

**Trois questions pour placer une règle**

1. Doit-elle tenir quoi qu'il arrive ? → permission ou hook, **en plus** de l'instruction.
2. Vaut-elle pour toutes les tâches ? → `CLAUDE.md`. Pour une partie du code → `.claude/rules/` avec `paths:`. Pour une procédure → Skill.
3. Pollue-t-elle le contexte principal ? → sous-agent (seul son résumé revient).

## 2. Ce qui est chargé, et quand

| Élément | Chargé |
|---|---|
| `CLAUDE.md` (racine et dossiers parents) | À chaque session, en entier ; relu après `/compact` |
| `CLAUDE.md` d'un sous-dossier | Quand Claude lit un fichier de ce sous-dossier |
| `.claude/rules/x.md` avec `paths:` | Quand un fichier correspondant est lu ou modifié |
| Description d'une Skill automatique | À chaque session |
| Skill avec `disable-model-invocation: true` | Rien tant qu'on ne tape pas `/nom` |
| Corps d'une Skill | À l'appel, puis il **reste** dans la conversation |
| Fichiers d'appui d'une Skill | Seulement si Claude les ouvre |
| Script d'une Skill | Jamais : seule sa sortie entre dans le contexte |
| Sous-agent | Dans **sa propre** fenêtre ; CLAUDE.md hérité, pas la conversation |
| Outils MCP | Noms seulement ; définitions à la demande |

Vérifier : `/context`, `/memory`, `/hooks`, `/agents`, `/mcp`, `/permissions`.

## 3. Anatomie d'une Skill

```
.claude/skills/octet-statut-uds/
├── SKILL.md          # frontmatter + consignes courtes
├── reference.md      # détail, lu à la demande
└── scripts/
    └── decode_statut.py   # calcul exact, exécuté
```

| Champ | Effet | Exemple dans `fleetdiag` |
|---|---|---|
| `description` | Seul texte vu avant de choisir la Skill : cas d'usage en premier | toutes |
| `when_to_use` | Phrases de déclenchement en plus | `octet-statut-uds` |
| `disable-model-invocation: true` | Appel manuel seulement (`/nom`) | `context-brief`, `revue-diff`, `compte-rendu` |
| `allowed-tools` | Pré-approuve des outils **pendant le tour d'appel** ; ne restreint rien d'autre | `context-brief`, `octet-statut-uds` |
| `argument-hint`, `$ARGUMENTS` | Arguments de `/nom …` | toutes |
| `` !`commande` `` ou bloc ` ```! ` | Exécuté avant que Claude lise la Skill ; la sortie remplace la ligne | `revue-diff`, `compte-rendu` |
| `context: fork` + `agent` | La Skill devient la mission d'un sous-agent | `revue-diff` → `relecteur` |
| `background: false` | Avec `fork` : attendre le résultat | `revue-diff` |
| `paths` | Déclenchement automatique seulement pour certains fichiers ; la Skill peut être **invisible** au début de la session | (non utilisé, voir l'anatomie annotée) |

## 4. Écrire une Skill : checklist et pièges

1. **Une Skill = un résultat identifiable** (un brief, une revue, un décodage). Pas « aide pour les tests ».
2. **Description précise, cas d'usage d'abord.** Trop vague → déclenchement trop fréquent ou jamais.
3. **Effet de bord** (écriture, commit, appel externe) → `disable-model-invocation: true`.
4. **`allowed-tools` étroit** : commandes exactes, jamais `Bash` entier.
5. **Corps court et durable** : il reste dans la conversation et coûte à chaque tour. Le détail va dans un fichier d'appui.
6. **Ce qui doit être exact → un script**, pas le raisonnement du modèle.
7. **Entrées en `$ARGUMENTS`, état réel par `!`** ; jamais un ticket écrit en dur.
8. **Format de sortie imposé**, avec une rubrique « non vérifié ».
9. **`context: fork` seulement avec une mission explicite** : des conseils sans tâche donnent une sortie inutile.
10. **Noms de champs exacts** : `disable_model_invocation` (souligné) est ignoré sans message. Vérifier avec `/` ou `claude --debug`.
11. **Tester** : mêmes demandes avec et sans la Skill, dans des sessions neuves. `/skill-doctor` signale les Skills peu utilisées qui coûtent du contexte.
12. **Dépôt inconnu** : relire les `!` et `allowed-tools` des Skills avant de lancer Claude.

## 5. Sous-agents

| Champ | Effet | `fleetdiag` |
|---|---|---|
| `tools` | Liste blanche d'outils | `explorateur` : lecture + Jira, **pas d'écriture possible** |
| `disallowedTools` | Liste noire | `relecteur` : pas d'Edit / Write |
| `skills` | Skills injectées en entier au démarrage | `relecteur` : `octet-statut-uds` |
| `maxTurns` | Borne le nombre de tours ; au-delà, résultat partiel | 30 / 40 |
| `model` | `inherit`, ou un modèle plus léger pour l'exploration | `inherit` |

- Contexte de départ : **neuf**. Ni la conversation, ni les fichiers déjà lus. Répéter les règles critiques dans la demande.
- Déclenchement : par la description, par « Utilise le sous-agent… », garanti avec `@agent-nom`, ou via une Skill `context: fork`.
- Pas pour un échange itératif : chaque délégation repart de zéro.

## 6. Hooks : contrat d'un hook `command`

| Code de sortie | Effet |
|---|---|
| `0` | OK ; un objet JSON sur la sortie standard est interprété |
| `2` | **Bloque** (PreToolUse : l'outil n'est pas exécuté ; Stop : Claude continue) ; la sortie d'erreur est renvoyée comme raison |
| autre | Erreur **non bloquante** : l'action continue. Un hook qui plante ne protège rien |

Validation humaine depuis un hook `PreToolUse` : code 0 et
`{"hookSpecificOutput": {"hookEventName": "PreToolUse", "permissionDecision": "ask", "permissionDecisionReason": "…"}}`.

Hook `Stop` : le champ `stop_hook_active` évite la boucle infinie ; au second
arrêt, `stop_gate.py` laisse passer → lire la réponse, qui doit dire « non terminé ».

Tester sans modèle : `python3 scripts/test_hooks.py`.

## 7. Ce que chaque contrôle garantit

| Contrôle | Garantit | Ne garantit pas |
|---|---|---|
| Instruction (`CLAUDE.md`, règle, Skill) | Que le modèle connaît la règle | Qu'il la respecte |
| `deny: Read(./config/secrets.env)` | Que l'outil Read ne lit pas le fichier | Que `cat` via Bash ne le lit pas |
| Hook à motifs | Les refus des cas reconnus | Les variantes (`cat config/secr*.env` passe) |
| Sous-agent sans Edit | Qu'il ne peut pas écrire | Que sa revue est juste |
| MCP sans outil d'écriture | Aucune écriture possible | Que le contenu lu est fiable (injection) |
| Validation humaine | Une décision par la bonne personne | Rien, si l'humain valide sans lire |
