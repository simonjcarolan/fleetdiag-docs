# Gabarit d'architecture du workflow agentique

**Groupe :** __________________ **Mission :** FD-142

Il n'est **pas** obligatoire d'avoir quatre sous-agents. Une fonction peut rester
dans la conversation principale ou être portée par une Skill. Supprimez tout
rôle qui n'apporte ni isolation de contexte, ni compétence spécialisée, ni
contrôle supplémentaire.

## Schéma

```
 [Humain] ──brief──▶ [Session principale : arbitre & planifie]
                          │            ▲
              mission     │            │ synthèse (pas l'historique !)
                          ▼            │
                     [Explorateur]─────┘         ← lecture seule
                          │
                 plan validé (POINT D'ARRÊT HUMAIN)
                          ▼
                    [Implémentation] ──diff──▶ [Tests] ──résultats──▶ [Relecteur]
                                                                      │ rapport
                                                                      ▼
                                                              [Humain : accepte / reprend]
```

## Rôles

| Rôle | Mission | Informations reçues | Outils accessibles | Peut modifier des fichiers ? | Sortie attendue | Condition de fin | Destinataire |
|---|---|---|---|---|---|---|---|
| Session principale | | | | | | | |
| Explorateur | | | | Non | | | |
| Implémentation | | | | | | | |
| Tests | | | | | | | |
| Relecteur | | | | Non | | | |

## Transmissions

| De → vers | Contenu transmis | Format | Taille max | Vérifié par |
|---|---|---|---|---|
| Explorateur → principale | | | | |
| Principale → implémentation | | | | |
| Tests → relecteur | | | | |
| Relecteur → humain | | | | |

> Le développeur reçoit-il une **synthèse exploitable** ou tout l'historique
> d'exploration ?

## Trois règles imposées

1. L'explorateur travaille par défaut **en lecture seule**.
2. Le relecteur **ne modifie pas** le code qu'il évalue.
3. Toute connexion externe ou action irréversible est **explicitement identifiée** :

| Connexion / action | Mode (lecture / écriture) | Contrôle associé |
|---|---|---|
| | | |

## Points d'arrêt

| Moment | Ce que l'humain vérifie | Ce qui est présenté |
|---|---|---|
| | | |
