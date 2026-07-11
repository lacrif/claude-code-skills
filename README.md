# Claude Code Skills

Marketplace GitHub pour Claude Code, contenant le plugin `project-init`.

## Installation

Dans Claude Code, exécute :

```text
/plugin marketplace add lacrif/claude-code-skills
/plugin install project-init@lacrif-claude-code-skills
/reload-plugins
```

Ou, en CLI :

```bash
claude plugin marketplace add lacrif/claude-code-skills
claude plugin install project-init@lacrif-claude-code-skills
```

## Skills fournies

- `/project-init:project-init [contexte]` : initialise la configuration
  Claude Code d'un projet.
- `/project-init:project-init-codex [contexte]` : même initialisation, avec
  installation et utilisation du plugin officiel `openai/codex-plugin-cc`.

## Développement local

Depuis ce dépôt :

```bash
claude plugin validate .
claude plugin marketplace add .
claude plugin install project-init@lacrif-claude-code-skills
```

Après une modification du manifeste ou de la structure du plugin, lance
`/reload-plugins` dans Claude Code.
