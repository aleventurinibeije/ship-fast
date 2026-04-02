# copilot-template

Un framework portabile per GitHub Copilot Agent, pensato per progetti BE + FE. Fornisce agent, skill, istruzioni e prompt generici che si adattano a qualsiasi progetto leggendo una cartella `context/` da compilare una sola volta. Le integrazioni esterne (ticket manager, piattaforme di code review, tool di design) sono pluggabili tramite **provider config**.

---

## Indice

- [Quick Start](#quick-start)
- [Architettura: il modello a slot](#architettura-il-modello-a-slot)
- [Setup passo-passo](#setup-passo-passo)
- [Provider: esempi pratici](#provider-esempi-pratici)
- [Cosa contiene il template](#cosa-contiene-il-template)
- [Workflow disponibili](#workflow-disponibili)
- [Come creare un nuovo provider](#come-creare-un-nuovo-provider)
- [Convenzioni chiave](#convenzioni-chiave)
- [FAQ](#faq)

---

## Quick Start

```bash
# 1. Copia il template nel tuo progetto
cp -r copilot-template/.github /path/to/your-project/

# 2. Scegli le best-practice per il tuo stack
cp .github/variants/backend-java-spring.md .github/context/best-practices/backend.md
cp .github/variants/frontend-react-ts.md   .github/context/best-practices/frontend.md

# 3. Scegli i provider per le tue integrazioni
cp .github/variants/providers/ticket-jira-atlassian-mcp.md .github/context/providers/ticket-manager.md
cp .github/variants/providers/review-gitlab-mcp.md         .github/context/providers/code-review.md
cp .github/variants/providers/design-figma-mcp.md          .github/context/providers/design-tool.md

# 4. Compila i file context/ e sostituisci <<PROJECT_NAME>> in copilot-instructions.md

# 5. Fatto!
```

---

## Architettura: il modello a slot

Il template usa un modello a **3 slot** per le integrazioni esterne. Ogni slot è riempito da un singolo file di configurazione provider, copiato da un variant.

```
┌─────────────────────────────────────────────────────────────┐
│                     copilot-instructions.md                  │
│                  (hub: agent registry + regole)              │
└──────────┬──────────────────┬──────────────────┬────────────┘
           │                  │                  │
    ┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
    │ ticket-     │   │ code-       │   │ design-     │
    │ manager.md  │   │ review.md   │   │ tool.md     │
    │             │   │             │   │ (opzionale) │
    └──────┬──────┘   └──────┬──────┘   └──────┬──────┘
           │                  │                  │
   ┌───────┴───────┐  ┌──────┴───────┐  ┌──────┴──────┐
   │ Jira          │  │ GitLab MR    │  │ Figma       │
   │ Redmine       │  │ GitHub PR    │  │ Sketch      │
   │ GitHub Issues │  │ Bitbucket PR │  │ ...         │
   │ Linear        │  │ Azure DevOps │  │             │
   │ ...           │  │ ...          │  │             │
   └───────────────┘  └──────────────┘  └─────────────┘
        variants/           variants/        variants/
        providers/           providers/       providers/
```

**Regola fondamentale:** le skill NON hardcodano mai nomi di tool MCP. Leggono il provider config → caricano i tool → chiamano le operazioni.

| Slot | File config | Scopo |
|------|------------|-------|
| **Ticket manager** | `context/providers/ticket-manager.md` | Issue tracker (Jira, Redmine, GitHub Issues, Linear, YouTrack, ecc.) |
| **Code review** | `context/providers/code-review.md` | Piattaforma MR/PR (GitLab, GitHub, Bitbucket, ecc.) |
| **Design tool** | `context/providers/design-tool.md` | Tool di design (Figma, Sketch, ecc.) — **opzionale** |

---

## Setup passo-passo

### 1. Copia `.github/` nel tuo progetto

```bash
cp -r copilot-template/.github /path/to/your-project/
```

### 2. Compila i file `context/`

Ogni file ha un commento che spiega cosa inserire.

| File | Cosa compilare |
|------|----------------|
| `context/PRD.md` | Requisiti di prodotto — feature, user stories, criteri di accettazione |
| `context/architecture.md` | Tech stack, layout dei moduli, concetti chiave, confini del sistema |
| `context/integrations.md` | Comandi di test, API esterne, variabili d'ambiente |
| `context/best-practices/backend.md` | Copiato da `variants/backend-<stack>.md` |
| `context/best-practices/frontend.md` | Copiato da `variants/frontend-<stack>.md` |

### 3. Scegli i variant per le best-practice

```bash
# Esempio: Java/Spring BE + React/TS FE
cp .github/variants/backend-java-spring.md .github/context/best-practices/backend.md
cp .github/variants/frontend-react-ts.md   .github/context/best-practices/frontend.md
```

Variant disponibili:

| Variant | Stack |
|---------|-------|
| `backend-java-spring.md` | Java 17, Spring Boot, JUnit 5, Mockito |
| `backend-node.md` | Node.js, Express/Fastify, TypeScript, Jest *(DRAFT)* |
| `frontend-react-ts.md` | React + TypeScript, Jest, React Testing Library |
| `frontend-vue.md` | Vue 3 + TypeScript, Vitest *(DRAFT)* |

### 4. Configura i provider

Copia il variant che corrisponde ai tuoi strumenti e compila le costanti del progetto.

```bash
# Vedi sezione "Provider: esempi pratici" per le combinazioni complete
cp .github/variants/providers/<variant>.md .github/context/providers/<slot>.md
```

### 5. Sostituisci il marker globale

Apri `.github/copilot-instructions.md` e sostituisci:

| Marker | Sostituisci con |
|--------|----------------|
| `<<PROJECT_NAME>>` | Il nome del tuo progetto (es. `MyApp`) |

> I marker specifici dei provider (es. `<<GITLAB_PROJECT_ID>>`, `<<JIRA_PROJECT_KEY>>`) sono nei file provider copiati al punto 4.

### 6. Fatto!

Agent, skill e istruzioni leggono tutto da `context/` e `context/providers/`. Nessuna altra configurazione necessaria.

---

## Provider: esempi pratici

### Esempio 1: Jira + GitLab + Figma (stack classico enterprise)

```bash
# Ticket manager → Jira
cp .github/variants/providers/ticket-jira-atlassian-mcp.md .github/context/providers/ticket-manager.md

# Code review → GitLab MR
cp .github/variants/providers/review-gitlab-mcp.md .github/context/providers/code-review.md

# Design tool → Figma
cp .github/variants/providers/design-figma-mcp.md .github/context/providers/design-tool.md
```

Poi apri ciascun file e compila le costanti:

**`ticket-manager.md`** — sostituisci:
- `<<JIRA_PROJECT_KEY>>` → es. `MYAPP`
- Base URL → es. `https://jira.mycompany.com`

**`code-review.md`** — sostituisci:
- `<<GITLAB_PROJECT_ID>>` → es. `1234` (ID numerico, mai il path!)
- Base URL → es. `https://gitlab.mycompany.com`

**`design-tool.md`** — compila:
- Team ID e file key di Figma

---

### Esempio 2: GitHub Issues + GitHub PR (progetto open-source)

```bash
# Ticket manager → GitHub Issues
cp .github/variants/providers/ticket-github-issues.md .github/context/providers/ticket-manager.md

# Code review → GitHub PR
cp .github/variants/providers/review-github.md .github/context/providers/code-review.md

# Design tool → nessuno (lascia il placeholder)
```

Compila le costanti in entrambi i file:
- `<<GITHUB_OWNER>>` → es. `my-org`
- `<<GITHUB_REPO>>` → es. `my-app`

> Non serve toccare `design-tool.md` — il placeholder indica alle skill di saltare l'integrazione design.

---

### Esempio 3: Redmine + GitLab (PA / legacy)

```bash
# Ticket manager → Redmine
cp .github/variants/providers/ticket-redmine.md .github/context/providers/ticket-manager.md

# Code review → GitLab MR
cp .github/variants/providers/review-gitlab-mcp.md .github/context/providers/code-review.md
```

> **Nota:** il variant Redmine è in stato DRAFT — i nomi dei tool MCP sono placeholder `<<TODO>>`. Dovrai compilarli quando il tuo server MCP Redmine sarà disponibile.

---

### Esempio 4: Solo sviluppo, nessun MCP

Se non hai nessun server MCP configurato, **lascia tutti i placeholder** così come sono. Le skill rileveranno che il provider non è configurato e chiederanno input manuale (es. "incolla qui la descrizione del ticket").

Funziona tutto:
- `analyze-ticket` → ti chiede i dettagli del ticket manualmente
- `review-changeset` → legge i diff locali con `git diff`
- `write-ticket-description` → genera la descrizione da un analysis doc locale

---

## Cosa contiene il template

### Agent (6)

Gli agent sono subagent specializzati con personalità, tool e workflow dedicati.

| Agent | Quando usarlo | Trigger |
|-------|--------------|---------|
| `dev` | Implementare feature, fix bug, scrivere test | "implement", "fix", "build", "write tests" |
| `analysis` | Creare analisi tecniche per un ticket | "analyze ticket", "create analysis" |
| `design` | Convertire design Figma in specifiche componenti | "design to spec", "Figma", "component spec" |
| `review` | Revisione di MR/PR contro gli standard del progetto | "review MR", "review PR", "code review" |
| `ticket-description` | Scrivere descrizioni ticket nel formato del tracker | "write ticket description" |
| `changeset-description` | Scrivere descrizioni MR/PR | "write MR description", "write PR description" |

### Skill (8)

Le skill sono workflow riusabili caricati dagli agent.

| Skill | Cosa fa |
|-------|---------|
| `analyze-ticket` | Crea/aggiorna un documento di analisi tecnica per un ticket |
| `design-to-spec` | Converte frame Figma in specifiche componenti strutturate |
| `review-changeset` | Revisione di un changeset (MR/PR) contro gli standard |
| `write-ticket-description` | Scrive la descrizione nel formato del ticket manager |
| `write-changeset-description` | Scrive la descrizione di un changeset (MR/PR) |
| `write-tests-be` | Genera classi di test backend |
| `write-tests-fe` | Genera file di test frontend |
| `run-test-coverage` | Esegue i test e analizza la coverage |

### Prompt (9)

I prompt sono comandi slash che attivano agent + skill specifici.

| Prompt | Agent | Input |
|--------|-------|-------|
| `analyze-ticket` | `analysis` | Ticket ID |
| `implement-ticket` | `dev` | Ticket ID |
| `design-to-spec` | `design` | URL design + Ticket ID (opz.) |
| `review-changeset` | `review` | URL changeset |
| `write-ticket-description` | `ticket-description` | Ticket ID |
| `write-changeset-description` | `changeset-description` | Ticket ID + URL (opz.) |
| `write-tests-be` | `dev` | Classe/modulo target |
| `write-tests-fe` | `dev` | Componente/hook target |
| `run-test-coverage` | (standalone) | Scope (opz.) |

### Instruction (6)

Le instruction sono contesto sempre attivo, iniettato automaticamente in base al pattern `applyTo`.

| Instruction | Si attiva su | Scopo |
|-------------|-------------|-------|
| `core` | `**/*` (sempre) | Workflow SDD, letture obbligatorie, spec-before-code |
| `development` | `src/**` | Pattern di sviluppo, convenzioni, test |
| `analysis` | `docs/AI-analysis-plan-docs/**` | Formato dell'analysis doc |
| `ticket-description` | `**/TICKET-DESC-*.md` | Regole per descrizioni ticket |
| `changeset-description` | `**/CHANGESET-DESC-*.md` | Regole per descrizioni changeset |
| `review` | (caricata esplicitamente) | Checklist per la revisione del codice |

---

## Workflow disponibili

I 5 workflow fondamentali sono **entry point composabili**, non una pipeline forzata. Puoi partire da qualsiasi punto.

```
     ┌──────────────┐     ┌──────────────┐
     │  Analisi      │     │  Design       │
     │  (ticket mgr) │     │  (design tool)│
     └──────┬───────┘     └──────┬────────┘
            │   input opzionale   │
            ▼                     ▼
     ┌──────────────────────────────────────┐
     │         Sviluppo (dev agent)          │
     │  legge: analysis doc, design spec,    │
     │  architecture.md, best-practices      │
     └──────────────┬───────────────────────┘
                    │
                    ▼
     ┌──────────────────────────────────────┐
     │       Unit Test (dev agent)           │
     │  skill: write-tests-be, write-tests-fe│
     │  run-test-coverage                    │
     └──────────────┬───────────────────────┘
                    │
                    ▼
     ┌──────────────────────────────────────┐
     │     Review (review agent)             │
     │  legge: code-review provider          │
     │  skill: review-changeset              │
     └─────────────────────────────────────┘
```

**Esempi di flusso:**

1. **Flusso completo:** Analisi → Sviluppo → Test → Review → Descrizione MR
2. **Partendo dal design:** Design-to-spec → Sviluppo → Test
3. **Solo review:** Review di una MR esistente, senza analisi preventiva
4. **Solo test:** Scrivi i test per un modulo esistente
5. **Senza MCP:** Tutto funziona anche senza integrazioni esterne

---

## Come creare un nuovo provider

Se il tuo tool non ha un variant predefinito (es. Linear, Azure DevOps, Shortcut), puoi crearne uno seguendo il formato standard.

### 1. Crea il file variant

Crea un nuovo file in `.github/variants/providers/`, ad esempio `ticket-linear.md`.

### 2. Segui il formato standard

```markdown
# Ticket Manager: Linear

## MCP Loading

- **Load pattern:** `mcp_linear`
- **Tool prefix:** `mcp_linear___`

> **Deferred tools:** Call `tool_search_tool_regex` with the load pattern
> above **before** any tool call.

## Constants

| Constant | Value |
|----------|-------|
| Team ID | `<<LINEAR_TEAM_ID>>` |
| Base URL | `https://linear.app` |
| Ticket pattern | `[A-Z]+-\d+` |

## Operations

### fetch-ticket

Fetch an issue's full details.

- **Tool:** `mcp_linear___get_issue`
- **Parameters:**
  - `issue_id`: Issue identifier (e.g. `ENG-123`)
- **Response fields:**
  - `title` → issue title
  - `description` → full description (Markdown)
  - `state.name` → current status
  - `assignee.name` → assignee

### update-ticket

Update an issue's fields.

- **Tool:** `mcp_linear___update_issue`
- **Parameters:**
  - `issue_id`: Issue identifier
  - `description`: Updated description

## Output Format

Linear uses **Markdown**.

### Validation Rules

- [ ] Pure Markdown only
- [ ] Issue references use identifier format (e.g. `ENG-123`)
```

### 3. Le operazioni necessarie per ogni slot

**Ticket manager** (obbligatorie):
- `fetch-ticket` — recupera un ticket per ID
- `update-ticket` — aggiorna i campi di un ticket

**Code review** (obbligatorie):
- `fetch-changeset` — recupera metadata MR/PR
- `fetch-changeset-diff` — recupera i diff dei file
- `fetch-changeset-discussions` — recupera discussioni/commenti esistenti
- `post-changeset-comment` — pubblica un commento
- `update-changeset` — aggiorna campi (es. descrizione)

**Design tool** (obbligatorie):
- `fetch-design-file` — recupera struttura del file
- `fetch-frame` — recupera dettagli di un frame/componente
- `fetch-component-styles` — recupera stili condivisi

### 4. Copia e attiva

```bash
cp .github/variants/providers/ticket-linear.md .github/context/providers/ticket-manager.md
# Compila <<LINEAR_TEAM_ID>> e gli altri placeholder
```

---

## Struttura completa del template

```
.github/
├── copilot-instructions.md              ← Hub principale: agent registry + regole MCP
├── agents/
│   ├── dev.agent.md                     ← Sviluppo, fix, test
│   ├── analysis.agent.md               ← Analisi tecnica ticket
│   ├── design.agent.md                 ← Design → component spec
│   ├── review.agent.md                 ← Review changeset (MR/PR)
│   ├── ticket-description.agent.md     ← Descrizione ticket
│   └── changeset-description.agent.md  ← Descrizione changeset (MR/PR)
├── skills/
│   ├── analyze-ticket/SKILL.md
│   ├── design-to-spec/SKILL.md
│   ├── review-changeset/SKILL.md
│   ├── write-ticket-description/SKILL.md
│   ├── write-changeset-description/SKILL.md
│   ├── write-tests-be/SKILL.md
│   ├── write-tests-fe/SKILL.md
│   └── run-test-coverage/SKILL.md
├── instructions/
│   ├── core.instructions.md             ← Sempre attivo su tutti i file
│   ├── development.instructions.md      ← Attivo su src/**
│   ├── analysis.instructions.md         ← Attivo su docs/AI-analysis-plan-docs/**
│   ├── ticket-description.instructions.md
│   ├── changeset-description.instructions.md
│   └── review.instructions.md
├── prompts/
│   ├── analyze-ticket.prompt.md
│   ├── implement-ticket.prompt.md
│   ├── design-to-spec.prompt.md
│   ├── review-changeset.prompt.md
│   ├── write-ticket-description.prompt.md
│   ├── write-changeset-description.prompt.md
│   ├── write-tests-be.prompt.md
│   ├── write-tests-fe.prompt.md
│   └── run-test-coverage.prompt.md
├── context/
│   ├── PRD.md                           ← Requisiti di prodotto (tu lo compili)
│   ├── architecture.md                  ← Architettura (tu lo compili)
│   ├── integrations.md                  ← Comandi test, API, env vars (tu lo compili)
│   ├── best-practices/
│   │   ├── README.md
│   │   ├── backend.md                   ← Copiato da variants/
│   │   └── frontend.md                  ← Copiato da variants/
│   └── providers/
│       ├── README.md                    ← Documentazione formato provider
│       ├── ticket-manager.md            ← Slot: tracker (copiato da variants/)
│       ├── code-review.md              ← Slot: code review (copiato da variants/)
│       └── design-tool.md             ← Slot: design (copiato da variants/, opzionale)
└── variants/
    ├── backend-java-spring.md
    ├── backend-node.md
    ├── frontend-react-ts.md
    ├── frontend-vue.md
    └── providers/
        ├── ticket-jira-atlassian-mcp.md  ← Jira via Atlassian MCP
        ├── ticket-redmine.md             ← Redmine (DRAFT)
        ├── ticket-github-issues.md       ← GitHub Issues (DRAFT)
        ├── review-gitlab-mcp.md          ← GitLab MR via GitLab MCP
        ├── review-github.md              ← GitHub PR (DRAFT)
        └── design-figma-mcp.md           ← Figma via Figma MCP
```

---

## Convenzioni chiave

- **Nomi file fissi** — Le skill leggono sempre `context/best-practices/backend.md` e `frontend.md`. Seleziona un variant copiandolo con quei nomi.
- **Tool MCP deferred** — I tool MCP non sono disponibili finché non vengono caricati. Le skill leggono il provider config per scoprire il pattern di caricamento e i nomi dei tool.
- **Formato provider standard** — Ogni provider config segue lo stesso formato: MCP Loading, Constants, Operations, Output Format. Vedi `context/providers/README.md`.
- **Ticket ID dal provider** — Il pattern per riconoscere i ticket ID viene letto dal provider `ticket-manager.md` → Constants → Ticket pattern.
- **Spec-before-code (SDD)** — La skill `analyze-ticket` obbliga a leggere `context/PRD.md` e `context/architecture.md` prima di qualsiasi analisi del codice.
- **Workflow composabili** — Analisi, Design, Sviluppo, Test e Review sono entry point indipendenti. Non c'è una pipeline obbligatoria.

---

## FAQ

### Posso usare il template senza nessun server MCP?

**Sì.** Lascia i file placeholder in `context/providers/`. Le skill rileveranno che il provider non è configurato e chiederanno i dati manualmente (es. "descrivi il ticket", "incolla il diff").

### Cosa succede se il mio strumento non ha un variant?

Crea il tuo variant seguendo il [formato standard](#come-creare-un-nuovo-provider). Le operazioni necessarie per ogni slot sono documentate sopra. Puoi anche partire da un variant DRAFT (es. `ticket-redmine.md`) come modello.

### Posso usare GitHub per i PR e Jira per i ticket?

**Sì.** Ogni slot è indipendente. Puoi combinare liberamente:
- `ticket-manager.md` ← Jira
- `code-review.md` ← GitHub PR
- `design-tool.md` ← Figma

### Il marker `<<PROJECT_NAME>>` va sostituito ovunque?

No, **solo** in `copilot-instructions.md`. I marker specifici dei provider (es. `<<GITLAB_PROJECT_ID>>`) vanno sostituiti nei rispettivi file provider in `context/providers/`.

### Dove finiscono i documenti generati?

Tutti sotto `docs/AI-analysis-plan-docs/{TICKET-ID}/`:
- `ANALYSIS-{TICKET-ID}.md` — analisi tecnica
- `DESIGN-SPEC-{TICKET-ID}.md` — specifica componenti da design
- `TICKET-DESC-{TICKET-ID}.md` — descrizione per il ticket manager
- `CHANGESET-DESC-{TICKET-ID}.md` — descrizione per MR/PR
- `REVIEW-{CHANGESET-ID}.md` — risultato della code review
