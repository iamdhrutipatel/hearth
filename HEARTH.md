# hearth: instructions for AI assistants

hearth is `iamdhrutipatel/hearth`, a personal fork of `securo-finance/securo`
that carries lovenest's code (`ADolkun/lovenest`). This folder is both the
development clone and the install that runs the live finance app.

`AGENTS.md` belongs to lovenest and changes with its releases, so it is kept
unedited apart from a pointer to this file (see Fork changes to lovenest
files). Where the two differ, this file wins. In particular, these `AGENTS.md` sections do not apply here: Branches,
Normal PR flow, Upstream sync flow, Verify before PR, and GitNexus. Its Safety
section still applies.

## Branches and remotes

| Branch | Role | Changes only by |
| --- | --- | --- |
| `hearth` | Default branch, the code the app runs | A merged PR |
| `lovenest` | Mirror of `ADolkun/lovenest:lovenest` | Fast-forward from `adolkun/lovenest` |
| `main` | Mirror of `securo-finance/securo:main` | Fast-forward from `upstream/main` |

| Remote | Repository |
| --- | --- |
| `origin` | `iamdhrutipatel/hearth` |
| `upstream` | `securo-finance/securo` |
| `adolkun` | `ADolkun/lovenest` |

Pass `--repo` to every `gh` command: no default repository is set, and in a
fork `gh` may pick the parent. Issues are disabled on `iamdhrutipatel/hearth`.

## Making a change

1. Branch from an up-to-date `hearth` as `<type>/<short-description>`, for
   example `fix/dashboard-chart-axis`.
2. Commit with a Conventional Commit subject, for example
   `docs: add hearth instructions`. Add no AI attribution: no
   `Co-Authored-By` trailer and no "Generated with" line.
3. Run the checks in `.pr-preparation.yml` for the areas you changed.
4. Push to `origin` and open a PR into `iamdhrutipatel/hearth:hearth`.

The `pr-preparation` skill automates these steps and reads its project rules
from `.pr-preparation.yml`.

Open a PR to an upstream only when the user asks for it:

- Securo: branch from `main`, then PR to `securo-finance/securo:main`.
- lovenest: branch from `lovenest`, then PR to `ADolkun/lovenest:lovenest`.

These PRs are public. They must not contain hearth-only commits or files,
such as this file or `.pr-preparation.yml`.

## Checks

The `checks` in `.pr-preparation.yml` mirror `.github/workflows/lovenest-ci.yml`.
Backend `pytest` runs only in CI, because it needs the Postgres and Redis
services the workflow starts. `uv` may be missing; ask before installing it.

## Updating from upstream

1. Fast-forward the mirrors:

   ```bash
   git fetch upstream && git fetch adolkun && git fetch origin
   git switch main && git merge --ff-only upstream/main && git push origin main
   git switch lovenest && git merge --ff-only adolkun/lovenest && git push origin lovenest
   ```

2. Merge `lovenest` into `hearth` through a PR, using a merge commit (not
   squash or rebase) so later syncs stay conflict-free:

   ```bash
   git switch hearth && git pull --ff-only origin hearth
   git switch -c chore/sync-lovenest-YYYYMMDD
   git merge lovenest
   git push -u origin HEAD
   gh pr create --repo iamdhrutipatel/hearth --base hearth --head chore/sync-lovenest-YYYYMMDD
   ```

3. After the PR merges, the user decides when to update the running app:

   ```bash
   docker compose -f docker-compose.prod.yml exec -T db pg_dump -U postgres -Fc securo \
     > ~/hearth-backups/db-before-update-$(date +%Y%m%d-%H%M).dump
   git switch hearth && git pull --ff-only origin hearth
   docker compose -f docker-compose.prod.yml up -d --build
   docker compose -f docker-compose.prod.yml ps
   ```

   The rebuild runs database migrations. Previous images stay tagged by
   version, so a rollback is possible; restoring a backup needs the user's
   approval.

## Fork changes to lovenest files

hearth keeps its edits to lovenest's files to these few lines so that syncs
merge cleanly. Put new hearth-only content in hearth-only files instead.

| File | hearth change |
| --- | --- |
| `AGENTS.md` | Pointer to this file on the first line |
| `README.md` | Note above the title that names the fork |
| `frontend/index.html` | App name `Hearth` in `<title>` and the name meta tags |
| `frontend/src/locales/*.json` | `app.name` is `Hearth`, and `setup.title` welcomes to Hearth |

If a sync conflicts on one of these lines, keep lovenest's other changes and
reapply the hearth change. If lovenest adds a language file, set its
`app.name` and `setup.title` the same way. The tax planner (`extras/tax/`)
still says Lovenest on purpose.

## Safety

- Run `docker compose up`, `build`, `down`, `restart` or `stop`, or any
  `alembic` migration, only when the user asks. Rebuild only from `hearth`.
- Keep the Compose project name `securo`: it names the `securo_*` volumes
  that hold all app data (database, attachments and agent knowledge).
- Never read `.env` values, the live database or `~/hearth-backups/` without
  the user's permission. Never commit them.
- Never change `SECRET_KEY` in `.env`: it encrypts the stored bank connection
  tokens.
- Write account numbers as `XXXX` in commits, PRs and logs.

## Local environment

- Docker Desktop. Shells that don't load `~/.zprofile` need
  `/Applications/Docker.app/Contents/Resources/bin` on `PATH`.
- GitHub access uses HTTPS through `gh auth git-credential`; SSH isn't set up.
- GitNexus isn't installed, so its tools and `.gitnexus/` commands are
  unavailable.
