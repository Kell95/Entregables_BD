
# 4.Crear las consultas identificadas o acciones del módulo para las HU correspondientes al Sprint 1 y 2 vs modelo físico.


**SPRINT 1**
Base del sistema

**HU-018 — Autenticación de usuarios**

Tablas: app_user, authentication_log, token_blacklist, audit_log

 Consultas:
 
- Buscar un usuario
  
```sql

SELECT id, username, email, password_hash, role, enabled, account_locked
FROM app_user
WHERE username = 'analista1'
  AND enabled = true;
```

 - Registrar un intento de login exitoso
 
```sql
INSERT INTO authentication_log (id, user_id, username, action, ip_address, created_at)
VALUES (
  '11111111-1111-1111-1111-111111111111',
  'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa',
  'analista01',
  'LOGIN_SUCCESS',
  '192.168.1.10',
  NOW()
);

```
- Login fallido
  
```sql
INSERT INTO authentication_log (id, user_id, username, action, ip_address, details, created_at)
VALUES (
  '22222222-2222-2222-2222-222222222222',
  NULL,
  'analista01',
  'LOGIN_FAILURE',
  '192.168.1.10',
  'Contraseña incorrecta',
  NOW()
);
```
- intentos fallidos
  
```sql
UPDATE app_user
SET failed_login_attempts = failed_login_attempts + 1,
    updated_at = NOW()
WHERE username = 'analista01';
```
- Bloquear usuario

```sql
UPDATE app_user
SET account_locked = true,
    lock_time = NOW(),
    updated_at = NOW()
WHERE username = 'analista01'
  AND failed_login_attempts >= 5;
```
- Reset login
  
```sql
UPDATE app_user
SET failed_login_attempts = 0,
    account_locked = false,
    lock_time = NULL,
    last_login_at = NOW(),
    updated_at = NOW()
WHERE id = 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa';
```
- Agregar token blacklist

```sql
INSERT INTO token_blacklist (id, jti, user_id, expires_at, blacklisted_at, reason, created_by)
VALUES (
  '33333333-3333-3333-3333-333333333333',
  'jwt-token-123456',
  'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa',
  '2026-05-07 10:00:00',
  NOW(),
  'USER_LOGOUT'
  'analista01'
);
```

- Verificar token bloqueado
  
```sql
SELECT COUNT(*) AS esta_bloqueado
FROM token_blacklist
WHERE jti = 'jwt-token-123456'
  AND expires_at > NOW();
```
**HU-002 ROLES**

Tablas:app_user, role_permission

- Permisos
  
```sql
SELECT resource, action
FROM role_permission
WHERE role = 'ANALYST';
```

- Validar permiso

```sql
SELECT COUNT(*) AS tiene_permiso
FROM role_permission
WHERE role = 'ANALYST'
  AND resource = 'APPLICANT'
  AND action = 'CREATE';
```
-Crear usuario

```sql
INSERT INTO app_user (id, username, email, password_hash, role, enabled, password_changed_at, created_at, created_by)
VALUES (
  'bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb',
  'analista02',
  'analista02@crediscan.com',
  '$2b$10$hashseguroejemplo',
  'ANALYST',
  true,
  NOW(),
  NOW(),
  'admin'
);
```
**HU-003 SOLICITANTE**

Tablas: applicant

- Ver duplicado
  
```sql
SELECT COUNT(*) AS ya_existe
FROM applicant
WHERE identification_hash = 'hash123456789';
```
- Crear solicitante
  
```sql
INSERT INTO applicant (
  id, name, identification_encrypted, identification_hash,
  birth_date, employment_type, monthly_income,
  work_experience_months, phone, address, email,
  created_at, created_by
)
VALUES (
  'cccccccc-cccc-cccc-cccc-cccccccccccc',
  'Carlos Perez',
  'dato_encriptado_123',
  'hash123456789',
  '1990-05-15',
  'EMPLOYEE',
  3500000,
  48,
  '3001234567',
  'Medellin',
  'carlos@email.com',
  NOW(),
  'analista01'
);
```
**HU-004 EDICIÓN**

Tablas: applicant, applicant_edit_audit


- Actualizar

```sql
UPDATE applicant
SET employment_type = 'INDEPENDENT',
    monthly_income = 4200000,
    phone = '3009876543',
    updated_at = NOW(),
    updated_by = 'analista01'
WHERE id = 'cccccccc-cccc-cccc-cccc-cccccccccccc';
```

-Auditoría cambio

```sql
INSERT INTO applicant_edit_audit (
  id, applicant_id, field_name, old_value, new_value, changed_at, changed_by
)
VALUES (
  'dddddddd-dddd-dddd-dddd-dddddddddddd',
  'cccccccc-cccc-cccc-cccc-cccccccccccc',
  'monthly_income',
  '3500000',
  '4200000',
  NOW(),
  'analista01'
);

```
**HU-005 FINANCIERO**

Tablas: financial_data

-Insertar datos

```sql
INSERT INTO financial_data (
  id, applicant_id, version,
  annual_income, monthly_expenses, current_debts,
  assets_value, declared_patrimony,
  has_outstanding_defaults,
  credit_history_months,
  defaults_last_12m,
  defaults_last_24m,
  external_bureau_score,
  active_credit_products,
  created_at, created_by
)
VALUES (
  'eeeeeeee-eeee-eeee-eeee-eeeeeeeeeeee',
  'cccccccc-cccc-cccc-cccc-cccccccccccc',
  1,
  42000000,
  1800000,
  5000000,
  80000000,
  75000000,
  false,
  36,
  0,
  1,
  720,
  2,
  NOW(),
  'analista01'

);
```

# Sprint 2

**HU-012 EVALUACIÓN**

Tablas:evaluation, evaluation_detail

- Guardar evaluación

```sql
INSERT INTO evaluation (
  id, applicant_id, model_id, financial_data_id,
  total_score, risk_level, knocked_out,
  evaluated_at, evaluated_by, created_at, created_by
)
VALUES (
  'ffffffff-ffff-ffff-ffff-ffffffffffff',
  'cccccccc-cccc-cccc-cccc-cccccccccccc',
  'model-001',
  'eeeeeeee-eeee-eeee-eeee-eeeeeeeeeeee',
  74.5,
  'LOW',
  false,
  NOW(),
  'analista01',
  NOW(),
  'analista01'
);
```
Detalle

```sql
INSERT INTO evaluation_detail (
  id, evaluation_id, variable_id, variable_name,
  raw_value, score, weight, weighted_score, created_at
)
VALUES (
  '99999999-9999-9999-9999-999999999999',
  'ffffffff-ffff-ffff-ffff-ffffffffffff',
  'var-001-uuid-fffff-fff-fffff'
  'Ingreso Anual',
  '48000000',
  70,
  0.30,
  21.0,
  NOW(),
);

```

**HU-013 DECISIÓN**

Tablas:credit_decision

```sql
INSERT INTO credit_decision (
  id, evaluation_id, decision,
  observations,
  approved_amount, interest_rate, term_months,
  decided_at, decided_by,
  created_at, created_by
)
VALUES (
  'abababab-abab-abab-abab-abababababab',
  'ffffffff-ffff-ffff-ffff-ffffffffffff',
  'APPROVED',
  'Solicitante cumple requisitos de ingresos y tiene historial crediticio sólido.',
  15000000,
  1.8,
  24,
  NOW(),
  'analista01',
  NOW(),
  'analista01'
);
```












