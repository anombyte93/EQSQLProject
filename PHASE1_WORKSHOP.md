# Phase 1: Database Foundations — Building Your First Production Database

**Workshop Series**: Architecture Firm Database Project
**Phase**: 1 of 4
**Duration**: 3-4 hours
**Difficulty**: Beginner to Intermediate SQL
**Prerequisites**: Basic command line knowledge, PostgreSQL installed

---

## 🎯 Learning Objectives

By the end of this workshop, you will be able to:
1. **Design** a normalized database schema with proper constraints
2. **Implement** CRUD operations with parameterized queries
3. **Perform** backups and understand disaster recovery basics
4. **Apply** security principles to prevent SQL injection
5. **Monitor** database activity and performance
6. **Identify** compliance requirements for sensitive data

## 📚 Materials Needed

- PostgreSQL 14+ installed ([Download](https://www.postgresql.org/download/))
- Terminal/Command Prompt access
- Text editor (VS Code, Notepad++, vim, etc.)
- This worksheet (save it locally!)
- Coffee ☕ (optional but recommended)

## 🏗️ Project Context

You've been hired by **ArchitectPro**, an international architecture firm, to build their database system from scratch. They currently use spreadsheets and need a proper database to manage clients, projects, and employees.

**Why this matters in industry**: 60% of businesses still rely on spreadsheets for critical data. Moving to a proper database reduces errors by 94% and improves data integrity significantly.

---

## Part 1: Database Setup and Connection

### Step 1.1: Create Your Database

Open your terminal and connect to PostgreSQL:

```bash
# Connect to PostgreSQL as the default superuser
psql -U postgres

# If prompted for a password, enter the one you set during installation
```

**🔍 What's happening?** `psql` is the PostgreSQL interactive terminal. The `-U` flag specifies the username.

Now create your database:

```sql
-- Create the database
CREATE DATABASE arch_firm;

-- List all databases to confirm creation
\l

-- Connect to your new database
\c arch_firm

-- Check your current database connection
SELECT current_database();
```

**💡 Industry Insight**: In production, you'd never use the postgres superuser for application access. We'll create a limited user later for security.

### Step 1.2: Environment Setup for Security

Create a `.env` file in your project directory:

```bash
# Exit psql temporarily
\q

# Create a project directory
mkdir ~/architectpro_project
cd ~/architectpro_project

# Create environment file
echo "DB_HOST=localhost
DB_PORT=5432
DB_NAME=arch_firm
DB_USER=postgres
DB_PASSWORD=yourpassword" > .env

# IMPORTANT: Add .env to .gitignore to prevent credential leaks
echo ".env" >> .gitignore
```

**🛡️ Security Check**: Never commit `.env` files to version control! This prevents credential exposure in GitHub/GitLab.

### 🧪 Mini Quiz 1
**Question**: What would happen if you accidentally pushed your `.env` file to GitHub?
<details>
<summary>Click for answer</summary>

Your database credentials would be exposed publicly. Attackers could connect to your database if it's internet-accessible. Always rotate credentials immediately if this happens and use GitHub's secret scanning to detect leaks.
</details>

---

## Part 2: Schema Design and Implementation

### Step 2.1: Understanding the Business Requirements

ArchitectPro needs to track:
- **Clients**: Companies that hire them for projects
- **Projects**: Building designs and construction oversight
- **Employees**: Architects, engineers, project managers

### Step 2.2: Create the Tables

Connect back to your database:
```bash
psql -U postgres -d arch_firm
```

Now create your schema with industry-standard constraints:

```sql
-- Enable UUID extension for future API integration
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Create clients table
CREATE TABLE clients (
    client_id SERIAL PRIMARY KEY,
    company_name VARCHAR(200) NOT NULL,
    contact_email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    address TEXT,
    country VARCHAR(100) NOT NULL DEFAULT 'Australia',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT true,

    -- Business constraint: Email must be valid format
    CONSTRAINT valid_email CHECK (contact_email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z]{2,}$')
);

-- Create employees table
CREATE TABLE employees (
    employee_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role VARCHAR(50) NOT NULL,
    department VARCHAR(50) DEFAULT 'Architecture',
    hire_date DATE NOT NULL DEFAULT CURRENT_DATE,
    salary DECIMAL(10,2) CHECK (salary > 0),
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0,

    -- Security: Track login attempts for breach detection
    CONSTRAINT reasonable_salary CHECK (salary BETWEEN 30000 AND 500000)
);

-- Create projects table with foreign key
CREATE TABLE projects (
    project_id SERIAL PRIMARY KEY,
    client_id INTEGER NOT NULL,
    project_name VARCHAR(200) NOT NULL,
    project_type VARCHAR(50) NOT NULL,
    budget DECIMAL(12,2) CHECK (budget > 0),
    start_date DATE NOT NULL DEFAULT CURRENT_DATE,
    end_date DATE,
    status VARCHAR(20) DEFAULT 'Planning',
    created_by INTEGER,

    -- Foreign key relationships
    CONSTRAINT fk_client
        FOREIGN KEY (client_id)
        REFERENCES clients(client_id)
        ON DELETE RESTRICT,

    CONSTRAINT fk_creator
        FOREIGN KEY (created_by)
        REFERENCES employees(employee_id)
        ON DELETE SET NULL,

    -- Business logic constraints
    CONSTRAINT valid_dates CHECK (end_date IS NULL OR end_date > start_date),
    CONSTRAINT valid_status CHECK (status IN ('Planning', 'Active', 'On Hold', 'Completed', 'Cancelled')),
    CONSTRAINT valid_project_type CHECK (project_type IN ('Residential', 'Commercial', 'Industrial', 'Public', 'Mixed-Use'))
);

-- Create indexes for better query performance
CREATE INDEX idx_projects_client ON projects(client_id);
CREATE INDEX idx_projects_status ON projects(status) WHERE status = 'Active';
CREATE INDEX idx_employees_email ON employees(email);

-- View your tables
\dt

-- Describe table structure
\d clients
\d employees
\d projects
```

**🔍 What's happening with these constraints?**
- `SERIAL PRIMARY KEY`: Auto-incrementing unique identifier
- `NOT NULL`: Prevents missing critical data
- `UNIQUE`: Ensures no duplicate emails
- `CHECK`: Validates data (salary > 0, valid email format)
- `FOREIGN KEY`: Maintains referential integrity
- `ON DELETE RESTRICT`: Prevents deleting clients with active projects
- `DEFAULT`: Provides sensible defaults

### Step 2.3: Test Your Constraints (Learning from Errors)

Let's intentionally break things to understand our safety nets:

```sql
-- Test 1: Try to insert invalid email
INSERT INTO clients (company_name, contact_email, country)
VALUES ('Bad Email Corp', 'not-an-email', 'Australia');
-- ERROR: Email validation will fail!

-- Test 2: Try to insert negative salary
INSERT INTO employees (email, first_name, last_name, role, salary)
VALUES ('john@architectpro.com', 'John', 'Doe', 'Architect', -5000);
-- ERROR: Salary must be positive!

-- Test 3: Try to insert project with non-existent client
INSERT INTO projects (client_id, project_name, project_type, budget)
VALUES (999, 'Ghost Project', 'Residential', 100000);
-- ERROR: Foreign key violation!
```

### 🧪 Mini Quiz 2
**Question**: What happens if you try to DELETE a client that has projects?
<details>
<summary>Click for answer</summary>

The deletion will fail due to `ON DELETE RESTRICT`. This prevents orphaned projects. In production, you might use `ON DELETE CASCADE` for automatic cleanup or `ON DELETE SET NULL` to keep historical records.
</details>

---

## Part 3: Seed Data and CRUD Operations

### Step 3.1: Insert Realistic Seed Data

```sql
-- Insert clients (Australian companies)
INSERT INTO clients (company_name, contact_email, phone, address, country) VALUES
('Westfield Group', 'contracts@westfield.com.au', '+61-2-9358-7000', '85 Castlereagh St, Sydney NSW 2000', 'Australia'),
('Lendlease Corporation', 'info@lendlease.com', '+61-2-9236-6111', 'Level 14, Tower Three, Barangaroo', 'Australia'),
('Mirvac Limited', 'enquiries@mirvac.com', '+61-2-9080-8000', '200 George St, Sydney NSW 2000', 'Australia'),
('Crown Resorts', 'projects@crownresorts.com.au', '+61-3-9292-8888', '8 Whiteman St, Melbourne VIC 3006', 'Australia'),
('Stockland Property', 'development@stockland.com.au', '+61-2-9035-2000', '133 Castlereagh St, Sydney NSW 2000', 'Australia');

-- Verify insertion
SELECT client_id, company_name, country FROM clients;

-- Insert employees with different roles
INSERT INTO employees (email, first_name, last_name, role, department, hire_date, salary) VALUES
('sarah.chen@architectpro.com', 'Sarah', 'Chen', 'Principal Architect', 'Architecture', '2019-03-15', 150000),
('james.wilson@architectpro.com', 'James', 'Wilson', 'Project Manager', 'Management', '2020-07-01', 95000),
('maria.garcia@architectpro.com', 'Maria', 'Garcia', 'Senior Architect', 'Architecture', '2018-11-20', 110000),
('alex.kumar@architectpro.com', 'Alex', 'Kumar', 'Junior Architect', 'Architecture', '2022-01-10', 65000),
('emma.brown@architectpro.com', 'Emma', 'Brown', 'Structural Engineer', 'Engineering', '2021-05-15', 85000);

-- Insert projects with foreign key relationships
INSERT INTO projects (client_id, project_name, project_type, budget, start_date, end_date, status, created_by) VALUES
(1, 'Westfield Sydney Tower Renovation', 'Commercial', 2500000.00, '2024-01-15', '2024-12-31', 'Active', 1),
(2, 'Barangaroo South Residential Complex', 'Residential', 8000000.00, '2024-03-01', '2025-06-30', 'Planning', 2),
(3, 'Mirvac Office Sustainability Retrofit', 'Commercial', 1500000.00, '2024-02-01', '2024-08-31', 'Active', 3),
(4, 'Crown Casino Hotel Extension', 'Mixed-Use', 12000000.00, '2024-06-01', '2025-12-31', 'Planning', 1),
(5, 'Stockland Green Shopping Centre', 'Commercial', 5000000.00, '2024-04-15', '2025-03-31', 'Active', 2),
(1, 'Westfield Bondi Junction Upgrade', 'Commercial', 3500000.00, '2023-07-01', '2024-06-30', 'Completed', 3);

-- Count records in each table
SELECT
    'clients' as table_name, COUNT(*) as row_count FROM clients
UNION ALL
SELECT 'employees', COUNT(*) FROM employees
UNION ALL
SELECT 'projects', COUNT(*) FROM projects;
```

### Step 3.2: Read Operations (SELECT Queries)

```sql
-- Basic SELECT with filtering
SELECT project_name, budget, status
FROM projects
WHERE status = 'Active'
ORDER BY budget DESC;

-- JOIN query to see clients and their projects
SELECT
    c.company_name,
    p.project_name,
    p.budget,
    p.status,
    e.first_name || ' ' || e.last_name AS project_creator
FROM clients c
INNER JOIN projects p ON c.client_id = p.client_id
LEFT JOIN employees e ON p.created_by = e.employee_id
ORDER BY c.company_name, p.start_date;

-- Aggregation: Total budget by project type
SELECT
    project_type,
    COUNT(*) as project_count,
    SUM(budget) as total_budget,
    AVG(budget) as avg_budget,
    MAX(budget) as max_budget
FROM projects
GROUP BY project_type
ORDER BY total_budget DESC;

-- Find clients without projects (useful for sales team)
SELECT c.company_name, c.contact_email
FROM clients c
LEFT JOIN projects p ON c.client_id = p.client_id
WHERE p.project_id IS NULL;
```

### Step 3.3: Update Operations

```sql
-- Update project status
UPDATE projects
SET status = 'On Hold',
    updated_at = CURRENT_TIMESTAMP
WHERE project_name = 'Crown Casino Hotel Extension';

-- Give employees a 5% raise
UPDATE employees
SET salary = salary * 1.05
WHERE hire_date < '2022-01-01';

-- Verify updates
SELECT first_name, last_name, salary
FROM employees
ORDER BY salary DESC;
```

### Step 3.4: Delete Operations (with safety discussion)

```sql
-- Soft delete approach (preferred in production)
UPDATE clients
SET is_active = false
WHERE company_name = 'Test Client';

-- Hard delete (use with caution!)
-- First, check what would be affected
SELECT * FROM projects WHERE client_id = 5;

-- If safe to delete:
DELETE FROM projects WHERE status = 'Cancelled';

-- Attempting to delete a client with projects
DELETE FROM clients WHERE client_id = 1;
-- ERROR: This will fail due to foreign key constraint!
```

### 🧪 Mini Quiz 3
**Question**: Why use soft deletes (is_active = false) instead of hard deletes?
<details>
<summary>Click for answer</summary>

Soft deletes preserve historical data for audit trails, allow recovery from mistakes, maintain referential integrity, and support compliance requirements (some regulations require data retention). Hard deletes are irreversible and can break historical reports.
</details>

---

## Part 4: Security and SQL Injection Prevention

### Step 4.1: Understanding SQL Injection

```sql
-- Create a demo table for security testing
CREATE TABLE user_logins (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    password VARCHAR(100)  -- Note: Never store plain text passwords!
);

INSERT INTO user_logins (username, password) VALUES
('admin', 'super_secret_password'),
('user1', 'password123');
```

**❌ VULNERABLE CODE (Never do this!):**
```python
# Python example of vulnerable code
user_input = "admin' OR '1'='1"  # Malicious input
query = f"SELECT * FROM user_logins WHERE username = '{user_input}'"
# This becomes: SELECT * FROM user_logins WHERE username = 'admin' OR '1'='1'
# Result: Returns ALL users! Security breach!
```

**✅ SECURE CODE (Always do this!):**
```sql
-- Using prepared statements in PostgreSQL
PREPARE secure_login (text) AS
    SELECT * FROM user_logins WHERE username = $1;

EXECUTE secure_login('admin');  -- Safe!
EXECUTE secure_login('admin'' OR ''1''=''1');  -- Still safe! Treats as literal string

-- Clean up
DEALLOCATE secure_login;
DROP TABLE user_logins;  -- Remove demo table
```

### Step 4.2: Implement Audit Logging

```sql
-- Create audit log table
CREATE TABLE audit_log (
    log_id SERIAL PRIMARY KEY,
    table_name VARCHAR(50) NOT NULL,
    operation VARCHAR(10) NOT NULL,
    user_name VARCHAR(100) DEFAULT CURRENT_USER,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    row_data JSONB
);

-- Create audit trigger function
CREATE OR REPLACE FUNCTION audit_trigger_function()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (table_name, operation, row_data)
    VALUES (
        TG_TABLE_NAME,
        TG_OP,
        CASE
            WHEN TG_OP = 'DELETE' THEN row_to_json(OLD)
            ELSE row_to_json(NEW)
        END
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply audit trigger to projects table
CREATE TRIGGER projects_audit_trigger
AFTER INSERT OR UPDATE OR DELETE ON projects
FOR EACH ROW EXECUTE FUNCTION audit_trigger_function();

-- Test the audit log
UPDATE projects SET budget = budget * 1.1 WHERE project_id = 1;

-- View audit entries
SELECT * FROM audit_log ORDER BY timestamp DESC LIMIT 5;
```

**🔍 Why audit logs matter**: Compliance requirements (SOC 2, ISO 27001) mandate tracking data changes. Audit logs help with debugging, security investigations, and regulatory audits.

---

## Part 5: Backup and Recovery

### Step 5.1: Create Your First Backup

```bash
# Exit psql
\q

# Create backup directory
mkdir -p ~/architectpro_project/backups

# Full database backup
pg_dump -U postgres -d arch_firm -F custom -f ~/architectpro_project/backups/arch_firm_$(date +%Y%m%d_%H%M%S).backup

# Human-readable SQL backup (useful for version control)
pg_dump -U postgres -d arch_firm --no-owner --clean --if-exists > ~/architectpro_project/backups/arch_firm_schema_data.sql

# Table-specific backup
pg_dump -U postgres -d arch_firm -t clients -f ~/architectpro_project/backups/clients_backup.sql
```

### Step 5.2: Test Recovery Process

```bash
# Create a test database for recovery
psql -U postgres -c "CREATE DATABASE arch_firm_recovery_test;"

# Restore from backup
pg_restore -U postgres -d arch_firm_recovery_test ~/architectpro_project/backups/arch_firm_*.backup

# Verify restoration
psql -U postgres -d arch_firm_recovery_test -c "SELECT COUNT(*) FROM projects;"

# Clean up test database
psql -U postgres -c "DROP DATABASE arch_firm_recovery_test;"
```

### Step 5.3: Automate Backups with Cron (Linux/Mac) or Task Scheduler (Windows)

Create a backup script `backup.sh`:

```bash
#!/bin/bash
# Save as ~/architectpro_project/backup.sh

DB_NAME="arch_firm"
DB_USER="postgres"
BACKUP_DIR="$HOME/architectpro_project/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup
pg_dump -U $DB_USER -d $DB_NAME -F custom -f "$BACKUP_DIR/${DB_NAME}_${DATE}.backup"

# Keep only last 7 days of backups
find $BACKUP_DIR -name "*.backup" -mtime +7 -delete

echo "Backup completed: ${DB_NAME}_${DATE}.backup"
```

Make it executable and schedule:
```bash
chmod +x ~/architectpro_project/backup.sh

# Add to crontab (runs daily at 2 AM)
(crontab -l 2>/dev/null; echo "0 2 * * * $HOME/architectpro_project/backup.sh") | crontab -
```

**💡 Industry Standard**: The 3-2-1 backup rule: 3 copies of data, 2 different storage types, 1 offsite location.

---

## Part 6: Monitoring and Performance

### Step 6.1: Basic Monitoring Queries

```sql
-- Check active connections
SELECT
    datname as database,
    pid,
    usename as username,
    application_name,
    client_addr as client_ip,
    state,
    query_start,
    SUBSTRING(query, 1, 50) as current_query
FROM pg_stat_activity
WHERE state != 'idle'
ORDER BY query_start;

-- Table sizes and row counts
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size,
    n_live_tup AS row_count
FROM pg_stat_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Slow queries (requires pg_stat_statements extension)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Index usage statistics
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Cache hit ratio (should be > 90%)
SELECT
    sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS cache_hit_ratio
FROM pg_statio_user_tables;
```

### Step 6.2: Create Monitoring Views for Easy Access

```sql
-- Create a monitoring schema
CREATE SCHEMA IF NOT EXISTS monitoring;

-- View for database health
CREATE OR REPLACE VIEW monitoring.database_health AS
SELECT
    current_database() as database_name,
    pg_size_pretty(pg_database_size(current_database())) as database_size,
    (SELECT count(*) FROM pg_stat_activity) as connection_count,
    (SELECT count(*) FROM pg_stat_activity WHERE state = 'active') as active_queries,
    (SELECT max(now() - query_start) FROM pg_stat_activity WHERE state = 'active') as longest_query_time,
    now() as check_timestamp;

-- View for table statistics
CREATE OR REPLACE VIEW monitoring.table_stats AS
SELECT
    schemaname,
    tablename,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows,
    ROUND(n_dead_tup::numeric / NULLIF(n_live_tup + n_dead_tup, 0) * 100, 2) as dead_row_percent,
    last_vacuum,
    last_autovacuum,
    last_analyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- Quick health check
SELECT * FROM monitoring.database_health;
SELECT * FROM monitoring.table_stats;
```

---

## Part 7: Command Aliases and Productivity

### Step 7.1: Create psql Shortcuts

Add to your `~/.psqlrc` file:

```sql
-- Useful psql configurations
\set QUIET 1
\timing on
\set ON_ERROR_ROLLBACK interactive
\pset null '(null)'

-- Custom commands
\set tables 'SELECT tablename FROM pg_tables WHERE schemaname = ''public'' ORDER BY tablename;'
\set connections 'SELECT datname, count(*) FROM pg_stat_activity GROUP BY datname;'
\set locks 'SELECT pid, relation::regclass, mode, granted FROM pg_locks WHERE relation IS NOT NULL;'
\set sizes 'SELECT tablename, pg_size_pretty(pg_total_relation_size(tablename::regclass)) FROM pg_tables WHERE schemaname = ''public'' ORDER BY pg_total_relation_size(tablename::regclass) DESC;'

\set QUIET 0
\echo 'Custom commands loaded: :tables :connections :locks :sizes'
```

Now you can use:
```sql
:tables      -- List all tables
:connections -- Show connections per database
:locks       -- View current locks
:sizes       -- Show table sizes
```

### Step 7.2: Create Shell Aliases

Add to your `~/.bashrc` or `~/.zshrc`:

```bash
# Database shortcuts
alias dbconnect='psql -U postgres -d arch_firm'
alias dbbackup='~/architectpro_project/backup.sh'
alias dbsize='psql -U postgres -d arch_firm -c "SELECT pg_size_pretty(pg_database_size(current_database()));"'
alias dbhealth='psql -U postgres -d arch_firm -c "SELECT * FROM monitoring.database_health;"'
```

---

## Part 8: Compliance and Data Protection

### Step 8.1: Identify Sensitive Data

```sql
-- Create a data classification table
CREATE TABLE data_classification (
    table_name VARCHAR(100),
    column_name VARCHAR(100),
    classification VARCHAR(50),
    contains_pii BOOLEAN,
    retention_days INTEGER,
    PRIMARY KEY (table_name, column_name)
);

-- Classify your data
INSERT INTO data_classification VALUES
('clients', 'contact_email', 'PII-Email', true, 2555),      -- 7 years
('clients', 'phone', 'PII-Phone', true, 2555),
('employees', 'email', 'PII-Email', true, 2555),
('employees', 'salary', 'Confidential', false, 2555),
('projects', 'budget', 'Internal', false, 1825);           -- 5 years

-- Query to find all PII columns
SELECT * FROM data_classification WHERE contains_pii = true;
```

### Step 8.2: Implement Data Masking Function

```sql
-- Function to mask sensitive data for non-production use
CREATE OR REPLACE FUNCTION mask_email(email TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN SUBSTRING(email FROM 1 FOR 2) || '****' || SUBSTRING(email FROM POSITION('@' IN email));
END;
$$ LANGUAGE plpgsql;

CREATE OR REPLACE FUNCTION mask_phone(phone TEXT)
RETURNS TEXT AS $$
BEGIN
    RETURN SUBSTRING(phone FROM 1 FOR 3) || '-XXX-XXXX';
END;
$$ LANGUAGE plpgsql;

-- Test masking functions
SELECT
    contact_email,
    mask_email(contact_email) as masked_email,
    phone,
    mask_phone(phone) as masked_phone
FROM clients
LIMIT 3;
```

**📋 Compliance Checklist:**
- [ ] Identified all PII (Personally Identifiable Information)
- [ ] Documented data retention policies
- [ ] Implemented data masking for test environments
- [ ] Enabled audit logging for sensitive tables
- [ ] Created backup and recovery procedures
- [ ] Documented who has access to what data

---

## Part 9: Professional Extras (Claude's Industry Insights)

### Extra 1: Migration Tracking System

```sql
-- Professional migration tracking (used by tools like Flyway, Liquibase)
CREATE TABLE IF NOT EXISTS schema_migrations (
    version VARCHAR(50) PRIMARY KEY,
    description TEXT,
    script_name VARCHAR(255),
    executed_on TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    execution_time_ms INTEGER,
    success BOOLEAN DEFAULT true
);

-- Record your initial schema as version 1.0.0
INSERT INTO schema_migrations (version, description, script_name)
VALUES ('1.0.0', 'Initial schema: clients, employees, projects tables', 'phase1_initial.sql');

-- Future migrations will add entries here
SELECT * FROM schema_migrations ORDER BY executed_on;
```

**Why this matters**: Professional teams never modify production databases directly. Every change goes through versioned migration scripts that can be tested, reviewed, and rolled back if needed.

### Extra 2: Connection Pool Configuration

```sql
-- Create application user with limited privileges
CREATE USER app_user WITH PASSWORD 'secure_password_here';
GRANT CONNECT ON DATABASE arch_firm TO app_user;
GRANT USAGE ON SCHEMA public TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO app_user;

-- Set connection limits to prevent resource exhaustion
ALTER USER app_user CONNECTION LIMIT 20;

-- Create read-only user for reporting
CREATE USER report_user WITH PASSWORD 'another_secure_password';
GRANT CONNECT ON DATABASE arch_firm TO report_user;
GRANT USAGE ON SCHEMA public TO report_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO report_user;
ALTER USER report_user CONNECTION LIMIT 5;
```

**Why this matters**: Using separate users with minimal privileges follows the Principle of Least Privilege (POLP). If the application is compromised, damage is limited.

---

## 📝 Reflection Questions

Please answer these questions in your own words (aim for 2-3 sentences each):

1. **Constraints**: Why did we use `ON DELETE RESTRICT` for the client foreign key instead of `ON DELETE CASCADE`? What are the trade-offs?

2. **Security**: Explain how parameterized queries prevent SQL injection. Why can't the attacker's input break out of the query structure?

3. **Backups**: Your backup from Monday is corrupt, and Tuesday's backup fails. It's now Wednesday. What's your recovery plan? What would you change to prevent this scenario?

4. **Performance**: You notice queries are getting slower as more data is added. What monitoring queries would you run first? What solutions might you consider?

5. **Compliance**: A client requests deletion of all their data under GDPR's "right to be forgotten". How do you balance this with maintaining audit logs and financial records that may be legally required?

6. **Real-world scenario**: The CEO wants to give all employees direct database access "to make reports easier". Write a brief response explaining the security implications and suggesting alternatives.

---

## ✅ Phase 1 Deliverables Checklist

Before moving to Phase 2, ensure you have completed:

### Database Implementation
- [ ] Created `arch_firm` database
- [ ] Implemented all three tables with proper constraints
- [ ] Inserted at least 5 rows of seed data per table
- [ ] Successfully tested constraint violations
- [ ] Created at least one index beyond primary keys

### Queries Demonstrated
- [ ] Basic SELECT with WHERE and ORDER BY
- [ ] At least one JOIN query
- [ ] One aggregation query with GROUP BY
- [ ] UPDATE statement executed
- [ ] DELETE attempted (and understood why it might fail)

### Security & Compliance
- [ ] Created `.env` file for credentials (not in version control!)
- [ ] Understood SQL injection and parameterized queries
- [ ] Implemented audit log trigger
- [ ] Identified PII columns
- [ ] Created data masking functions

### Operations
- [ ] Successfully created a backup
- [ ] Tested restore process
- [ ] Set up backup automation (cron job or script)
- [ ] Created monitoring views
- [ ] Configured psql shortcuts

### Documentation
- [ ] Created README.md with:
  - Database schema diagram or description
  - List of tables and their purposes
  - Instructions to recreate your database
  - Backup and restore commands
  - Key learnings from the workshop

### Reflection
- [ ] Answered all 6 reflection questions
- [ ] Identified at least 3 things that surprised you
- [ ] Listed 2 topics you want to learn more about

---

## 🎉 Congratulations!

You've built your first production-grade database with industry-standard practices! You've implemented:

- **Normalized schema** with referential integrity
- **Security controls** preventing SQL injection
- **Backup strategy** with automation
- **Monitoring** for performance tracking
- **Compliance awareness** for data protection

### Quick Stats Check
Run this to see what you've built:
```sql
SELECT
    'Total Clients' as metric, COUNT(*) as value FROM clients
UNION ALL
SELECT 'Total Projects', COUNT(*) FROM projects
UNION ALL
SELECT 'Total Employees', COUNT(*) FROM employees
UNION ALL
SELECT 'Active Projects', COUNT(*) FROM projects WHERE status = 'Active'
UNION ALL
SELECT 'Total Budget (Active)', SUM(budget) FROM projects WHERE status = 'Active';
```

### What's Next?

Phase 2 will cover:
- Complex relationships with junction tables
- Advanced security with Role-Based Access Control (RBAC)
- Query optimization and EXPLAIN plans
- Stored procedures and functions
- More sophisticated backup strategies

### Final Commands to Remember

```bash
# Connect to your database
psql -U postgres -d arch_firm

# Quick backup
pg_dump -U postgres -d arch_firm -F custom -f quick_backup.backup

# Check database size
psql -U postgres -d arch_firm -c "SELECT pg_size_pretty(pg_database_size('arch_firm'));"
```

---

## 📚 Additional Resources

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [OWASP SQL Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [The Art of PostgreSQL](https://theartofpostgresql.com/) - Excellent book for going deeper
- [pgAdmin](https://www.pgadmin.org/) - GUI tool if you prefer visual management

## 🆘 Troubleshooting

Common issues and solutions:

**"Permission denied" when running psql**
- Solution: Use `sudo -u postgres psql` on Linux or run as administrator on Windows

**"Database does not exist"**
- Solution: Create it first with `CREATE DATABASE arch_firm;`

**Foreign key violation when inserting**
- Solution: Ensure parent record exists first (insert client before project)

**Backup command not found**
- Solution: Ensure PostgreSQL bin directory is in your PATH

---

## Instructor Notes

**Facilitation Tips:**
- Students often struggle with foreign keys initially - demonstrate the error and explain
- Emphasize WHY we use constraints, not just HOW
- For the SQL injection demo, use a whiteboard to show how the string concatenation breaks
- If students finish early, have them create a view joining all three tables

**Common Student Questions:**
1. "Why not use MongoDB?" - Discuss ACID properties and when relational vs NoSQL makes sense
2. "Is SERIAL the same as AUTO_INCREMENT?" - Yes, PostgreSQL's version
3. "Why learn command line when GUIs exist?" - Automation, scripting, and remote server access

**Assessment Guide:**
- Check that constraints are properly defined (not just tables created)
- Verify backup script actually works
- Look for understanding in reflection questions, not just completion
- Bonus points for creative seed data or additional constraints

---

*End of Phase 1 Workshop - Total time: 3-4 hours*

**Remember**: Every expert was once a beginner. If something doesn't work, read the error message carefully - PostgreSQL gives excellent error descriptions!