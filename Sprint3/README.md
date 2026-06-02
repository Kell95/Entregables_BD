# Entregables_BD

# Sprint 3

# EAV01

Kelly Julieth Arango Henao

Carlos Andrés Cordoba

# Sistema: Motor de Scoring de Riesgo Crediticio

1. Refinar el Modelo Entidad Relación (MER).

   
<img width="21360" height="10052" alt="image" src="https://github.com/user-attachments/assets/d20a4816-5301-4202-bdef-bb18577ba014" />



2. Crear o refinar el script de creación de objetos en general con Trigger y procedimientos para las HU desarrolladas. 

# Auditoría de Solicitantes 

El trigger compara los valores anteriores y nuevos de cada atributo. Cuando detecta una diferencia, registra el cambio en la tabla de auditoría.

```sql

CREATE FUNCTION fn_applicant_edit_audit()
RETURNS TRIGGER
AS
BEGIN

    IF OLD.identification_encrypted <> NEW.identification_encrypted THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'identification_encrypted',
            OLD.identification_encrypted,
            NEW.identification_encrypted,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.identification_hash <> NEW.identification_hash THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'identification_hash',
            OLD.identification_hash,
            NEW.identification_hash,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.name <> NEW.name THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'name',
            OLD.name,
            NEW.name,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.birth_date <> NEW.birth_date THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'birth_date',
            OLD.birth_date,
            NEW.birth_date,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.employment_type <> NEW.employment_type THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'employment_type',
            OLD.employment_type,
            NEW.employment_type,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.monthly_income <> NEW.monthly_income THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'monthly_income',
            OLD.monthly_income,
            NEW.monthly_income,
            CURRENT_TIMESTAMP
        );

    END IF;

    IF OLD.work_experience_months <> NEW.work_experience_months THEN

        INSERT INTO applicant_edit_audit (
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES (
            NEW.id,
            'work_experience_months',
            OLD.work_experience_months,
            NEW.work_experience_months,
            CURRENT_TIMESTAMP
        );

    END IF;

    RETURN NEW;

END;
```

```sql
CREATE TRIGGER trg_applicant_edit_audit
AFTER UPDATE
ON applicant
FOR EACH ROW
EXECUTE FUNCTION fn_applicant_edit_audit();
```

# Escalamiento

Este trigger asigna automáticamente una fecha límite de resolución cuando una solicitud es escalada para revisión por parte de un supervisor. De esta forma, se asegura el seguimiento oportuno de los casos pendientes.

```sql
CREATE FUNCTION fn_set_escalation_deadline()
RETURNS TRIGGER
AS
BEGIN

    IF NEW.decision = 'ESCALATED'
       AND NEW.resolution_deadline_at IS NULL THEN

        SET NEW.resolution_deadline_at =
            CURRENT_TIMESTAMP + INTERVAL '48 HOURS';

    END IF;

    RETURN NEW;

END;
```

```sql
CREATE TRIGGER trg_credit_decision_escalated
BEFORE INSERT OR UPDATE
ON credit_decision
FOR EACH ROW
EXECUTE FUNCTION fn_set_escalation_deadline();
```

# Auditoría de Decisiones Crediticias

Este trigger registra automáticamente cualquier modificación realizada sobre una decisión de crédito, permitiendo conocer cuándo ocurrió el cambio y cuál fue el nuevo estado asignado.

```sql
CREATE FUNCTION fn_credit_decision_audit()
RETURNS TRIGGER
AS
BEGIN

    INSERT INTO audit_log (
        entity_type,
        entity_id,
        action,
        event_time
    )
    VALUES (
        'credit_decision',
        NEW.id,
        'UPDATE',
        CURRENT_TIMESTAMP
    );

    RETURN NEW;

END;
```

```sql
CREATE TRIGGER trg_credit_decision_audit
AFTER UPDATE
ON credit_decision
FOR EACH ROW
EXECUTE FUNCTION fn_credit_decision_audit();
```

3. Refinar el análisis del volumen de datos de las entidades identificadas

   <img width="1305" height="750" alt="img1" src="https://github.com/user-attachments/assets/ca1fd5bd-3ee6-47c2-809d-f3e5aa2fbb38" />
   <img width="1298" height="854" alt="img2" src="https://github.com/user-attachments/assets/a49f201d-ec94-400f-afaa-713fef6adf06" />
   <img width="1292" height="389" alt="img3" src="https://github.com/user-attachments/assets/adefadef-c234-44f2-86c3-485530a2329f" />


