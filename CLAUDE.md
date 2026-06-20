This project is Postiz, a tool to schedule social media and chat posts to 28+ channels.
You can add posts to the calendar, they will be added into a workflow and posted at the right time.
You can find things like:
- Schedule posts
- Calendar view
- Analytics
- Team management
- Media library

This project is a monorepo with a root-only `package.json`. Requires **Node 22.12.x** and **pnpm 10.6.1**.
Workspaces use the `@gitroom/*` package name prefix.

> Gotcha: `package.json` has a stale `volta.node: 20.17.0` pin that conflicts with `engines` (Node 22.12.x). If you use Volta, override it to a 22.12.x toolchain — Node 20 will fail `engines`.

## Quick Start

```bash
pnpm install            # also runs prisma-generate via postinstall
pnpm dev                # backend + orchestrator + frontend + extension in parallel
pnpm dev:docker         # start Postgres/Redis via docker-compose.dev.yaml
pnpm test               # jest --coverage --detectOpenHandles (junit reporter)
pnpm build              # frontend + backend + orchestrator
pnpm lint               # lint only from the root, never from a subpackage
pnpm prisma-db-push     # destructive: --accept-data-loss
pnpm prisma-generate    # regenerate Prisma client
```

## Workspaces

```text
apps/
  backend/         NestJS API (controllers/services/repositories)
  orchestrator/    NestJS + Temporal workflows and activities
  frontend/        Vite + React (config: apps/frontend/tailwind.config.cjs)
  commands/        NestJS CLI app (build with `pnpm commands:build:development`)
  extension/       Browser extension (built with `pnpm build:extension`)
  sdk/             Published as @gitroom/sdk (`pnpm publish-sdk`)
libraries/
  nestjs-libraries/       Server-side: Prisma schema, integrations, services
  helpers/                Auth, decorators, fetch hooks (@gitroom/react/helpers/...)
  react-shared-libraries/  Shared React components and UI primitives
```

## Prisma

Schema lives at `libraries/nestjs-libraries/src/database/prisma/schema.prisma`.
Always run `pnpm prisma-generate` after schema changes.

We are using only pnpm, don't use any other dependency manager.
Never install frontend components from npmjs, focus on writing native components.

The project uses tailwind 3. Before writing any component look at:
- `apps/frontend/src/app/colors.scss`
- `apps/frontend/src/app/global.scss`
- `apps/frontend/tailwind.config.cjs`

Shared UI primitives live in `libraries/react-shared-libraries/src/`, not in
`apps/frontend/src/components/ui/` (that folder is mostly icons and small utilities).

All the --color-custom* are deprecated, don't use them.

And check other components in the system before to get the right design.

When working on the backend we need to pass the 3 layers:
Controller >> Service >> Repository (no shortcuts)
In some cases we will have
Controller >> Mananger >> Service >> Repository.

Most of the server logic should be inside of libs/server.
The backend repository is mostly used to write controller, and import files from libs.server.

For the frontend follow this:
- Shared UI primitives live in `libraries/react-shared-libraries/src/`. `apps/frontend/src/components/ui/` only contains icons and small utilities.
- Routing is in /apps/frontend/src/app
- Feature components are in /apps/frontend/src/components
- always use SWR to fetch stuff, and use "useFetch" hook from /libraries/helpers/src/utils/custom.fetch.tsx

When using SWR, each one have to be in a seperate hook and must comply with react-hooks/rules-of-hooks, never put eslint-disable-next-line on it.

It means that this is valid:
const useCommunity = () => {
   return useSWR....
}

This is not valid:
const useCommunity = () => {
  return {
    communities: () => useSWR<CommunitiesListResponse>("communities", getCommunities),
    providers: () => useSWR<ProvidersListResponse>("providers", getProviders),
  };
}

- Linting of the project can run only from the root.
- Use only pnpm.
- The system is in production with many users, if you want to change something, you need to be sure that you are not breaking anything for existing users and a migration might be needed

## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
