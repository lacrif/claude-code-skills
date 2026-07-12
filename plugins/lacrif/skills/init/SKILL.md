---
name: init
description: Initialise la configuration Claude Code complète d'un nouveau projet — CLAUDE.md, rules modulaires, sous-agents spécialisés avec mémoire, skills et settings.json
disable-model-invocation: true
argument-hint: [contexte optionnel sur le projet]
---
 
# Initialisation de projet Claude Code
 
Tu vas initialiser la configuration Claude Code complète de ce projet.
Contexte fourni par l'utilisateur : $ARGUMENTS
 
Suis les phases dans l'ordre. Ne passe à la phase suivante qu'après avoir
terminé la précédente. À la fin de chaque phase, résume en 1-2 lignes ce
qui a été créé.
 
---
 
## Phase 0 — Prérequis de version
 
Vérifie la version installée avec `claude --version` :
 
- **v2.1.33 minimum** requis pour le frontmatter `memory:` des sous-agents
- **v2.1.59 minimum** requis pour la mémoire auto (~/.claude/projects/)
Si la version est inférieure, signale-le à l'utilisateur et propose deux
options : mettre à jour Claude Code, ou continuer en générant les agents
SANS le champ `memory:` (et en omettant les mentions de mémoire dans
leurs system prompts). N'écris jamais de configuration qui référence une
fonctionnalité indisponible dans la version détectée.
 
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
 
## Phase 4 — Sous-agents spécialisés (.claude/agents/)
 
Objectif : garder le contexte de l'agent principal propre en déléguant
les tâches isolables à des sous-agents avec leur propre fenêtre de
contexte.
 
IMPORTANT — ne duplique PAS les agents natifs : Claude Code embarque déjà
Explore (recherche read-only dans la codebase, sur Haiku), Plan (recherche
en plan mode) et general-purpose (tâches complexes multi-étapes). Ils sont
invoqués automatiquement. Ne crée donc AUCUN agent d'exploration ou de
recherche générique — c'est déjà couvert nativement.
 
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
Crée par défaut ces deux agents, adaptés à la stack détectée :
 
### code-reviewer.md
```markdown
---
name: code-reviewer
description: Revue de code en lecture seule — correction, sécurité, performance, respect des conventions du projet. À utiliser après toute modification significative ou avant un commit.
tools: Read, Grep, Glob, Write, Edit
memory: project
---
 
Tu es un reviewer senior sur ce projet. Analyse le diff ou les fichiers
indiqués. Vérifie dans l'ordre : bugs et cas limites, failles de sécurité,
respect des conventions du projet (CLAUDE.md et rules), performance.
Rends un rapport structuré : bloquants, importants, suggestions.
Tu ne modifies JAMAIS un fichier du projet. Tes outils Write/Edit servent
EXCLUSIVEMENT à gérer ta mémoire. Enrichis-la avec les patterns
récurrents du projet que tu découvres. Garde-la concise : l'essentiel
dans les 200 premières lignes, purge ce qui devient obsolète.
```
 
### test-runner.md
```markdown
---
name: test-runner
description: Lance les tests, analyse les échecs et propose des correctifs. À utiliser après chaque modification de code ou quand des tests échouent.
tools: Read, Grep, Glob, Bash, Write, Edit
memory: project
---
 
Tu lances les tests du projet avec [COMMANDE VÉRIFIÉE EN PHASE 1].
Pour chaque échec : isole la cause racine (code vs test obsolète),
propose le correctif minimal, et distingue clairement les deux cas.
Tu ne modifies JAMAIS un fichier du projet : tu proposes, l'agent
principal applique. Tes outils Write/Edit servent EXCLUSIVEMENT à ta
mémoire. Ne suggère jamais de corriger un test pour le faire passer
artificiellement. Mémorise les commandes de test utiles (test isolé,
watch, coverage) — concis, l'essentiel dans les 200 premières lignes.
```
 
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
 
Ajoute ensuite les agents spécifiques à la stack si pertinent
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
Ne crée PAS plus de 3-4 agents à l'init. Avant de créer un agent,
vérifie qu'il n'est pas redondant avec Explore, Plan ou general-purpose.
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
4. Vérifie la mémoire des agents : invoque brièvement le code-reviewer
   sur un petit fichier et contrôle qu'un `MEMORY.md` apparaît dans
   `.claude/agent-memory/code-reviewer/`. S'il n'apparaît pas, signale-le
   à l'utilisateur (bug connu selon les versions) au lieu de laisser une
   mémoire silencieusement inerte
5. Ajoute au `.gitignore` si absents : `.claude/settings.local.json`,
   `CLAUDE.local.md`, `.claude/agent-memory-local/`
6. Propose un commit incluant : `CLAUDE.md`, `.claude/rules/`,
   `.claude/agents/`, `.claude/skills/`, `.claude/settings.json`,
   `docs/decisions/`
7. Termine par un court récapitulatif des prochaines étapes manuelles :
   - Lancer `/retro` en fin de session de travail significative pour
     faire évoluer les règles
   - Si un pattern d'orchestration multi-agents revient régulièrement,
     le figer dans un skill projet dédié (généré avec Claude plutôt
     qu'écrit à la main)
   - Préférences personnelles → `CLAUDE.local.md` ou `~/.claude/`
   - Vérifier le chargement avec `/memory` et `/context`
