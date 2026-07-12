---
name: project-init-codex
description: Initialise la configuration Claude Code d'un nouveau projet et son intégration Codex — CLAUDE.md, rules modulaires, délégation/revue Codex, skills et settings.json
disable-model-invocation: true
argument-hint: [contexte optionnel sur le projet]
---

# Initialisation de projet Claude Code avec Codex

Tu vas initialiser la configuration Claude Code complète de ce projet.
Contexte fourni par l'utilisateur : $ARGUMENTS

Suis les phases dans l'ordre. Ne passe à la phase suivante qu'après avoir
terminé la précédente. À la fin de chaque phase, résume en 1-2 lignes ce
qui a été créé.

---

## Phase 0 — Prérequis et plugin Codex

Vérifie la version installée avec `claude --version` :

- **v2.1.33 minimum** requis pour le frontmatter `memory:` des sous-agents
- **v2.1.59 minimum** requis pour la mémoire auto (~/.claude/projects/)
Si la version est inférieure, signale-le à l'utilisateur et propose deux
options : mettre à jour Claude Code, ou continuer en générant les agents
SANS le champ `memory:` (et en omettant les mentions de mémoire dans
leurs system prompts). N'écris jamais de configuration qui référence une
fonctionnalité indisponible dans la version détectée.

Vérifie également que Node.js est en version **18.18 ou supérieure** : le
plugin Codex en dépend. Si Node.js est absent ou trop ancien, explique le
blocage et n'essaie pas d'installer le plugin.

Le projet doit utiliser le plugin officiel
[`openai/codex-plugin-cc`](https://github.com/openai/codex-plugin-cc), qui
permet de lancer Codex depuis Claude Code pour les revues et les tâches
déléguées. Vérifie s'il est déjà disponible. S'il ne l'est pas, installe-le
dans cet ordre :

```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
/reload-plugins
/codex:setup
```

N'ajoute pas deux fois la marketplace ou le plugin. `/codex:setup` est la
source de vérité : il vérifie l'installation de Codex et l'authentification
(compte ChatGPT ou clé API). Si cette étape échoue, indique l'action manuelle
nécessaire et continue l'initialisation Claude Code sans prétendre que les
flux Codex sont opérationnels.

Ne crée pas de configuration `.codex/` par défaut. Respecte la configuration
Codex utilisateur existante ; n'ajoute un réglage projet que si l'utilisateur
demande explicitement de changer le modèle ou l'effort de raisonnement.

---

## Phase 1 — Analyse du projet

Pour garder le contexte principal propre, délègue l'exploration à l'agent
natif Explore quand le projet est volumineux ; ne récupère que la synthèse.

Explore le dépôt avant d'écrire quoi que ce soit :

- Stack et langages : lis les manifestes (package.json, pyproject.toml,
  go.mod, Cargo.toml, composer.json...)
- Gestionnaire de paquets réellement utilisé (lockfile présent)
- Structure des dossiers : identifie où vivent le code source, les tests,
  la config, la doc
- Commandes : extrais build, test, lint, dev, typecheck depuis les
  scripts du manifeste, Makefile, justfile ou CI
- Framework de test et conventions de nommage des fichiers de test
- Linter / formateur configuré (eslint, prettier, ruff, gofmt...)
- CI existante (.github/workflows, .gitlab-ci.yml...)
- README et doc d'architecture existante — à référencer, jamais à dupliquer
RÈGLE ABSOLUE : ne devine aucune commande. Chaque commande écrite dans la
configuration doit avoir été vérifiée dans un fichier du projet. Si une
commande standard est absente (pas de script lint par exemple), ne
l'invente pas — signale-le à l'utilisateur.

Si le projet est vide ou quasi vide, demande à l'utilisateur la stack
prévue avant de continuer.

---

## Phase 2 — CLAUDE.md racine (< 80 lignes)

Crée `CLAUDE.md` à la racine du projet. Contenu STRICTEMENT limité à :

1. **Description** : 3-5 bullets — quoi, pour qui, stack principale
2. **Commandes** : uniquement les commandes vérifiées en Phase 1
3. **Carte du projet** : où va quoi (5-10 lignes max), pas un listing
   exhaustif de l'arborescence
4. **Règles invariantes** : conventions non négociables, y compris les
   interdits explicites (fichiers à ne jamais modifier, librairies
   bannies...)
Contraintes d'écriture :

- Formulation impérative : « Utilise X », jamais « Le projet utilise X »
- Chaque ligne doit prévenir une erreur concrète que Claude ferait sans
  elle. Test : « si je supprime cette ligne, quelle erreur spécifique se
  produit ? » Pas de réponse → supprime la ligne
- Aucune information inférable en lisant le code (pas de paraphrase du
  README, pas de description des dépendances évidentes)
- Si un README ou une doc d'architecture existe, référence le chemin au
  lieu de copier le contenu
- Maximum 80 lignes. Si tu dépasses, déplace le surplus en Phase 3
---

## Phase 3 — Rules modulaires (.claude/rules/)

Crée des rules UNIQUEMENT si des conventions substantielles sont
spécifiques à un domaine. Ne crée jamais une rule vide ou de remplissage.

Chaque rule scopée utilise le frontmatter `paths:` pour ne se charger que
lorsque Claude touche les fichiers concernés :

```markdown
---
paths:
  - "**/*.test.ts"
  - "tests/**"
---

# Conventions de test
- ...
```

Candidates typiques (à adapter, pas à créer systématiquement) :

- `testing.md` — scopée sur les fichiers de test : framework, structure
  AAA/GWT, mocking, commande pour lancer un test isolé
- `api.md` — scopée sur le backend : format d'erreurs, validation,
  conventions de routes
- `frontend.md` — scopée sur les composants : state management,
  conventions de nommage, styling
- `db.md` — scopée sur les migrations/modèles : ORM, règles de migration
Une rule SANS `paths:` se charge à chaque session comme CLAUDE.md —
réserve ce cas aux règles transverses trop longues pour le fichier racine.

---

## Phase 4 — Délégation Codex et sous-agents spécialisés (.claude/agents/)

Objectif : garder le contexte de l'agent principal propre en déléguant les
tâches isolables. Utilise d'abord les capacités fournies par Claude Code et
par le plugin Codex ; ne crée un sous-agent projet que lorsqu'il apporte une
expertise durable non couverte.

IMPORTANT — ne duplique PAS les capacités existantes : Claude Code embarque
Explore (recherche read-only), Plan et general-purpose. Le plugin ajoute
`codex:codex-rescue` pour déléguer à Codex. Ne crée donc aucun agent générique
d'exploration, de revue, de test ou de dépannage.

Quand le plugin est opérationnel, utilise les commandes suivantes :

- `/codex:review --background` après une modification significative ; ajoute
  `--base main` pour la revue d'une branche. Cette revue est read-only.
- `/codex:adversarial-review --background <angle de risque>` avant une mise
  en production ou pour challenger une décision (sécurité, concurrence,
  rollback, perte de données). Cette revue est également read-only.
- `/codex:rescue --background <tâche précise>` pour investiguer un bug,
  tenter un correctif ou reprendre une tâche Codex. Utilise `--resume` pour
  poursuivre la dernière tâche du dépôt et `--fresh` quand ce contexte n'est
  plus pertinent.
- `/codex:status`, `/codex:result` et `/codex:cancel` pour suivre, récupérer
  ou annuler les tâches en arrière-plan. Ne bloque pas le travail principal
  en attendant une tâche longue sans raison.
- `/codex:transfer` seulement si l'utilisateur veut poursuivre dans Codex le
  contexte de la session Claude Code en cours.

Formule chaque délégation avec le symptôme, le périmètre, les contraintes et
le résultat attendu. Pour une tâche qui peut modifier le code, demande un
patch minimal et les commandes de vérification réellement présentes dans le
projet. Ne délègue jamais de secrets ni le contenu de fichiers `.env`.

RÈGLES POUR TOUT AGENT AVEC `memory:` :

- Inclus TOUJOURS `Write, Edit` dans la liste `tools:`, même pour un
  agent conceptuellement read-only. La doc annonce que Read/Write/Edit
  sont auto-activés pour la gestion de la mémoire, mais en pratique une
  allowlist `tools:` explicite qui les exclut empêche l'agent d'écrire
  son MEMORY.md (bug connu). Compense dans le system prompt : l'agent
  n'écrit QUE dans sa mémoire, jamais dans le code du projet.
- Seules les ~200 premières lignes du MEMORY.md sont injectées au
  démarrage de l'agent : chaque system prompt doit exiger une mémoire
  concise, l'essentiel en tête de fichier.
Ne crée pas `code-reviewer.md` ni `test-runner.md` : ils sont remplacés par
`/codex:review` et `/codex:rescue`. Si le plugin est indisponible, signale
l'absence de cette automatisation plutôt que de recréer ces agents par défaut.

### Optionnel — codebase-analyst.md (uniquement si gros projet / monorepo)
Le Explore natif est one-shot et sans mémoire. Si et seulement si le
projet est volumineux ou complexe (monorepo, > ~50k lignes, architecture
non triviale), un analyste custom avec mémoire persistante apporte une
vraie plus-value : il capitalise la cartographie du projet entre sessions
au lieu de repartir de zéro.

```markdown
---
name: codebase-analyst
description: Analyse d'architecture APPROFONDIE avec mémoire persistante — tracer des flux transverses, cartographier les dépendances entre modules. Pour les recherches simples de fichiers ou de code, ne pas utiliser (l'agent Explore natif s'en charge).
tools: Read, Grep, Glob, Write, Edit
memory: project
---

Tu es l'analyste d'architecture de ce projet. Consulte d'abord ta
mémoire avant d'explorer. Réponds avec les chemins de fichiers exacts.
Tu ne modifies JAMAIS un fichier du projet ; Write/Edit servent
exclusivement à ta mémoire. Enrichis-la avec la cartographie durable du
projet (frontières de modules, flux principaux, points d'entrée), jamais
avec des détails volatils. Essentiel dans les 200 premières lignes.
```

Ajoute uniquement les agents spécifiques à la stack si pertinent
(ex : `migration-writer` pour un projet avec ORM, `doc-writer` si le
projet a une doc structurée). Pour chaque agent :

- `description` rédigée pour l'auto-délégation : elle doit dire QUAND
  l'utiliser, pas seulement ce qu'il fait
- `tools` restreint au strict nécessaire (read-only par défaut pour tout
  agent d'analyse, plus Write/Edit si `memory:` est utilisé — cf. règle
  ci-dessus)
- `memory: project` pour que la mémoire soit committée et partagée avec
  l'équipe (utiliser `memory: local` si elle ne doit pas l'être)
- System prompt court, spécialisé, impératif
Ne crée PAS plus de 2-3 agents à l'init. Avant de créer un agent,
vérifie qu'il n'est pas redondant avec Explore, Plan, general-purpose ou
`codex:codex-rescue`.
Les agents supplémentaires se créent quand un pattern de délégation
revient réellement.

---

## Phase 5 — Skills projet (.claude/skills/)

N'utilise JAMAIS `.claude/commands/` (mécanisme fusionné dans les skills ;
les anciens fichiers commands fonctionnent encore mais les nouveaux
workflows vont dans .claude/skills/).

Crée un skill uniquement si le projet a un workflow répétable évident
détecté en Phase 1 (script de déploiement, procédure de release,
génération de migration...). Format : un dossier par skill contenant un
`SKILL.md` avec frontmatter `description`, plus fichiers de support si
besoin.

Pour un workflow déclenché uniquement par l'humain (déploiement, release),
ajoute `disable-model-invocation: true`.

S'il n'y a pas de workflow évident, ne crée rien et dis-le.

---

## Phase 6 — Boucle d'auto-amélioration

Installe le mécanisme qui permet à la configuration d'évoluer avec le
projet au lieu de rester figée. Trois pièces :

### 1. Méta-règle de promotion (à ajouter à la fin du CLAUDE.md racine)

```markdown
## Amélioration continue
- Si l'utilisateur te corrige deux fois sur le même sujet, propose-lui
  explicitement d'ajouter une règle dans CLAUDE.md ou .claude/rules/
  (avec le texte exact de la règle). N'ajoute jamais une règle sans
  son accord.
- Après toute décision d'architecture significative (choix de librairie,
  changement de pattern, refonte de module), propose une ADR numérotée dans
  `docs/decisions/NNNN-slug.md` et lie-la à l'issue ou à la PR concernée.
- Si tu découvres qu'une règle existante est obsolète ou contredite par
  le code actuel, signale-le au lieu de l'appliquer silencieusement.
```

### 2. ADR versionnées (`docs/decisions/`)

Crée le dossier `docs/decisions/` et son `README.md` décrivant cette
convention. Ne crée pas d'ADR vide : crée `0001-slug.md` seulement si une
décision d'architecture a été validée pendant l'initialisation. Utilise ce
format pour chaque ADR :

```markdown
# ADR 0001 — Titre court

Date : AAAA-MM-JJ
Statut : Acceptée
Liens : #123, PR #456

## Contexte

## Décision

## Conséquences
```

Numérote les ADR de façon séquentielle et ne renomme jamais un fichier déjà
committé. Les ADR ne sont pas chargées à chaque session : référence seulement
`docs/decisions/` dans la carte du projet du `CLAUDE.md` et consulte l'ADR
concernée à la demande.

### 3. Skill de rétrospective (.claude/skills/retro/SKILL.md)

```markdown
---
name: retro
description: Rétrospective de session — extraire les leçons de la session en cours et proposer des mises à jour de la configuration
disable-model-invocation: true
---

Analyse la session en cours et rends un rapport en trois sections :

1. **Corrections récurrentes** : ce que l'utilisateur a dû corriger ou
   répéter. Pour chaque cas, propose la règle exacte à ajouter
   (CLAUDE.md si transverse, .claude/rules/ avec `paths:` si thématique).
2. **Décisions prises** : les choix d'architecture ou de convention
   faits pendant la session. Propose une ADR distincte par décision dans
   `docs/decisions/NNNN-slug.md`, avec les liens vers l'issue ou la PR.
3. **Nettoyage** : règles existantes devenues obsolètes ou contredites,
   à modifier ou supprimer.

Applique uniquement ce que l'utilisateur valide, point par point.
Ne modifie jamais la configuration sans validation explicite.
```

Rappels sur ce qui existe déjà nativement (ne pas le recréer) :
- La mémoire auto (~/.claude/projects/) apprend seule les patterns,
  commandes et insights — locale, non partagée
- Les mémoires d'agents (memory: project) capitalisent par domaine et
  sont committées
La boucle ci-dessus sert à PROMOUVOIR ces apprentissages implicites en
règles explicites, versionnées et partagées.
---

## Phase 7 — settings.json (.claude/settings.json)

Crée `.claude/settings.json` avec :

1. **permissions.allow** : les commandes sûres et fréquentes vérifiées en
   Phase 1 (test, lint, build, typecheck) pour éviter les confirmations
   répétées
2. **permissions.deny** : au minimum
   - `Bash(rm -rf*)`
   - `Read(./.env)`, `Read(./.env.*)` et tout fichier de credentials
     détecté dans le projet
3. **hooks** : si un formateur est configuré dans le projet, ajoute un
   hook PostToolUse qui formate les fichiers après Edit/Write avec la
   commande vérifiée du formateur
Rappel de la répartition : CLAUDE.md et rules = guidage (Claude peut
dévier) ; settings.json = garanties mécaniques (permissions, hooks).
Tout interdit critique doit être dans settings.json, pas seulement
dans CLAUDE.md.

---

## Phase 8 — Validation et livraison

1. Liste tous les fichiers créés sous forme d'arborescence
2. Vérifie que CLAUDE.md fait moins de 80 lignes (méta-règle de la
   Phase 6 incluse) — sinon, refactore vers les rules
3. Vérifie qu'aucune commande écrite n'a été inventée
4. Si des agents projet avec `memory:` ont été créés, invoque brièvement
   chacun sur un petit fichier et contrôle qu'un `MEMORY.md` apparaît dans
   son répertoire `.claude/agent-memory/`. S'il n'apparaît pas, signale-le
   à l'utilisateur (bug connu selon les versions) au lieu de laisser une
   mémoire silencieusement inerte
5. Si le plugin Codex est opérationnel, lance `/codex:review --background`
   sur les changements de configuration, puis consulte `/codex:result` avant
   la livraison. Traite les problèmes bloquants ; rapporte séparément les
   suggestions non appliquées. Si le plugin n'est pas prêt, indique que cette
   revue n'a pas été exécutée.
6. Ajoute au `.gitignore` si absents : `.claude/settings.local.json`,
   `CLAUDE.local.md`, `.claude/agent-memory-local/`
7. Propose un commit incluant : `CLAUDE.md`, `.claude/rules/`,
   `.claude/agents/`, `.claude/skills/`, `.claude/settings.json`,
   `docs/decisions/`
8. Termine par un court récapitulatif des prochaines étapes manuelles :
   - Lancer `/retro` en fin de session de travail significative pour
     faire évoluer les règles
   - Si un pattern d'orchestration multi-agents revient régulièrement,
     le figer dans un skill projet dédié (généré avec Claude plutôt
     qu'écrit à la main)
   - Préférences personnelles → `CLAUDE.local.md` ou `~/.claude/`
   - Vérifier le chargement avec `/memory` et `/context`
   - Après une modification importante, lancer `/codex:review --background`,
     puis `/codex:result`; utiliser `/codex:rescue --background` pour un
     diagnostic ou correctif isolé
