# tech-codelens — Backend Code Review Workflow

An agentic, multi-phase backend code review workflow that auto-detects your tech stack and drives a full review cycle: code quality, security (OWASP Top 10), performance analysis, staged fix proposals, and a senior-developer report.

**The agent never commits. All proposed changes are staged for human review.**

---

## What It Does

| Phase | What Happens |
|-------|-------------|
| 0 — Auto-Detection | Detects language, framework, DB, ORM, auth, API style |
| 1 — Discovery | Maps entry points, data flows, dependency CVEs, secrets scan |
| 2 — Code Quality | Architecture, error handling, complexity, dead code, observability |
| 3 — Security | Full OWASP Top 10 (2021) audit adapted to your stack |
| 4 — Performance | N+1 queries, caching, async issues, memory leaks, algorithm efficiency |
| 5 — Staged Fixes | Writes `.review-staging/` with diffs, patches, and review annotations |
| 6 — Report | Generates `review-report.md` — executive summary, risk matrix, action plan |
| 7 — Validation | Self-checks for completeness and consistency before handing off |

Supports: Java/Spring, Python/Django/FastAPI/Flask, .NET/ASP.NET Core, Node.js/Express/NestJS, Go/Gin, Ruby/Rails, PHP/Laravel, Rust/Axum, and more.

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/your-org/tech-codelens.git
cd tech-codelens
```

### 2. Install for your agent

Pick the section below that matches your AI coding tool.

---

## Claude Code

**File:** `tech-codelens.md` (Claude Code skill)

### Install

Copy the skill file to your Claude Code commands directory:

```bash
# macOS / Linux
cp tech-codelens.md ~/.claude/commands/tech-codelens.md

# Windows (PowerShell)
Copy-Item tech-codelens.md "$env:USERPROFILE\.claude\commands\tech-codelens.md"
```

> If the `commands/` directory does not exist, create it first:
> ```bash
> mkdir -p ~/.claude/commands          # macOS / Linux
> mkdir "$env:USERPROFILE\.claude\commands"  # Windows PowerShell
> ```

### Invoke

In any Claude Code session, run:

```
/tech-codelens <path>
```

**Examples:**
```
/tech-codelens src/main/java/
/tech-codelens app/
/tech-codelens .
```

**With flags:**
```
/tech-codelens . --security-only
/tech-codelens . --propose --severity critical
/tech-codelens . --performance-only --report
```

---

## Windsurf

**File:** `.windsurf/rules/tech-codelens.md`

### Install

Copy the rules directory into the root of your project:

```bash
# macOS / Linux
cp -r .windsurf/ /path/to/your/project/

# Windows (PowerShell)
Copy-Item -Recurse .windsurf\ "C:\path\to\your\project\"
```

Or clone this repo directly inside your project root and move the folder:

```bash
git clone https://github.com/your-org/tech-codelens.git _tcl
mv _tcl/.windsurf .windsurf
rm -rf _tcl
```

### Invoke

Open the **Cascade** chat panel in Windsurf and type:

```
Run the tech-codelens review on <path>
```

**Examples:**
```
Run the tech-codelens review on src/
Run the tech-codelens review on . with security only
Run the tech-codelens review on app/ and propose fixes
Run the tech-codelens review on . severity critical only
```

> Windsurf will automatically pick up the rule from `.windsurf/rules/tech-codelens.md` because it is marked `trigger: manual`.

---

## GitHub Copilot

**File:** `.github/prompts/tech-codelens.prompt.md`

### Requirements

- VS Code with the **GitHub Copilot Chat** extension (v1.156+)
- Copilot Chat must have **Agent mode** enabled

### Install

Copy the prompts directory into your project's `.github/` folder:

```bash
# macOS / Linux
cp -r .github/prompts/ /path/to/your/project/.github/prompts/

# Windows (PowerShell)
Copy-Item -Recurse .github\prompts\ "C:\path\to\your\project\.github\prompts\"
```

Or if `.github/` already exists in your project, just copy the prompt file:

```bash
# macOS / Linux
cp .github/prompts/tech-codelens.prompt.md /path/to/your/project/.github/prompts/

# Windows (PowerShell)
Copy-Item .github\prompts\tech-codelens.prompt.md "C:\path\to\your\project\.github\prompts\"
```

### Invoke

1. Open **Copilot Chat** in VS Code (`Ctrl+Alt+I` / `Cmd+Alt+I`)
2. Switch to **Agent mode** (dropdown at the top of the chat panel)
3. Type:

```
@workspace #file:.github/prompts/tech-codelens.prompt.md — review <path>
```

**Examples:**
```
@workspace #file:.github/prompts/tech-codelens.prompt.md — review src/main/java/
@workspace #file:.github/prompts/tech-codelens.prompt.md — review . --security-only
@workspace #file:.github/prompts/tech-codelens.prompt.md — review app/ --propose --report
```

> Copilot will load the prompt file as agent instructions and begin Phase 0 auto-detection.

---

## Cursor

**File:** `.cursor/rules/tech-codelens.mdc`

### Install

Copy the rules directory into your project root:

```bash
# macOS / Linux
cp -r .cursor/ /path/to/your/project/

# Windows (PowerShell)
Copy-Item -Recurse .cursor\ "C:\path\to\your\project\"
```

Or just the rule file if `.cursor/rules/` already exists:

```bash
# macOS / Linux
cp .cursor/rules/tech-codelens.mdc /path/to/your/project/.cursor/rules/

# Windows (PowerShell)
Copy-Item .cursor\rules\tech-codelens.mdc "C:\path\to\your\project\.cursor\rules\"
```

### Invoke

Open the **Agent** chat panel in Cursor and type:

```
@tech-codelens review <path>
```

**Examples:**
```
@tech-codelens review src/
@tech-codelens review . --security-only
@tech-codelens review app/ --propose --severity critical
@tech-codelens review . --performance-only --report
```

> Because `alwaysApply: false` is set in the rule's frontmatter, the rule only activates when you explicitly reference `@tech-codelens` in your message.

---

## Available Flags

All agents support the same set of flags. Include them in your invocation message:

| Flag | Effect |
|------|--------|
| `--security-only` | Run Phase 3 only — skip quality and performance |
| `--performance-only` | Run Phase 4 only — skip quality and security |
| `--propose` | Write proposed fix files to `.review-staging/` |
| `--severity critical` | Surface only Critical and High findings |
| `--depth deep` | Expand dependency analysis to transitive dependencies |
| `--report` | Write `review-report.md` to the project root |

> Without `--propose`, the agent performs analysis and outputs findings to the chat only — no files are written.

---

## Output

### Review Report (`review-report.md`)

Generated when `--report` is passed (or always in full runs). Contains:

- Executive summary and ship-readiness assessment
- Full tech profile
- Risk matrix (CRITICAL / HIGH / MEDIUM / LOW / INFO)
- Complete finding registry with file:line references
- OWASP Top 10 coverage table
- Performance hotspot ranking
- Code quality scorecard
- Recommended action plan (immediate / short-term / long-term)

### Staging Directory (`.review-staging/`)

Generated when `--propose` is passed. Contains:

```
.review-staging/
├── README.md           — How to apply changes
├── CHANGES.md          — Summary of every proposed change
├── patches/
│   ├── 01-quality-fixes.patch
│   ├── 02-security-fixes.patch
│   └── 03-performance-fixes.patch
├── proposed/           — Full modified files mirroring source tree
└── annotations/        — Per-file review notes explaining each change
```

**To apply a patch:**
```bash
git apply .review-staging/patches/02-security-fixes.patch
```

**To apply a single file:**
```bash
cp .review-staging/proposed/src/auth.py src/auth.py
```

---

## Supported Tech Stacks

| Language | Frameworks | ORMs / DBs |
|----------|-----------|------------|
| Java | Spring Boot, Micronaut, Quarkus | Hibernate, JPA, PostgreSQL, MySQL, MongoDB |
| Python | Django, FastAPI, Flask, Starlette | SQLAlchemy, Django ORM, Redis, PostgreSQL |
| .NET | ASP.NET Core, Minimal APIs | Entity Framework Core, Dapper, MSSQL, PostgreSQL |
| Node.js | Express, NestJS, Fastify, Koa | Sequelize, Prisma, Mongoose, TypeORM |
| Go | Gin, Echo, Chi, Fiber | GORM, sqlx, pgx, MongoDB |
| Ruby | Rails, Sinatra | ActiveRecord, PostgreSQL, MySQL |
| PHP | Laravel, Symfony | Eloquent, Doctrine, MySQL, PostgreSQL |
| Rust | Axum, Actix-web, Rocket | Diesel, SQLx, SeaORM |

---

## Repository Structure

```
tech-codelens/
├── README.md                              — This file
├── tech-codelens.md                       — Claude Code skill (install to ~/.claude/commands/)
├── .windsurf/
│   └── rules/
│       └── tech-codelens.md               — Windsurf Cascade rule
├── .github/
│   └── prompts/
│       └── tech-codelens.prompt.md        — GitHub Copilot agent prompt
└── .cursor/
    └── rules/
        └── tech-codelens.mdc              — Cursor agent rule
```

---

## Important Notes

- **The agent never modifies source files directly.** All changes go to `.review-staging/` for human review.
- **The agent never commits, pushes, or merges** any code.
- **`.review-staging/` should be added to `.gitignore`** — it is ephemeral output for a single review session.
- After applying and testing approved patches, delete `.review-staging/` before committing.

Add to your `.gitignore`:
```
.review-staging/
review-report.md
```

---

## License

MIT — use freely in personal or commercial projects.
