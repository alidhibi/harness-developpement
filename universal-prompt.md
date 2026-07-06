# Prompt universel — Générer le harness `.claude/` d'un projet

> Copiez-collez ce prompt dans Claude Code (extension VS Code ou CLI), à la
> racine de **n'importe quel** projet, dans **n'importe quelle** techno, de
> **n'importe quelle** ampleur. Claude détecte votre stack, propose un plan
> (que vous validez), puis génère le harness complet.

---

```markdown
# MISSION : Générer le harness `.claude/` de ce projet

Tu es un architecte spécialisé dans l'ingénierie de contexte pour agents de code.
Ta mission : créer une arborescence `.claude/` complète, adaptée à CE projet,
en appliquant strictement les bonnes pratiques de harness de développement
guidé par l'IA (maîtrise des coûts + fiabilité).

Travaille en 3 PHASES séquentielles. NE PASSE PAS à la phase suivante sans mon
accord explicite.

═══════════════════════════════════════════════════════════════════
PHASE 1 — ANALYSE (lecture seule, sois économe)
═══════════════════════════════════════════════════════════════════
Explore le projet SANS rien écrire. Utilise Glob/Grep et des lectures ciblées
(pas de fichiers entiers inutiles, jamais les lockfiles/dist/node_modules).
Détecte et rapporte de façon CONCISE :
- Langage(s) et framework(s) principaux
- Gestionnaire de paquets + commandes RÉELLES (install, dev, build, test, lint, typecheck)
- Runner de tests et convention de nommage des tests
- Structure des dossiers (où vit le code source, les tests, la config)
- Conventions visibles (lint/format, style de commit, patterns d'archi)
- Zones sensibles (auth, secrets/.env, migrations, infra/IaC, paiements…)
- Ampleur du projet (mono-fichier, petit, moyen, large, monorepo)
- Présence éventuelle d'un `.claude/` ou d'un `CLAUDE.md` existants

Termine la phase par : "ANALYSE TERMINÉE — voici le plan proposé" suivi du plan.

═══════════════════════════════════════════════════════════════════
PHASE 2 — PLAN (validation obligatoire)
═══════════════════════════════════════════════════════════════════
Propose l'arborescence `.claude/` que tu vas créer, ADAPTÉE à l'ampleur détectée :
- Petit projet → harness minimal (CLAUDE.md + settings + 1-2 agents + session_state).
- Projet moyen/large → harness complet (agents, commands, skills, MCP).
Liste précisément chaque fichier + son rôle en une ligne.
Indique quels agents/skills sont pertinents POUR CE PROJET (ne crée pas de skill
inutile). Termine par : "PLAN PRÊT — valider avant génération."
STOP et attends ma validation.

═══════════════════════════════════════════════════════════════════
PHASE 3 — GÉNÉRATION (après validation uniquement)
═══════════════════════════════════════════════════════════════════
Crée les fichiers en respectant IMPÉRATIVEMENT ces standards :

STRUCTURE CIBLE (adapter selon le plan validé) :
.
├── CLAUDE.md
├── session_state.md
├── .mcp.json                      (si pertinent)
└── .claude/
    ├── settings.json
    ├── agents/
    │   ├── architect.md           (Plan : modèle le + capable, read-only)
    │   ├── codebase-explorer.md   (Explore : modèle rapide/économe, read-only)
    │   ├── code-reviewer.md        (Review : modèle rapide, read-only)
    │   └── test-runner.md          (Verify : modèle rapide, maxTurns borné)
    ├── commands/
    │   └── review-pr.md
    └── skills/                     (uniquement si un besoin réel existe)
        └── <skill-adapté-au-projet>/
            ├── SKILL.md
            ├── scripts/            (logique déterministe = zéro token)
            └── references/         (templates chargés à la demande)

RÈGLES PAR ARTEFACT :

1) CLAUDE.md — manuel opératoire CONCIS (chaque ligne est rechargée à chaque tour) :
   - Stack + commandes RÉELLES détectées (pas d'inventé).
   - Conventions de code + style de commit.
   - Zones sensibles "NE PAS MODIFIER sans validation humaine".
   - Section "Mémoire & discipline" : lire CLAUDE.md + session_state.md en début
     de phase, y écrire les décisions en fin, faire /clear entre les phases.
   - Règles de travail : PLAN avant implémentation, périmètre strict, STOP +
     "BLOCKED: [raison]" après 2 échecs, choix de modèle selon la tâche.
   - Interdiction d'inventer des commandes ou chemins non vérifiés.

2) Sous-agents (.claude/agents/*.md) — frontmatter YAML obligatoire :
   - `name`, `description` (dit QUAND l'utiliser), `model`, `maxTurns`, `tools`.
   - Choix du modèle : le plus capable pour l'architecture (Plan), un modèle
     rapide/économe pour explorer/reviewer/tester.
   - `tools` RESTREINTS au strict nécessaire (les agents de lecture = read-only).
   - Corps du prompt : 100–300 mots max, avec une CONDITION D'ARRÊT explicite.
   - Utilise les noms de modèles réels disponibles dans cet environnement ; si tu
     n'es pas certain, utilise des alias génériques (opus/sonnet/haiku) et
     documente-le.

3) settings.json — `env` + `permissions` :
   - `env.DISABLE_NON_ESSENTIAL_MODEL_CALLS = "1"`.
   - `permissions.allow` : commandes de test/lint/typecheck RÉELLES + git status/diff
     + exécution des scripts de skills.
   - `permissions.deny` : lecture de .env/secrets, commandes destructrices
     (rm -rf), git push.

4) commands (.claude/commands/*.md) — slash commands réutilisables, courtes,
   lecture seule quand c'est de l'analyse (ex. review-pr délègue à code-reviewer).

5) skills (.claude/skills/*/SKILL.md) — SEULEMENT si un besoin répétitif réel
   existe dans CE projet (ex. scaffolding d'endpoint/composant/module) :
   - `description` claire sur le déclenchement (progressive disclosure).
   - Logique répétitive dans `scripts/` (déterministe, zéro token).
   - Détails volumineux dans `references/` (chargés à la demande).

6) session_state.md — mémoire externalisée : objectif, avancement (checklist),
   tableau des fichiers concernés, hors-périmètre, décisions prises, bloquants,
   prochaine action prête à coller.

7) .mcp.json — uniquement si pertinent :
   - Serveurs MCP SCOPÉS (jamais la racine du repo).
   - Pour la mémoire de codebase et le pré-filtrage sémantique, PROPOSE
     (sans l'imposer) un serveur MCP dédié, par exemple :
     codebase-memory-mcp — https://github.com/DeusData/codebase-memory-mcp
   - Ajoute-le en placeholder COMMENTÉ. Indique que l'équipe doit vérifier la
     commande d'installation et les variables d'environnement RÉELLES dans le
     README du serveur choisi avant de committer.
   - N'invente JAMAIS la config d'un serveur MCP tiers : laisse des placeholders
     explicites si tu n'as pas la doc réelle.

PRINCIPES TRANSVERSAUX À RESPECTER PARTOUT :
- Mémoire externalisée (fichiers) plutôt que contexte conversationnel.
- Pré-filtrage sémantique / mémoire de codebase : privilégier un serveur MCP
  dédié (ex. codebase-memory-mcp) plutôt que des lectures de fichiers entiers.
- Discipline de session : phases + reset de contexte, état écrit noir sur blanc.
- Économie de tokens : outils restreints, modèles adaptés, maxTurns bornés,
  conditions d'arrêt explicites.
- Ne crée AUCUN fichier superflu : un harness minimal et juste vaut mieux qu'un
  harness surchargé.

LIVRABLE FINAL :
- Crée réellement les fichiers.
- Termine par un récapitulatif : arborescence générée + 1 ligne par fichier +
  les points à personnaliser par l'équipe (modèles, commandes, skills métier,
  config réelle des serveurs MCP).

Commence maintenant par la PHASE 1.
```

---

## Comment l'utiliser

- Lance-le depuis la **racine du projet**. Privilégie un modèle capable pour les
  phases Analyse/Plan (`/model opus` ou `/model sonnet`), puis redescends pour la
  génération.
- Le prompt **impose une validation** entre le plan et la génération — tu gardes
  le contrôle et tu évites les skills inutiles.
- Il **s'auto-adapte** : harness minimal pour un petit script, harness complet
  pour un monorepo.
- **Réutilisable partout** : aucune techno n'est codée en dur, tout est détecté.
- Pour la mémoire de codebase / le pré-filtrage sémantique, vois les
  « Serveurs MCP recommandés » dans le guide.
