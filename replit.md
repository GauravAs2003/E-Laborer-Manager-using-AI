# E-Laborer Management System

A full-stack workforce management system for construction/labor operations, combining a Java console app with a modern React web dashboard.

## Architecture

### Monorepo Structure (pnpm workspaces)
- `artifacts/e-laborer` — React+Vite frontend dashboard (port 21112, served at `/`)
- `artifacts/api-server` — Express REST API backend (port 8080, served at `/api`)
- `lib/db` — Drizzle ORM schema + PostgreSQL client
- `lib/api-spec` — OpenAPI v0.2 specification (`openapi.yaml`)
- `lib/api-zod` — Generated Zod schemas from OpenAPI spec
- `lib/api-client-react` — Generated React Query hooks from OpenAPI spec
- `e-laborer-management-system/src/` — Java console application (OOP/ArrayList-based)

### Database (PostgreSQL via Drizzle ORM)
Tables:
- `laborers` — id, name, skill, experience (nullable), dailyWage, assignedWork (nullable), status (default: 'available')
- `work_assignments` — id, laborerId (FK), workName, location, startDate, endDate, wage, createdAt

### API Endpoints
- `GET /api/health` — Health check
- `GET /api/laborers` — List all laborers (supports query filters: name, skill, minWage, maxWage, available)
- `POST /api/laborers` — Create a new laborer
- `GET /api/laborers/:id` — Get a single laborer profile
- `PATCH /api/laborers/:id` — Update a laborer
- `DELETE /api/laborers/:id` — Delete a laborer
- `GET /api/laborers/:id/history` — Get work assignment history for a laborer
- `POST /api/assign-work` — Create a new work assignment (also updates laborer status to 'active')
- `GET /api/assignments` — List all work assignments (joined with laborer names)

## Web Dashboard Features

### Pages (all accessible from the dark sidebar nav)
1. **Dashboard** (`/`) — 4 stat cards (total/active/available/avg wage) + Recharts charts (skill distribution bar, workforce status pie, top wage earners bar) + recent laborers list
2. **Add Laborer** (`/add`) — Form with name, skill, experience, daily wage + toast notification on success
3. **View Laborers** (`/laborers`) — Full table with status badges, clickable profiles, delete and assign-work actions
4. **Active Assignments** (`/assignments`) — Filtered view of laborers with active work assignments
5. **Advanced Search** (`/search`) — Multi-filter search (name, skill, wage range, availability dropdown)
6. **Assign Work** (`/assign-work`) — Form to assign work with laborer dropdown, work name, location, start/end dates, wage
7. **Laborer Profile** (`/laborers/:id`) — Hero card with stats + full work history table
8. **Reports** (`/reports`) — Summary stats + CSV export buttons for laborers and assignments

### Tech Stack (Frontend)
- React 19 + Vite 7
- Wouter (client-side routing)
- TanStack Query (React Query) with generated hooks from `@workspace/api-client-react`
- Recharts (charts/data visualization)
- React Hook Form + Zod (form validation)
- Framer Motion (animations)
- Tailwind CSS (styling)
- Lucide React (icons)
- shadcn/ui components (Card, Toast, etc.)

## Java Console App

**Location:** `e-laborer-management-system/src/`  
**Run:** `cd e-laborer-management-system/src && java Main`

9-item menu:
1. Add Laborer
2. View All Laborers
3. Assign Work
4. Calculate Wage
5. Search Laborer
6. Update Laborer
7. Remove Laborer
8. Wage Summary
9. Exit

## Development

### Running the project
- Both workflows start automatically: API Server and Web frontend
- To run manually:
  - API: `pnpm --filter @workspace/api-server run dev`
  - Frontend: `pnpm --filter @workspace/e-laborer run dev`

### Regenerating API client after spec changes
```bash
pnpm --filter @workspace/api-zod run generate
pnpm --filter @workspace/api-client-react run generate
```

### Pushing DB schema changes
```bash
pnpm --filter @workspace/db run push
```

## Key Notes
- Do NOT add artifact packages to the root `tsconfig.json` references
- Use `catalog:` for shared deps in `pnpm-workspace.yaml`
- Use `zod/v4` import style across the project
- Frontend base URL is read from `import.meta.env.BASE_URL` (Vite)
- API client hooks use `getGetLaborersQueryKey()` for cache invalidation after mutations
- CSV export is client-side only (no backend needed)
