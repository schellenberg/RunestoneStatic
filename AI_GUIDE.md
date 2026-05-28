# RunestoneComponents AI Guide (Static Fork Focus)

## Purpose
This file captures research and decisions for creating a static-first fork (`runestone-static`) from this repository.

## Repository Snapshot
- Repo: `schellenberg/RunestoneComponents`
- Current base seen in this workspace: `6.6.2` era, commit `d05ee26` lineage.
- Upstream RunestoneComponents is archived; this fork is the practical target for static-focused work.

## Backend Coupling: Where It Lives
1. **Build-time config (`pavement.py` templates)**
   - Main control is `template_args` in `/tmp/workspace/schellenberg/RunestoneComponents/runestone/common/project_template/pavement.tmpl`.
   - Key flags: `use_services`, `dynamic_pages`, `login_required`, `loglevel`, `dburl`, `jobe_server`, `proxy_uri_runs`, `proxy_uri_files`.
2. **Build-time DB sync**
   - `/tmp/workspace/schellenberg/RunestoneComponents/runestone/server/componentdb.py`
   - DB writes are guarded by `dburl`/session checks; static builds can proceed without DB.
3. **Runtime JS behavior**
   - In static mode, runtime falls back to local behavior (`localStorage` flows) and avoids service logging/calls when flags are off.
4. **Template/UI service wiring**
   - `/tmp/workspace/schellenberg/RunestoneComponents/runestone/common/project_template/_templates/plugin_layouts/sphinx_bootstrap/layout.html`
   - Service/account menu items are gated by `use_services == 'true'`.

## Static-Compatible vs Service-Dependent Features

### Works in static mode
- Core client-side interactives (many activecode modes, mchoice, fitb, parsons, codelens, reveal/tabbed UI, spreadsheet, etc.) generally degrade to local-only behavior.

### Limited or service-dependent
- `selectquestion` needs dynamic services.
- Timed/group/assignment flows are service-oriented (collection/grading/reporting).
- Non-Python `activecode` backends may require `jobe_server` infrastructure.

## Static Defaults to Use
Set defaults in project templates to:
- `login_required: 'false'`
- `loglevel: 0`
- `use_services: 'false'`
- `dynamic_pages: False`
- `dburl: ''`
- `jobe_server: ''`
- `proxy_uri_runs: ''`
- `proxy_uri_files: ''`

## Template/UI Notes
- In static mode, avoid login-targeted navbar links.
- Gate ad/service scripts behind dynamic/service checks.
- Keep account/progress service UI hidden when `use_services` is false.

## Changes Already Applied in This Branch
1. `/tmp/workspace/schellenberg/RunestoneComponents/runestone/common/project_template/pavement.tmpl`
   - Defaulted generated projects to static-only values above.
   - Removed DBURL override in this template path for static default behavior.
2. `/tmp/workspace/schellenberg/RunestoneComponents/runestone/common/project_template/_templates/plugin_layouts/sphinx_bootstrap/layout.html`
   - Brand logo link points to book index instead of login URL.
   - EthicalAds script now loads only when `dynamic_pages == 'True'`.

## Suggested Next Steps (When Continuing Static Fork Work)
1. Consider making SQLAlchemy optional for strict static packaging.
2. Audit remaining UI/service references for static-only forks.
3. Decide policy for partially service-dependent directives (disable, warn, or keep degraded behavior).
4. Add a lightweight static-mode validation workflow once dependencies are stabilized.

## Current Validation/Environment Constraints Seen Here
- `npm run build` currently fails due missing JS dependency resolution and invalid dependency name entry in `package.json` (`"-": "0.0.1"`).
- `pytest` is unavailable in this runtime (`pytest: command not found`).

## Workflow Constraint for This Repository
- Remote rules can block pushes to newly created branch names; use the pre-provisioned `copilot/*` branch for push operations in this environment.
