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

CREATE OR REPLACE FUNCTION fn_applicant_edit_audit()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN

    IF OLD.identification_encrypted IS DISTINCT FROM NEW.identification_encrypted THEN
        INSERT INTO applicant_edit_audit(
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES(
            NEW.id,
            'identification_encrypted',
            OLD.identification_encrypted::text,
            NEW.identification_encrypted::text,
            CURRENT_TIMESTAMP
        );
    END IF;

    IF OLD.identification_hash IS DISTINCT FROM NEW.identification_hash THEN
        INSERT INTO applicant_edit_audit(
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES(
            NEW.id,
            'identification_hash',
            OLD.identification_hash::text,
            NEW.identification_hash::text,
            CURRENT_TIMESTAMP
        );
    END IF;

    IF OLD.name IS DISTINCT FROM NEW.name THEN
        INSERT INTO applicant_edit_audit(
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES(
            NEW.id,
            'name',
            OLD.name,
            NEW.name,
            CURRENT_TIMESTAMP
        );
    END IF;

    IF OLD.birth_date IS DISTINCT FROM NEW.birth_date THEN
        INSERT INTO applicant_edit_audit(
            applicant_id,
            field_name,
            old_value,
            new_value,
            changed_at
        )
        VALUES(
            NEW.id,
            'birth_date',
            OLD.birth_date::text,
            NEW.birth_date::text,
            CURRENT_TIMESTAMP
        );
    END IF;

    RETURN NEW;

END;
$$;
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
CREATE OR REPLACE FUNCTION fn_set_escalation_deadline()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN

    IF NEW.decision = 'ESCALATED'
    AND NEW.resolution_deadline_at IS NULL THEN

        NEW.resolution_deadline_at :=
            CURRENT_TIMESTAMP + INTERVAL '48 hours';

    END IF;

    RETURN NEW;

END;
$$;

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
CREATE OR REPLACE FUNCTION fn_credit_decision_audit()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN

    IF ROW(OLD.*) IS DISTINCT FROM ROW(NEW.*) THEN

        INSERT INTO audit_log(
            entity_type,
            entity_id,
            action,
            event_time
        )
        VALUES(
            'credit_decision',
            NEW.id,
            'UPDATE',
            CURRENT_TIMESTAMP
        );

    END IF;

    RETURN NEW;

END;
$$;

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


