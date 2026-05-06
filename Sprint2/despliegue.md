
3. Crear el script de despliegue de estructuras para el motor de Base de Datos.

# SEGURIDAD

```sql

CREATE TABLE app_user (
    id                    UUID          PRIMARY KEY DEFAULT
    username              VARCHAR(50)   NOT NULL UNIQUE,
    email                 VARCHAR(255)  NOT NULL UNIQUE,
    password_hash         TEXT          NOT NULL,
    role                  VARCHAR(30)   NOT NULL,
    enabled               BOOLEAN       NOT NULL DEFAULT TRUE,
    account_locked        BOOLEAN       NOT NULL DEFAULT FALSE,
    failed_login_attempts INT           NOT NULL DEFAULT 0,
    password_changed_at   TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_at            TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_by            VARCHAR(100)  NOT NULL,
    updated_at            TIMESTAMPTZ,
    updated_by            VARCHAR(100)
);

CREATE TABLE role_permission (
    id         UUID        PRIMARY KEY DEFAULT 
    role       VARCHAR(30) NOT NULL,
    resource   VARCHAR(50) NOT NULL,
    action     VARCHAR(30) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    UNIQUE (role, resource, action)
);

CREATE TABLE token_blacklist (
    id             UUID         PRIMARY KEY DEFAULT
    jti            VARCHAR(255) NOT NULL UNIQUE,
    user_id        UUID         NOT NULL REFERENCES app_user(id) ON DELETE CASCADE,
    expires_at     TIMESTAMPTZ  NOT NULL,
    blacklisted_at TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    reason         VARCHAR(100)

CREATE TABLE audit_log (
    id          UUID         PRIMARY KEY DEFAULT 
    created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    entity_type VARCHAR(50)  NOT NULL,
    entity_id   UUID,
    action      VARCHAR(30)  NOT NULL,
    actor       VARCHAR(100) NOT NULL,
    actor_ip    VARCHAR(45),
    result      VARCHAR(20),
    data_before JSONB,
    data_after  JSONB,
    details     VARCHAR(1000)
);

```

# SOLICITANTE

```sql

CREATE TABLE applicant (
    id                       UUID          PRIMARY KEY DEFAULT,
    name                     VARCHAR(150)  NOT NULL,
    identification_encrypted VARCHAR(700)  NOT NULL,
    identification_hash      VARCHAR(128)  NOT NULL UNIQUE,
    birth_date               DATE          NOT NULL,
    employment_type          VARCHAR(30)   NOT NULL,
    monthly_income           NUMERIC(19,2) NOT NULL,
    work_experience_months   INT           NOT NULL,
    phone                    VARCHAR(20),
    address                  VARCHAR(500),
    email                    VARCHAR(255),
    created_at               TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_by               VARCHAR(100)  NOT NULL,
    updated_at               TIMESTAMPTZ,
    updated_by               VARCHAR(100)
);
 
CREATE TABLE applicant_edit_audit (
    id           UUID         PRIMARY KEY DEFAULT,
    applicant_id UUID         NOT NULL REFERENCES applicant(id) ON DELETE CASCADE,
    field_name   VARCHAR(100) NOT NULL,
    old_value    VARCHAR(500),
    new_value    VARCHAR(500),
    changed_at   TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    changed_by   VARCHAR(100) NOT NULL
);
```

# DATOS FINANCIEROS


```sql
CREATE TABLE financial_data (
    id                       UUID          PRIMARY KEY DEFAULT 
    applicant_id             UUID          NOT NULL REFERENCES applicant(id) ON DELETE CASCADE,
    version                  INT           NOT NULL DEFAULT 1,
    annual_income            NUMERIC(19,2) NOT NULL,
    monthly_expenses         NUMERIC(19,2) NOT NULL,
    current_debts            NUMERIC(19,2) NOT NULL,
    assets_value             NUMERIC(19,2) NOT NULL,
    declared_patrimony       NUMERIC(19,2) NOT NULL,
    has_outstanding_defaults BOOLEAN       NOT NULL DEFAULT FALSE,
    credit_history_months    INT           NOT NULL DEFAULT 0,
    defaults_last_12m        INT           NOT NULL DEFAULT 0,
    defaults_last_24m        INT           NOT NULL DEFAULT 0,
    external_bureau_score    INT,
    active_credit_products   INT           NOT NULL DEFAULT 0,
    created_at               TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_by               VARCHAR(100)  NOT NULL,
    updated_at               TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    updated_by               VARCHAR(100),
    UNIQUE (applicant_id, version)
);


```
# VARIABLES DE SCORING

 ```sql

CREATE TABLE scoring_variable (
    id            UUID         PRIMARY KEY DEFAULT 
    name          VARCHAR(100) NOT NULL UNIQUE,
    description   TEXT,
    variable_type VARCHAR(30)  NOT NULL,
    peso          NUMERIC(5,4),
    enabled       BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by    VARCHAR(100) NOT NULL,
    updated_at    TIMESTAMPTZ,
    updated_by    VARCHAR(100)
);
 
CREATE TABLE variable_range (
    id          UUID          PRIMARY KEY DEFAULT 
    variable_id UUID          NOT NULL REFERENCES scoring_variable(id) ON DELETE CASCADE,
    min_value   NUMERIC(19,4) NOT NULL,
    max_value   NUMERIC(19,4) NOT NULL,
    score       INT           NOT NULL,
    label       VARCHAR(100),
    created_at  TIMESTAMPTZ   NOT NULL DEFAULT NOW()
);
 
CREATE TABLE variable_category (
    id             UUID         PRIMARY KEY DEFAULT,
    variable_id    UUID         NOT NULL REFERENCES scoring_variable(id) ON DELETE CASCADE,
    category_value VARCHAR(100) NOT NULL,
    score          INT          NOT NULL,
    label          VARCHAR(100),
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    UNIQUE (variable_id, category_value)
);

```
# MODELO DE SCORING
 
```sql


CREATE TABLE scoring_model (
    id            UUID         PRIMARY KEY DEFAULT,
    name          VARCHAR(150) NOT NULL,
    description   TEXT,
    version       INT          NOT NULL DEFAULT 1,
    status        VARCHAR(20)  NOT NULL DEFAULT 'DRAFT',
    min_score     NUMERIC(5,2) NOT NULL DEFAULT 0,
    max_score     NUMERIC(5,2) NOT NULL DEFAULT 100,
    activated_at  TIMESTAMPTZ,
    deprecated_at TIMESTAMPTZ,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by    VARCHAR(100) NOT NULL,
    updated_at    TIMESTAMPTZ,
    updated_by    VARCHAR(100)
);
 
CREATE TABLE model_variable (
    id              UUID         PRIMARY KEY DEFAULT,
    model_id        UUID         NOT NULL REFERENCES scoring_model(id) ON DELETE CASCADE,
    variable_id     UUID         NOT NULL REFERENCES scoring_variable(id),
    weight          NUMERIC(5,4) NOT NULL,
    ranges_snapshot JSONB,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by      VARCHAR(100) NOT NULL,
    updated_at      TIMESTAMPTZ,
    updated_by      VARCHAR(100),
    UNIQUE (model_id, variable_id)
);
 
CREATE TABLE knockout_rule (
    id              UUID         PRIMARY KEY DEFAULT,
    model_id        UUID         REFERENCES scoring_model(id) ON DELETE SET NULL,
    name            VARCHAR(100) NOT NULL,
    field           VARCHAR(100) NOT NULL,
    operator        VARCHAR(20)  NOT NULL,
    threshold_value VARCHAR(255) NOT NULL,
    description     TEXT,
    priority        INT          NOT NULL DEFAULT 1,
    enabled         BOOLEAN      NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by      VARCHAR(100) NOT NULL,
    updated_at      TIMESTAMPTZ,
    updated_by      VARCHAR(100)
);
 

```

# EVALUACION

```sql

CREATE TABLE evaluation (
    id                UUID         PRIMARY KEY DEFAULT,
    applicant_id      UUID         NOT NULL REFERENCES applicant(id),
    model_id          UUID         NOT NULL REFERENCES scoring_model(id),
    financial_data_id UUID         NOT NULL REFERENCES financial_data(id),
    total_score       NUMERIC(5,2) NOT NULL,
    risk_level        VARCHAR(20)  NOT NULL,
    knocked_out       BOOLEAN      NOT NULL DEFAULT FALSE,
    knockout_reasons  VARCHAR(1000),
    evaluated_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    evaluated_by      VARCHAR(100) NOT NULL,
    created_at        TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by        VARCHAR(100) NOT NULL
);
 
CREATE TABLE evaluation_detail (
    id             UUID         PRIMARY KEY DEFAULT,
    evaluation_id  UUID         NOT NULL REFERENCES evaluation(id) ON DELETE CASCADE,
    variable_id    UUID         NOT NULL REFERENCES scoring_variable(id),
    variable_name  VARCHAR(100) NOT NULL,
    raw_value      VARCHAR(255) NOT NULL,
    score          NUMERIC(5,2) NOT NULL,
    weight         NUMERIC(5,4) NOT NULL,
    weighted_score NUMERIC(5,2) NOT NULL,
    created_at     TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
 
CREATE TABLE evaluation_knockout (
    id            UUID         PRIMARY KEY DEFAULT,
    evaluation_id UUID         NOT NULL REFERENCES evaluation(id) ON DELETE CASCADE,
    rule_id       UUID         NOT NULL REFERENCES knockout_rule(id),
    rule_name     VARCHAR(100) NOT NULL,
    field_value   VARCHAR(255) NOT NULL,
    triggered     BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
 ```

# DECISION CREDITICIA

```sql

CREATE TABLE credit_decision (
    id                     UUID          PRIMARY KEY DEFAULT,
    evaluation_id          UUID          NOT NULL UNIQUE REFERENCES evaluation(id),
    decision               VARCHAR(20)   NOT NULL,
    observations           VARCHAR(2000) NOT NULL,
    decided_by             VARCHAR(100)  NOT NULL,
    decided_at             TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_at             TIMESTAMPTZ   NOT NULL DEFAULT NOW(),
    created_by             VARCHAR(100)  NOT NULL,
    supervisor_id          VARCHAR(100),
    resolution_deadline_at TIMESTAMPTZ
);

``` 
 # SIMULACION

```sql
CREATE TABLE simulation_scenario (
    id              UUID         PRIMARY KEY DEFAULT,
    model_id        UUID         NOT NULL REFERENCES scoring_model(id) ON DELETE CASCADE,
    name            VARCHAR(150) NOT NULL,
    description     TEXT,
    values_snapshot JSONB        NOT NULL,
    created_at      TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    created_by      VARCHAR(100) NOT NULL
);

```
