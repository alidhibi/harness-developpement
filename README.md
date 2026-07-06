# Claude Code Harness 🦾

> A ready-to-use **development harness** for Claude Code: cost-optimized, reliable, and adaptable to **any project, any stack, any scale**.

**🌐 Languages:** [English](#english) · [Français](#français)

---

## English

### What is this?

This repository packages battle-tested **best practices for AI-guided development** with Claude Code (CLI or VS Code extension). It helps engineering teams **control token costs** and **improve reliability** on long-running tasks — through externalized memory, semantic pre-filtering, session discipline, specialized sub-agents, Skills, and a lean `.claude/` harness.

### What's inside

| File | Purpose |
| --- | --- |
| `universal-prompt.md` | A copy-paste **meta-prompt** that makes Claude generate the right `.claude/` structure for *your* project — auto-detecting stack and scale. |
| `harness-development-guide.md` | The full **best-practices guide**: cost control, memory architecture, models, Skills, sub-agents, monitoring, and ready-to-commit template files. |
| `README.md` | This file. |

### Quick start

1. Open your project in Claude Code (VS Code extension or CLI).
2. Copy the content of **`universal-prompt.md`** into the chat.
3. Let Claude run the 3 phases: **Analyze → Plan → (you validate) → Generate**.
4. Review, adapt, and **commit the generated `.claude/` folder** so your whole team benefits.

### Core principles

- **Externalized memory** — state lives in files (`CLAUDE.md`, `session_state.md`), not in an ever-growing conversation.
- **Semantic pre-filtering** — query a vector store first; load only relevant snippets (often ~10× fewer tokens).
- **Session discipline** — sequential phases with context reset (`/clear`) between them.
- **Right model for the task** — Haiku to explore, Sonnet to build, Opus to reason.
- **Guardrails everywhere** — restricted tools, bounded `maxTurns`, explicit stop conditions.

### Recommended workflow

`Explore (Haiku) → Plan (Opus, validated) → Implement (Sonnet) → Verify (Haiku, bounded)`

### License

MIT — free to use, share, and adapt.

---

## Français

### C'est quoi ?

Ce dépôt regroupe des **bonnes pratiques éprouvées du développement guidé par l'IA** avec Claude Code (CLI ou extension VS Code). Il aide les équipes à **maîtriser les coûts en tokens** et à **gagner en fiabilité** sur les tâches longues — grâce à la mémoire externalisée, au pré-filtrage sémantique, à la discipline de session, aux sous-agents spécialisés, aux Skills et à un harness `.claude/` épuré.

### Contenu du dépôt

| Fichier | Rôle |
| --- | --- |
| `universal-prompt.md` | Un **méta-prompt** à copier-coller qui fait générer par Claude la bonne arborescence `.claude/` pour *votre* projet — détection automatique de la stack et de l'ampleur. |
| `harness-development-guide.md` | Le **guide complet des bonnes pratiques** : maîtrise des coûts, architecture mémoire, modèles, Skills, sous-agents, monitoring, et fichiers types prêts à committer. |
| `README.md` | Ce fichier. |

### Démarrage rapide

1. Ouvrez votre projet dans Claude Code (extension VS Code ou CLI).
2. Copiez le contenu de **`universal-prompt.md`** dans le chat.
3. Laissez Claude dérouler les 3 phases : **Analyse → Plan → (vous validez) → Génération**.
4. Relisez, adaptez, et **committez le dossier `.claude/` généré** pour toute l'équipe.

### Principes fondamentaux

- **Mémoire externalisée** — l'état vit dans des fichiers (`CLAUDE.md`, `session_state.md`), pas dans une conversation qui gonfle.
- **Pré-filtrage sémantique** — interroger une base vectorielle d'abord ; ne charger que les extraits utiles (souvent ~10× moins de tokens).
- **Discipline de session** — phases séquentielles avec reset de contexte (`/clear`) entre chaque.
- **Le bon modèle pour la tâche** — Haiku pour explorer, Sonnet pour coder, Opus pour raisonner.
- **Garde-fous partout** — outils restreints, `maxTurns` bornés, conditions d'arrêt explicites.

### Cycle recommandé

`Explore (Haiku) → Plan (Opus, validé) → Implement (Sonnet) → Verify (Haiku, borné)`

### Licence

MIT — libre d'utilisation, de partage et d'adaptation.

---

## Author / Auteur

**Ali DHIBI** — Fractional CTO & Strategic Advisor · AI & LLM Expert (Granite, LangGraph) · Cloud Architecture & Cybersecurity

🔗 [alidhibi.me](https://alidhibi.me)
