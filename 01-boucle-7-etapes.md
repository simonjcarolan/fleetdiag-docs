# La boucle de travail en 7 étapes

```
 Understand ─▶ Explore ─▶ Plan ─▶ Implement ─▶ Test ─▶ Review ─▶ Accept / Rework
     ▲                       │                                         │
     └──────── nouvelle information, contrainte, échec ◀───────────────┘
```

| Étape | Question à se poser | Mode Claude Code typique | Sortie observable | Qui décide du passage ? |
|---|---|---|---|---|
| **1. Understand** | Qu'est-ce que Claude devra deviner ? | Conversation, `/context-brief` | Context Brief, questions au demandeur | Humain |
| **2. Explore** | Qu'existe-t-il déjà ? Où sont les contradictions ? | Mode plan, sous-agent `explorateur` (lecture seule) | Synthèse ciblée avec sources `fichier:ligne` | Humain |
| **3. Plan** | Dans quel ordre, avec quels contrôles ? | Mode plan | Plan vérifiable (résultat, dépendances, tests, critères) | **Humain — point d'arrêt** |
| **4. Implement** | Le plus petit incrément utile ? | `acceptEdits` ou validation à chaque édition | Diff limité, justifié par un critère | Agent, sous contrôle des hooks |
| **5. Test** | Comment sait-on que c'est réussi ? | Commandes de test, hook Stop | Sorties de tests, échecs qualifiés | Automatique + humain |
| **6. Review** | Qu'est-ce qui pourrait être faux ? | Sous-agent `relecteur`, `/revue-diff` | Rapport de revue, critères démontrés / non démontrés | Rôle distinct |
| **7. Accept / Rework** | Les critères sont-ils démontrés ? Que reste-t-il d'incertain ? | `/compte-rendu` | Synthèse exploitable par un développeur | **Humain** |

**Règle d'or** — Une tâche n'est pas terminée parce que l'agent a produit du
code. Elle est terminée lorsque le résultat respecte des critères observables,
que les contrôles ont été exécutés et que les incertitudes restantes sont
visibles.

## Repères Claude Code

- `Shift+Tab` : basculer entre les modes de permission (dont le **mode plan**).
- `claude --permission-mode plan` : démarrer en lecture seule.
- `/agents`, `/hooks`, `/mcp`, `/permissions` : voir ce qui est actif.
- `/compact` et `/clear` : gérer la fenêtre de contexte entre deux étapes.
- `Esc` : interrompre ; `Esc Esc` : revenir à un message précédent.
