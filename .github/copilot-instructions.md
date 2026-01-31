# VS Code Copilot Instructions

## Project Overview

This repository contains **application design proposals and pitch materials** - not source code. Each project idea evolves through documentation phases: concept → MVP plan → architecture → development plan.

## Repository Structure

```
project-ideas/
├── .github/
│   ├── instructions/     # Scoped instruction files (applyTo patterns)
│   └── prompts/          # Reusable prompt templates for planning workflows
├── {project-name}/       # Self-contained project idea folder
│   ├── README.md         # Problem, solution, features, audience, stack
│   ├── MVP_plan.md       # Constraints, user scenarios, service architecture
│   ├── Architecture-Document.md  # Technical deep-dive, data models, APIs
│   ├── Development-Plan.md       # Sprints, work items, testing strategy
│   ├── presentation/     # Pitch decks, slides
│   ├── diagrams/         # PlantUML, Mermaid diagrams
│   └── assets/           # Images, videos, mockups
```

**Reference example:** `resume-fine-tuner/` demonstrates the complete documentation evolution.

## Document Conventions

### README.md (Project Overview)
Must include these sections in order: Problem Statement → Proposed Solution → Key Features → Target Audience → Technology Stack → Challenges/Risks → Future Enhancements. See `.github/instructions/idea-overview.instructions.md`.

### Architecture Documents
- Use Mermaid diagrams for system context, component relationships, and data flows
- Define service interfaces with Go-style signatures (language-agnostic concepts)
- Include cost analysis tables for infrastructure decisions
- Document LLM prompt templates inline with expected JSON outputs

### Development Plans
- Use Mermaid Gantt charts for timelines
- Structure work items as: Epic → Work Item → Tasks → Acceptance Criteria
- Include sprint review demo commands (curl examples for APIs)

## Working with This Repo

### Creating New Project Ideas
1. Create folder: `{short-kebab-case-name}/`
2. Start with `README.md` using the template structure above
3. Add subfolders as content develops: `presentation/`, `diagrams/`, `assets/`

### Evolving Existing Ideas
Progress documents in order: README.md → MVP_plan.md → Architecture-Document.md → Development-Plan.md

### Using Prompt Templates
- `plan-deep.prompt.md`: For complex planning - asks clarifying questions first
- `plan-fast.prompt.md`: For quick iterations - shorter implementation plans

## Design Principles

- **MVP-first**: Validate core value quickly with minimal infrastructure (e.g., single-node VM, <$100/month)
- **Language-agnostic architecture**: Focus on service responsibilities, not specific tech stacks
- **User scenarios drive design**: Document concrete user workflows before technical solutions
- **Cost-conscious**: Include budget constraints and cost breakdowns in architecture docs

## Learnings

- Prefer Mermaid over PlantUML for diagrams (better GitHub/VS Code rendering)
- Include "why" reasoning alongside technical decisions in architecture docs