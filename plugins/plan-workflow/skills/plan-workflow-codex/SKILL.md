---
name: plan-workflow-codex
description: Créer un plan d'exécution actionnable dans un fichier Markdown et le challenger avec l'intégration openai/codex-plugin-cc dans Claude Code. Utiliser quand l'utilisateur demande un plan, une feuille de route, des phases ou une stratégie d'implémentation qui bénéficie d'une analyse ou d'une revue Codex.
---

# Plan d'exécution avec Codex

Produis un plan concret, fondé sur le dépôt, puis enregistre-le dans un fichier Markdown. Dans Claude Code, utilise `openai/codex-plugin-cc` lorsque sa contribution améliore réellement l'analyse ou le contrôle du plan. Ne livre jamais uniquement le plan dans la conversation.

## Phase 0 — Vérifier l'intégration Codex

1. Si la session s'exécute dans Claude Code, vérifie `claude --version` et une version Node.js 18.18 ou supérieure avant de dépendre du plugin.
2. Vérifie si `openai/codex-plugin-cc` est déjà disponible. Sinon, propose ou exécute, selon les autorisations, les commandes suivantes dans cet ordre :

   ```text
   /plugin marketplace add openai/codex-plugin-cc
   /plugin install codex@openai-codex
   /reload-plugins
   /codex:setup
   ```

3. Ne duplique jamais la marketplace ou le plugin. Si `/codex:setup` échoue, explique l'action manuelle requise et continue le plan sans présenter Codex comme opérationnel.
4. Dans Codex hors de Claude Code, n'essaie pas d'invoquer les commandes `/codex:*` : applique le même workflow de planification avec les outils disponibles localement.

## Phase 1 — Cadrer et explorer

1. Identifie l'objectif, le périmètre, les contraintes, les livrables et les critères de réussite.
2. Explore les fichiers, la documentation et la configuration pertinents. Cite les chemins précis ; ne déduis pas une commande ou une architecture sans preuve dans le dépôt.
3. Distingue les faits vérifiés, les hypothèses et les questions bloquantes. Si une réponse change sensiblement le périmètre, demande-la avant d'écrire le plan.

## Phase 2 — Solliciter Codex si utile

Dans Claude Code avec le plugin opérationnel, utilise au plus une délégation ciblée par défaut :

- `/codex:rescue --background <périmètre>` pour investiguer une zone complexe ou un bug ; demande une synthèse, des chemins précis et les risques, sans modifier de fichier.
- `/codex:adversarial-review --background <angle de risque>` pour challenger une décision de plan (sécurité, migration de données, concurrence, rollback ou compatibilité).
- `/codex:review --background` uniquement lorsqu'un diff ou une implémentation existe déjà à examiner.

Formule chaque délégation avec le symptôme ou l'objectif, les fichiers concernés, les contraintes et le résultat attendu. Ne délègue ni secrets ni contenu de fichiers `.env`. Récupère le résultat avec `/codex:result` avant de finaliser un plan qui en dépend ; sinon, indique clairement que le plan reste à challenger. Utilise `/codex:status` ou `/codex:cancel` pour suivre ou arrêter une tâche.

## Phase 3 — Écrire le fichier de plan

1. Utilise le chemin fourni par l'utilisateur. À défaut, crée `docs/plans/YYYY-MM-DD-<sujet>-plan.md`, avec un sujet court en kebab-case, et crée le dossier nécessaire.
2. N'écrase jamais un fichier existant sans demande explicite. En cas de collision, choisis un suffixe descriptif ou demande confirmation.
3. Écris le contenu complet avant de répondre, avec cette structure :

```markdown
# Plan — <objectif>

## Objectif
<résultat attendu et valeur livrée>

## Périmètre
- Inclus : ...
- Exclu : ...

## Contexte vérifié
- `<chemin>` — <constat utile>

## Analyse Codex
- <résultat exploité, ou « Non utilisée / non disponible »>

## Hypothèses et décisions ouvertes
- <hypothèse ou décision, impact>

## Phase 1 — <résultat>
1. <action atomique, emplacement et résultat attendu>

**Validation :** <commande vérifiée ou contrôle observable>

## Phase 2 — <résultat>
1. ...

**Validation :** ...

## Risques et atténuations
- <risque> — <prévention, rollback ou décision>

## Critères d'acceptation
- [ ] <critère testable>

## Ordre d'exécution et dépendances
1. <dépendance ou jalon>
```

## Phase 4 — Contrôler la qualité

1. Numérote toutes les actions exécutables et commence une phase à chaque jalon, changement de responsabilité ou de type de travail.
2. Décris pour chaque étape le quoi, le où et le pourquoi ; évite les formulations vagues.
3. Ordonne les décisions, contrats et migrations avant l'implémentation ; termine par tests, documentation et déploiement quand ils sont pertinents.
4. Ajoute une validation par phase, uniquement avec des commandes vérifiées dans le dépôt.
5. Couvre, selon le contexte, compatibilité, données, sécurité, performances, observabilité et rollback.
6. Vérifie que le fichier contient l'objectif, le périmètre, les phases numérotées, les validations, les risques, les dépendances et des critères d'acceptation testables.

Ne réalise pas l'implémentation pendant cette tâche, sauf demande explicite. Réponds de façon concise avec le chemin du fichier créé, l'objectif couvert, l'usage de Codex et les éventuelles décisions à confirmer.
