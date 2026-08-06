# Audit Log PoC — PostgreSQL Partitioning + JSONB

Research project: a high-performance audit log system on PostgreSQL 16.

## Quick Start (from a clean machine)

### Option 1: Docker (Recommended)

```bash
# 1. Clone repository
git clone <repo-url>
cd csdl

# 2. Copy and edit the configuration (optional)
cp .env.example .env
# Adjust .env as needed

# 3. Start PostgreSQL via Docker
docker-compose up -d

# 4. Run the benchmark
bash bench/run_baseline.sh
bash bench/run_proposed.sh

# 5. View performance results
bash bench/analyze_performance.sh
```

### Option 2: Local PostgreSQL

```bash
# 1. Prerequisites
#    PostgreSQL 16, psql, pgbench — available on Ubuntu 22.04 WSL2

# 2. Bootstrap the database (run as superuser)
#    Roles must exist BEFORE creating the database with OWNER db_admin
psql "postgresql://postgres:postgres@localhost/postgres" -f sql/00_roles.sql
psql "postgresql://postgres:postgres@localhost/postgres" -c "CREATE DATABASE audit_poc OWNER db_admin;"

# 3. Schema
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/01_schema_audit.sql
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/02_schema_business.sql

# 4. Seed data (~5-8 GB, runs in the background)
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/03_seed_data.sql

# 5. Audit function + triggers
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/04_audit_function.sql

# 6. Immutability
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/05_security_immutability.sql

# 7. (Optional) Hash chain
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/06_hash_chain.sql

# 8. (Optional) DDL Audit - audit schema changes (requires superuser — event trigger creation)
psql "postgresql://postgres:postgres@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/09_audit_ddl.sql

# 9. Indexes
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/07_indexes.sql

# 10. GRANT/REVOKE
psql "postgresql://db_admin:db_admin_pass@localhost/audit_poc" -v ON_ERROR_STOP=1 -f sql/08_grants.sql

# 11. Benchmark
bash bench/run_baseline.sh
bash bench/run_proposed.sh
```

## Recent Enhancements

### 📊 Performance Monitoring
- **pg_stat_statements**: Automatically enabled during benchmarks to track performance
- **analyze_performance.sh**: Analyzes slow queries, call frequency, and audit trigger impact
- **monitor.sh**: Real-time monitoring during benchmark runs (connections, TPS, WAL, cache hit ratio)

### 🔍 DDL Audit Support
- **sql/09_audit_ddl.sql**: Event trigger that records schema changes (CREATE, ALTER, DROP)
- Stored in the `audit_ddl_logs` table with user information and the SQL statement

### 🐳 Docker Support
- **docker-compose.yml**: Starts PostgreSQL 16 with pg_stat_statements ready to use
- **Adminer**: Web interface for managing the database at http://localhost:8080
- Configure via the `.env` file (copy from `.env.example`)

### 🗂️ Partition Management
- **scripts/manage_partitions.sh**: Automatic partition management
  - `create`: Create partitions for the next 3 months
  - `drop_old`: Drop old partitions (default: older than 6 months)
  - `list`: List all partitions and their sizes

### 📖 Enhanced Documentation
- **docs/GETTING_STARTED.md**: Step-by-step guide for Docker and local setup
- Performance monitoring and DDL audit are integrated into the workflow

## Reset

```bash
psql "postgresql://postgres:postgres@localhost/postgres" -f sql/99_cleanup.sql
```

## Documentation

- `6-baocao.md` — Research report outline
- `7-huongdan-xay-dung-source.md` — Detailed technical guide
- `8-ke-hoach-xay-dung.md` — Phase-by-phase implementation plan
- `docs/results-summary.md` — Experimental results summary (fill after running the benchmark)
