# Squad Copilot Plugins

A collection of reusable GitHub Copilot plugins and skills for engineering teams that want opinionated, repeatable review and development workflows.

This repository is designed to centralize custom Copilot capabilities that support code reviews, architecture guidance, and domain-specific quality checks.

## Overview

The project includes:

- Plugin manifests under `plugins/`
- Reusable custom Copilot plugin sources under `plugins/`
- Specialized skills under `skills/`
- Code review skills for Java (Spring Boot), Node.js, and Python backend reviews

## Repository structure

```text
.
├── README.md
├── plugins/
│   ├── reviews/          (reviews-java)
│   │   └── plugin.json
│   ├── reviews-nodejs/
│   │   └── plugin.json
│   └── reviews-python/
│       └── plugin.json
├── skills/
│   ├── springboot-code-review/
│   │   └── SKILL.md
│   ├── nodejs-code-review/
│   │   └── SKILL.md
│   └── python-code-review/
│       └── SKILL.md
└── .github/
```

## Included capabilities

### Spring Boot Code Review (Java)

Located at `skills/springboot-code-review/SKILL.md`, registered by the `reviews-java` plugin.

### Node.js Code Review

Located at `skills/nodejs-code-review/SKILL.md`, registered by the `reviews-nodejs` plugin.
s

Each `plugins/<name>/plugin.json` defines that plugin's metadata and registers:

- the plugin name and version
- author metadata
- keywords
- the skill(s) included in the plugin package

These are the entry points used by Copilot to recognize each folder
- observability and operational readiness
- missing or weak test coverage

Each skill gives a structured review workflow and a severity-based finding format using P0–P3 priorities.

## Plugin manifest

The root `plugin.json` defines the repository metadata and registers:

- the project name and version
- author metadata
- keywords
- plugin entry points
- skills included in the plugin package

This is the entry point used by Copilot to recognize the repository as a plugin bundle.

## How to use this repository

1. Clone or copy this repository into your Copilot plugin workspace.
2. Keep the `plugin.json` and child directories in place.
3. Use teach `plugins/<name>/plugin.json` and its referenced skill directories in place.
3. Use the included custom skills when prompting Copilot for reviews or engineering guidance.
4. Extend the repository by adding more plugin folders or specialized skills for your team.

## Example usage

When reviewing a Spring Boot pull request:

```text
Review this Spring Boot change using the springboot-code-review skill.
```

When reviewing a Node.js pull request:

```text
Review this Node.js change using the nodejs-code-review skill.
```

When reviewing a Python pull request:

```text
Review this Python change using the python

This gives a risk-focused review that prioritizes real defects and likely production issues over cosmetic concerns.

## Customization

This repository is a starting point for creating team-specific Copilot workflows. You can add more:

- domain-specific review skills
- architecture validation prompts
- code generation patterns
- team standards and review checklists

## License

This project is licensed under the MIT license.

## Author

Fabio Almeida
