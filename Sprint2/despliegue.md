
3. Crear el script de despliegue de estructuras para el motor de Base de Datos.

SEGURIDAD

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
