# Squad Copilot Plugins

A collection of reusable GitHub Copilot plugins and skills for engineering teams that want opinionated, repeatable review and development workflows.

This repository is designed to centralize custom Copilot capabilities that support code reviews, architecture guidance, and domain-specific quality checks.

## Overview

The project includes:

- A plugin manifest in `plugin.json`
- Reusable custom Copilot plugin sources under `plugins/`
- Specialized skills under `skills/`
- A Spring Boot code review skill for Java backend reviews

## Repository structure

```text
.
├── README.md
├── plugin.json
├── plugins/
│   └── documents/
├── skills/
│   └── springboot-code-review/
│       └── SKILL.md
└── .github/
```

## Included capability

### Spring Boot Code Review

The repository includes a Spring Boot-focused review skill located at `skills/springboot-code-review/SKILL.md`.

It helps Copilot evaluate Java Spring Boot changes for:

- correctness and API behavior
- security and authorization concerns
- transaction and persistence issues
- reliability and integration risks
- observability and operational readiness
- missing or weak test coverage

The skill gives a structured review workflow and a severity-based finding format using P0–P3 priorities.

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
3. Use the included custom skills when prompting Copilot for reviews or engineering guidance.
4. Extend the repository by adding more plugin folders or specialized skills for your team.

## Example usage

When reviewing a Spring Boot pull request, ask Copilot to use the Spring Boot review skill:

```text
Review this Spring Boot change using the springboot-code-review skill.
```

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
