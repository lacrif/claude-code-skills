# Claude Code Skills

Marketplace GitHub pour Claude Code, contenant le plugin `lacrif`.

## Installation

Dans Claude Code, exécute :

```text
/plugin marketplace add lacrif/claude-code-skills
/plugin install lacrif@lacrif-claude-code-skills
/reload-plugins
```

Ou, en CLI :

```bash
claude plugin marketplace add lacrif/claude-code-skills
claude plugin install lacrif@lacrif-claude-code-skills
```

## Skills fournies

- `/lacrif:init [contexte]` : initialise la configuration
  Claude Code d'un projet.
- `/lacrif:plan [objectif]` : crée un plan d'exécution par
  phases et l'enregistre dans un fichier Markdown.

## Désinstallation

Dans Claude Code, désinstalle d'abord les plugins, puis retire la marketplace :

```text
/plugin uninstall lacrif@lacrif-claude-code-skills
/plugin marketplace remove lacrif-claude-code-skills
```

Ou, en CLI :

```bash
claude plugin uninstall lacrif@lacrif-claude-code-skills
claude plugin marketplace remove lacrif-claude-code-skills
```

## Développement local

Depuis ce dépôt :

```bash
claude plugin validate .
claude plugin marketplace add .
claude plugin install lacrif@lacrif-claude-code-skills
```

Après une modification du manifeste ou de la structure du plugin, lance
`/reload-plugins` dans Claude Code.
