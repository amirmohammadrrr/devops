# DevOps

This repository contains DevOps-related infrastructure, scripts, and documentation
used to provision, deploy, and maintain services for the project.

## Contents

- `infra/` — Infrastructure-as-code (Terraform, CloudFormation, etc.)
- `ci/` — CI pipeline definitions and helper scripts
- `scripts/` — Utility scripts for maintenance and automation
- `migrations/` — Database migration files and migration tooling

## Getting Started

1. Clone the repo:

	 git clone <repo-url>

2. Review the `infra/` and `ci/` directories for environment-specific configurations.

3. For database migrations, see the `migrations/` directory and the `Schema Changes` section below.

## Conventions

- Branching: follow `main` as the protected production branch; create feature branches for changes.
- Commits: use conventional commit messages (e.g., `feat:`, `fix:`, `docs:`, `chore:`).
- Migrations: keep migration files in `migrations/` with timestamped filenames.

**Schema Changes**

When you need to modify the database schema (add tables, alter columns, create indexes), follow the process below.

- **Overview:** All schema changes must be applied via migration files stored in `migrations/` and executed by the project's migration tool (Flyway, Liquibase, Alembic, or a custom runner).
- **Migration file naming:** Use a timestamped, ordered naming convention so migrations apply deterministically. Example:

	migrations/20251122_120000_add_users_table.sql

- **Migration structure:** Each migration should be idempotent if practical and include a corresponding rollback where supported. Include a brief comment header describing intent and any required application-side changes.

- **Example migration (Postgres):**

	-- migrations/20251122_120000_add_users_table.sql
	-- Add a users table used by the auth service
	BEGIN;
	CREATE TABLE IF NOT EXISTS users (
		id UUID PRIMARY KEY,
		email TEXT NOT NULL UNIQUE,
		hashed_password TEXT NOT NULL,
		created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
	);
	COMMIT;

- **Rollback (example):**

	-- migrations/20251122_120000_add_users_table_down.sql
	BEGIN;
	DROP TABLE IF EXISTS users;
	COMMIT;

- **Applying migrations:** Add a CI/CD step to run migrations against the target environment before or during deployments. Example command (if using a generic SQL runner):

	./scripts/run-migrations.sh --database-url "$DATABASE_URL" --migrations-dir migrations/

- **Best practices:**
	- Keep migrations small and focused to simplify rollbacks and troubleshooting.
	- Avoid long-running table locks — prefer non-blocking schema changes when possible.
	- Test migrations on a staging database before applying to production.

## CI/CD

- The CI pipeline should lint, run tests, and validate infrastructure changes. For deployments, ensure migrations run as a distinct step and that database backups are taken prior to production migrations.

## Contact

For questions about infrastructure or migrations, contact the DevOps team owner.

---

Generated sample README.