# AGENTS.md

This is the canonical set of instructions for AI coding agents (GitHub Copilot, Codex, Claude, or any other agent) working in this repository. It follows the [agents.md](https://agents.md/) convention. Tool-specific configs (e.g. `.github/copilot-instructions.md`) simply point back here — keep this file up to date and they'll stay accurate automatically.

## Project overview

Tailspin Toys is a crowdfunding platform for games with a developer theme. The application is a single **Astro 7** site (fully prerendered/static output) styled with **Tailwind CSS v4**. Data is stored in a local SQLite database accessed at build time through **Drizzle ORM + Node.js's built-in SQLite driver**; pages query the database directly in frontmatter — there is no separate backend API or client-side UI framework.

## Agent notes

- Explore the project before beginning code generation
- Create todo lists for long operations
  - Before each step in a todo list, reread the instructions to ensure you always have the right directions
- Always use instructions files when available, reviewing before generating code
- Do not generate summary markdown files upon completion of a task
- Always use absolute paths when running scripts and BASH commands
- **NEVER commit or push to main automatically unless explicitly instructed to do so**

## Code standards

### Required Before Each Commit

#### Testing guidelines

- **Always run tests and lint through the `quality-checks` skill — never invoke `npm run test:unit`, `npm run test:e2e`, or `npm run lint` directly.** The skill wraps environment setup, ordering, and troubleshooting. (Starting the app for manual validation is not a quality check — run `npm run dev` directly for that.) If your agent/tool doesn't support skills, run the npm scripts directly instead.
- Run Vitest unit tests to verify the data layer and transforms, and Playwright tests to verify e2e and frontend functionality
- Run ESLint to check frontend code quality before committing
- Review the existing tests to ensure we're not duplicating efforts
- Test code should be of the same quality as the rest of the project, and follow DRY principles
- For frontend changes, verify the build (`npm run build`) directly, and run the end-to-end tests through the `quality-checks` skill, to ensure everything works correctly
- When changing the data layer (schema, helpers, transforms), update and run the corresponding unit tests

#### Project guidelines

- When updating the database schema, generate and commit the drizzle-kit migration (`npm run db:generate`)
- When adding new functionality, make sure you update the README
- Make sure all guidance in this file is updated with any relevant changes, including to project structure and scripts, and programming guidance

### Code formatting requirements

- Use TypeScript with explicit types for function parameters and return values, especially in the data layer (`db/`, `src/lib/`)
- Frontend code (TypeScript, Astro) must pass ESLint checks (`npm run lint`)

### Data Layer Patterns (Drizzle + Node SQLite)

- Define tables in `db/schema.ts`; manage schema changes with drizzle-kit migrations - see `.github/instructions/drizzle.instructions.md`
- Keep data-access helpers in `src/lib/` with an **injectable `db`** argument so they're testable
- Keep CSV/seed logic as pure functions in `db/transforms.ts`
- Seed-derived values must be deterministic (no `Math.random`) so static builds are reproducible

### Astro Patterns

- **Astro Pages/Components**: routing, layouts, content, and components are all `.astro` - see `.github/instructions/astro.instructions.md`
- Query data directly in page frontmatter via the `src/lib/` helpers (build-time, static output)
- Dynamic routes use `getStaticPaths()` + `export const prerender = true`
- Provide a branded `404.astro` (unknown routes are real 404s under static output)
- Only add a scoped Astro `<script>` when genuine client interactivity is required

### Styling

- Use Tailwind CSS utility classes exclusively - see `.github/instructions/style.instructions.md`
- Dark theme colors: slate palette (`bg-slate-800`, `text-slate-100`, etc.)
- Rounded corners and modern UI patterns
- Follow modern UI/UX principles with clean, accessible interfaces

### Testing

- See `.github/instructions/unit-tests.instructions.md` for Vitest guidelines and `.github/instructions/playwright.instructions.md` for Playwright guidelines
- See `.github/instructions/ui.instructions.md` for the broader component/testability/accessibility strategy interactive elements must follow (e.g. `data-testid` on every interactive element)

### GitHub Actions workflows

- Follow good security practices
- Make sure to explicitly set the workflow permissions
- Add comments to document what tasks are being performed

## Scripts

- The project uses **npm scripts** for all development tasks — there is no `scripts/` directory.
- **Skills take precedence.** Before running a command directly, check whether a skill covers the task (e.g. the `quality-checks` skill wraps tests and lint). If one applies, follow it.
- Key npm scripts:
  - `npm run dev` — start the Astro dev server (`predev` migrates + seeds the local SQLite database)
  - `npm run build` — build the static site (`prebuild` migrates + seeds the local SQLite database)
  - `npm run preview` — serve the built `dist/` output
  - `npm run lint` — ESLint
  - `npm run test:unit` — Vitest unit tests
  - `npm run test:e2e` — Playwright E2E tests (builds + previews first)
  - `npm run typecheck` — type-check the pure TypeScript with `tsgo` (TypeScript 7 native compiler, via `@typescript/native-preview`) using `tsconfig.tsgo.json`
  - `npm run typecheck:astro` — type-check `.astro` files with `astro check` (classic TypeScript package)
  - `npm run typecheck:all` — run both type-check scripts (used by the CI `type-check` job)
  - `npm run db:generate` / `db:migrate` / `db:seed` / `db:setup` — Drizzle schema/migration/seed tasks

> [!NOTE]
> TypeScript 7 (`tsgo`) is adopted **side-by-side** for type checking only; it does not affect linting. ESLint + `typescript-eslint` and `astro check` still resolve the classic `typescript` package (kept at v6) because the native compiler's API isn't ready for them yet. Do **not** bump the classic `typescript` package to 7 (a Dependabot `ignore` holds it) until `typescript-eslint` + `@astrojs/check` support the native API. `tsgo` is `--noEmit` only; the site is still built by `astro build`.

## Repository Structure

The application lives at the repository root:

- `db/`: Drizzle schema, migrations, transforms, seed, and `games.csv`
- `src/lib/`: Node SQLite client (`db.ts`) and data-access helpers (`games.ts`)
- `src/components/`: reusable `.astro` components
- `src/layouts/`: Astro layout templates
- `src/pages/`: Astro page routes (`index.astro` listing, `game/[id].astro`, `404.astro`, `about.astro`)
- `src/styles/`: CSS and Tailwind configuration
- `src/types/`: TypeScript interfaces (Game, Publisher, Category)
- `e2e-tests/`: Playwright E2E tests (home, games, accessibility)
- `drizzle.config.ts`, `vitest.config.ts`, `astro.config.mjs`, `playwright.config.ts`: tooling config
- `README.md`: Project documentation

## Instruction files

Path-scoped guidance lives in `.github/instructions/` (auto-applied by path for tools that support it, e.g. Copilot; other agents should read the relevant file directly before editing matching paths):

| Pattern | File | Covers |
|---|---|---|
| `**/*.astro` | `.github/instructions/astro.instructions.md` | Astro pages, layouts, components, routing |
| `db/**/*.ts`, `src/lib/*.ts` | `.github/instructions/drizzle.instructions.md` | Drizzle ORM + Node SQLite data layer |
| `**/*.spec.ts` | `.github/instructions/playwright.instructions.md` | Playwright test generation |
| `**/*.{astro,css}` | `.github/instructions/style.instructions.md` | Tailwind CSS v4 / dark theme |
| `**/*.test.ts` | `.github/instructions/unit-tests.instructions.md` | Vitest unit tests |
| all UI work | `.github/instructions/ui.instructions.md` | Component strategy, testability, accessibility |

## PR / commit instructions

- Never commit or push to `main` automatically unless explicitly instructed to do so.
- Do not generate summary markdown files on task completion unless explicitly requested.
- Keep commits scoped and precise; don't fix unrelated pre-existing issues.
- Follow `CONTRIBUTING.md` and the repository's issue/PR templates for the contribution process itself.
