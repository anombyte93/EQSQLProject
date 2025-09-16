# Architecture Firm Database Project — University Workshop Series

## Project Overview

This four-phase workshop series guides university students through the complete lifecycle of building a production-grade database system for **ArchitectPro**, a fictitious international architecture firm. Students will progressively develop SQL skills while implementing industry-standard practices for security, compliance, backups, and governance. Starting with PostgreSQL fundamentals and culminating in a full-stack Supabase application, participants will experience the real-world challenges and solutions of enterprise database management.

**Target Audience**: University students (beginner to intermediate SQL developers)
**Duration**: 4 workshops (3-4 hours each)
**Teaching Style**: Self-contained, worksheet-based with step-by-step instructions
**Final Deliverable**: Working database-backed web application with industry-grade practices

---

## Global Learning Objectives

### Technical Skills
- Design and normalize relational database schemas following 3NF principles
- Write efficient SQL queries including complex JOINs, aggregations, and CTEs
- Implement database migrations and schema evolution strategies
- Integrate databases with modern web applications using Supabase
- Perform backup, restore, and disaster recovery operations

### Security & Compliance Competencies
- Apply **ISO 27001/27002** information security controls to database systems
- Implement **OWASP Top 10** database security practices
- Design Role-Based Access Control (RBAC) and Row-Level Security (RLS) policies
- Handle PII data according to **GDPR** principles and retention policies
- Practice secure DevOps with secrets management and audit logging

### Professional Development
- Document technical systems for stakeholder communication
- Conduct security assessments and risk analysis
- Follow IEEE/ACM code of ethics for data handling
- Present technical solutions to non-technical audiences
- Work with version control and collaborative development workflows

---

## Industry Standards & Best Practices

### 1. Backup & Recovery Standards
- **RTO/RPO Targets**: Recovery Time Objective < 4 hours, Recovery Point Objective < 1 hour
- **3-2-1 Rule**: 3 copies of data, 2 different media types, 1 offsite location
- **Tools**: `pg_dump`, `pg_restore`, automated cron jobs, Supabase point-in-time recovery
- **Testing**: Monthly restore drills, backup integrity verification

### 2. Security Framework (ISO 27001/OWASP)
- **Access Control**: Principle of least privilege, segregation of duties
- **Encryption**: TLS for transit, AES-256 for storage, key rotation policies
- **SQL Injection Prevention**: Parameterized queries, stored procedures, input validation
- **Monitoring**: Failed login tracking, privilege escalation alerts, query performance logs

### 3. Compliance & Governance
- **GDPR Requirements**:
  - Explicit consent for data collection
  - Right to erasure implementation
  - Data portability exports
  - 72-hour breach notification
- **Audit Requirements**: Immutable audit logs, change tracking, compliance reports
- **Data Classification**: Public, Internal, Confidential, Restricted

### 4. Development Operations
- **Version Control**: Git-based migration history
- **CI/CD Pipeline**: Automated testing, staging deployments
- **Monitoring**: Prometheus/Grafana dashboards, alert thresholds
- **Documentation**: API specs, ERD diagrams, runbooks

### 5. Performance Standards
- **Query Performance**: < 100ms for OLTP, < 5s for reports
- **Indexing Strategy**: B-tree for equality, GiST for ranges, covering indexes
- **Connection Pooling**: PgBouncer configuration, max connections management
- **Caching**: Redis for session data, materialized views for reports

---

## Workshop Design Philosophy — IMPORTANT FOR ALL PHASES

### Self-Sufficient Learning Principles
Each workshop phase MUST be designed for completely autonomous learning:

1. **Explain Before Execute**: Every command must have:
   - WHAT it does (plain English explanation)
   - WHY we're doing it (business/technical justification)
   - HOW it works (technical breakdown)
   - WHAT TO EXPECT (expected output/errors)
   - COMMON MISTAKES (with fixes)

2. **Conceptual Foundation First**: Before any hands-on work:
   - Explain the theory and concepts
   - Use analogies and real-world examples
   - Include diagrams where helpful
   - Define all technical terms on first use

3. **Progressive Disclosure**:
   - Start with simplified explanations
   - Add complexity gradually
   - Reference back to earlier concepts
   - Build mental models step by step

4. **Active Learning Elements**:
   - "Think About It" boxes before revealing answers
   - "Try It Yourself" challenges with hints
   - "What Would Happen If..." scenarios
   - Common misconceptions addressed proactively

5. **Troubleshooting Integration**:
   - Every section includes "If This Doesn't Work" boxes
   - Common error messages explained
   - Multiple approaches to solve problems
   - Verification steps after each major task

## Phase 1: Foundations — Core Schema & CRUD Operations

### Overview
Students establish the fundamental database structure for ArchitectPro, implementing core tables for clients, projects, and employees. Focus on normalization, constraints, and basic query patterns while establishing backup and monitoring practices.

**CRITICAL**: This phase must include extensive explanations of database fundamentals, not just commands to run. Students should understand:
- What a database is and why we need it vs spreadsheets
- How tables relate to each other conceptually
- What each SQL command actually does behind the scenes
- Why each security/backup practice matters with real examples

### Schema Design
```sql
-- Core entities with industry constraints
clients (
    client_id SERIAL PRIMARY KEY,
    company_name VARCHAR(200) NOT NULL,
    contact_email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    address TEXT,
    country VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    CONSTRAINT valid_email CHECK (contact_email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z]{2,}$')
)

projects (
    project_id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(client_id),
    project_name VARCHAR(200) NOT NULL,
    project_type VARCHAR(50) CHECK (project_type IN ('Residential', 'Commercial', 'Industrial', 'Public')),
    budget DECIMAL(12,2) CHECK (budget > 0),
    start_date DATE NOT NULL,
    end_date DATE,
    status VARCHAR(20) DEFAULT 'Planning',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT valid_dates CHECK (end_date IS NULL OR end_date > start_date),
    INDEX idx_client_status (client_id, status)
)

employees (
    employee_id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role VARCHAR(50) NOT NULL,
    department VARCHAR(50),
    hire_date DATE NOT NULL,
    salary DECIMAL(10,2) CHECK (salary > 0),
    is_active BOOLEAN DEFAULT true,
    last_login TIMESTAMP,
    failed_login_attempts INTEGER DEFAULT 0
)
```

### Key Exercises
1. **CRUD Operations**: INSERT 20+ records per table, UPDATE with conditions, DELETE with CASCADE
2. **Basic Queries**: SELECT with WHERE, ORDER BY, LIMIT/OFFSET pagination
3. **Joins**: INNER JOIN for active projects, LEFT JOIN for clients without projects
4. **Aggregations**: COUNT projects by type, AVG budget by client, SUM salaries by department
5. **Backup & Restore Lab**: Create backup script with pg_dump, test pg_restore to new database, automate with cron job
6. **Aliases & Productivity**: Configure \set shortcuts in psql, create .psqlrc file, shell aliases for common commands
7. **Monitoring Basics**: Use pg_stat_activity for current connections, pg_stat_statements for query performance, table size monitoring

### Security Implementation
- Create read-only role for reporting
- Implement connection limits per user
- Enable SSL connections
- Configure pg_hba.conf for network restrictions
- Demonstrate SQL injection prevention with parameterized queries

### Monitoring Setup
```sql
-- Current database activity
SELECT pid, usename, datname, state, query_start,
       SUBSTRING(query, 1, 50) as current_query
FROM pg_stat_activity
WHERE state != 'idle';

-- Query performance baseline (requires pg_stat_statements)
SELECT query, calls, mean_exec_time, max_exec_time
FROM pg_stat_statements
ORDER BY mean_exec_time DESC LIMIT 10;

-- Connection monitoring
SELECT datname, count(*),
       ROUND(100.0 * count(*) / (SELECT setting::int FROM pg_settings WHERE name = 'max_connections'), 2) as percent_used
FROM pg_stat_activity
GROUP BY datname;

-- Table sizes and bloat
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as total_size,
       pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) as table_size,
       pg_size_pretty(pg_indexes_size(schemaname||'.'||tablename)) as indexes_size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

### Compliance Awareness
Students learn to identify and handle sensitive data according to industry standards:
- **PII Identification**: Recognize emails, phone numbers, addresses as personally identifiable information
- **GDPR Principles**: Understand data minimization, purpose limitation, retention limits
- **ISO 27001 Context**: Introduction to information security management systems
- **Data Classification**: Categorize data by sensitivity (Public, Internal, Confidential, Restricted)
- **Audit Requirements**: Why we log data access and changes

### Deliverables
- [ ] Complete schema with constraints and indexes
- [ ] 50+ rows of realistic seed data
- [ ] 10 CRUD queries demonstrating all operations
- [ ] Verified backup/restore procedure with automated scheduling
- [ ] At least one working psql alias or .psqlrc configuration
- [ ] One monitoring query result demonstrating database activity
- [ ] Monitoring queries saved as views for ongoing use
- [ ] Data classification matrix identifying PII and sensitive fields
- [ ] README.md with ERD and setup instructions
- [ ] Reflection paper (500 words) on normalization decisions and compliance considerations

### Phase 2 Preview
Phase 2 builds directly on these foundations with:
- **Advanced Relationships**: Junction tables (project_assignments), many-to-many relationships
- **Role-Based Access Control (RBAC)**: Multiple user roles with granular permissions
- **Complex JOINs**: Multi-table queries, subqueries, CTEs, window functions
- **SQL Injection Prevention**: Hands-on lab with vulnerable vs secure code
- **Query Optimization**: Using EXPLAIN ANALYZE, index strategies, performance tuning
- **Audit Logging**: Complete change tracking with triggers and JSONB storage

### Claude Intuition Extras
- Add UUID columns for external API integration
- Implement soft delete with `deleted_at` timestamps
- Create audit trigger for tracking all changes
- Add full-text search indexes on project descriptions

---

## Phase 2: Relationships & Security — Complex Queries & RBAC

### Overview
Students expand the schema with junction tables and financial data while implementing comprehensive security controls including RBAC, audit logging, and injection prevention techniques.

### Schema Evolution
```sql
project_assignments (
    assignment_id SERIAL PRIMARY KEY,
    project_id INTEGER REFERENCES projects(project_id) ON DELETE CASCADE,
    employee_id INTEGER REFERENCES employees(employee_id),
    role_in_project VARCHAR(100) NOT NULL,
    hours_allocated DECIMAL(6,2) CHECK (hours_allocated > 0),
    hourly_rate DECIMAL(8,2) CHECK (hourly_rate > 0),
    start_date DATE NOT NULL,
    end_date DATE,
    UNIQUE(project_id, employee_id, role_in_project),
    INDEX idx_employee_assignments (employee_id, start_date)
)

invoices (
    invoice_id SERIAL PRIMARY KEY,
    project_id INTEGER REFERENCES projects(project_id),
    invoice_number VARCHAR(50) UNIQUE NOT NULL,
    issue_date DATE NOT NULL DEFAULT CURRENT_DATE,
    due_date DATE NOT NULL,
    amount DECIMAL(12,2) NOT NULL CHECK (amount > 0),
    tax_rate DECIMAL(4,2) DEFAULT 0.10,
    status VARCHAR(20) DEFAULT 'Draft',
    payment_date DATE,
    payment_method VARCHAR(50),
    notes TEXT,
    CHECK (due_date >= issue_date),
    CHECK (payment_date IS NULL OR payment_date >= issue_date)
)

audit_log (
    log_id BIGSERIAL PRIMARY KEY,
    table_name VARCHAR(50) NOT NULL,
    operation VARCHAR(10) NOT NULL,
    user_name VARCHAR(100) NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    row_id INTEGER,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    session_id VARCHAR(100)
) PARTITION BY RANGE (timestamp);
```

### Advanced Query Patterns
1. **Multi-table JOINs**: Employee utilization across projects
2. **Subqueries**: Projects exceeding average budget
3. **CTEs**: Recursive org chart, running totals
4. **Window Functions**: Rank projects by revenue, moving averages
5. **Set Operations**: UNION, EXCEPT, INTERSECT for reporting

### Security Implementation
```sql
-- Role-Based Access Control
CREATE ROLE architects WITH LOGIN;
CREATE ROLE project_managers WITH LOGIN;
CREATE ROLE accountants WITH LOGIN;
CREATE ROLE auditors WITH LOGIN;

-- Granular permissions
GRANT SELECT, INSERT, UPDATE ON projects TO project_managers;
GRANT SELECT ON projects TO architects;
GRANT ALL ON invoices TO accountants;
GRANT SELECT ON audit_log TO auditors;

-- Row-Level Security preview
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
CREATE POLICY project_isolation ON projects
    FOR ALL
    USING (project_id IN (
        SELECT project_id FROM project_assignments
        WHERE employee_id = current_setting('app.current_user_id')::INT
    ));
```

### SQL Injection Prevention Lab
```python
# Vulnerable code example (DO NOT USE)
def bad_query(project_name):
    return f"SELECT * FROM projects WHERE project_name = '{project_name}'"

# Secure parameterized query
def secure_query(project_name):
    return cursor.execute(
        "SELECT * FROM projects WHERE project_name = %s",
        (project_name,)
    )

# Stored procedure approach
CREATE FUNCTION get_project_safely(p_name VARCHAR)
RETURNS TABLE(...) AS $$
BEGIN
    RETURN QUERY
    SELECT * FROM projects WHERE project_name = p_name;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Performance Optimization
```sql
-- Indexing strategy
CREATE INDEX idx_covering_assignments
ON project_assignments(employee_id)
INCLUDE (project_id, hours_allocated);

-- Query optimization
EXPLAIN (ANALYZE, BUFFERS)
SELECT e.*, COUNT(pa.assignment_id), SUM(pa.hours_allocated)
FROM employees e
LEFT JOIN project_assignments pa ON e.employee_id = pa.employee_id
GROUP BY e.employee_id;

-- Materialized view for reports
CREATE MATERIALIZED VIEW project_summary AS
SELECT p.*, COUNT(pa.*) as team_size, SUM(i.amount) as total_invoiced
FROM projects p
LEFT JOIN project_assignments pa ON p.project_id = pa.project_id
LEFT JOIN invoices i ON p.project_id = i.project_id
GROUP BY p.project_id
WITH DATA;

CREATE UNIQUE INDEX ON project_summary(project_id);
```

### Deliverables
- [ ] Extended schema with all constraints
- [ ] 15 complex queries using all JOIN types
- [ ] RBAC implementation with 4+ roles
- [ ] Audit trigger capturing all DML operations
- [ ] SQL injection test cases and prevention code
- [ ] Performance analysis with EXPLAIN plans
- [ ] README.md with security matrix
- [ ] Reflection paper (750 words) on least privilege principle

### Claude Intuition Extras
- Implement data masking functions for PII
- Create suspicious activity detection queries
- Add rate limiting through function calls
- Build automated compliance report generator

---

## Phase 3: Schema Evolution & Cloud Migration — Supabase Integration

### Overview
Students migrate their local PostgreSQL database to Supabase, implementing cloud-native features including Auth, Row-Level Security, Edge Functions, and real-time subscriptions while managing schema migrations professionally.

### Schema Migration & Evolution
```sql
-- Migration 001: Add project phases
CREATE TABLE project_phases (
    phase_id SERIAL PRIMARY KEY,
    project_id INTEGER REFERENCES projects(project_id) ON DELETE CASCADE,
    phase_name VARCHAR(100) NOT NULL,
    phase_order INTEGER NOT NULL,
    start_date DATE,
    end_date DATE,
    status VARCHAR(20) DEFAULT 'Not Started',
    deliverables TEXT[],
    completion_percentage INTEGER DEFAULT 0 CHECK (completion_percentage BETWEEN 0 AND 100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(project_id, phase_order)
);

-- Migration 002: Add payments tracking
CREATE TABLE payments (
    payment_id SERIAL PRIMARY KEY,
    invoice_id INTEGER REFERENCES invoices(invoice_id),
    payment_amount DECIMAL(12,2) NOT NULL CHECK (payment_amount > 0),
    payment_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    payment_method VARCHAR(50) NOT NULL,
    transaction_id VARCHAR(100) UNIQUE,
    status VARCHAR(20) DEFAULT 'Pending',
    gateway_response JSONB,
    refund_amount DECIMAL(12,2) DEFAULT 0,
    notes TEXT
);

-- Migration 003: Add file attachments
CREATE TABLE project_documents (
    document_id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    project_id INTEGER REFERENCES projects(project_id) ON DELETE CASCADE,
    file_name VARCHAR(255) NOT NULL,
    file_size INTEGER NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    storage_path TEXT NOT NULL,
    uploaded_by INTEGER REFERENCES employees(employee_id),
    uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    version INTEGER DEFAULT 1,
    is_latest BOOLEAN DEFAULT true,
    checksum VARCHAR(64)
);
```

### Migration Management
```sql
-- Migrations tracking table
CREATE TABLE schema_migrations (
    version VARCHAR(50) PRIMARY KEY,
    applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    execution_time INTERVAL,
    checksum VARCHAR(64),
    applied_by VARCHAR(100),
    rollback_sql TEXT
);

-- Migration wrapper function
CREATE FUNCTION apply_migration(
    p_version VARCHAR,
    p_up_sql TEXT,
    p_down_sql TEXT
) RETURNS void AS $$
DECLARE
    v_start TIMESTAMP;
BEGIN
    v_start := clock_timestamp();

    -- Check if already applied
    IF EXISTS (SELECT 1 FROM schema_migrations WHERE version = p_version) THEN
        RAISE EXCEPTION 'Migration % already applied', p_version;
    END IF;

    -- Execute migration
    EXECUTE p_up_sql;

    -- Record migration
    INSERT INTO schema_migrations (version, execution_time, rollback_sql, applied_by)
    VALUES (p_version, clock_timestamp() - v_start, p_down_sql, current_user);
END;
$$ LANGUAGE plpgsql;
```

### Supabase Configuration
```javascript
// Initialize Supabase client
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
    auth: {
        persistSession: true,
        autoRefreshToken: true,
        detectSessionInUrl: true
    },
    global: {
        headers: { 'x-application-name': 'ArchitectPro' }
    }
})

// Auth implementation
const { data, error } = await supabase.auth.signUp({
    email: 'user@architectpro.com',
    password: 'SecurePass123!',
    options: {
        data: {
            employee_id: 1,
            role: 'architect'
        }
    }
})
```

### Row-Level Security Policies
```sql
-- Enable RLS on all tables
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE project_assignments ENABLE ROW LEVEL SECURITY;
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;

-- Projects: Users see only their assigned projects
CREATE POLICY "Users see own projects" ON projects
    FOR SELECT
    USING (
        project_id IN (
            SELECT project_id FROM project_assignments
            WHERE employee_id = auth.uid()::INTEGER
        )
        OR
        auth.jwt() ->> 'role' = 'admin'
    );

-- Invoices: Accountants see all, others see own project invoices
CREATE POLICY "Invoice visibility" ON invoices
    FOR SELECT
    USING (
        auth.jwt() ->> 'role' = 'accountant'
        OR
        project_id IN (
            SELECT project_id FROM project_assignments
            WHERE employee_id = auth.uid()::INTEGER
        )
    );

-- Documents: Write requires project membership
CREATE POLICY "Document upload" ON project_documents
    FOR INSERT
    WITH CHECK (
        project_id IN (
            SELECT project_id FROM project_assignments
            WHERE employee_id = auth.uid()::INTEGER
            AND end_date IS NULL
        )
    );
```

### Edge Functions
```typescript
// Supabase Edge Function: Invoice PDF Generation
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from '@supabase/supabase-js'
import PDFDocument from 'pdfkit'

serve(async (req: Request) => {
    const { invoice_id } = await req.json()

    // Create Supabase client with service role
    const supabase = createClient(
        Deno.env.get('SUPABASE_URL')!,
        Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    )

    // Fetch invoice data
    const { data: invoice } = await supabase
        .from('invoices')
        .select(`
            *,
            project:projects(
                project_name,
                client:clients(company_name, address)
            )
        `)
        .eq('invoice_id', invoice_id)
        .single()

    // Generate PDF
    const doc = new PDFDocument()
    doc.text(`Invoice #${invoice.invoice_number}`)
    // ... PDF generation logic

    return new Response(doc, {
        headers: { 'Content-Type': 'application/pdf' }
    })
})
```

### Real-time Subscriptions
```javascript
// Subscribe to project updates
const projectSubscription = supabase
    .channel('project-changes')
    .on('postgres_changes',
        { event: '*', schema: 'public', table: 'projects' },
        (payload) => {
            console.log('Project changed:', payload)
            updateUIWithNewData(payload.new)
        }
    )
    .subscribe()

// Subscribe to new assignments
const assignmentChannel = supabase
    .channel('my-assignments')
    .on('postgres_changes',
        {
            event: 'INSERT',
            schema: 'public',
            table: 'project_assignments',
            filter: `employee_id=eq.${currentUser.id}`
        },
        (payload) => notifyNewAssignment(payload.new)
    )
    .subscribe()
```

### Deliverables
- [ ] Complete schema migration to Supabase
- [ ] Migration scripts with rollback procedures
- [ ] RLS policies for all tables
- [ ] Supabase Auth integration with JWT
- [ ] Edge Function for report generation
- [ ] Real-time subscription demo
- [ ] Storage bucket for project files
- [ ] README.md with Supabase setup guide
- [ ] Reflection paper (1000 words) on cloud vs on-premise security

### Claude Intuition Extras
- Implement Supabase Vault for API keys
- Create database functions for complex business logic
- Add webhook integration for external services
- Build automated backup to external S3

---

## Phase 4: Capstone — Full-Stack Integration & Production Deployment

### Overview
Students build a complete web application integrating their Supabase backend with a modern frontend framework, implementing production-grade features including monitoring, cron jobs, disaster recovery, and compliance reporting.

### Frontend Application Structure
```typescript
// Next.js 14 App Structure
/app
  /api
    /webhooks
      stripe.ts        // Payment processing
      github.ts        // CI/CD triggers
  /(auth)
    /login
    /register
    /reset-password
  /(dashboard)
    /projects
      page.tsx         // Project list
      [id]/page.tsx    // Project details
    /invoices
    /reports
    /settings
  /layout.tsx
  /middleware.ts       // Auth middleware

// Key Components
interface Project {
    project_id: number
    project_name: string
    client: Client
    status: 'Planning' | 'Active' | 'Completed'
    budget: number
    phases: ProjectPhase[]
    team: ProjectAssignment[]
    documents: ProjectDocument[]
}

// Project Dashboard Component
export default function ProjectDashboard() {
    const [projects, setProjects] = useState<Project[]>([])
    const [filter, setFilter] = useState({ status: 'Active' })

    useEffect(() => {
        fetchProjects()
        subscribeToUpdates()
    }, [filter])

    const fetchProjects = async () => {
        const { data, error } = await supabase
            .from('projects')
            .select(`
                *,
                client:clients(*),
                phases:project_phases(*),
                team:project_assignments(
                    employee:employees(*)
                )
            `)
            .eq('status', filter.status)
            .order('start_date', { ascending: false })

        if (data) setProjects(data)
    }

    return (
        <DashboardLayout>
            <ProjectFilters onFilterChange={setFilter} />
            <ProjectGrid projects={projects} />
            <ProjectMetrics projects={projects} />
        </DashboardLayout>
    )
}
```

### Scheduled Jobs & Automation
```sql
-- Supabase Cron Jobs via pg_cron extension
SELECT cron.schedule(
    'invoice-reminders',
    '0 9 * * 1', -- Every Monday at 9 AM
    $$
    INSERT INTO notifications (recipient_id, type, message, related_id)
    SELECT
        c.primary_contact_id,
        'invoice_reminder',
        'Invoice #' || i.invoice_number || ' is due in 3 days',
        i.invoice_id
    FROM invoices i
    JOIN projects p ON i.project_id = p.project_id
    JOIN clients c ON p.client_id = c.client_id
    WHERE i.status = 'Sent'
    AND i.due_date = CURRENT_DATE + INTERVAL '3 days';
    $$
);

-- Automated report generation
SELECT cron.schedule(
    'monthly-reports',
    '0 0 1 * *', -- First day of month at midnight
    $$
    SELECT generate_monthly_report(
        EXTRACT(MONTH FROM CURRENT_DATE - INTERVAL '1 month')::INT,
        EXTRACT(YEAR FROM CURRENT_DATE - INTERVAL '1 month')::INT
    );
    $$
);

-- Database maintenance
SELECT cron.schedule(
    'vacuum-analyze',
    '0 2 * * 0', -- Sunday at 2 AM
    'VACUUM ANALYZE;'
);
```

### Monitoring & Alerting
```typescript
// Performance monitoring with Sentry
import * as Sentry from "@sentry/nextjs"

Sentry.init({
    dsn: process.env.SENTRY_DSN,
    tracesSampleRate: 0.1,
    environment: process.env.NODE_ENV,
    integrations: [
        new Sentry.Integrations.Postgres(),
        new Sentry.Integrations.Http({ tracing: true })
    ]
})

// Custom metrics collection
export async function trackDatabaseMetrics() {
    const metrics = await supabase.rpc('get_database_metrics')

    // Send to monitoring service
    await fetch('/api/metrics', {
        method: 'POST',
        body: JSON.stringify({
            timestamp: new Date(),
            connections: metrics.active_connections,
            slow_queries: metrics.slow_query_count,
            cache_hit_ratio: metrics.cache_hit_ratio,
            table_sizes: metrics.table_sizes,
            replication_lag: metrics.replication_lag
        })
    })
}

// Health check endpoint
export async function GET() {
    const checks = {
        database: await checkDatabase(),
        storage: await checkStorage(),
        auth: await checkAuth(),
        cache: await checkCache()
    }

    const healthy = Object.values(checks).every(c => c.status === 'ok')

    return NextResponse.json({
        status: healthy ? 'healthy' : 'degraded',
        checks,
        timestamp: new Date().toISOString()
    }, { status: healthy ? 200 : 503 })
}
```

### Disaster Recovery Plan
```yaml
# disaster-recovery-plan.yaml
recovery_objectives:
  rpo: 1 hour          # Maximum data loss
  rto: 4 hours         # Maximum downtime
  mttr: 2 hours        # Mean time to repair

backup_strategy:
  automated:
    - type: continuous
      destination: supabase_pitr
      retention: 7 days
    - type: daily
      destination: aws_s3
      retention: 30 days
    - type: weekly
      destination: azure_blob
      retention: 90 days

recovery_procedures:
  data_corruption:
    1. Identify corruption timestamp
    2. Stop application traffic
    3. Restore from PITR to new instance
    4. Validate data integrity
    5. Update DNS to new instance
    6. Resume application traffic

  regional_outage:
    1. Activate DR site
    2. Restore from latest backup
    3. Update DNS records
    4. Notify stakeholders
    5. Begin replication catch-up

testing_schedule:
  - type: backup_restoration
    frequency: monthly
    owner: dba_team
  - type: failover_drill
    frequency: quarterly
    owner: devops_team
  - type: full_dr_simulation
    frequency: annually
    owner: cto_office
```

### Compliance & Governance
```typescript
// GDPR Compliance Implementation
export class GDPRCompliance {
    // Right to access
    async exportUserData(userId: string) {
        const tables = ['employees', 'audit_log', 'project_assignments']
        const userData = {}

        for (const table of tables) {
            const { data } = await supabase
                .from(table)
                .select('*')
                .eq('employee_id', userId)

            userData[table] = data
        }

        return {
            exported_at: new Date(),
            user_id: userId,
            data: userData,
            format: 'json'
        }
    }

    // Right to erasure
    async deleteUserData(userId: string, confirmation: string) {
        if (confirmation !== `DELETE-${userId}`) {
            throw new Error('Invalid confirmation')
        }

        // Anonymize instead of delete for audit trail
        await supabase.rpc('anonymize_user_data', { user_id: userId })

        // Log the action
        await this.logCompliance('GDPR_ERASURE', userId)
    }

    // Consent management
    async updateConsent(userId: string, consents: ConsentUpdate[]) {
        return supabase.from('user_consents').upsert(
            consents.map(c => ({
                user_id: userId,
                consent_type: c.type,
                granted: c.granted,
                timestamp: new Date(),
                ip_address: c.ipAddress
            }))
        )
    }
}

// PCI Compliance for payment processing
export class PCICompliance {
    // Never store full card numbers
    maskCardNumber(cardNumber: string): string {
        return `****-****-****-${cardNumber.slice(-4)}`
    }

    // Tokenize sensitive payment data
    async tokenizePayment(paymentData: PaymentData) {
        const token = await stripeClient.tokens.create({
            card: {
                number: paymentData.cardNumber,
                exp_month: paymentData.expMonth,
                exp_year: paymentData.expYear,
                cvc: paymentData.cvc
            }
        })

        // Store only token reference
        return supabase.from('payment_tokens').insert({
            token_id: token.id,
            last_four: paymentData.cardNumber.slice(-4),
            expires_at: new Date(Date.now() + 3600000) // 1 hour
        })
    }
}
```

### Production Deployment Checklist
```markdown
## Pre-Deployment
- [ ] All tests passing (unit, integration, e2e)
- [ ] Security scan completed (OWASP ZAP, Snyk)
- [ ] Performance testing done (load, stress)
- [ ] Code review approved by 2+ developers
- [ ] Documentation updated
- [ ] Rollback plan documented

## Database
- [ ] Migrations tested on staging
- [ ] Indexes optimized
- [ ] Connection pooling configured
- [ ] RLS policies verified
- [ ] Backup job scheduled

## Application
- [ ] Environment variables secured
- [ ] Error tracking configured (Sentry)
- [ ] Logging aggregation setup (Datadog/ELK)
- [ ] Rate limiting enabled
- [ ] CORS properly configured

## Infrastructure
- [ ] SSL certificates valid
- [ ] CDN configured for static assets
- [ ] Auto-scaling policies set
- [ ] Monitoring alerts configured
- [ ] DDoS protection enabled

## Compliance
- [ ] Privacy policy updated
- [ ] Terms of service current
- [ ] Cookie consent implemented
- [ ] Data retention automated
- [ ] Audit logging verified
```

### Final Deliverables
- [ ] Full-stack application deployed to production
- [ ] Complete test suite with 80%+ coverage
- [ ] API documentation (OpenAPI/Swagger)
- [ ] User manual and admin guide
- [ ] Security assessment report
- [ ] Performance benchmarks
- [ ] Disaster recovery runbook
- [ ] 10-minute presentation video
- [ ] Reflection paper (1500 words) on lessons learned

---

## Capstone Project Rubric

### Technical Implementation (40%)
| Criteria | Excellent (90-100%) | Good (70-89%) | Satisfactory (50-69%) | Needs Improvement (<50%) |
|----------|-------------------|---------------|----------------------|-------------------------|
| **Schema Design** | Fully normalized, optimized indexes, proper constraints | Minor normalization issues, most indexes present | Some denormalization, basic indexes | Poor structure, missing constraints |
| **Query Complexity** | Advanced CTEs, window functions, optimized | Good use of JOINs and subqueries | Basic queries work correctly | Simple SELECTs only |
| **Frontend Integration** | Smooth UX, real-time updates, responsive | Functional UI, some delays | Basic CRUD interface | Minimal functionality |
| **API Design** | RESTful/GraphQL, versioned, documented | Clear endpoints, some docs | Working but unclear | Poorly structured |

### Security & Compliance (30%)
| Criteria | Excellent (90-100%) | Good (70-89%) | Satisfactory (50-69%) | Needs Improvement (<50%) |
|----------|-------------------|---------------|----------------------|-------------------------|
| **Authentication** | MFA, JWT, session management | Secure auth, tokens | Basic auth working | Weak/no authentication |
| **Authorization** | Granular RLS, RBAC implemented | Good permission system | Basic role checking | No authorization |
| **Data Protection** | Encryption, masking, secure storage | Most data protected | Some security measures | Plaintext sensitive data |
| **Compliance** | GDPR/PCI ready, audit complete | Good compliance effort | Basic compliance | No compliance consideration |

### Operations & Reliability (20%)
| Criteria | Excellent (90-100%) | Good (70-89%) | Satisfactory (50-69%) | Needs Improvement (<50%) |
|----------|-------------------|---------------|----------------------|-------------------------|
| **Backup/Recovery** | Automated, tested, documented | Regular backups, some testing | Manual backups | No backup strategy |
| **Monitoring** | Comprehensive metrics, alerts | Good monitoring coverage | Basic health checks | No monitoring |
| **Performance** | Optimized, <100ms response | Good performance | Acceptable speed | Slow/timeouts |
| **Documentation** | Complete runbooks, API docs | Good documentation | Basic README | Minimal documentation |

### Professional Skills (10%)
| Criteria | Excellent (90-100%) | Good (70-89%) | Satisfactory (50-69%) | Needs Improvement (<50%) |
|----------|-------------------|---------------|----------------------|-------------------------|
| **Code Quality** | Clean, maintainable, tested | Good structure, some tests | Working code | Messy/buggy code |
| **Presentation** | Clear, engaging, demonstrates mastery | Good explanation | Basic presentation | Poor communication |
| **Innovation** | Creative solutions, extra features | Some creativity | Meets requirements | Below requirements |
| **Teamwork** | Excellent collaboration (if applicable) | Good teamwork | Adequate cooperation | Poor collaboration |

---

## Claude Intuition Hooks

### Phase 1 Enhancements
When students complete Phase 1, Claude should consider adding:
- **Temporal tables** for audit history: `CREATE TABLE clients_history (LIKE clients)` with triggers
- **Database statistics collection**: Custom function to track query patterns
- **Backup verification script**: Automated restore test with data validation

### Phase 2 Enhancements
After Phase 2 security implementation, Claude should suggest:
- **Honeypot tables** to detect intrusions: Fake `credit_cards` table with alerts
- **Query performance baseline**: Stored procedure to capture typical execution times
- **Compliance checklist generator**: Automated report comparing schema to standards

### Phase 3 Enhancements
During Supabase migration, Claude should introduce:
- **Blue-green deployment** setup: Parallel environments for zero-downtime updates
- **Synthetic monitoring**: Edge function that runs test transactions
- **Cost optimization**: Query to identify unused indexes and bloated tables

### Phase 4 Enhancements
For the capstone, Claude should recommend:
- **Chaos engineering tests**: Randomly fail 10% of queries to test resilience
- **A/B testing framework**: Database support for feature flags
- **ML-ready data pipeline**: Materialized views formatted for analytics

### Industry Context Explanations
Claude should always explain WHY these practices matter:
- "We implement RLS because 60% of data breaches involve privilege misuse"
- "Backup testing matters because 75% of organizations find backup issues only during recovery"
- "Migration rollbacks are critical as 20% of schema changes cause production issues"

---

## Resources & References

### Documentation
- [PostgreSQL Official Docs](https://www.postgresql.org/docs/)
- [Supabase Documentation](https://supabase.com/docs)
- [OWASP Database Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Database_Security_Cheat_Sheet.html)
- [ISO 27001 Controls](https://www.iso.org/isoiec-27001-information-security.html)

### Tools
- **Development**: pgAdmin, DBeaver, VS Code with SQL extensions
- **Testing**: pgTAP, Jest for API testing, Cypress for E2E
- **Monitoring**: pgBadger, pg_stat_statements, Grafana
- **Security**: SQLMap (for testing), Vault for secrets

### Sample Datasets
- Realistic client data: Fortune 500 companies
- Project types: Based on real architectural categories
- Employee roles: Industry-standard positions
- Financial data: Typical project budgets and invoice amounts

### Support & Community
- Workshop Discord channel for peer support
- Weekly office hours with instructors
- Stack Overflow tag: #architectpro-workshop
- GitHub repository for code sharing

---

## Appendix: Common Pitfalls & Solutions

### Pitfall 1: N+1 Query Problem
**Problem**: Fetching related data in loops
**Solution**: Use JOINs or batch fetching
```sql
-- Bad: N+1 queries
SELECT * FROM projects;
-- Then for each project:
SELECT * FROM project_assignments WHERE project_id = ?;

-- Good: Single query
SELECT p.*, pa.*
FROM projects p
LEFT JOIN project_assignments pa ON p.project_id = pa.project_id;
```

### Pitfall 2: Missing Indexes on Foreign Keys
**Problem**: Slow JOINs and CASCADE operations
**Solution**: Always index foreign key columns
```sql
CREATE INDEX idx_fk_project_client ON projects(client_id);
CREATE INDEX idx_fk_assignment_project ON project_assignments(project_id);
```

### Pitfall 3: Storing Sensitive Data in Logs
**Problem**: Passwords, credit cards in audit logs
**Solution**: Redact sensitive fields
```sql
CREATE FUNCTION redact_sensitive(data JSONB) RETURNS JSONB AS $$
BEGIN
    RETURN data - ARRAY['password', 'credit_card', 'ssn', 'api_key'];
END;
$$ LANGUAGE plpgsql;
```

### Pitfall 4: Ignoring Time Zones
**Problem**: Incorrect timestamp handling
**Solution**: Always use TIMESTAMP WITH TIME ZONE
```sql
-- Bad
created_at TIMESTAMP

-- Good
created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
```

### Pitfall 5: Unlimited Result Sets
**Problem**: Memory exhaustion, slow queries
**Solution**: Always paginate
```sql
-- Add default limits
CREATE FUNCTION get_projects(p_limit INT DEFAULT 100, p_offset INT DEFAULT 0)
RETURNS SETOF projects AS $$
BEGIN
    RETURN QUERY
    SELECT * FROM projects
    ORDER BY created_at DESC
    LIMIT p_limit OFFSET p_offset;
END;
$$ LANGUAGE plpgsql;
```

---

## Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2024-01-15 | Initial workshop framework | Workshop Team |
| 1.1 | 2024-02-01 | Added Supabase integration | Cloud Team |
| 1.2 | 2024-02-15 | Enhanced security requirements | Security Team |
| 1.3 | 2024-03-01 | Added compliance frameworks | Governance Team |
| 2.0 | 2024-03-15 | Complete rewrite for Phase 4 | Full Team |

---

*This document serves as the authoritative reference for the Architecture Firm Database Project workshop series. It should be consulted for all phase implementations, grading decisions, and technical specifications. Updates require approval from the workshop committee.*